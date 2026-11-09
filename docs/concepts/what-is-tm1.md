# What is TM1?

**TM1** is an in-memory, multidimensional database built for planning, budgeting, forecasting and analysis. IBM sells it as the engine inside **IBM Planning Analytics**.

When people say "TM1" they usually mean one of two things:

- the **database engine** that stores and calculates the data, or
- the whole **product** built around it, including the user interfaces.

This book uses "TM1" for the engine and "Planning Analytics" for the product.

## What TM1 is good at

TM1 is designed for numbers that people need to slice, total and change, for example:

- monthly budgets by department and account
- sales forecasts by product, region and customer
- headcount and salary plans
- financial consolidation and reporting

Three features make it suit this work.

**It is multidimensional.** Data lives in **cubes**. Each cube is defined by a set of **dimensions**, such as Period, Region and Product. You can look at any combination of them without writing a separate query for each. See [Cubes and dimensions](cubes-and-dimensions.md).

**It totals on the fly.** You enter data only at the lowest level, such as Bikes in North in Jan. TM1 calculates totals such as Total Product, Total Region or Q1 when you ask for them, so totals always match the detail.

**It is writeable.** Unlike most reporting databases, TM1 lets users type numbers straight into a cube, and the totals update right away. You can even type into a total, and TM1 spreads the number down to the detail cells beneath it.

## In memory

TM1 keeps its cubes in RAM. Reads, writes and recalculations happen in memory, which is why it responds quickly even with large models. Data is also saved to disk, so it survives a restart.

Because everything lives in memory, model design directly affects how much RAM a database needs. The [Modelling](../modelling/index.md) chapter covers this.

## Where you work with TM1

You rarely talk to the engine directly. In v12 you usually work through:

- **Planning Analytics Workspace (PAW)**, the web interface for building models, writing rules and processes, and creating reports and dashboards
- **Planning Analytics for Microsoft Excel (PAfE)**, an Excel add-in for reports and data entry
- the **TM1 REST API**, for scripts and integrations, for example with the Python library TM1py

## A short history

TM1 was first developed in the 1980s; the name stood for "Table Manager 1". The product passed through Applix and Cognos, and came to IBM when IBM bought Cognos in 2008. IBM later brought it, together with its web and Excel tools, under the **Planning Analytics** name.

TM1 version 11 is the long-running "classic" engine. **TM1 v12** is a re-engineered version of the engine. [What's new in TM1 v12](tm1-v12.md) explains what changed.
