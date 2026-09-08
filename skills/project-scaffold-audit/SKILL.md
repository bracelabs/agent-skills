---
name: project-scaffold-audit
description: Scan an existing project against the Org Standard it was applied from, classify every difference as Local / Promote / Remove-Migrate / Needs-decision, and after explicit approval reflect Promote items into the Org Standard. Use to audit Org Standard drift and promote conventions; not to bootstrap or apply an Org Standard to a project.
license: MIT
---

# Project Scaffold Audit

Audit a Project against the **Org Standard** and propose improvements to that
standard. This is the **Project → Org Standard** direction. Applying an Org
Standard to a project is `project-scaffold`'s job.

This skill owns the scan-and-diff method; `project-scaffold` reuses it for
Bootstrap. See [analysis.md](references/analysis.md).

## Resolve the comparison baseline

1. Locate the Org Standard. Prefer the project's `.project-scaffold.json`
   `scaffoldSource`; otherwise `$PROJECT_SCAFFOLD_HOME/scaffold/`, else
   `~/.config/agent-skills/project-scaffold/scaffold/`.
2. Resolve the **applied** version and the payload prefix with
   [the resolution table](../project-scaffold/references/scope.md#resolving-a-standard-version).
   That table also covers a missing or unreadable marker, an unresolvable ref,
   and an unreachable remote.
3. Resolve the **current** version: the explicitly selected branch/ref, else the
   remote default branch after fetch for a remote source, else the on-disk
   content for a local source. Record its SHA and any uncommitted changes, or a
   timestamp for a non-Git source. Do not switch branches or discard edits to
   read a version.
4. If the Org Standard's identity is still ambiguous — no marker and more than
   one plausible standard — ask the user before proceeding.

Compare findings against the current version before proposing anything. If the
current version cannot be read, report the proposals as provisional and reflect
nothing.

Note whether the audited project is a Git worktree. If it is not, report file
comparisons without claiming Git status or history.

## Inspection boundary

Read only what [scope.md](../project-scaffold/references/scope.md) permits,
assembling the run's paths from
[the scope table](../project-scaffold/references/scope.md#assembling-the-scope-for-a-run)
— for audit that is the union of the applied and current versions'
`operationalFiles` plus the marker's `retainedOperationalFiles`, noting which
version lists each path. Preserve existing work; treat repo-specific rules as
intentional until evidence says otherwise.

## Classify every difference

Run the [analysis method](references/analysis.md) and put each finding in exactly
one bucket:

- **Local** — justified by the project's stack, domain, or delivery model; keep as-is.
- **Promote** — a durable, technology-neutral convention plausibly reusable in
  other projects; a candidate for the Org Standard even if observed in only one
  project.
- **Remove / Migrate** — outdated, duplicated, or contradicting the current
  standard; propose cleanup.
- **Needs-decision** — evidence is insufficient to tell whether the difference is
  intentional; list what would resolve it.

Each finding carries evidence, scope, benefit, compatibility risk, the smallest
proposed change, and its change target. Write it up per
[report.md](references/report.md).

## Route and reflect

Default output is read-only: the audit report and its proposals. The report file
is the only thing audit writes; it never modifies the audited project or the Org
Standard without approval of an exact item list.

After approval, follow [handoff.md](references/handoff.md): Org Standard changes
are made here on a branch, Project changes are handed to `project-scaffold`, and
the starting branch is restored afterwards.

## Global Promote candidates

If a Promote item is generic enough to belong in the Built-in Starter, flag it in
the report as a **Global Promote candidate** only. Integrating it into the starter
is the `project-scaffold-maintain` skill's job. Never edit `starter/` from here.
