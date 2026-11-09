# Database objects

A TM1 database contains more than cubes and dimensions. This page introduces the other objects you will meet.

## Processes and chores

A **TurboIntegrator process** (usually just "process" or "TI process") is a script that runs inside the database. Processes load data from files, views or other systems, build and update dimensions, copy and clear data, and run administrative tasks.

Each process has four sections that run in order:

| Section | Runs | Typically used for |
| --- | --- | --- |
| Prolog | Once, before the data source is read | Setup, clearing data, creating views |
| Metadata | Once for each record in the data source | Adding elements to dimensions |
| Data | Once for each record in the data source | Writing values into cubes |
| Epilog | Once, at the end | Cleanup, logging, calling other processes |

A **chore** runs one or more processes in sequence, on a schedule or on demand. Chores are how you automate overnight loads and other routine jobs.

See the [TurboIntegrator](../turbointegrator/index.md) chapter.

## Control objects

TM1 keeps its own settings and metadata in ordinary cubes and dimensions whose names start with a closing brace, `}`. These are **control objects**. They are hidden by default, but you can show them in Planning Analytics Workspace.

Some you will meet often:

| Object | Holds |
| --- | --- |
| `}Clients` | Every user of the database |
| `}Groups` | Every security group |
| `}ClientGroups` | Which users belong to which groups |
| `}ElementAttributes_<dimension>` | Attribute values for a dimension |
| `}CubeSecurity` | Group access rights to each cube |
| `}ElementSecurity_<dimension>` | Group access rights to individual elements |
| `}CellSecurity_<cube>` | Group access rights to individual cells |

Because control objects are ordinary cubes and dimensions, you can read and update them with rules, processes and views like any others.

## Security

TM1 security is based on **groups**. You give each group access rights to cubes, dimensions, elements, cells, processes and chores, and then add users to groups.

The main access rights are:

| Right | Allows the group to |
| --- | --- |
| None | Not see the object at all |
| Read | See values but not change them |
| Write | See and change values |
| Reserve, Lock | Hold an object so that others cannot change it |
| Admin | Change the object's definition and security |

TM1 also has built-in groups. The most important is **ADMIN**, whose members have full access to everything.

## Sandboxes

A **sandbox** is a private set of a user's changes to a database. When a user works in a sandbox, their edits are kept separate from the shared data, which is called the **base**. They can try out what-if scenarios and, when they are ready, **commit** the changes to the base or discard them.

Sandboxes are useful in planning, where people want to test numbers before others see them.
