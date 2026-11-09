# Concepts

This chapter explains the ideas you need before you build anything in TM1: what TM1 is, how it stores data, and what the objects in a TM1 database are called. Later chapters assume you know these terms.

## In this chapter

| Page | What you will learn |
| --- | --- |
| [What is TM1?](what-is-tm1.md) | What TM1 and Planning Analytics are, and what they are used for |
| [What's new in TM1 v12](tm1-v12.md) | How v12 differs from v11, and what that means for you |
| [Cubes and dimensions](cubes-and-dimensions.md) | How TM1 organises data into cubes, dimensions and cells |
| [Elements and hierarchies](elements-and-hierarchies.md) | Element types, consolidations, weights, hierarchies and attributes |
| [How cells get their values](cells-and-calculation.md) | Input cells, consolidations, rules and feeders |
| [Views and subsets](views-and-subsets.md) | How you select and look at slices of a cube |
| [Database objects](database-objects.md) | Processes, chores, control objects, security and sandboxes |
| [Glossary](glossary.md) | Short definitions of the terms used in this book |

## The sample model

Examples throughout this book use one small sales planning model. It has a single cube, **Sales**, with these dimensions:

| Dimension | Example elements |
| --- | --- |
| Version | Actual, Budget, Forecast |
| Year | 2025, 2026 |
| Period | Jan … Dec, Q1 … Q4, Full Year |
| Region | North, South, East, West, Total Region |
| Product | Bikes, Helmets, Gloves, Total Product |
| Sales Measure | Units, Price, Revenue |

A single number in this cube might be "actual units of bikes sold in the North region in January 2026".
