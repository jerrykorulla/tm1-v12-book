# Hands-on: build the Sales cube

In this exercise you build the [sample model](index.md#the-sample-model) yourself and use it to see each idea from this chapter at work. Most steps end with something to **observe**: a number to check, or a question to answer, that ties back to a concept page.

Allow about an hour.

## What you need

- **A TM1 v12 database** you can change, with admin rights. TM1 v12 runs on IBM Planning Analytics as a Service or Planning Analytics on IBM Software Hub. Use a development or sandbox database, never production.
- **Cubewise Arc**, version 4.4 or later. Arc is a development tool for TM1 made by Cubewise, a third-party company. It is licensed software, so you need a licence or a trial from Cubewise.

!!! info "Using a different tool"
    Each step describes **what** to build, not which buttons to press, so you can follow along in Planning Analytics Workspace instead of Arc. Menu names in Arc can also change between releases. If a step doesn't match what you see, look for the equivalent action in your version.

!!! tip "TM1 v11"
    Everything in this exercise also works on a TM1 v11 server, because the concepts are the same. Only the connection step differs.

## Part 1: Connect to your database

Add your TM1 v12 database as a connection in Arc. For Planning Analytics as a Service you need:

- the database's URL, from your Planning Analytics administrator or the IBM console
- an **API key**, which you generate in Planning Analytics as a Service

Once connected, expand the database in Arc. You should see empty or near-empty lists of cubes, dimensions and processes.

**Observe:** Arc connects to TM1 over the REST API. In v12 that is the only way any tool can connect. See [What's new in TM1 v12](tm1-v12.md#the-rest-api-is-the-only-interface).

## Part 2: Create the dimensions

Create these six dimensions. Each tree shows the hierarchy to build: indented elements are children of the element above them. Elements marked **(C)** are consolidated; everything else is numeric unless marked **(S)**. Leave every weight at 1.

**Version**

```text
Actual
Budget
Forecast
```

**Year**

```text
2025
2026
```

**Period**

```text
Full Year (C)
├── Q1 (C)
│   ├── Jan
│   ├── Feb
│   └── Mar
├── Q2 (C)
│   ├── Apr
│   ├── May
│   └── Jun
├── Q3 (C)
│   ├── Jul
│   ├── Aug
│   └── Sep
└── Q4 (C)
    ├── Oct
    ├── Nov
    └── Dec
```

**Region**

```text
Total Region (C)
├── North
├── South
├── East
└── West
```

**Product**

```text
Total Product (C)
├── Bikes
├── Helmets
└── Gloves
```

**Sales Measure**

```text
Units
Price
Revenue
Comment (S)
```

**Observe:** Open the Period dimension and check the levels. Jan should be level 0, Q1 level 1, and Full Year level 2. See [Levels](elements-and-hierarchies.md#levels).

## Part 3: Add an alias

1. On the **Product** dimension, create an attribute named `Code` of type **Alias**.
2. Give each product a code:

    | Element | Code |
    | --- | --- |
    | Bikes | P-100 |
    | Helmets | P-200 |
    | Gloves | P-300 |
    | Total Product | P-ALL |

3. Try to give Gloves the code `P-100` as well.

**Observe:** TM1 refuses the duplicate, because alias values must be unique in the dimension. See [Attributes](elements-and-hierarchies.md#attributes).

## Part 4: Create the cube

Create a cube named **Sales** with the dimensions in this order:

```text
Version, Year, Period, Region, Product, Sales Measure
```

**Observe:** The cube has 3 × 2 × 17 × 5 × 4 × 4 = 8,160 possible cells, but it takes almost no memory, because none of them holds data yet. See [Sparsity](cubes-and-dimensions.md#sparsity).

## Part 5: Enter data and watch totals

Open the Sales cube in Arc's cube viewer and set up this layout:

- **Rows:** Region
- **Columns:** Sales Measure
- **Context:** Version = Actual, Year = 2026, Period = Jan, Product = Bikes

Type in these values:

| Region | Units | Price |
| --- | --- | --- |
| North | 10 | 500 |
| South | 20 | 450 |

**Observe:**

1. Total Region Units shows **30**. You never typed it; TM1 [consolidated](cells-and-calculation.md#consolidated-cells) it from the leaf cells.
2. Total Region Price shows **950**. Adding prices makes no sense. You'll fix this in Part 8.
3. Try typing into Total Region Units. Depending on your tool's settings, TM1 either refuses or [spreads](cells-and-calculation.md#consolidated-cells) the value across North, South, East and West. If it spreads, set North and South back to 10 and 20 and clear East and West before you go on.
4. Type `Strong month` into North's Comment cell. It accepts text because Comment is a string element in the last dimension. See [Element types](elements-and-hierarchies.md#element-types).

## Part 6: Write your first rule

Open the rules editor for the Sales cube and enter this rule. It deliberately leaves out `N:`:

```text
['Revenue'] = ['Units'] * ['Price'];
```

Save, then look at the view again.

**Observe:**

| Region | Revenue |
| --- | --- |
| North | 5,000 |
| South | 9,000 |
| Total Region | **28,500** |

Total Region is wrong. Without `N:` the rule also runs on the consolidated cell, and multiplies total units by the meaningless total price: 30 × 950. See [Why the N: matters](cells-and-calculation.md#why-the-n-matters).

Now add `N:` and save:

```text
['Revenue'] = N: ['Units'] * ['Price'];
```

**Observe:** Total Region Revenue is now **14,000**, the sum of the leaf results. Try typing into a Revenue cell: you can't, because it is [rule-calculated](cells-and-calculation.md#rule-calculated-cells-are-read-only).

## Part 7: Break and fix the feeders

Add `SKIPCHECK;` as the first line of the rules, and save:

```text
SKIPCHECK;

['Revenue'] = N: ['Units'] * ['Price'];
```

**Observe:** North and South still show 5,000 and 9,000, but Total Region Revenue drops to **0**. TM1 now skips cells that are empty or unfed, and nothing feeds Revenue. This is **underfeeding**. See [The problem feeders solve](cells-and-calculation.md#the-problem-feeders-solve).

Add the feeder and save:

```text
SKIPCHECK;

['Revenue'] = N: ['Units'] * ['Price'];

FEEDERS;

['Units'] => ['Revenue'];
```

**Observe:** Total Region Revenue is **14,000** again.

Now enter Units of 5 for East, with no price.

**Observe:** East Revenue is fed, because East has Units, but it calculates to 0 because Price is empty. Totals are still right. Feeding a cell that ends up as 0 wastes a little memory but never gives a wrong answer.

## Part 8: Fix the total price

Add a `C:` rule **above** the Revenue rule, so the rules read:

```text
SKIPCHECK;

['Price'] = C: ['Revenue'] \ ['Units'];

['Revenue'] = N: ['Units'] * ['Price'];

FEEDERS;

['Units'] => ['Revenue'];
```

**Observe:** Total Region Price is now **400**: revenue of 14,000 divided by units of 35, now that East has 5 units. The leaf prices are untouched, because the rule only applies to consolidated cells.

Set East Units back to empty. Total Region Price becomes 14,000 ÷ 30 ≈ **466.67**.

## Part 9: Protect actual revenue with STET

Suppose actual revenue comes from the sales system and should not be calculated. Add a `STET` rule **above** the Revenue rule:

```text
SKIPCHECK;

['Price'] = C: ['Revenue'] \ ['Units'];

['Actual', 'Revenue'] = N: STET;

['Revenue'] = N: ['Units'] * ['Price'];

FEEDERS;

['Units'] => ['Revenue'];
```

**Observe:**

1. Actual Revenue for North and South is now **empty**, and you can type into it. The first matching rule wins, and `STET` makes these ordinary input cells again. See [Rule order](cells-and-calculation.md#rule-order).
2. Switch the context to Version = Budget and enter some Units and Prices. Budget Revenue is still calculated.
3. Move the `STET` line **below** the Revenue rule and save. Actual Revenue is calculated again, because the general rule now matches first. Move it back.

## Part 10: Build a view and a dynamic subset

1. Add a new product, **Lights**, as a child of Total Product, with code `P-400`.
2. On the Product dimension, create a **dynamic** subset named `All Products` with this MDX:

    ```text
    {TM1FILTERBYLEVEL({TM1SUBSETALL([Product].[Product])}, 0)}
    ```

3. Create a **static** subset named `Original Products` containing Bikes, Helmets and Gloves.
4. Save a public view of the Sales cube named `Revenue by Product`:
    - **Rows:** Product, using `All Products`
    - **Columns:** Period, Jan to Mar
    - **Context:** Version = Budget, Year = 2026, Region = Total Region, Sales Measure = Revenue

    Some cells may be empty, depending on the Budget data you entered in Part 9. This part is about which **rows** appear, not the values.
5. Add another product, **Pumps**, under Total Product.

**Observe:** Reopen the view. Pumps appears, because `All Products` is dynamic. Open `Original Products`: it still lists only three products, because static subsets don't change. See [Subsets](views-and-subsets.md#subsets).

## Part 11: Look behind the scenes

Turn on the display of control objects in Arc, then open:

1. the **`}ElementAttributes_Product`** cube. You'll find the `Code` values you typed in Part 3, stored as ordinary cube data.
2. the **`}Clients`** dimension. Your own user is listed.

**Observe:** TM1 stores its own metadata in ordinary cubes and dimensions. See [Control objects](database-objects.md#control-objects).

## Check your understanding

??? question "1. Why did Total Region Revenue show 28,500 in Part 6?"
    The rule had no `N:`, so it also calculated the consolidated cell: 30 total units × 950 total price. With `N:`, the consolidated cell is a normal sum of the leaf results.

??? question "2. In Part 7, why did adding SKIPCHECK make the total 0 while the leaf cells still showed values?"
    Leaf cells are calculated when you ask for them directly. Totals are built by visiting only cells that hold data or are fed. With `SKIPCHECK;` and no feeders, no Revenue cell was fed, so the total skipped them all.

??? question "3. Why feed Revenue from Units and not from Price?"
    Revenue is non-zero only when Units is non-zero. Prices often exist for combinations that never sell, so feeding from Price would feed many cells that always calculate to 0. That is overfeeding.

??? question "4. A colleague adds a general rule for Revenue at the top of the file, and actual revenue starts being calculated again. Why?"
    TM1 uses the first rule that matches a cell. A general Revenue rule above the `STET` rule matches Actual revenue first, so the `STET` is never reached.

??? question "5. Where does TM1 store the alias P-100?"
    In the `}ElementAttributes_Product` control cube, at the cell for element Bikes and attribute Code.

## Clean up

To remove what you built, delete the Sales cube first, then the six dimensions. TM1 won't delete a dimension that a cube still uses.

Or keep the model: later chapters build on it.
