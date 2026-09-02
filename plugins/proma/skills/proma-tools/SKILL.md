---
name: proma-tools
description: Use when a request touches Proma - reading or changing a Space, System, Sheet, column, view or row, setting up an automation, or creating or reading form submissions - including when the user says "in Proma", names a Proma space/system/sheet, pastes a proma.ai link, or asks where some record lives in their workspace.
---

# Working with Proma

Proma is reached exclusively through the `proma` MCP server that this plugin
registers (`https://server.proma.ai/mcp/connectors`). There is no CLI and no
local database - if the server is not connected, nothing in this skill works.

> **Status of this file:** every place that needs a real MCP tool name is marked
> `TODO(tool-name)`. Those markers are deliberate placeholders, not tool names.
> Never guess a tool name in their place: list the tools the `proma` server
> actually exposes and use those. If a marker is still present when you need it,
> say so instead of inventing a call.

## Before anything else

1. Confirm the `proma` server is connected in this session. If it is not, stop
   and tell the user - a failed connect is a connection problem, not a missing
   capability, and it is usually fixed with `/mcp` (reconnect / re-authorize).
2. Read the server's actual tool list and prefer it over anything written here.
   The tool set is the source of truth; this file is the map of the territory.
3. Authentication is OAuth, negotiated by Claude Code at connect time. Never ask
   the user for a token, and never put one in a file.

## The data model

Learn these five nouns before making any call - most mistakes are addressing
mistakes, not argument mistakes.

- **Space** - the top-level container, roughly "a workspace or a team area".
  Everything else lives inside exactly one Space.
- **System** - a grouping of related Sheets inside a Space. Think of it as an
  app: a CRM, an intake pipeline, an ops tracker.
- **Sheet** - the actual table of records inside a System. This is where data
  lives.
- **Column** - a typed field on a Sheet. The type matters: writing a value in
  the wrong shape (a plain string into a select, a date in the wrong format, an
  unresolved reference into a relation) is the most common write failure.
- **View** - a saved lens over a Sheet (filters, sort, visible columns, grouping).
  A View never holds its own rows; it is a way of looking at the Sheet's rows.
- **Row** - one record in a Sheet, addressed by id.

On top of that:

- **Automation** - a trigger/condition/action rule attached to a Sheet (or to a
  System) that fires when records change or on a schedule.
- **Form** - a public or shared intake surface bound to a Sheet; a submission
  becomes a Row.

Addressing rule: identifiers are scoped downward. Resolve
Space -> System -> Sheet -> Column/View/Row rather than assuming a bare name is
unique. Two Spaces can both have a Sheet called "Tasks".

## Standard working loop

1. **Locate** - resolve names to ids before touching anything.
   `TODO(tool-name)` to list Spaces, `TODO(tool-name)` to list Systems in a
   Space, `TODO(tool-name)` to list Sheets in a System.
2. **Inspect the schema** - never write to a Sheet whose columns you have not
   read. `TODO(tool-name)` returns the Sheet's columns with their types and
   options; `TODO(tool-name)` lists its Views.
3. **Read** - `TODO(tool-name)` to query rows (filter, sort, paginate). Prefer
   reading through a View when the user refers to one by name, since the View
   already encodes the filter they mean.
4. **Write** - `TODO(tool-name)` to create a row, `TODO(tool-name)` to update
   one, `TODO(tool-name)` to delete one. Match each value to its column type.
5. **Verify** - read the row back after a write that mattered, and report the
   real result rather than assuming the write landed.

## Reading

- Filter server-side with the query tool's own arguments rather than pulling a
  whole Sheet and filtering locally; Sheets can be large.
- Paginate to completion when the user asks for a count or a total - a first
  page is not an answer.
- When the user names a View ("the Overdue view"), read through that View so
  their filter definition is honoured instead of re-deriving it.
- Reference/relation columns come back as ids. Resolve them to something human
  before quoting them back to the user.

## Writing

- Create: `TODO(tool-name)`. Update: `TODO(tool-name)`. Delete: `TODO(tool-name)`.
- Confirm with the user before deleting rows, before bulk edits, and before any
  change to a Sheet's structure (adding, retyping or removing a Column via
  `TODO(tool-name)`) - schema changes affect every row and every automation
  bound to that Sheet.
- Prefer one batched call over a loop of single-row writes where the server
  offers a batch form; check the tool list for it rather than assuming.
- Partial failure is real. If a multi-row write reports some rows rejected, say
  which ones and why - do not report the operation as clean.

## Automations

Automations are trigger -> condition -> action rules. List them with
`TODO(tool-name)`, read one with `TODO(tool-name)`, create or change one with
`TODO(tool-name)`, and enable/disable one with `TODO(tool-name)`.

- Treat an automation as production wiring: read the existing rule before
  editing it, and describe the behaviour change to the user before applying it.
- Remember that your own writes can trip automations. A bulk import into a Sheet
  with a "on row created" rule may fire it once per row - warn the user first.

## Forms

A Form is an intake surface bound to a Sheet; each submission lands as a Row.
List forms with `TODO(tool-name)`, read a form's definition with
`TODO(tool-name)`, create or update one with `TODO(tool-name)`, and read
submissions with `TODO(tool-name)`.

- A form's fields are backed by the Sheet's Columns - a field can only collect
  what a Column can store, so check the Sheet schema first.
- Sharing a form link makes it reachable by whoever holds the link. Confirm the
  intended audience with the user before creating or publishing one.

## Reporting back

- Quote ids alongside names when you report what you changed, so the user can
  find the record.
- If a tool errors, report the server's actual message. Auth errors ("re-connect
  in `/mcp`"), permission errors ("your account cannot write to that Space") and
  validation errors ("that column expects one of ...") need different fixes.
- Never fabricate a row, a count, or a successful write.
