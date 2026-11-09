# How cells get their values

Every cell in a cube gets its value in one of three ways.

| Kind of cell | Where the value comes from | Can users type into it? |
| --- | --- | --- |
| Input cell | Stored data, typed in or loaded | Yes |
| Consolidated cell | Calculated by adding up child cells | Yes, by spreading (see below) |
| Rule-calculated cell | Calculated by a rule | No |

## Input cells

An input cell is a cell whose address uses only leaf elements, and that no rule calculates. These are the only cells whose values TM1 stores.

Values arrive in input cells when users type them in, when a TurboIntegrator process loads them, or when a script writes them through the REST API.

## Consolidated cells

A cell is consolidated when any element in its address is a consolidated element. This cell is consolidated because Q1 and Total Region are consolidations:

```text
Sales: Actual, 2026, Q1, Total Region, Bikes, Units
```

TM1 calculates consolidated cells when they are requested, adding up the non-empty leaf cells beneath them using the weights in each hierarchy. It keeps results in memory, so asking again is fast.

If a user types a number into a consolidated cell, TM1 can **spread** it down to the leaf cells beneath, for example equally or in proportion to existing values. Spreading changes the input cells; the consolidated cell still has no stored value of its own.

## Rules

A **rule** is a formula, written for a cube, that calculates cell values. Rules handle anything that is not a simple sum, such as prices, ratios, allocations and lookups into other cubes.

A rule names the cells it applies to on the left and the formula on the right:

```text
['Revenue'] = N: ['Units'] * ['Price'];
```

This says: for every leaf (`N:`) cell whose Sales Measure is Revenue, calculate Units multiplied by Price at the same address.

Rule-calculated cells are read-only. They update automatically when the cells they depend on change.

## Feeders

Rules and sparsity need to cooperate. Because TM1 skips empty cells when it builds totals, it needs to know which rule-calculated cells will have a value. Without that, a total of Revenue would skip every calculated Revenue cell and show zero.

**Feeders** solve this. A feeder tells TM1: "when this cell has data, mark that other cell as having data too."

```text
SKIPCHECK;

['Revenue'] = N: ['Units'] * ['Price'];

FEEDERS;

['Units'] => ['Revenue'];
```

Here, whenever a Units cell holds a value, the matching Revenue cell is fed, so totals include it.

Getting feeders right is one of the main skills in TM1 development. Too few feeders give wrong totals; too many waste memory and slow calculation. The [Rules](../rules/index.md) chapter covers both in detail.
