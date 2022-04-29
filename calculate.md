# CALCULATE

What the second argument does?

It modifies the filter context.

It can be a predicate or a table.

If it is a predicate then DAX will transform this into a table
that is pushed into the filter context.

or the second argument can be a table.

## Examples

```
% Brand Sales =
divide (SalesAmount,
Calculate(
SalesAmount,
All('Product'[Brand])
))
```

This will remove the filters on Product['Brand'] ONLY

If you want to move the filter of all Product columns then pass the table in the
ALL() like.

```
% Product Sales =
divide (SalesAmount,
Calculate(
SalesAmount,
All('Product')
))
```

But still this will not remove filters for other dimension tables.

You can either individually add ALL(TableName) arguments.
And Calculate take a 3rd, 4th, nth Element.

Or you can just pass ALL(Sales) then all the
filters on the dimensions will be removed.

Also the ALLSELECTED() can be used if the intention is to
have a dynamic % as the end user filter the model using slicer or any visuals.

Every argument in `CALCULATE` is a table.

## KEEPFILTERS

Allow you to add new filters without ignoring the filters applied to the filter context.

This is possible if you "wrap" the `CALCULATE` filter argument with `KEEPFILTERS`

then the filter context will "merge" with the `CALCULATE` filter argument.

Only columns in both the filter context and the filter argument will be
considered.

This function can only be used as a filter argument in `CALCULATE`.

## What is the order of evaluation in nested `CALCULATE` ?

The inner calculate filters overwrites the outer calculated when there is a "clash" in the
same table column

unless `KEEPFILTERS` is used. Then the filter is intersected (merged) instead of overridden.

## Manipulate filter in `CALCULATE`

Intersect -> Multiple filters in `CALCULATE` arguments
Overwrite -> Nested `CALCULATE` inner overwrites out
Remove Filters -> use `ALL`
Add filters -> use `KEEPFILTERS`

## Efficiently passing filter arguments to calculate

for example when passing a filter argument to `CALCULATE`
only pass the necessary columns that are used to evaluate the
filter argument.

So DO NOT do this...

```sql
BigSales =
    CALCULATE(
    [Sales Amount],
    FILTER(
        Sales,
        (Sales[quantity] * Sales[UnitPrice]) > 1000

    )
)
```

Instead write like below.

```sql
BigSales =
    CALCULATE(
        [Sales Amount],
        KEEPFILTERS(
            FILTER(
                ALL(Sales[Quantity], Sales(UnitPrice)),
                        (Sales[quantity] * Sales[UnitPrice]) > 1000

                    )
                )
            )
```

The reason to use `KEEPFILTERS` to wraps the whole filter argument is to make sure that the
filters impacting the evaluation of `CALCULATE` intersect with the filter argument.

Both will produce the same result however the second is more efficient.

It is best practice to NEVER have a whole table as a the `FILTER` function.
Only use the columns necessary to return the `CALCULATE` filter argument.

## Variables in evaluation context

Variables are Constants.

They cannot be changed by using calculate

-- TODO expand mode with an example
