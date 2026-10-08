---
name: proma-tools
description: Use when a request touches Proma through the Proma MCP server - reading or changing a Space, System, Dataset (sheet), column, interface (view), form or row; building a new system or adding to an existing one; setting up, checking or debugging an automation and its runs; or whenever the user says "in Proma", names a Proma space, system or dataset, or pastes a proma.ai link.
---

# Working with Proma

## What Proma is

Proma is a no-code work and data platform. Everything is addressed top-down:

```
Space ─► System ─► Dataset ─► Column      (users may still say "sheet" for dataset)
                      │
                      ├─► Interface       (grids, kanban, calendar, forms … ; old word: "view")
                      └─► Row
System ─► Role, Automation
A Form is an Interface type.
```

- A **System** is an app: a CRM, an intake pipeline, an ops tracker.
- A **Dataset** is the table where rows live. Its **Columns** are typed.
- An **Interface** is how an audience sees and edits a dataset. It holds no rows
  of its own. **Roles** decide which interfaces each person opens.
- Names are only unique within their parent. Two systems can both have a dataset
  called "Tasks", so resolve Space → System → Dataset before trusting a bare name.

Every tool runs with the user's own Proma permissions.

## Before anything else

1. **The Proma connection must be connected.** If the Proma tools are missing or
   fail with an authentication error, stop and say so. That is a connection
   problem, not a missing capability, and the user fixes it by reconnecting or
   re-authorizing Proma in their client. `describe_proma` is the only tool that
   works without sign-in.
2. **Never ask the user for a token**, API key or password, and never write one
   to a file. Sign-in is OAuth, handled by the client.
3. **Scopes.** A connection is granted `proma:read` and/or `proma:write`.
   - A read-only connection still *lists* the write tools, but calling one is
     refused, and the error names the missing scope.
   - Scopes are fixed when the connection opens. To get more, the user
     re-authorizes the connection in their client.
   - On a scope refusal, do not retry and do not look for a workaround. Tell the
     user which scope is missing and how to add it.

## Look before building

`get_context` resolves the numeric ids everything else needs: spaces, systems,
datasets, columns, roles, interfaces, designs. Pick the `kind`, then filter with
`query` and `parent_id` rather than paging. `limit` is 50 at most, so a broad
listing can be cut short.

Two things `get_context` returns that are easy to miss:

- A column item carries its `type`. That is what decides whether it can be a
  `match_column`, a `group_by`, or a lookup target.
- An options / states / checklist column carries `domain`, the exact labels it
  accepts. **Use them verbatim.** Comparisons are literal, so a wrong label or a
  case difference builds fine, runs green, and never fires.
  `domain.truncated` means you are seeing a prefix, not the whole set.

Then `get_system` (`id`) for a snapshot of one system before revising it. Set
`include_automation_flows` only when you need the full flows.

For any field that takes an AI model, pick from `list_ai_models`, which lists
the models and Proma tiers this organization can use.

## Reading data

Two tools, for different questions:

| Use | When |
| --- | --- |
| `read_rows` | Structured rows from one dataset (`system_id`, `dataset_id`), up to 100 per page. Pass `interface_id` to read the rows exactly as that interface's audience sees them, or `column_ids` to narrow the columns instead. |
| `run_sql` | Aggregates, group-bys and joins across the datasets of one system. SELECT only. Table names are the dataset names in double quotes. Check the dialect with `get_reference('sql')`. |

- `run_sql` **ignores interface and role filters.** Never use it to answer "what
  does this role see"; use `read_rows` with that `interface_id`. When a SQL total
  could differ from what a user sees in their interface, say so.
- When the user names an interface ("the Overdue board"), read through it so
  their filter definition is honoured instead of re-deriving it.
- When asked for a count or a total, paginate `read_rows` to the end or use a SQL
  aggregate. A first page is not an answer.

## Writing rows

`write_rows` writes up to 50 rows per call.

- `mode` is `append` or `upsert`. Upsert needs a `match_column` and
  **overwrites** the matching row's values.
- Resolve dataset and column ids with `get_context` first, and shape each value
  to its column `type` and `domain`. Writing against a guessed column fails.
- Formula columns are refused. Leave them out.
- It needs admin access on the system. That is a Proma permission, separate from
  the connection's scope.
- **Dry-run imports** with `validate_only` first. Fix what it reports, then write.
- **Confirm with the user** before bulk writes and before any upsert.
- **Writes fire automations.** Automations on the dataset run for rows you write,
  so a bulk import into a dataset with a row-created automation can fire it once
  per row. Check `list_automations` and warn the user first.
- Problems come back per row in `issues`. Report which rows were rejected and
  why. **Never call a partial write clean.**
- Read back the rows after a write that mattered, and report what is actually
  there.

## Building a system

```
get_spec_guide  →  write the spec  →  validate_spec  →  apply_spec
   (call FIRST)                          (loop until valid)
```

