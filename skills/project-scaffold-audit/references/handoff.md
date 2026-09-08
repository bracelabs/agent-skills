# Routing and reflecting approved changes

Audit's default output is read-only. Nothing here happens without the user
approving an exact list of items.

## Route by change target

| Change target | Who acts |
| --- | --- |
| `Org Standard` | This skill, after approval — see below. |
| `Project` | Hand off to `project-scaffold`, which presents its own change plan and takes its own approval. Its removal path exists for exactly this. |
| `none` | Nobody. Reported only. |

Do not edit the Org Standard to work around a Project-only problem, and do not
edit the audited project. Split a proposal that touches both into two approvable
items.

## Reflecting into a Git-managed Org Standard

1. Record the starting branch — or the starting commit, for a detached HEAD — so
   it can be restored.
2. Branch from the current standard version you actually checked during the
   audit. Recheck each approved item against it first: skip anything already
   incorporated, and re-present anything that has materially changed.
3. Apply the smallest change and commit only approved items. Leave unrelated
   working-tree edits alone.
4. Hand off:
   - **GitHub remote configured** (github.com or GitHub Enterprise): push the
     branch and open a PR against the intended base with `gh pr create`,
     describing source evidence, scope, and risk. Resolve an ambiguous remote or
     base before pushing. Do not merge.
   - **No remote, or no remote the available PR tooling supports:** finish
     locally. Report the branch, the commit, the diff, and why no PR was opened.
     Do not create a remote or push to complete the audit. Use another host's
     review workflow only if the user asks and tooling exists.
   - Authentication, network, or permission failures on a supported remote leave
     the handoff **incomplete**, not locally complete. Keep the commit and report
     the remaining step.
5. Return to the recorded starting branch or commit and verify it — including
   when PR creation failed. Keep the proposal branch and commit for review; do
   not merge them. If returning would overwrite existing work, stop and report
   the current branch and what recovery it needs.

A proposal branch left checked out is never reported as the accepted current
standard.

## Reflecting into a standard that is not Git-managed

Apply the smallest approved change directly to `scaffold/`, re-read the changed
files, and report what changed.

## Never

- Auto-apply anything.
- Push or open a PR without approval of the exact item list.
- Edit `starter/` — that is `project-scaffold-maintain`'s job.
