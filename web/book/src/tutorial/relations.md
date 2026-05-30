# Relations

PRQL is designed on top of _relational algebra_, which is the established data
model used by modern SQL databases. A _relation_ has a rigid mathematical
definition, which can be simplified to "a table of data". For example, the
`invoices` table from the Chinook database
([https://github.com/lerocha/chinook-database](https://github.com/lerocha/chinook-database))
looks like this:

| invoice_id | customer_id | billing_city | _other columns_ | total |
| ---------- | ----------- | ------------ | :-------------: | ----- |
| 1          | 2           | Stuttgart    |       ...       | 1.98  |
| 2          | 4           | Oslo         |       ...       | 3.96  |
| 3          | 8           | Brussels     |       ...       | 5.94  |
| 4          | 14          | Edmonton     |       ...       | 8.91  |
| 5          | 23          | Boston       |       ...       | 13.86 |
| 6          | 37          | Frankfurt    |       ...       | 0.99  |

A relation is composed of rows. Each row in a relation contains a value for each
of the relation's columns. Each column in a relation has a unique name and a
designated data type. The table above is a relation, and has columns named
`invoice_id`and `customer_id` each with a data type of "integer number", a
`billing_city` column with a data type of "text", several other columns, and a
`total` column that contains floating-point numbers.

## Queries

The main purpose of PRQL is to build queries that combine and transform data
from relations such as the `invoices` table above. Here is the most basic query:

```prql no-eval
from invoices
```

<!-- prettier-ignore -->
> [!NOTE]
> Try each of these examples here in the
> [Playground.](https://prql-lang.org/playground/) Enter the query on the
> left-hand side, and click the **Query Results** tab on the right-hand side to
> see the result.

The result of the query above is not terribly interesting, it's just the same
relation as before.

### `select` transform

The `select` function picks the columns to pass through based on a list and
discards all others. Formally, that list is a _tuple_ of comma-separated
expressions wrapped in `{ ... }`.

Suppose we only need the `order_id` and `total` columns. Use `select` &mdash; the named columns pass through
to the next transform.
None of the other columns do.
_(Try it in the
[Playground.](https://prql-lang.org/playground/))_

```prql no-eval
from invoices
select { order_id, total }
```

We can write the items in the tuple on one or several lines: trailing commas are
ignored. In addition, we can assign any of the expressions to a _variable_ that
becomes the name of the resulting column in the SQL output.

```prql no-eval
from invoices
select {
  OrderID = invoice_id,
  Total = total,
}
```

This is the same query as above, rewritten on multiple lines, and assigning
`OrderID` and `Total` names to the columns.

Once we `select` certain columns,
subsequent transforms have access only to
those columns named in the tuple.

### `derive` transform

To add columns to a relation, we can use the `derive` function. Let's define a
new column for Value Added Tax, set at 19% of the invoice total.

```prql no-eval
from invoices
derive { VAT = total * 0.19 }
```

The result of this transform is to pass through
all of the existing columns as well as the new
"derived" column.
The values for the new column can be a constant
(such as a number or a string), or
can be computed from the value of an existing column.
NB In this example, the new column
is assigned the name `VAT`.

### `join` transform

The `join` transform also adds columns to the relation by combining the rows
from two relations "side by side". To determine which rows from each relation
should be joined, `join` has match criteria, written in `( ... )`.

```prql no-eval
from invoices
join customers ( ==customer_id )
```

This example "connects" the `invoices` and `customers` relations
by creating new rows that include all columns from each relation.
The `customer_id` column is used to match the rows.
If there are identical values in the `customer_id` 
column in each relation, the rows are joined.

It is frequently useful to assign an _alias_ to both relations being joined
together so that each relation's columns can be referred to uniquely.

```prql no-eval
from inv=invoices
join cust=customers ( inv.customer_id == cust.customer_id)
```

In the example above, the alias `inv` represents the `invoices` relation and
`cust` represents the `customers` relation.
Aliases also make it possible to refer to
`inv.billing_city` and `cust.last_name` unambiguously.

The query above has the same result as the previous query: `==customer_id` is shorthand for using the same column name from both relations. 

### Summary

PRQL manipulates relations (tables) of data. The `derive`, `select`, and `join`
transforms change the number of columns in a table. The first two never affect
the number of rows in a table. `join` may change the number of rows, depending
on the chosen type of join.

This final example combines these transforms into a single query. It illustrates _a pipeline_ -
the fundamental basis of PRQL. We simply add new lines (transforms)
at the end of the query. Each transform modifies the relation produced by the
statement above to produce the final result.

```prql no-eval
from inv=invoices                    # start with invoices
join cust=customers (==customer_id)  # join with customers
derive { VAT = inv.total * 0.19 }    # add a "VAT" column
select {                             # and select four columns
  OrderID = inv.invoice_id,          #   giving many of them
  CustomerName = cust.last_name,     #   different column names
  Total = inv.total,
  VAT,
}
```
