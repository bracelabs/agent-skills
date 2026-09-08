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
   - `scaffoldSource` is a local path → use it directly.
   - `scaffoldSource` is a git remote → if the local Org Standard directory is a
     clone of that remote, `git fetch` it; otherwise `git clone` it into a cache
     (`$PROJECT_SCAFFOLD_HOME/.cache/<sanitized-remote>/`) and read from there.
2. Pick the ref to diff against:
   - `.project-scaffold.json` present with a git-SHA `scaffoldRef`, and the Org
     Standard is git-managed → diff against that commit (`git show <ref>:<path>`).
   - No marker, `scaffoldRef` is a timestamp, the Org Standard is not git-managed,
     or the ref cannot be resolved/fetched → diff against the current Org Standard
     and state in the report that the baseline is "current", not the exact applied
     version.
3. If the Org Standard's identity is still ambiguous (e.g. no marker and more than
   one plausible Org Standard), ask the user before proceeding.
4. Also resolve the current standard: use the explicitly selected branch/ref,
   otherwise the remote default branch after fetch for a remote source, or the
   on-disk content for a local source. Record its SHA and any uncommitted changes
   (or timestamp for a non-Git source). Do not switch branches or discard edits
   to read a version. Compare findings with this current version before proposing
   changes; if unavailable, report proposals as provisional and do not reflect
   them until the current version can be checked.

For each version, resolve the payload directory: the repo root when it contains
the standard, or `scaffold/` when the parent is Git-managed. If both are plausible,
ask which is intended. Use that repository-relative prefix with `git show`.

## Inspection boundary

Inspect only: shallow layout, root AGENTS.md, root README.md, root .gitignore,
listed operational files from `scaffold.config.json` and the marker's
`retainedOperationalFiles` (see
[analysis.md](references/analysis.md)), docs/README.md, docs/AGENTS.md, shallow docs
directories, and reusable files under `docs/00_templates/`. Do not read
application source, dependency trees, generated output, secrets, or product
specifications. Preserve existing work; treat repo-specific rules as intentional
until evidence says otherwise.

## Classify every difference

Run the [analysis method](references/analysis.md) and put each finding in exactly
one bucket:

- **Local** — justified by the project's stack, domain, or delivery model; keep as-is.
- **Promote** — a durable, technology-neutral convention plausibly reusable in other projects; a candidate for the Org Standard even if observed in only one project.
- **Remove / Migrate** — outdated, duplicated, or contradicting the current standard; propose cleanup.
- **Needs-decision** — evidence is insufficient to tell whether the difference is intentional; list what would resolve it.

For each finding give: evidence (exact files), affected scope, benefit,
compatibility risk, the smallest proposed change, and its change target (Project,
Org Standard, or none). A single-project domain rule
is never a common convention.

## Route proposals and reflect approved Org Standard changes

Default output is read-only: an audit report plus a proposal. Do not modify the
audited project.

Route each proposal by its change target. Project-side Remove-Migrate items are
handoffs to `project-scaffold` for its application plan and approval flow; do not
edit the Org Standard to resolve a Project-only issue. Org Standard cleanup can
remain in audit. Split changes affecting both targets for separate approval.

After the user explicitly approves specific Promote / Remove-Migrate items
targeting the Org Standard:

- **Org Standard is git-managed:** record the starting branch (normally `main`,
  or the starting commit for detached HEAD), then create a branch from the
  checked current standard, apply the smallest change, and commit only approved changes. Preserve
  unrelated working-tree edits. Then choose the handoff:
  - **A GitHub remote (including GitHub Enterprise) is configured:** push the
    branch and open a PR (`gh pr create`) against the intended base, describing
    source evidence, scope, and risk. Resolve ambiguous remote/base choices before
    pushing. Do not merge.
  - **No remote, or no remote supported by the available PR tooling:** finish
    locally and report the branch, commit, diff, and reason no PR was opened.
    Do not create a remote or push merely to complete the audit. Use another
    host's review workflow only when the user requests it and tooling is available.
  Authentication, network, or permission failures on a supported remote are
  incomplete PR handoffs, not local-only completion; preserve the local commit
  and report the remaining step.
  After the PR or local handoff, return to the recorded starting branch/commit
  and verify it, including when PR creation fails. Keep the proposal branch and
  commit for review; do not merge them into the starting branch. If returning
  would overwrite existing work, stop and report the current branch and required
  recovery instead of forcing the switch. A proposal branch left checked out must
  not be reported as the accepted current standard.
- **Org Standard is not git-managed:** apply the smallest change directly to
  `scaffold/` files, then re-read them and report what changed.

Recheck approved items against the current standard immediately before writing;
skip already-incorporated items and re-present materially changed proposals.

Never auto-apply. Never push or open a PR without approval of the exact item list.

## Global Promote candidates

If a Promote item is generic enough to belong in the Built-in Starter, flag it in
the report as a **Global Promote candidate** only. Integrating it into the starter
is the `project-scaffold-maintain` skill's job. Never edit `starter/` from here.
