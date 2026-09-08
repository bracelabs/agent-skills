# Audit report

The audit's deliverable. Its header is fixed so that reports from many projects,
collected over months, can be compared without re-reading each one.

## Where it goes

Write the report to the project's own untracked working-artifact location — for
the Built-in Starter convention, `tmp/project-scaffold-audit/`, named
`<YYYY-MM-DD>-audit.md`. Follow the project's own convention when it differs.

This is the one write audit may make. It does not change the project: the report
is a deliverable for whoever owns the Org Standard, not project content. Keep it
untracked unless the organization decides otherwise — audit findings quote
internal operating conventions, which do not belong in a repository shared with
a customer by default.

If the project has no untracked working-artifact location, or the natural one is
tracked, say so and ask where to write instead. Also report the findings in the
session; the file is a record, not a substitute for the conversation.

## Header

Open the report with this block, filled in:

```yaml
---
schemaVersion: 1
auditedAt: <ISO 8601 timestamp>
project:
  path: <absolute path>
  remote: <Git remote URL, or null>
  isGitWorktree: <true|false>
standard:
  name: <marker standardName, else how the resolved standard is identified>
  source: <the Org Standard actually compared against — always recorded>
  appliedRef: <marker scaffoldRef; null when unknown>
  appliedRefExact: <marker scaffoldRefExact; true when absent, null when unknown>
  currentRef: <SHA, timestamp, or null>
  currentDirty: <true|false>
baseline: <applied+current | current-only>
scope:
  inspected: [<path>, ...]
  missing: [<listed path that does not exist>, ...]
  coverageLimits: [<what could not be resolved and why>, ...]
findings:
  local: <n>
  promote: <n>
  removeMigrate: <n>
  needsDecision: <n>
  globalPromoteCandidates: <n>
  alreadyIncorporated: <n>
---
```

`source` always names the standard the audit actually compared against, including
when there is no marker and it was resolved from `$PROJECT_SCAFFOLD_HOME`. A
report that cannot say what it was measured against is not comparable with any
other report. "The applied version is unknown" is expressed by
`appliedRef: null`, not by dropping the source.

`baseline: current-only` means the applied version could not be resolved, so the
origin of a difference cannot be established. Say that in the prose too, with the
reason from the resolution table's "Report as" column in `coverageLimits`.

`appliedRefExact` carries the marker's `scaffoldRefExact` through under the
header's naming. When it is `false`, the standard had uncommitted changes when it
was applied, so `appliedRef` does not name what actually landed: differences
traceable to that are Needs-decision, not project drift. Note this next to any
finding it touches.

## Findings

One row per finding:

| Field | Content |
| --- | --- |
| Bucket | `Local` / `Promote` / `Remove-Migrate` / `Needs-decision` |
| Change target | `Project`, `Org Standard`, or `none` |
| Evidence | Exact file paths, and the rule or structure at issue |
| Scope | What the change would affect |
| Benefit | Why it is worth doing |
| Compatibility risk | What could break, for whom |
| Smallest change | The minimal edit that resolves it |

`Change target` is `none` for `Local` findings and for anything recorded as
already incorporated — they are reported, not acted on. A finding that would
change both the Project and the Org Standard is split into two rows so each can
be approved on its own.

Flag any Promote item generic enough for the Built-in Starter as a **Global
Promote candidate**. That is a flag on the row, not a fifth bucket, and it is
`project-scaffold-maintain`'s input — never acted on here.

## Also include

- Items already incorporated into the current standard, listed separately from
  the actionable findings, with no change proposed.
- Current-standard changes that have not reached the project, reported as
  available migrations rather than as violations.
- Conflicts between a project change and a current-standard change, explained
  before any recommendation.
