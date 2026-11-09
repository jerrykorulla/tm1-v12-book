# How cells get their values

Every cell in a cube gets its value in one of three ways.

| Kind of cell | Where the value comes from | Can users type into it? |
| --- | --- | --- |
| Input cell | Stored data, typed in or loaded | Yes |
| Consolidated cell | Calculated by adding up child cells | Yes, by spreading (see below) |
| Rule-calculated cell | Calculated by a rule | No |

## Input cells

An input cell is a cell whose [address](cubes-and-dimensions.md#cells) (the list of elements, one from each dimension, that identifies it) uses only leaf elements, and that no rule calculates. These are the only cells whose values TM1 stores.

Values arrive in input cells when users type them in, when a TurboIntegrator process loads them, or when a script writes them through the REST API.

## Consolidated cells

A cell is consolidated when any element in its address is a consolidated element. This cell is consolidated because Q1 and Total Region are consolidations:

```text
Sales: Actual, 2026, Q1, Total Region, Bikes, Units
```

TM1 calculates consolidated cells when they are requested, adding up the non-empty leaf cells beneath them using the weights in each hierarchy. It keeps results in memory, so asking again is fast.

If a user types a number into a consolidated cell, TM1 can **spread** it down to the leaf cells beneath, for example equally or in proportion to existing values. Spreading changes the input cells; the consolidated cell still has no stored value of its own.

## Rules

Consolidations can only add things up. Many numbers in a model are not sums: revenue is units times price, a margin is a ratio, an allocation splits one number across many. For these, TM1 uses **rules**.

A **rule** is a formula that calculates cell values in a cube. Each cube can have one set of rules, which you edit in Planning Analytics Workspace. Rule-calculated values are not stored. TM1 works them out when someone asks for the cell, and keeps the result in memory until something it depends on changes.

### Anatomy of a rule

```text
['Revenue'] = N: ['Units'] * ['Price'];
```

A rule has three parts:

| Part | In the example | Meaning |
| --- | --- | --- |
| **Area** | `['Revenue']` | Which cells the rule applies to. Here: every cell whose address includes Revenue, whatever the other elements are. |
| **Qualifier** | `N:` | Whether the rule applies to leaf cells (`N:`), consolidated cells (`C:`), or, if left out, both. |
| **Formula** | `['Units'] * ['Price']` | How to calculate the value. `['Units']` means "the Units cell at the same address as the cell being calculated". |

The rule ends with a semicolon.

So for the cell `Actual, 2026, Jan, North, Bikes, Revenue`, TM1 reads `Actual, 2026, Jan, North, Bikes, Units` and `Actual, 2026, Jan, North, Bikes, Price` and multiplies them. One line of rule covers every version, year, period, region and product.

### Why the `N:` matters

Suppose the leaf cells for Bikes in Jan hold these values:

| Region | Units | Price | Revenue (rule) |
| --- | --- | --- | --- |
| North | 10 | 500 | 5,000 |
| South | 20 | 450 | 9,000 |
| **Total Region** | **30** | **950** | **14,000** |

With `N:`, the rule calculates Revenue only for leaf cells. Total Region revenue is then a normal consolidation of the leaf results: 5,000 + 9,000 = 14,000, which is correct.

Without `N:`, the rule would also apply at Total Region and calculate 30 × 950 = 28,500. That is wrong, because the "price" at Total Region is just the two prices added together.

The table also shows a second problem: a total Price of 950 is meaningless. A `C:` rule can replace it with an average price:

```text
['Price'] = C: ['Revenue'] \ ['Units'];
```

At Total Region this gives 14,000 ÷ 30 ≈ 466.67. The `\` operator divides like `/`, but returns 0 instead of an error when dividing by zero.

### Narrower areas

An area can name more than one element, to target a smaller set of cells:

```text
['Budget', 'Revenue'] = N: ['Units'] * ['Price'];
```

This rule applies only to Budget revenue. Actual revenue, loaded from the sales system, is left alone.

### Rule order

When more than one rule could apply to a cell, TM1 uses the **first** one in the rule file and ignores the rest. Put specific rules before general ones:

```text
# Actual revenue is loaded from the sales system, so leave it as input.
['Actual', 'Revenue'] = N: STET;

# Every other version calculates revenue.
['Revenue'] = N: ['Units'] * ['Price'];
```

`STET` means "don't calculate this cell; treat it as an ordinary input cell". Lines starting with `#` are comments.

### Reading from other cubes

Rules can read values from other cubes with the `DB` function. Suppose prices are kept in a separate **Price List** cube with Year, Product and a measure dimension:

```text
['Price'] = N: DB('Price List', !Year, !Product, 'Unit Price');
```

`!Year` and `!Product` mean "the Year and Product of the cell being calculated". So each Price cell looks up the price for its own year and product.

### Rule-calculated cells are read-only

Users cannot type into a rule-calculated cell, and processes cannot write to it, because its value always comes from the formula. To change it, change the cells the formula reads from. The [Rules](../rules/index.md) chapter covers the full rule language, including conditions, string rules and performance.

## Feeders

### The problem feeders solve

TM1 is fast because of sparsity: when it adds up a total, it visits only the leaf cells that hold data and skips the empty ones.

Rule-calculated cells don't hold stored data. As far as TM1's sparse totalling is concerned, every Revenue cell looks empty. So when you ask for Revenue at Total Region, TM1 skips all the Revenue leaf cells and returns 0, even though each one would calculate to a value if asked.

TM1 needs to be told in advance which rule-calculated cells might hold a value. That is what **feeders** do.

### How feeders work

A feeder says: "whenever this cell holds a value, flag that other cell as possibly holding a value too". A flagged cell is said to be **fed**. When TM1 totals, it visits stored cells and fed cells, and skips the rest.

Feeders go at the end of the rules, after the `FEEDERS;` line. The rules also need `SKIPCHECK;` at the top, which tells TM1 to use sparse totalling, and therefore feeders, for this cube.

```text
SKIPCHECK;

['Revenue'] = N: ['Units'] * ['Price'];

FEEDERS;

['Units'] => ['Revenue'];
```

`['Units'] => ['Revenue']` reads: "for any Units cell with a value, feed the Revenue cell at the same address".

With the earlier example:

1. A process loads 10 into `… North, Bikes, Units`.
2. The feeder flags `… North, Bikes, Revenue` as fed.
3. A user asks for Revenue at Total Region.
4. TM1 visits the fed Revenue cells for North and South, calculates 5,000 and 9,000, and returns 14,000.

### Choosing what to feed from

A rule usually reads several cells, but you only need to feed from one that has a value whenever the result does. Revenue is Units × Price, so it is non-zero only when Units is non-zero. Units is the right source.

Feeding from Price would also work, but it would be wasteful. A price list often holds a price for every product in every region and month, including combinations that never sell. Feeding from Price would flag thousands of Revenue cells that always calculate to 0.

### Underfeeding and overfeeding

| Problem | Cause | What you see |
| --- | --- | --- |
| **Underfeeding** | Some rule-calculated cells that hold values are not fed | Totals are too low or 0, even though the leaf cells show the right values |
| **Overfeeding** | Cells are fed that will never hold a value | Totals are correct, but the database uses more memory and calculates more slowly |

Underfeeding gives wrong numbers, so it is the more serious problem. Overfeeding costs memory and speed. A common cause of overfeeding is feeding a consolidated element: feeding a consolidation feeds every leaf beneath it.

Also note that a feeder flag is not removed when the source cell is cleared. The target stays fed until TM1 recalculates feeders, for example when the cube's rules are saved.

### Without SKIPCHECK

If a cube's rules leave out `SKIPCHECK;`, TM1 doesn't skip empty cells in areas with rules. Totals are always correct without any feeders, but calculation can be very slow on large cubes. Most production cubes with rules use `SKIPCHECK;` and feeders.

The [Rules](../rules/index.md) chapter covers feeders in more depth, including feeding across cubes and how to find feeder problems.
