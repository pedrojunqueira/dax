# Evaluation context

There is always 2 contexts happening at the same time

1. Filter Context
2. Row Context

The filter context "filters" the Row context "iterates". They do different things.

## Filter context

It is any filters coming from

    - Visuals
    - Slicers
    - Filters
    - Matrix cell interception

The filter will "reduce" the number of rows while evaluating measures.

## Row Context

It happens during iteration of a table row by row.

Happens at.

    1. Automatic created for calculated column. It does need to be created in a iteration.
    2. Row iteration functions with "X"

Although there is an automatic row context in a calculated column, in a measure
the row context needs to be created by an iterator function like `sumx` and the
other X functions.

The row context "kicks in" after the filter context when a measure is evaluated.

## RELATED

Access column in related tabled from the many to one side.

## Outer and Inner row context.

### Example with a ranking created with row context.

```sql
Products[RankofPrice] =
Var CurrentListPrice = Product[ListPrice] --outer row context

return
Countrows (
    Filter (
        Products,
        Products[ListPrice] > CurrentListPrice -- inner row context
    )
)
```

The outer context is the price of the current iterated row
The inner context is the outer row context that is iterated in the whole product table to
return the Filtered table.

When there was no variable to separate the inner and outer context was used the EARLIER function.

The same above measure would have been written

```sql
Countrows (
    Filter (
        Products,
        Products[ListPrice] > EARLIER(Products[ListPrice]) -- inner row context
    )
)
```

The former formula will return a dense rank, because is counting
the number of product with a price greater than the current product price.
To do this it is possible to return a distinct list of prices.

For this we can use ALL

```sql
Products[RankofPrice] =
Var CurrentListPrice = Product[ListPrice] --outer row context

return
Countrows (
    Filter (
        ALL(Products[ListPrice],
        Products[ListPrice] > CurrentListPrice -- inner row context
    )
)
```

Interesting that the `FILTER` took an argument a ALL(COLUMN) and still
can filter with a condition.

## Evaluation context and relationships

There are four scenarios

Row Context Many side
Row Context One Site

Filter Context Many Side
Filter Context Once Side

The row context does not happens across relationships.
For this you need `RELATED` in the many side

In the one side you will need `RELATEDTABLE` to fix the "problem" of
row context not working across tables.

The filter context automatically flows relationships
from the one side to the many side.

If you need to filter from the many to one you need to explicitly
configure a bi-directional relationship or
do it via Crossfilter() modifier.
