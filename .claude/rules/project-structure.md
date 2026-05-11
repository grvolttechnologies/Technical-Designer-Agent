# Project Folder Structure

version 8

`.volt/` is a **1:1 mirror of the project's Notion workspace**. Built and
maintained by the `volt-notion-sync` CLI (`.volt/.cli/volt-notion-sync.cjs`,
sources in `Volt-Notion-Sync` repo). Edits flow both ways: pull rewrites
local files from Notion, push sends local edits back to Notion.

If something doesn't belong in Notion (AL source, raw test data, generated
artifacts), it does **not** belong in `.volt/`. Put it in the repo root
under `extensions/<task-id>/...`. See "Code & raw test data" below.

---

## Top-level layout

```
.volt/
├── .cli/
│   └── volt-notion-sync.cjs            ← vendored sync engine (~1.2 MB)
├── .env                                ← NOTION_TOKEN — never committed
├── .sync-state.json                    ← sync engine bookkeeping
├── .volt-config.json                   ← minimal: client + Notion ids
├── .volt-sync.yml                      ← Notion ↔ local mapping (notionId pinned)
├── AGENTS.md                           ← agent context (per-project copy)
├── README.md                           ← local human-facing intro
├── docs/                               ← Notion page trees mirrored as markdown
│   ├── process-flows/index.md
│   ├── project-definition/index.md
│   └── waterfall-tasks/
│       ├── index.md
│       ├── extensions/index.md
│       ├── integrations/index.md
│       ├── migrations/index.md
│       ├── reports/index.md
│       └── issues/index.md
└── projectmanagement/                  ← Notion databases — one folder per DB
    ├── meetings/_index.json
    ├── waterfall-tasks/
    │   ├── _index.json                 ← schema (databaseId, dataSourceId, properties)
    │   ├── extension/                  ← Extension-type rows (SINGULAR — see note below)
    │   │   ├── email-4.md              ← row body (= the Overview tab in the platform UI)
    │   │   ├── email-4/                ← child-page folder, same slug as the row file
    │   │   │   ├── FDD.md              ← Functional Design Document
    │   │   │   ├── TDD.md              ← Technical Design Document
    │   │   │   ├── Documentation.md    ← end-user documentation
    │   │   │   ├── test-report.md      ← latest test run summary
    │   │   │   └── assets/             ← screenshots / images linked from the .md files
    │   │   ├── fa.md                   ← another row — early-stage (only FDD started)
    │   │   └── fa/
    │   │       └── FDD.md
    │   ├── reports/                    ← Report-type rows (PLURAL — naming quirk, not a typo)
    │   ├── integrations/index.md       ← resource intro page (no rows of this type yet)
    │   ├── migrations/index.md         ← resource intro page
    │   └── issues/index.md             ← resource intro page (open issues live in the `issues/` DB)
    ├── issues/_index.json
    ├── sprints/_index.json
    ├── milestones/_index.json
    ├── phases/_index.json
    ├── project-members/_index.json
    ├── process-flows/_index.json
    ├── abstract-process-flows/_index.json
    ├── notes/_index.json
    ├── sprint-goals/_index.json
    ├── transcripts/_index.json
    ├── project-notes/_index.json
    └── subtasks/_index.json
```

The 14 databases above match the **Volt project Notion template** (Project
Home → Settings → Quick Start Databases + the 3 user DBs under Settings).
A given project may have a different list — always check the project's
`.volt-sync.yml` for the authoritative database set.

---

## What each Notion folder is (read this before deciding where to look)

These folders mirror the QuickStart methodology. If you don't know what
something is, this is the short version — the long version lives in the
`PM-Agent` quickstart-methodology skill.

### Under `docs/` (Notion page trees, not databases)