- `get_spec_guide` first, every time. Per-block detail is not in it: open the
  drawer you need with `get_reference` (columns, interfaces, roles, sidebar,
  connections, form_detail, knowledge_base, agents, campaigns, dashboards,
  designs, sample_data, editing, automations, logic, sql). Whole worked specs
  come from `get_example_system`.
- `validate_spec` is a dry run. Fix all `issues` (each has a JSON path), validate
  again, repeat. `provisional_findings` were judged against a still-broken
  document: read them for direction, do not chase each one.
- When it goes valid you get a `plan`. **Read it against what the user actually
  asked for.** A spec can be valid and still be the wrong system.
- If the spec defines roles you also get an `audience_plan`. Show it to the user:
  "here is what each person sees when they log in" is a question they can answer.
  `apply_spec` requires that response's `spec_digest` for any spec with roles.
- `apply_spec` returns `audience_check`. `role_opens_nothing` and
  `home_shows_no_rows` both mean somebody logs in to a blank screen. Report them.
- **Sample data:** only pass `sample_data` after the user explicitly confirms they
  want demo rows. Otherwise build the empty structure.
- Pass a stable `client_request_id` so a retry after a timeout does not build a
  second copy of the system.

## Editing and deleting structure

**Edits are additive only.** To add to an existing system, call `apply_spec`
with `system.id` set, give the dataset its real numeric `id`, and list only the
new columns and interfaces. `get_reference('editing')` covers the details.

Removal goes through three tools, and each one is a **dry run by default**:

| Tool | Removes |
| --- | --- |
| `delete_columns` | Columns. The report says what would break. |
| `delete_interfaces` | Interfaces. |
| `delete_datasets` | Datasets **and their rows, which cannot be recovered.** Blocked while a lookup in another dataset points into it. |

1. Call the delete tool without `confirm` and read the report.
2. Show the user the report: what goes, and what would break.
3. Call again with `confirm: true` only after the user agrees to that report.

Never delete as part of a tidy-up the user did not ask for. For
`delete_datasets`, say plainly that the rows are gone for good.

## Forms

A Form is an Interface type, so find it with `get_context` like any interface.

```
get_form_design (form_id)  →  update_form_design
```

- `get_form_design` returns everything about one form. Read it before changing
  anything.
- `update_form_design` changes settings, questions, `show_when` conditions,
  sections and branding.
- `required` is a **column** property. Setting it changes the column, so it
  affects every form on that dataset. Tell the user before you set it.

## Automations

```
get_automation_guide → validate_automation_spec → apply_automation_spec → check_automation
```

- `get_automation_guide` first. It covers triggers, control flow and the native
  action palette.
- Prefer the guide's native action palette. Only reach for `search_actions` when
  it genuinely cannot express the action. Then `get_action_schema` for the chosen
  action's inputs, `get_field_options` for a dynamic field's valid choices, and
  `list_connections` for the organization's authenticated connections to that
  piece.
- Call `get_action_output` to see what a trigger or step outputs **before**
  writing `{{...}}` templates against it.
- `validate_automation_spec` is a dry run against the live action schemas.
- `apply_automation_spec` rolls back **per automation, not per spec**. On
  `applied: 'partial'`, everything in `created` is already live and published.
  Retry from `failed_at` onward, never the whole document.
- To change an existing automation, give it its `id` from `list_automations`;
  its flow is rebuilt in place. Applying a name that already exists in the
  system without an `id` is refused. Never rename it to get past that refusal:
  that publishes a second automation firing on the same event.
- A `schedule` automation is created **switched off**. Say so to the user rather
  than reporting it as running.
- `check_automation` after building. ActivePieces reports a step that silently
  wrote nothing as SUCCEEDED, so a green run does not mean a working automation.
  This is the only way to catch a bad `{{...}}` template without waiting for a run.
- Once it has run, `get_automation_run` shows what each template actually
  resolved to against real data.

Managing what already exists:

| Tool | Use |
| --- | --- |
| `list_automations` | The automations of one system (`system_id`). |
| `list_automation_runs` | Run history. `status: 'failures'` returns every failing kind. |
| `get_automation_run` | One run, step by step. It contains real customer data: quote only what is needed. |
| `toggle_automation` | Enable or disable one automation. |
| `delete_automation` | Delete one automation. It also removes the underlying flow. Confirm with the user first. |

To debug: `list_automations` → `list_automation_runs` with `status: 'failures'`
→ `get_automation_run` on a failing run → fix the spec →
`validate_automation_spec` → `apply_automation_spec` → `check_automation`.

**Treat an automation as live wiring.** Read the existing automation
(`get_system` with `include_automation_flows`) before editing it, describe the
behaviour change to the user before applying it, and remember that your own row
writes can trigger it.

## Reporting back

- Quote ids alongside names when you report what you changed, so the user can
  find the record.
- If a tool errors, report the server's actual message. Connection errors
  (reconnect or re-authorize Proma in the client), scope errors (the error names
  the missing scope), permission errors ("your account cannot write to that
  system") and validation errors ("that column expects one of ...") need
  different fixes.
- Never fabricate a row, a count, or a successful write.
