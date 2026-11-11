# Elements and hierarchies

## Element types

Every element in a dimension has one of three types.

| Type | Letter | What it holds |
| --- | --- | --- |
| Numeric | N | A number. Numeric elements without children are the lowest level of the dimension, often called **leaf** elements. |
| Consolidated | C | A total of other elements, called its **children**. TM1 calculates its value. |
| String | S | Text, such as comments and labels. |

In the Region dimension, North, South, East and West are numeric leaf elements, and Total Region is a consolidated element.

!!! note "String cells"
    A cell holds text only when the element from the **last** dimension of the cube is a string element. This is one reason the measures dimension usually comes last: you can then add a string measure such as `Comment`.

## Consolidations and weights

A consolidated element adds up its children. Each parent–child link has a **weight**, which the child's value is multiplied by before it is added. The default weight is 1.

A weight of -1 subtracts a child, which is common in financial models:

```text
Gross Profit (C)
├── Revenue        weight  1
└── Cost of Sales  weight -1
```

A child can itself be a consolidation, so totals can be many levels deep:

```text
Full Year (C)
├── Q1 (C)
│   ├── Jan
│   ├── Feb
│   └── Mar
├── Q2 (C)
│   └── …
```

An element can also have more than one parent. For example, Bikes can roll up to both Total Product and a separate Cycling Products total.

## Levels

TM1 describes where an element sits in two ways:

- **Level** counts up from the bottom. Leaf elements are level 0, their parents are level 1, and so on.
- **Depth** counts down from the top. Elements with no parent are at depth 0.

Many functions and MDX expressions use levels, for example to select every leaf element (level 0).

## Hierarchies

A **hierarchy** is one way of arranging a dimension's elements into totals. Every dimension has at least one hierarchy, which has the same name as the dimension.

A dimension can have extra hierarchies that group the same leaf elements in different ways. For example, the Product dimension might have:

- the default **Product** hierarchy, grouping products by category
- a **Product by Supplier** hierarchy, grouping the same products by supplier

Leaf elements are shared across a dimension's hierarchies. Units of Bikes are stored once, whichever hierarchy you look at them through. Each hierarchy only changes how the totals are built.

Because the same element name can appear in more than one hierarchy, you sometimes need a fully qualified name in MDX:

```text
[Product].[Product by Supplier].[Bikes]
```

## Attributes

An **attribute** is extra information stored against each element of a dimension, like a column in a table. Attributes have one of three types.

| Type | Example |
| --- | --- |
| String | Product colour, regional manager's name |
| Numeric | Product weight, sort order |
| Alias | An alternative name for the element |

**Aliases** let users see and use a different name for an element. For example, a product might have the element name `P-1001` and the alias `Mountain Bike`. Alias values must be unique in the dimension, among themselves and among element names.

Attribute values are stored in a hidden cube named `}ElementAttributes_<dimension>`, for example `}ElementAttributes_Product`. See [control objects](database-objects.md#control-objects).

!!! example "Try it"
    Build consolidations, check levels and add an alias in [Parts 2 and 3 of the hands-on exercise](exercise.md#part-2-create-the-dimensions).
