# Sandboxes

A **sandbox** is a private workspace for cube data. When you write values in a sandbox, they are stored only in that sandbox. The shared data, called the **base**, and other users' sandboxes don't change until you **publish** the sandbox into base or **merge** it into another sandbox.

The [Concepts chapter](../concepts/database-objects.md#sandboxes) introduces sandboxes. This page shows how to work with them through the [REST API](index.md) and TurboIntegrator, and lists the behaviour that catches people out.

!!! note "v11 and v12"
    Sandboxes existed in TM1 v11 and work the same way in v12. If something here doesn't match your database, check IBM's documentation for your version.

## How a sandbox works

Suppose you want to try a higher January bike budget for the North region without anyone else seeing it.

| Cell (Sales cube) | Base | Your sandbox "Budget Plan 1" |
| --- | --- | --- |
| Budget, 2026, Jan, North, Bikes, Units | 10 | 15 |
| Budget, 2026, Jan, South, Bikes, Units | 20 | 20 (read from base) |

You wrote only one cell, so the sandbox stores only that one change. Every other cell is read through from base. A sandbox is not a copy of the cube.

Some other facts about sandboxes:

- **Each sandbox belongs to the user who created it.** Other users, including administrators, can't see it. A request for another user's sandbox returns "not found" (404).
- **Sandboxes persist.** Your changes are still there after you log out and back in.
- **Names are unique per user.** Two users can each have a sandbox called "Budget Plan 1". The name `Base` is reserved and can't be used.

## The sandbox object

`GET` on a sandbox returns an object like this:

```json
{
  "Name": "Budget Plan 1",
  "IsLoaded": false,
  "IsActive": false,
  "IsQueued": false,
  "IncludeInSandboxDimension": true
}
```

| Property | Meaning |
| --- | --- |
| `Name` | The sandbox's name. Unique for each user. |
| `IsLoaded` | The sandbox's data is in memory. |
| `IsActive` | It is the current session's active sandbox. |
| `IsQueued` | A publish or merge is waiting to run. |
| `IncludeInSandboxDimension` | The sandbox appears in the user's [sandbox dimension](#the-sandbox-dimension). |

## Lifecycle

The paths below are relative to the database, for example `.../api/v1/Databases('Sales')`. The exact start of the URL depends on how your database is hosted.

| Action | Request | Success |
| --- | --- | --- |
| Create | `POST /Sandboxes` with `{"Name": "Budget Plan 1"}` | 201 |
| List | `GET /Sandboxes` | 200 |
| Read one | `GET /Sandboxes('Budget Plan 1')` | 200 |
| Change the dimension flag | `PATCH /Sandboxes('Budget Plan 1')` with `{"IncludeInSandboxDimension": false}` | 200 |
| Load into memory | `POST /Sandboxes('Budget Plan 1')/tm1.Load` | |
| Save to disk and free memory | `POST /Sandboxes('Budget Plan 1')/tm1.Unload` | |
| Discard changes | `POST /Sandboxes('Budget Plan 1')/tm1.DiscardChanges` | 204 |
| Delete | `DELETE /Sandboxes('Budget Plan 1')` | 204 |

Notes on each step:

- **Create.** An empty name, or a name you already use, returns 400. If you leave out `IncludeInSandboxDimension`, it defaults to `true`.
- **Load and unload.** You rarely need these. Any request that names a sandbox loads it automatically, and it stays loaded afterwards.
- **Discard changes vs delete.** `tm1.DiscardChanges` empties the sandbox, so every cell reads from base again, but the sandbox itself remains. It takes no parameters; sending any returns 400. `DELETE` removes the sandbox entirely, and a later `GET` returns 404.

## Reading and writing in a sandbox

To run a request in a sandbox, add the query option `!sandbox=<name>`. If the sandbox doesn't exist, the server returns 400.

For example, to write the North bike units from the table above, send the cell write to:

```text
POST /Cubes('Sales')/tm1.UpdateCells?!sandbox=Budget%20Plan%201
```

The sandbox option works with:

| Operation | Example |
| --- | --- |
| Run MDX | `POST /ExecuteMDX?$expand=Cells&!sandbox=Budget%20Plan%201` |
| Run a saved view | `POST /Cubes('Sales')/Views('Budget')/tm1.Execute?!sandbox=...` |
| Write cells | `POST /Cubes('Sales')/tm1.UpdateCells?!sandbox=...` |
| Read or write cellset cells | `GET` or `PATCH /Cellsets('<id>')/Cells?!sandbox=...` |
| Change sets | `POST /BeginChangeSet?!sandbox=...` (see [below](#change-sets-and-undo)) |
| Run a TI process | `POST /Processes('Load Budget')/tm1.ExecuteWithReturn?!sandbox=...` |

!!! warning "Encode the name"
    Spaces and special characters in a sandbox name must be URL-encoded. `Budget Plan 1` becomes `Budget%20Plan%201`, and `R&D` becomes `R%26D`.

## Publishing and merging

Your sandbox changes reach other users only when you move them somewhere else:

- **Publish** copies the sandbox's changes into base: `POST /Sandboxes('Budget Plan 1')/tm1.Publish`. Planning Analytics Workspace calls this **committing** the sandbox.
- **Merge** copies them into another of your sandboxes: `POST /Sandboxes('Budget Plan 1')/tm1.Merge`, with a `Target` naming the other sandbox.

Both accept these parameters:

| Parameter | Effect |
| --- | --- |
| `CleanAfter` | Empty the source sandbox after the operation. |
| `AllowQueueing` | Let the server queue the operation if it can't run now. A queued publish returns 202 instead of completing straight away. |
| `ConflictResolution` | What to do when the target cell has also changed: `0` keep the source value (default), `1` keep the target value, `2` resolve each conflict individually. |

`Publish` doesn't accept `Target`; sending one returns 400. `Merge` requires it, and merging a sandbox into itself also returns 400. Either operation fails if you don't have enough rights on one of the cubes involved.

## Change sets and undo

A **change set** groups cell writes so you can undo them together, much like an undo button for a batch of edits.

1. `POST /BeginChangeSet?!sandbox=...` returns a change set ID.
2. Write cells, adding `&!ChangeSet=<id>` to each request.
3. `POST /EndChangeSet?!sandbox=...` with `{"ChangeSetID": "<id>"}` closes the set.
4. `POST /UndoChangeSet?!sandbox=...` reverts every cell in the set.

## The sandbox dimension

If the database setting **`EnableSandboxDimension`** is on, each user gets a **virtual dimension called `Sandboxes`**. It contains `Base` plus each of the user's own sandboxes that have `IncludeInSandboxDimension` set to true. The setting is off by default.

The sandbox dimension lets you compare sandboxes side by side in one query. This MDX puts base and two sandboxes on the columns:

```text
SELECT
  {[Sandboxes].[Sandboxes].[Base],
   [Sandboxes].[Sandboxes].[Budget Plan 1],
   [Sandboxes].[Sandboxes].[Budget Plan 2]} ON COLUMNS,
  {[Region].[Region].[North], [Region].[Region].[South]} ON ROWS
FROM [Sales]
WHERE ([Version].[Version].[Budget], [Year].[Year].[2026],
       [Period].[Period].[Jan], [Product].[Product].[Bikes],
       [Sales Measure].[Sales Measure].[Units])
```

Rules for the sandbox dimension:

- **It is read-only.** You can't add elements, hierarchies, subsets or attributes, rename or change `Base`, delete the dimension, or save another dimension under its name. Each of these returns 400. To change its contents, create, delete or update sandboxes.
- **Use it on one axis only.** Putting `[Sandboxes].[Sandboxes]` on two axes returns 400 ("Hierarchy ... encountered more than once in the query").
- **You can't spread across sandboxes.** A [spreading](../concepts/cells-and-calculation.md#consolidated-cells) request that would write to several sandboxes at once fails.

### Effect on cellset IDs

A **cellset** is the result of an MDX query or view held on the server. Clients get a cellset ID and use it to read or write the cells later.

With `EnableSandboxDimension` on, creating or deleting a sandbox can invalidate your existing cellsets. The next request with an old ID returns **404** (error 278: "can not be found in collection of type 'Cellset'"). The server doesn't warn you in advance.

| Action, with `EnableSandboxDimension` on | Existing cellset ID afterwards |
| --- | --- |
| Create a sandbox, included in the dimension | 404 |
| Delete a sandbox that is in the dimension | 404 |
| Create a sandbox with `IncludeInSandboxDimension: false` | **404** |
| Delete a sandbox that is not in the dimension | Still works |

With `EnableSandboxDimension` off, none of these actions affect cellsets.

The third row is the surprise: creating a sandbox invalidates cellsets even when the sandbox is not added to the dimension.

!!! tip "Handling this in your code"
    If your client keeps cellset IDs, be ready for a 404 after any sandbox is created or deleted: run the query again to get a new ID. Release cellsets you no longer need with `DELETE /Cellsets('<id>')`.

## Rules, MDX and security in a sandbox

- **Rules** calculate using the sandbox's values. If you change North bike Units in a sandbox, the rule-calculated Revenue for that cell changes in the sandbox too. [Feeders](../concepts/cells-and-calculation.md) work within the sandbox.
- **Rule-calculated cells can't be written**, in a sandbox or anywhere else. The write is reported with a `HasMinorErrors` status.
- **Calculated members** in MDX (`WITH MEMBER`) see sandbox values. A calculated member can check which sandbox it is in with `[Sandboxes].CURRENTMEMBER.NAME`.
- **Cell security still applies.** Writing a cell you have no write access to returns 400 with `CubeCellWriteStatusElementNoWriteAccess`.
- **`NON EMPTY` can hide sandbox values.** If you use `!sandbox=` but don't put the sandbox in the `WHERE` clause, `NON EMPTY` may treat cells that have values only in the sandbox as empty, and drop their rows. Put the sandbox in `WHERE` to avoid this.

## Sandboxes in TurboIntegrator

A [TurboIntegrator](../turbointegrator/index.md) process can read and write a sandbox instead of base:

```text
# Point CellGetN, CellPutN and similar functions at a sandbox
ServerActiveSandboxSet('Budget Plan 1');
SetUseActiveSandboxProperty(1);

CellPutN(15, 'Sales', 'Budget', '2026', 'Jan', 'North', 'Bikes', 'Units');

# Switch back to base
SetUseActiveSandboxProperty(0);
```

To tidy up old sandboxes in bulk, use `ServerSandboxesDelete`. It takes a filter made of `attribute:operator:value` conditions joined by commas, and deletes only **unloaded** sandboxes that match.

| Attribute | Operators |
| --- | --- |
| `name` | `:=:`, `:containing:` |
| `client` | `:=:` |
| `group` | `:=:` |
| `creationdate`, `accessdate`, `updatedate` | `:<:`, `:=:`, `:>:`, with dates as `YYYY-MM-DD` |
| `size` | `:<:`, `:>:`, with units KB, MB or GB |

For example, to delete one user's sandboxes not used since the start of 2026:

```text
ServerSandboxesDelete('client:=:Ana, accessdate:<:2026-01-01');
```

If a sandbox name contains `:`, `,` or `=`, pass your own delimiters as the second and third arguments.

!!! warning
    Check the `ServerSandboxesDelete` filter syntax, including spaces after commas, against IBM's TurboIntegrator reference before running it on a real database.

## Gotchas

| Symptom | Cause |
| --- | --- |
| A cellset ID suddenly returns 404 | `EnableSandboxDimension` is on and a sandbox was created or deleted. Run the query again. See [Effect on cellset IDs](#effect-on-cellset-ids). |
| `Publish` returns 400 | You passed `Target`. Only `Merge` takes one. |
| `Merge` returns 400 | `Target` is missing or names the source sandbox. |
| `DiscardChanges` returns 400 | You passed a parameter. It takes none. |
| A request returns 400 naming the sandbox | The sandbox doesn't exist, or its name isn't URL-encoded. |
| `ServerSandboxesDelete` leaves some sandboxes | It skips sandboxes that are loaded. |
| `NON EMPTY` drops rows you changed | The sandbox isn't in the `WHERE` clause. |

??? question "1. You write one cell in a new sandbox. How many cells does the sandbox store?"
    One. A sandbox stores only the cells you changed; every other cell reads through from base.

??? question "2. What is the difference between `tm1.DiscardChanges` and `DELETE`?"
    `tm1.DiscardChanges` empties the sandbox but keeps it, so you can carry on using it. `DELETE` removes the sandbox altogether.

??? question "3. `EnableSandboxDimension` is on. You create a sandbox with `IncludeInSandboxDimension: false`. Will your open cellset still work?"
    No. Creating any sandbox invalidates existing cellsets while the setting is on, so your cellset ID returns 404. Run the query again.
