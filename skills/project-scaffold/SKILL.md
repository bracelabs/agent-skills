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
  Org Standard and ref were applied. See
  [org-standard.md](references/org-standard.md).

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

- every `*.tmpl` and `gitignore` resolves to a rename-map target; no target
  collides with the scaffold's own `README.md` / `.gitignore`
- the payload's `.gitignore` keeps an anchored `/tmp/` rule
- internal Markdown links inside the scaffold resolve
- if `.project-scaffold.json` will be read (re-apply / audit), its `scaffoldSource`
  is reachable

## Apply an Org Standard to a project

Do not modify a project until the user approves a concrete plan.

1. Determine mode: **new** (empty or newly requested target) or **existing**.
   Preserve current user work either way.
2. Inspect only permitted metadata of the target: shallow layout, root AGENTS.md,
   root README.md, root .gitignore, listed operational files, docs/README.md,
   docs/AGENTS.md, and shallow docs directory names. Do not read application
   source, dependency trees, generated output, secrets, or product specifications.
3. Select only what the user requested or an approved `project-scaffold-audit`
   report identifies. An existing Org Standard is not permission to rewrite every
   difference.
4. Present the plan: each file/dir to create or change, the convention and its
   source, compatibility impact, intentional local exceptions left untouched,
   whether `/tmp/` is added to `.gitignore`, and the matching AGENTS.md
   temporary-work rule.
5. On approval, make only the listed changes. If the plan changes materially,
   re-present and ask again.
6. Write or update `.project-scaffold.json` at the project root: `scaffoldSource`,
   `scaffoldRef` (git commit SHA of the Org Standard when git-managed, else an ISO
   8601 timestamp), `appliedAt`.
7. Verify the changed operational files. Report target location, applied
   conventions, and remaining exceptions. If the target is not a git worktree,
   report verification without claiming git status.

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

## Temporary working files

Use `tmp/<work-item>/` at the project root for artifacts that must not be
committed: scratch notes, one-off exports, rendered previews, logs, agent handoff
files. `docs/` is the versioned source of truth; never use `docs/tmp/`.

- Every Org Standard's root `.gitignore` keeps an anchored `/tmp/` rule,
  preserving existing rules.
- The root AGENTS.md documents the `tmp/<work-item>/` convention and its `docs/`
  boundary. For an existing project, include that AGENTS.md change in the approved
  plan.
- Create `tmp/<work-item>/` only when needed; it is untracked, so add no
  placeholders.
- Do not use a system `/tmp/...` path for artifacts shared between agents in one
  repo.

## Finish

- Verify the created or changed tree and read the key operational files.
- When `.gitignore` is created or changed, verify the anchored `/tmp/` rule.
- When AGENTS.md is created or changed, verify it documents the `tmp/<work-item>/`
  convention and its `docs/` boundary.
- Report target location, main files, and intentional omissions.

## Do not edit the Built-in Starter during a run

`starter/` is generic template content. Never modify it as a side effect of
creating or applying an Org Standard — improvements flow Project → Org Standard via
`project-scaffold-audit`. Updating `starter/` itself is the
`project-scaffold-maintain` skill (skills-repo maintainers only), out of scope
here.
