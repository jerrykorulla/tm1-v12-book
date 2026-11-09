# Glossary

Address
:   The list of elements, one from each of a cube's dimensions in the cube's dimension order, that identifies a single cell. For example, `Actual, 2026, Jan, North, Bikes, Units`. Also called a tuple. See [Cells](cubes-and-dimensions.md#cells).

Alias
:   An alternative name for an element, stored as an attribute. Must be unique in the dimension.

Attribute
:   Extra information stored against each element of a dimension. Can be string, numeric or alias.

Cell
:   One value in a cube, identified by its address: one element from each of the cube's dimensions.

Child
:   An element that rolls up into a consolidated element, its parent.

Chore
:   A scheduled or on-demand sequence of TurboIntegrator processes.

Consolidated element
:   An element whose value is the weighted sum of its children. Type C.

Control object
:   A cube, dimension or other object that TM1 uses to store its own settings. Names start with `}`.

Cube
:   A multidimensional set of data, defined by an ordered list of dimensions.

Database
:   In TM1 v12, one complete model: its cubes, dimensions, rules, processes, chores and security. Called a "server" in v11.

Dimension
:   A list of elements that describe one aspect of data, such as Period or Region.

Element
:   One item in a dimension, such as Jan in the Period dimension.

Feeder
:   A statement in a rule that tells TM1 which rule-calculated cells will hold values, so that totals include them.

Hierarchy
:   One arrangement of a dimension's elements into totals. A dimension can have several.

Leaf element
:   An element with no children, at level 0.

Level
:   How far an element sits above the bottom of a hierarchy. Leaf elements are level 0.

MDX
:   Multidimensional Expressions, a query language for cubes. TM1 uses it for dynamic subsets, MDX views and reports.

PAfE
:   Planning Analytics for Microsoft Excel, the Excel add-in for TM1.

PAW
:   Planning Analytics Workspace, the web interface for TM1.

Process
:   A TurboIntegrator script that runs inside the database. Also called a TI process.

REST API
:   The HTTP interface to TM1. In v12, the only programming interface.

Rule
:   A formula that calculates cell values in a cube.

Sandbox
:   A private set of a user's changes, kept separate from the shared base data until committed.

Sparsity
:   The fact that most possible cells in a cube are empty. TM1 stores only non-empty cells.

Spreading
:   Entering a value into a consolidated cell and letting TM1 distribute it to the leaf cells below.

Subset
:   A list of elements chosen from one hierarchy, either fixed (static) or defined by MDX (dynamic).

TurboIntegrator (TI)
:   TM1's built-in scripting language for loading data and maintaining the database.

View
:   A saved layout of a cube: which dimensions are on rows and columns, and which elements are shown.

Weight
:   The factor a child's value is multiplied by when it rolls up into its parent. Default 1.
