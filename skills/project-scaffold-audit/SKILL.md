---
name: project-scaffold-audit
description: Scan an existing project against the Org Scaffold it was applied from, classify every difference as Local / Promote / Remove-Migrate / Needs-decision, and after explicit approval reflect Promote items into the Org Scaffold. Use to audit scaffold drift and promote conventions; not to bootstrap or apply a scaffold to a project.
license: MIT
---

# Project Scaffold Audit

Audit a Project against the **Org Scaffold** and propose improvements to that
scaffold. This is the **Project → Scaffold** direction. Applying a scaffold to a
project is `project-scaffold`'s job.

This skill owns the scan-and-diff method; `project-scaffold` reuses it for
Bootstrap. See [analysis.md](references/analysis.md).

## Resolve the comparison baseline

1. Locate the Org Scaffold. Prefer the project's `.project-scaffold.json`
   `scaffoldSource`; otherwise `$PROJECT_SCAFFOLD_HOME/scaffold/`, else
   `~/.config/agent-skills/project-scaffold/scaffold/`.
   - `scaffoldSource` is a local path → use it directly.
   - `scaffoldSource` is a git remote → if the local scaffold directory is a clone
     of that remote, `git fetch` it; otherwise `git clone` it into a cache
     (`$PROJECT_SCAFFOLD_HOME/.cache/<sanitized-remote>/`) and read from there.
2. Pick the ref to diff against:
   - `.project-scaffold.json` present with a git-SHA `scaffoldRef`, and the scaffold
     is git-managed → diff against that commit (`git show <ref>:<path>`).
   - No marker, `scaffoldRef` is a timestamp, the scaffold is not git-managed, or
     the ref cannot be resolved/fetched → diff against the current scaffold and
     state in the report that the baseline is "current", not the exact applied
     version.
3. If the scaffold identity is still ambiguous (e.g. no marker and more than one
   plausible scaffold), ask the user before proceeding.

## Inspection boundary

Inspect only: shallow layout, root AGENTS.md, root README.md, root .gitignore,
listed operational files, docs/README.md, shallow docs directories, and reusable
files under `docs/00_templates/`. Do not read application source, dependency
trees, generated output, secrets, or product specifications. Preserve existing
work; treat repo-specific rules as intentional until evidence says otherwise.

## Classify every difference

Run the [analysis method](references/analysis.md) and put each finding in exactly
one bucket:

- **Local** — justified by the project's stack, domain, or delivery model; keep as-is.
- **Promote** — a durable, technology-neutral convention the Org Scaffold should adopt.
- **Remove / Migrate** — outdated, duplicated, or contradicting the current standard; propose cleanup.
- **Needs-decision** — evidence is insufficient to tell whether the difference is intentional; list what would resolve it.

For each finding give: evidence (exact files), affected scope, benefit,
compatibility risk, and the smallest proposed change. A single-project domain rule
is never a common convention.

## Reflect approved Promote items

Default output is read-only: an audit report plus a proposal. Do not modify the
audited project.

After the user explicitly approves specific Promote / Remove-Migrate items:

- **Org Scaffold is git-managed:** create a branch in the scaffold repo, apply the
  smallest change, commit, and open a PR (`gh pr create`) describing source
  evidence, scope, and risk. Do not merge.
- **Org Scaffold is not git-managed:** apply the smallest change directly to
  `scaffold/` files, then re-read them and report what changed.

Never auto-apply. Never push or open a PR without approval of the exact item list.

## Global Promote candidates

If a Promote item is generic enough to belong in the Built-in Starter, flag it in
the report as a **Global Promote candidate** only. Integrating it into the starter
is the `project-scaffold-maintain` skill's job. Never edit `starter/` from here.
