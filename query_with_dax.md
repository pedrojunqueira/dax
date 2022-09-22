# Query with DAX

What are the tools to query DAX?

SSMS (possible but not ideal)

[DAX Studio](https://daxstudio.org/) is ideal

With DAX studio is possible to connect to a PBIX file or a Tabular Server

What is a query in DAX?

Is a way to query a Tabular model using DAX 

# Structure of a query

```SQL
DEFINE

    Measure <table> [<col>] = <expression>

EVALUATE <Table Expression>

ORDER BY <expression> [ASC/DESC]

START AT <value>, ...
```

## EVALUATE

This is the only required argument to run/evaluate a DAX query


```SQL
EVALUATE
	Customer
```

it will return the whole table

works like

```SQL
SELECT * from Customer
```

It is possible to sort it by using `ORDER BY`



```SQL
EVALUATE
	Customer
ORDER BY Customer[Gender] DESC
```

## CALCULATETABLE

```SQL
EVALUATE
	CALCULATETABLE(
		'Product',
		FILTER( ALL('Product'[Color]),
			'Product'[Color] = "red")
		)
```

CALCULATETABLE is the recommended method to filter a table.

It is faster then using `FILTER`

The order of evaluation of CALCULATETABLE is outer first then inner which overrides the outer.

## FILTER

THe order of execution of the `FILTER` of nested it is first inner then outer

## SELECTCOLUMNS

IF you want to add columns you can use `SELECTCOLUMNS`

Works similar to the SELECT in SQL language.

```SQL
EVALUATE
SELECTCOLUMNS(
	'Product', --can be any table expression
	"Category", RELATED( 'Product Category'[Category]),
	"Color", 'Product'[Color],
	"Sales Amount", [Sales Amount]
	)
```

It will ALWAYS return all the rows of the selected table. It dos not aggregate.

It only reduced the column of the evaluated table.

## ADDCOLUMNS

This one extends the columns of a table.

```SQL
EVALUATE
	ADDCOLUMNS(
		'Product Category',
		"Sales Amount", [Sales Amount]
)
```

You can combine both
```SQL
EVALUATE
	ADDCOLUMNS(
		SELECTCOLUMNS(
	'Product',
	"Color", 'Product'[Color],
	"Sales Amount SELECTED", [Sales Amount]
	),
	"Sales Amount ADDCOL", [Sales Amount]
)


```

The ADDCOLUMNS will aggregate values of distinct combination of columns

## SUMMARIZE

Used to aggregate - not ideal to use. Use with care

Enable to get few columns in a unique way. Able to add columns of different table if there
is a relationship many to one.

Behind the scenes it joins all table relationship

It works similar to SELECT GROUP BY in SQL. BUT DO NOT DO THIS...

Use other techniques.

Because of performance issues and also semantics.

```SQL
EVALUATE

SUMMARIZE(
	Sales,
	'Product Category'[Category],
	'Product'[Color],
	"Sales Amount", [Sales Amount]
	)
```

Only use to get distinct values and then add columns with `ADDCOLUMNS`

A better way is

```SQL
EVALUATE

ADDCOLUMNS(
    SUMMARIZE(
        Sales,
        'Product Category'[Category],
        'Product'[Color]),
        
        "Sales Amount", [Sales Amount]
        )

```

## SUMMARIZECOLUMNS

It is similar to SUMMARIZE with a more advanced version.

Issue is that cannot be used in a measure because practically impossible to used in a measure.

Works similar to SUMMARIZE however you do not need to specify a starting table. 

```SQL

EVALUATE
SUMMARIZECOLUMNS(
	'Product Category'[Category],
	Product[Color],
	"sales amount", [Sales Amount])
```
Table can be used as filters for the summarizecolumns

SUMMARIZE can group only one table
SUMMARIZECOLUMNS can aggregate measures coming from different tables

Article of SQLBI about summarize columns ->

You will hardly use SUMMARIZECOLUMNS unless writing report for reporting server.

## CROSSJOIN

Used to create a cartesian product 

```SQL
EVALUATE
ADDCOLUMNS (
    CROSSJOIN (
        DISTINCT ( 'Product'[Brand] ),
        DISTINCT ( 'Product'[Color] )
    ),
    "Sales", [Sales Amount]
)
```

## TOPN

Works like limit/top in SQL

```SQL
EVALUATE
TOPN (
    10,
    ADDCOLUMNS (
        VALUES ( 'Product'[Product Name] ),
        "Sales", [Sales Amount]
    )
)

```


## GENERATE

it is similar to `APPLY` in SQL

Can be used with TOPN

```SQL
EVALUATE
GENERATE (
    VALUES ( Customer[Continent] ),
    TOPN (
        3,
        ADDCOLUMNS (
            VALUES ( 'Product'[Product Name] ),
            "Sales", [Sales Amount]
        ),
        [Sales]
    )
)
```

It returns the top 3 products for all countinents

It repeats the values of the first argument and combines with the result of the second 
executed in a row context.

```SQL
EVALUATE
GENERATE (
    VALUES ( 'Product Category'[Category] ),
    CALCULATETABLE (
        TOPN (
            3,
            ADDCOLUMNS (
                VALUES ( Product[Product Name] ),
                "Sales", [Sales Amount]
            ),
            [Sales]
        )
    )
)
```

## ROW

Used to generate new table

Create a table with 1 row 1 column

```SQL

EVALUATE
	ROW("Sales", [Sales Amount])
```

To create a anonymous table

use curly braces

```SQL
EVALUATE
{ [Sales Amount] }
```

## DATATABLE

Now you can control the name of rows and columns

```SQL

EVALUATE
DATATABLE (
    "Name", STRING,
    "Amount", INTEGER,
    {
        { "Pedro", 10 },
        { "Elena", 20 }
    }
)
```

This is a bit useless as accept only constants and not dynamic values.

### Tables and relationships

When we use tables we can always use relationship in a intuitive way.

## UNION 

Performs set addition

```SQL
EVALUATE
VAR DatesWithSales =
    SUMMARIZE ( Sales, 'Date'[Date] )
VAR MondayDates =
    CALCULATETABLE ( VALUES ( 'Date'[Date] ), 'Date'[Day of Week] = "Monday" )
VAR Result =
    UNION ( DatesWithSales, MondayDates )
RETURN
    Result
ORDER BY 'Date'[Date]

```


## INTERSECT 

performs set intersection

```SQL

EVALUATE
CALCULATETABLE (
    VAR FirstDay =
        MIN ( 'Date'[Date] )
    VAR CurrentCustomers =
        VALUES ( Sales[CustomerKey] )
    VAR PreviousCustomers =
        CALCULATETABLE ( VALUES ( Sales[CustomerKey] ), 'Date'[Date] < FirstDay )
    VAR ReturningCustomers =
        INTERSECT ( CurrentCustomers, PreviousCustomers )
    RETURN
        ReturningCustomers,
    'Date'[Calendar Year Number] = 2007,
    'Date'[Month Number] = 2
)

```

## GROUP BY

Used when want to group by from 2 "temp" queries.

```SQL
EVALUATE
VAR SalesPriceLevels =
    ADDCOLUMNS (
        ALL ( Sales[Unit Price] ),
        "Sales", [Sales Amount],
        "Price Level", SWITCH (
            TRUE,
            Sales[Unit Price] < 100, "LOW",
            Sales[Unit Price] < 1000, "MEDIUM",
            "HIGH"
        )
    )
RETURN
    GROUPBY (
        SalesPriceLevels,
        [Price Level],
        "Sales", SUMX (
            CURRENTGROUP (),
            [Sales]
        )
    )
```

## ADDING MEASURES

```SQL
DEFINE
    MEASURE Sales[Subcategories] =
        COUNTROWS ( RELATEDTABLE ( 'Product Subcategory' ) )
    MEASURE Sales[Products] =
        COUNTROWS ( RELATEDTABLE ( 'Product' ) )
EVALUATE
ADDCOLUMNS (
    'Product Category',
    "SubCategories", [Subcategories],
    "Products Count", [Products]
)
```

If an existing measure is defined in the query it will overwrite the measure in the model

It is useful to debug

