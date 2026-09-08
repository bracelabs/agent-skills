---
name: project-scaffold
description: Create an Org Standard (a team's shared project convention set) and apply it to a project — from a bundled Built-in Starter, by bootstrapping from existing assets, or from an existing Org Standard. Use to bootstrap a project or roll out maintained structural and operational conventions; not to audit an existing project (use project-scaffold-audit) or curate the Built-in Starter (use project-scaffold-maintain).
license: MIT
---

# Project Scaffold

Build a project standard and apply it. This skill is a mechanism, not a fixed
template: it creates an **Org Standard** (the standard a user or organization
grows over time) and applies it to projects. A small **Built-in Starter** ships
with the skill only as a starting point. The Org Standard itself is grown by
`project-scaffold-audit` (Project → Org Standard) and `project-scaffold-maintain`
(Org Standard → Built-in Starter).

```
Built-in Starter   (bundled, generic; curated via project-scaffold-maintain)
      ↓  create
Org Standard       ($PROJECT_SCAFFOLD_HOME, grown via project-scaffold-audit)
      ↓  apply
Project
```

Non-goals: transferring product code, dependencies, domain rules, or unrequested
technology decisions.

## Locations

- **Built-in Starter:** `starter/` in this skill directory. Read-only at runtime.
- **Org Standard:** `$PROJECT_SCAFFOLD_HOME` if set, else
  `~/.config/agent-skills/project-scaffold/`. The Org Standard's content is
  `scaffold/` under that directory. It lives outside any skill install directory,
  so `gh skill install` / `gh skill update` never touch it.
- **Project marker:** `.project-scaffold.json` at a project root records which
  Org Standard and ref were applied. Schema and rules: [scope.md](references/scope.md#project-marker).

What may be read, in this skill and the other two, is defined once in
[scope.md](references/scope.md). Do not widen it from here.

## Choose an init pattern

Before applying, resolve which Org Standard to use. Offer the user these three
patterns and follow the one chosen. Details and the git-management follow-up are
in [org-standard.md](references/org-standard.md).

1. **Built-in Starter** — copy `starter/` into a new Org Standard, adjusting only
   what the user requests. Fast, generic.
2. **Bootstrap from existing assets** — scan the user's existing projects,
   AGENTS.md files, handbook, or guidelines once, synthesize an Org Standard, and
   present it for approval. Uses the analysis method in
   [../project-scaffold-audit/references/analysis.md](../project-scaffold-audit/references/analysis.md)
   (requires the `project-scaffold-audit` skill installed). See
   [bootstrap.md](references/bootstrap.md).
3. **Use existing Standard** — an Org Standard already exists at
   `$PROJECT_SCAFFOLD_HOME/scaffold/`, or the user points at one (local path or
   git remote). Apply it, applying the rename map in
   [org-standard.md](references/org-standard.md) (a scaffold source is not copied
   verbatim — `*.tmpl` and `gitignore` are renamed, and the scaffold's own
   `README.md` / `.gitignore` are not copied).

For patterns 1 and 2, after the `scaffold/` content is approved, offer to place it
under git and, if the user wants, create a remote repository. "Not git-managed" is
a valid choice. Never create a remote or push without explicit approval.

## Preflight (read-only, before applying)

Validate the scaffold source and report any failure instead of applying:

- payload paths resolve through the rename map without duplicate destinations;
  exclude root scaffold metadata before mapping
- `scaffold.config.json`, if present, satisfies
  [the config rules](references/scope.md#operationalfiles)
- ignore rules and operational instructions agree on the organization's chosen
  temporary-artifact boundary (the Starter uses an anchored `/tmp/` rule)
- internal Markdown links resolve in the mapped project layout
- on re-application, the applied version and its scope resolve as described in
  [scope.md](references/scope.md#resolving-a-standard-version); report
  unavailable history as a coverage limit, not as a failure of the new source
- the standard's working tree is clean, or the plan will carry the dirty-standard
  notice and record `scaffoldRefExact: false`

## Apply an Org Standard to a project

Do not modify a project until the user approves a concrete plan.

1. Determine mode: **new** (empty or newly requested target) or **existing**.
   Preserve current user work either way.
2. Inspect the target within [the resolved scope](references/scope.md#assembling-the-scope-for-a-run).
   On re-application that is the union of the recovered applied scope and the
   selected standard's scope, including old-only paths.
3. Select only what the user requested or an approved `project-scaffold-audit`
   report identifies. An existing Org Standard is not permission to rewrite every
   difference.
4. Present the plan: each file/dir to create, change, or remove; the convention
   and its source; compatibility impact; intentional local exceptions left
   untouched; any ignore-rule changes; and the matching operational rule
   separating temporary artifacts from durable documentation. List
   [payload that runs](references/org-standard.md#payload-that-runs) as its own
   category, with its trigger and effect, for separate approval.
5. On approval, make only the listed changes. If the plan changes materially,
   re-present and ask again.
6. Write or update `.project-scaffold.json` per
   [the marker contract](references/scope.md#project-marker): record the standard's
   Git remote as `scaffoldSource` when it has one, set `scaffoldRefExact`, and
   carry `retainedOperationalFiles` forward. Name any decision to retire retained
   inspection scope in the approved plan.
7. Verify the changed operational files. Report target location, applied
   conventions, and remaining exceptions. If the target is not a git worktree,
   report verification without claiming git status.

### Removing project files

Preserving the user's work is the default, so this skill creates and changes
files; it does not tidy a project on its own initiative. Removal or consolidation
of an existing project file is permitted only when it implements a
`Remove-Migrate` finding that a `project-scaffold-audit` report targeted at the
Project and the user has approved.

Even then: quote the finding, show the current content of each affected file in
the plan, prefer merging content into its replacement over deleting it, and get
approval for the removals separately from the rest of the plan. Never remove a
file merely because the Org Standard has no counterpart for it.

## Built-in Starter shape

`starter/` provides a generic application skeleton. Treat it as a pattern, not a
mandatory structure — the Org Standard and the target project's own rules win.

```
AGENTS.md            # from starter/AGENTS.md.tmpl
README.md            # from starter/README.md.tmpl
.gitignore           # from starter/gitignore; must keep an anchored /tmp/ rule
tmp/                 # ignored; created only when temporary work is needed
docs/
├── README.md
├── AGENTS.md         # how to use docs/ — directory roles, workflow, checklist
├── 00_templates/    # reusable document templates (requirements, spec, decision-record, discussion)
├── 01_product/      # purpose, users, value
├── 02_requirements/ # functional/non-functional requirements, acceptance criteria
├── 03_spec/         # architecture, data, APIs, operations
├── 04_decisions/    # decision records: context, decision, rationale, rejected options
├── 05_discussions/  # unresolved questions and research
└── 06_execution/    # delivery plans, QA, releases, operations
```

`docs/AGENTS.md` carries the rules for using those directories, which keeps them
in the default read scope without listing the product-bearing files inside them.

## Temporary working files

Follow the target project's existing rules for working artifacts during the run.
The Starter defaults to `tmp/<work-item>/`, an anchored `/tmp/` ignore rule, and
`docs/` for durable documentation. It excludes `docs/tmp/` and system `/tmp/`
for shared repo artifacts. An Org Standard may choose other locations and place
its instructions in other operational files. Preserve those choices; include any
boundary changes in the approved plan. Create untracked working directories only
when needed, without placeholders.

## Finish

- Verify the created or changed tree and read the key operational files.
- Verify changed ignore rules and operational instructions preserve the chosen
  temporary-artifact / durable-documentation boundary. For the unchanged Starter
  convention, check `tmp/<work-item>/`, anchored `/tmp/`, and the `docs/` boundary.
- Report target location, main files, and intentional omissions.

## Do not edit the Built-in Starter during a run

`starter/` is generic template content. Never modify it as a side effect of
creating or applying an Org Standard — improvements flow Project → Org Standard via
`project-scaffold-audit`. Updating `starter/` itself is the
`project-scaffold-maintain` skill (skills-repo maintainers only), out of scope
here.
