# Views and subsets

A cube can have millions of cells. To work with one, you choose which part of it to look at.

## Subsets

A **subset** is a list of elements chosen from one hierarchy of a dimension. For example:

- the four leaf regions: North, South, East, West
- every month in Q1
- the ten products with the highest revenue

Subsets come in two kinds.

| Kind | How it is defined | When the dimension changes |
| --- | --- | --- |
| Static | A fixed list of elements | Stays the same |
| Dynamic | An MDX expression | Re-evaluated, so new elements can appear |

For example, this MDX expression selects every leaf product, and picks up new products automatically:

```text
{TM1FILTERBYLEVEL({TM1SUBSETALL([Product].[Product])}, 0)}
```

Subsets can be **public**, visible to every user with access to the dimension, or **private**, visible only to the user who created them.

## Views

A **view** is a saved way of looking at a cube. It says which dimensions go on the rows and columns, and which elements are chosen for each dimension.

A typical view of the Sales cube:

- **Rows:** Product, showing every product
- **Columns:** Period, showing Jan to Dec
- **Context** (also called titles or filters): Version = Actual, Year = 2026, Region = Total Region, Sales Measure = Revenue

Each dimension not on the rows or columns is fixed to one element in the context.

TM1 has two kinds of saved view:

- **Native views**, which store the layout and a subset for each dimension
- **MDX views**, which store an MDX query

Views are used by people, for reports and data entry, and by processes, as a data source for TurboIntegrator.

Like subsets, views can be public or private.