| Folder | What it holds |
| --- | --- |
| `docs/project-definition/` | Top-level scope artifacts from diagnostics: business model, systems model (As-Is / To-Be), scope, assumptions, diagnostics presentation. The "why this project exists" reference. |
| `docs/process-flows/` | Narrative write-ups of the customer's business processes (driving vs. supporting, As-Is and To-Be). Pair these with the `process-flows` database rows when you need full context on a flow. |
| `docs/waterfall-tasks/` | Resource pages introducing the four waterfall task types — see below — plus an `issues/` resource page. The actual rows live under `projectmanagement/waterfall-tasks/`. |

### Under `projectmanagement/` (Notion databases — one folder per DB)

| Database | Purpose |
| --- | --- |
| `waterfall-tasks/` | The build backlog. Four kinds, identified by the `Type` property on each row: **Extension** (BC AL customizations), **Integration** (data flows in/out of D365 — source, destination, direction, protocol), **Migration** (data loads from legacy/spreadsheet/email sources with cutover assumptions), **Report** (workspaces, Power BI, SSRS, etc.). Story points = total days of effort (design + doc + dev + test). |
| `issues/` | Defects and gaps discovered during shadow/model/CRP. Frontmatter `properties.Is Complete` flips when closed. Gaps tracked by severity A>20d / B>10d / C>5d / D<5d. |
| `phases/` | Top-level project phases (Diagnostics, Sprints, CRPs, Hypercare). Each phase has a `recVersion` for concurrency-safe updates. |
| `milestones/` | Major dated checkpoints inside phases (CRP 1 / CRP 2 / CRP 3 / Go-Live, etc.). |
| `sprints/` | The 4-week iterations under the Sprints phase. Each sprint has shadow / model / review-rehearse / show-and-tell weeks. |
| `sprint-goals/` | Specific, measurable goals attached to a sprint ("model order entry and order modification"). Used to judge sprint pass/fail and CRP readiness. |
| `process-flows/` | Database rows for each business process flow — driving or supporting. Joined to sprints via `sprint-process-flow-mapping` (see PM-Agent's `readSprintProcessFlowMapping`). |
| `abstract-process-flows/` | Phase-level abstractions of process flows (used in multi-phase scoping at diagnostics). |
| `project-members/` | Team roster — Sunrise consultants + customer team — with roles. |
| `meetings/` | Scheduled meeting records (status, steering committee, show-and-tell, CRP, etc.). |
| `transcripts/` | Raw meeting/discovery session transcripts. Heavy text — most useful as upstream input to a waterfall task (drop a relevant transcript into `<row-slug>/input/` if you need to feed it to an agent). |
| `notes/` | Free-form project notes — typically created via `createProjectNote()` for daily/weekly summaries. |
| `project-notes/` | Companion to `notes/` for longer-running project documentation. |
| `subtasks/` | Sub-rows under waterfall-task rows when work needs to be split below story-point granularity. |

**Where to look for what you need (developer agent perspective):**

- "What am I building?" → `projectmanagement/waterfall-tasks/extension/<slug>.md` (the row body is the overview) + the four sub-pages in `extension/<slug>/`.
- "Why are we building it?" → the row's frontmatter `properties` (Type, Phase, Sprint relation) plus the relevant `docs/process-flows/` and `docs/project-definition/` pages.
- "Has the customer already raised this?" → search `projectmanagement/issues/*.md` by `properties.Process` or `properties.Sub Workflow`.
- "Who owns this / when is it due?" → row frontmatter (`Sunrise Owner`, `Customer Owner`, `Sprint`, `Story Points`) and the linked sprint/milestone rows.
- "What was discussed about this in a meeting?" → grep `projectmanagement/transcripts/` for the feature name; transcripts are big — don't ingest blindly.

If a database in the table above is missing from a given project, that's
fine — projects can drop DBs they don't use. Always confirm against
`.volt-sync.yml` first.

---

## How the mirror is shaped

Two file types under `projectmanagement/<db-slug>/`:

### `_index.json`

Generated on every pull. Holds the database schema — property names,
types, select options, relation targets — plus row count. Read this
**first** to understand any database before reading rows.

```json
{
  "databaseId": "35200f55-…",
  "dataSourceId": "35200f55-…",
  "schema": {
    "Task": { "id": "title", "type": "title", "title": {} },
    "Type": { "id": "rhrf", "type": "select", "select": { "options": [{ "name": "Extension" }, …] } },
    "Phase": { "id": "kq^k", "type": "relation", "relation": { "database_id": "…", "type": "single_property" } }
  },
  "rowCount": 1
}
```

### `<row-slug>.md` — the row body

Frontmatter holds `notion_id`, `notion_url`, `last_edited_time`, `title`,
`data_source_id`, and all the row's properties. Body is the page body
verbatim.

```markdown
---
notion_id: 35200f55-…
notion_url: https://app.notion.com/…
last_edited_time: 2026-04-30T19:33:00.000Z
title: Restaurant Module 2
data_source_id: 35200f55-…
properties:
  Task: Restaurant Module 2
  Task Id: EXT-00099
  Type: Extension
  Status: 00-NotStarted
---

# Restaurant Module 2

(body content)
```

### `<row-slug>/<page>.md` — child pages of a row

For body-heavy database rows (extensions, issues, process flows, etc.) we
mirror Notion sub-pages of the row as nested files under a folder named
after the row slug. Frontmatter on each child carries `parent_row_id`
back-pointing at the row.

The convention for a Waterfall Tasks **Extension** row is exactly four
sub-pages, living inside a folder named after the row's slug. Concrete
example: the "Email 4" extension is stored at

```
projectmanagement/waterfall-tasks/extension/
├── email-4.md             ← row body (frontmatter + Overview content)
└── email-4/               ← child-page folder (same slug, no .md)
    ├── FDD.md
    ├── TDD.md
    ├── Documentation.md
    ├── test-report.md
    └── assets/            ← optional images referenced from the .md files
```

| Sub-page in Notion | Local file              | Audience / purpose               |
| ------------------ | ----------------------- | -------------------------------- |
| `FDD`              | `FDD.md`                | Functional Design Document — client + functional reviewer |
| `TDD`              | `TDD.md`                | Technical Design Document — tech lead, AL devs |
| `Documentation`    | `Documentation.md`      | End-user documentation           |
| `test-report`      | `test-report.md`        | Latest test run summary (auto-updated by CI) |

Rows that haven't reached every stage still follow the shape — e.g. the
`fa` extension currently has only `fa.md` + `fa/FDD.md`. Don't create
empty stub files; the platform's stage dots key off file presence.

**Type-folder naming quirk:** Extensions land under `extension/`
(singular), Reports under `reports/` (plural). This is wired in the
platform's `KIND_FOLDER` map (`app/src/pages/ExtensionDetail.tsx`) — do
not "normalize" it. Integrations and Migrations are folder placeholders
today (just an `index.md`); when those row types start landing, expect
them under `integrations/<slug>.md` and `migrations/<slug>.md` by the
same nesting pattern.

The row body itself (`<row-slug>.md`) carries the original brief +
problem/solution from the Extension Template; the four sub-pages are the
deeper artifacts. The platform UI surfaces these as tabs on the
Extension Detail page (`Overview | FDD | TDD | Documentation | Test Report
| Input`) — the row body is what shows up under "Overview".

**Overview must link to its sub-pages.** Whenever you create or update
any of FDD / TDD / Documentation / test-report, also update the **top of
the row body** (`<row-slug>.md`) so it carries a small navigation block
linking out to each artifact that exists. The customer reads the
extension record in Notion — without these links they have to hunt
through child pages manually. Put it directly under the H1, before any
other content. Example:

```markdown
# Restaurant Module 2

**Artifacts:** [FDD](./Restaurant-Module-2/FDD.md) ·
[TDD](./Restaurant-Module-2/TDD.md) ·
[Documentation](./Restaurant-Module-2/Documentation.md) ·
[Test Report](./Restaurant-Module-2/test-report.md)

> Original brief / problem / solution follows…
```

Only link the artifacts that exist on disk — don't fabricate dead links
for stages that haven't run yet. On push, Notion turns these relative
paths into proper sub-page links back to the child pages.

**Filename casing**: slugify preserves case for child pages — uppercase
acronyms in Notion (`FDD`, `TDD`) stay uppercase on disk; multi-word
titles can be controlled by the literal Notion title (rename the Notion
page to `test-report` and the file lands as `test-report.md`).

---

## Sync (volt-notion-sync)

Local commands (`set -a && . .volt/.env && set +a` first):

```bash
node .volt/.cli/volt-notion-sync.cjs pull --repo .       # Notion → repo
node .volt/.cli/volt-notion-sync.cjs push --repo .       # repo → Notion
node .volt/.cli/volt-notion-sync.cjs inspect --repo .    # list children of rootPageId
node .volt/.cli/volt-notion-sync.cjs bootstrap <starter-page-id>   # build a fresh template workspace
```

`.github/workflows/volt-notion-sync.yml` runs the same CLI on
`repository_dispatch` (Notion webhook) and on schedule. The
`defaultDirection` in `.volt-sync.yml` is usually `both`; flip to `pull`
to lock Notion → repo only.

Hash-skipping in both pull and push prevents the webhook ↔ workflow loop:
unchanged files are detected via `.sync-state.json` and skipped.

### Bootstrapping a new project's Notion workspace

`volt-notion-sync bootstrap <starter-page-id>` creates the entire Volt
template — Project Home, Settings, Quick Start Databases, 14 databases
with full schemas + relations, the 3 resource pages (Process Flows /
Project Definition / Waterfall Tasks), and the 5 Waterfall Tasks
sub-pages (Extensions / Integrations / Migrations / Reports / Issues).
Source: `Volt-Notion-Sync/src/bootstrap/{template,run}.ts`.

The starter page must be a real (empty) Notion page that the integration
already has access to — Notion's API refuses to create workspace-root
pages from internal integrations. Create the page in Notion, share it
with the integration, then run.

---

## Code & raw test data — outside `.volt/`

`.volt/` is Notion only. Anything machine-generated or too large for
Notion stays in the repo root, joined to its Notion row via the
`Task Id` property.

```
<repo-root>/
├── BC/                                 ← Main BC AL app (app.json + src/)
├── BC Test/                            ← Test BC AL app
├── extensions/                         ← per-Waterfall-Task code & artifacts
│   └── EXT-00099/                      ← matches Notion Task Id
│       ├── src/                        ← AL source files
│       └── test-results/               ← raw test data, JSON, screenshots, logs
│           └── run_<timestamp>_<id>/
│               ├── results.json
│               ├── report.md
│               └── assets/*.png
└── .volt/                              ← Notion mirror
```

When working on extension `EXT-00099`:

- **Read the spec**: `.volt/projectmanagement/waterfall-tasks/extension/<slug>.md` + `extension/<slug>/{FDD,TDD,Documentation,test-report}.md`
- **Write source code**: `extensions/EXT-00099/src/`
- **Write raw test results**: `extensions/EXT-00099/test-results/run_*/...`
- **Update test-report summary**: edit `extension/<slug>/test-report.md` (auto-pushes to Notion on next sync)

The `Task Id` frontmatter property on the row is the join key.

---

## BC AL apps — repo root

BC apps still live at the repo root (e.g. `BC/`, `BC Test/`) with their
`app.json` AL manifest. These are not `.volt/` content. The Volt
platform's BC compile/deploy pipeline reads them directly from the repo
checkout, not from `.volt/`.

Per-environment credentials (tenantId, clientId, clientSecret) are
**not** in `.volt/` either — they belong in CI secrets / the platform's
secret store. If a project has them locally, they live in
`.volt/.env` (gitignored) or a separate uncommitted file.

---

## Reading rules for agents

1. **Always read `.volt/.volt-sync.yml` first** to learn which databases
   and pages exist for this project. Never assume the 14 standard
   databases; the project may add or remove some.
2. **Then `.volt/projectmanagement/<db>/_index.json`** for any database
   you need to reason about. The schema there is canonical.
3. **Then row markdown files** under `<db>/`. Frontmatter `properties` is
   structured; body is prose.
4. **For body-heavy artifacts** (extension specs, issue write-ups, design
   notes), check whether `<db>/<row-slug>/` exists. If it does, the row
   has child pages — read them too.

## Writing rules for agents

1. **Never edit `_index.json`** — schema lives in Notion. Edit there
   (or via the bootstrap script) and re-pull.
2. **Edit row markdown freely**, then run push (or commit + let CI push).
   Hash-skip on both sides prevents loops.
3. **Adding a new extension row**: easiest is via the Notion UI (create a
   new Waterfall Tasks row, type = Extension), then `pull`. To do it
   programmatically, use the Notion API directly (`pages.create` with
   `parent: { database_id: <waterfall-tasks-db-id> }`) and a `children`
   array following the Extension Template structure.
4. **Adding the four extension sub-pages**: create them as **child
   pages** of the row (`parent: { page_id: <row-id> }`). Names must be
   exactly `FDD`, `TDD`, `Documentation`, `test-report` for the file
   names to land as documented above.
5. **Source code and raw test data** go to `extensions/<task-id>/...`,
   never to `.volt/`. The `localIgnore` in `.volt-sync.yml` already
   excludes typical code paths but you should still keep the discipline.
6. **Keep the overview's artifact links current.** Any time you create or
   modify `FDD.md`, `TDD.md`, `Documentation.md`, or `test-report.md`
   under a waterfall-task row, update the **Artifacts** line at the top
   of that row's body (`<row-slug>.md`) so the Notion record shows live
   links to every artifact that exists. Don't link files that don't
   exist yet. See the "Overview must link to its sub-pages" note above.

## Forbidden moves

1. ❌ Editing files under `.volt/docs/**` or `.volt/projectmanagement/**`
   directly with no intent to push. Pull will overwrite them.
2. ❌ Putting AL source, large CSVs, or binaries inside `.volt/` —
   they'll get pushed to Notion as page bodies and mangled.
3. ❌ Changing `notion_id` in any frontmatter. It's the join key.
4. ❌ Committing `.env` or `.sync-state.json` edits made by hand.
5. ❌ Running `push` direction from a stale local checkout — diff
   against the latest pull first, or rely on the GH Action.

---

## Finding your work

| Ask | Read |
|---|---|
| "What does the workspace look like?" | `.volt-sync.yml` + each `_index.json` |
| "What's the spec for EXT-00099?" | `projectmanagement/waterfall-tasks/extension/<row-slug>.md` + `extension/<row-slug>/{FDD,TDD}.md` |
| "Show me open issues" | `projectmanagement/issues/*.md` — filter by frontmatter `properties.Is Complete: false` |
| "Where's the AL code for EXT-00099?" | `extensions/EXT-00099/src/` (repo root, outside `.volt/`) |
| "What's the latest test result?" | `projectmanagement/waterfall-tasks/extension/<row-slug>/test-report.md` (summary) + `extensions/EXT-00099/test-results/` (raw) |
| "Add a new extension" | Notion UI: new row in Waterfall Tasks (type=Extension), then add 4 child pages: FDD / TDD / Documentation / test-report. Pull. |
| "What automation does PM Task X run?" | `projectmanagement/pm-tasks/<row-slug>.md` body — if that DB exists in this project |
| "Bootstrap a new client workspace" | `node .volt/.cli/volt-notion-sync.cjs bootstrap <starter-page-id>` |
