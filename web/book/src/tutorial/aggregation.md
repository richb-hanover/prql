# Aggregation

A key feature of analytics is reducing many values down to a summary. This act
is called "aggregation" and always includes a function &mdash; for example,
`average` or `sum` &mdash; that reduces values in the table to a single row.

### `aggregate` transform

The `aggregate` transform takes a tuple to create a single row with one or more
_new_ columns that "distill down" data from the named columns. `aggregate`
discards other columns that are not named in the tuple.

```prql no-eval
from invoices
aggregate { sum_of_orders = sum total }
```

The query above creates a single row with one column (named `sum_of_orders`) and
a value that is the sum of the values of the`total` column.

`aggregate` can produce multiple columns: one for each expression in the tuple.

```prql no-eval
from invoices
aggregate {
    num_orders = count this,
    sum_of_orders = sum total,
}
```

In the example above, the result is a single row with two columns. The `count`
function displays the number of rows in the table that was passed in; the `sum`
function adds up the values of the `total` column of all rows.

## Grouping

Suppose we want to produce summaries of invoices _for each city_ in the table.
We could create a query for each city, and aggregate its rows:

```prql no-eval
from invoices
filter billing_city == "Oslo"
aggregate { sum_of_orders = sum total }
```

But we would need to do it for each city: `London`, `Frankfurt`, etc. Of course
this is repetitive (and boring) and error prone (because we would need to type
each `billing_city` by hand). Moreover, we would need to create a list of each
`billing_city` before we started. There's a better alternative...

### `group` transform

The `group` transform uses information that's already present in the table to
separate the rows into groups (say, those rows having the same city). It then
applies a transform to each group, and combines the results back together, along
with the columns named in the `group` transform.

```prql no-eval
from invoices
group billing_city (
    aggregate {
        num_orders = count this,
        sum_of_orders = sum total,
    }
)
```

The result of this example is a relation with as many rows as there were cities
in the `invoices` relation. Each row has _three_ columns:

- `billing_city` (because the "grouping criteria" always get passed through),
- `num_orders` (calculated for each city), and
- `sum_of_orders` (the sum of the orders for each city.

Here's a more complex example that demonstrates that the group's transform - the
lines within `( ... )` - can have multiple steps. The following code groups
`invoices` by the billing city, then executes the multi-step transform that
sorts the `total` for _each_ city, then takes two rows (the two largest) for
each city. The query completes by recombining all those groups of two rows into
a single table to produce the final result.

```prql no-eval
from invoices
group billing_city (
    sort total
    take 2
)
```

The `aggregate` transform changes the number of columns: only the ones named in
the tuple appear in its result. The `group` transform passes all the columns
through.

<!-- prettier-ignore -->
> [!NOTE]
> Those familiar with SQL may have noticed that this has decoupled
> _aggregation_ from _grouping_.
> 
> Although those two operations are tightly connected in SQL, PRQL makes it
> straightforward to use `group` and `aggregate` separately from each other,
> while still being used in combination with other transforms.
