# handling errors

function `iserror` or `iferror`

the `divide` function handle errors

# variables

they are not actually variables they are constant because they cannot be changed
there is no global scope only local

```dax
VAR name_variable = sum(sales[quantity])

return name_variable

```

# Table Functions

## FILTER

`Filter` -> filters a table and returns a table.

It reduces the rows in a table given a condition

## ALL

`All` return the entire content of a table **_ignoring_** any filter

Returns the entire table.

Can take the whole table, one column or several columns.

Applied to columns it will return the distinct values of the column.
It works like `select distinct` in SQL.

## ALLEXCEPT

`Allexcept` it is like writing `All` but excluding a few columns

It takes at least two arguments. The table and the columns to be excluded.

## DISTINCT

`Distinct` returns the unique values of a table.

The difference of `all` and `distinct` is that distinct does not ignore filters.

# Dax treatment of relationship "referential integrity"

If there is a relationship between two tables, say sales and product, where it is a many to one (sales->product)
and there are missing products in product tables that are in sales.
In this case Dax add an additional blank row in the table in the one side of
the relationship i.e. the product table to accommodate the non matching products in the sales table.

Using `All` it will consider the blank row
`distinct` will not consider the blank row

## VALUES

if you want to have the same behavior of `distinct` but consider the blank row then
user `values`

in doubt use `values` to make sure you do not miss a row, unless
you want to for some reason.

## ALLNONBLANKROW

In this context `Allnonblankrow` has the same behavior of `All` but does not
return the blank rows.

## ALLSELECTED

`Allselected` -> returns the whole table, ignoring filters only for
what was selected in the filter **_outside_** the visual.

This is very usefully when calculating sales % and you want to
dynamically adjust the denominator of the % formula.
