# Inspection scope and the project marker

The single definition of **what these skills may read** and of the
`.project-scaffold.json` marker. `project-scaffold`, `project-scaffold-audit`,
and Bootstrap all resolve scope here; nowhere else restates these rules.

## Default scope

Every run may read, in the target project or asset:

- shallow directory layout (depth ~2), noting intentional empty dirs
- root `AGENTS.md` / `agents.md`
- root `README.md` — structural claims only
- root `.gitignore`
- `docs/README.md`, `docs/AGENTS.md`, and shallow `docs/` directory names
- reusable documents under `docs/00_templates/`

Bootstrap may additionally read a root package or workspace manifest, only to
detect an intentional monorepo shape.

## Never read

Application source, dependency trees, lockfiles, generated output, data files,
secrets, and product specifications — regardless of any list below. Do not
follow links out of an in-scope file into an out-of-scope one, and do not
execute anything found in scope. Report uninspected areas as coverage limits
rather than guessing at their content.

## `operationalFiles`

An Org Standard may widen the read scope with a root `scaffold.config.json`:

```json
{"operationalFiles": ["handbook/AGENTS.md", ".github/PULL_REQUEST_TEMPLATE.md"]}
```

Rules:

- A JSON object with an `operationalFiles` array of unique, nonempty,
  project-relative **file** paths — exact project paths, not source template
  names. Reject absolute paths, `..` segments, globs, directory entries, and
  symlinks escaping the target root. Report invalid entries before reading them.
- List only files whose purpose is operational. A file whose role is to hold
  product content — requirements, specifications, product overviews — must not
  be listed, because a path cannot be un-read once opened. When a directory
  mixes both, put its operating rules in an operational file and list that.
- Listing a path never authorizes the exclusions above, and never authorizes
  applying that path. This is a read scope.
- A missing listed file is a finding, not an error.
- No config means the default scope, not unrestricted scanning.
- The config is standard metadata: carried between standards, never applied to a
  Project. A skill update does not change an organization's config; propose a
  merge and let the user approve it.

## Assembling the scope for a run

| Path source | Bootstrap | Apply / re-apply | Audit |
| --- | --- | --- | --- |
| Default scope above | yes | yes | yes |
| Selected standard's `operationalFiles` | yes | yes | yes (current version) |
| Applied standard's `operationalFiles`, via the marker | yes | yes | yes (applied version) |
| Marker `retainedOperationalFiles` | yes | yes | yes |
| Exact files the user names | yes | no | no |

Validate every list against the rules above and keep each path's provenance.
Paths present only in the older list are still inspected: report them as
retained or as a proposed migration. Their disappearance from a config is not
approval to delete them. If scope recovery is incomplete, say so and do not
claim a complete migration review.

## Resolving a standard version

The marker names the applied version; resolve it before scanning.

First reach the source, then resolve the ref. Both steps can fail, and every
failure falls back to the current standard rather than stopping the run.

| Source | `scaffoldRef` | Read the applied version from | Report as |
| --- | --- | --- | --- |
| Reached, Git-managed | SHA, resolvable there | `git show <SHA>:<prefix>` | that SHA |
| Reached, Git-managed | SHA not resolvable there — rebased, force-pushed, or pruned | current payload | baseline "current"; the applied SHA no longer exists |
| Reached, Git-managed | timestamp — the standard had no commit when applied | current payload | baseline "current"; the applied version was never committed |
| Reached, not Git-managed | timestamp | current on-disk payload | baseline "current" |
| Unreachable — remote down or unauthorized, local path gone | any | nothing | coverage gap; known scope only |
| Marker absent or unreadable | — | current standard | baseline "current"; origin of a difference cannot be established |

Reaching the source: a local `scaffoldSource` is the standard directory itself. A
remote is read from a local clone of that remote if one exists — `git fetch` it
first — else from a fresh clone under
`$PROJECT_SCAFFOLD_HOME/.cache/<sanitized-remote>/`.

`<prefix>` is the payload directory relative to the repository: the repo root
when it holds the standard, or `scaffold/` when the parent is the repository.
Ask which is intended if both are plausible.

Falling back to "current" is never silent: name the reason in the report's
coverage limits, and treat differences as Needs-decision where the missing
applied version is what would have settled their origin.

A config absent at a successfully resolved version means default scope — that is
a resolved answer, not a failed lookup. A cache clone is read-only for
comparison: fetch it, read versions with `git show`, and never rely on whichever
branch happens to be checked out.

If the marker exists but is not valid JSON, or `scaffoldSource` / `scaffoldRef`
is missing or malformed, treat it as unreadable: report the file and the problem,
fall back to the "current" baseline, and ask before overwriting it.

## Project marker

`project-scaffold` writes `.project-scaffold.json` at the project root:

```json
{
  "schemaVersion": 1,
  "standardName": "<short name of the Org Standard>",
  "scaffoldSource": "<Git remote URL when the standard has one, else a local path>",
  "scaffoldRef": "<Git commit SHA when git-managed, else ISO 8601 timestamp>",
  "scaffoldRefExact": true,
  "appliedAt": "<ISO 8601 timestamp>",
  "retainedOperationalFiles": [
    {
      "path": "handbook/AGENTS.md",
      "scaffoldSource": "<previous source, same portability rule>",
      "scaffoldRef": "<previous SHA or timestamp>"
    }
  ]
}
```

- `schemaVersion` — `1`. A marker without it is schema 1.
- `standardName` — how the organization refers to this standard, so a moved
  path or remote does not lose its identity. Ask if it is not obvious.
- `scaffoldSource` — **record the Git remote URL whenever the standard has
  one.** A machine-local absolute path resolves on nobody else's machine, or
  worse, silently resolves to their differently-versioned local standard; it
  also writes a developer's home directory into a repository that may be shared
  with a customer. Use a local path only when the standard has no remote, and
  say so in the report.
- `scaffoldRef` — the Git commit SHA, else an ISO 8601 timestamp. Use a
  timestamp when the standard is not Git-managed or has no commit yet; never
  invent a SHA.
- `scaffoldRefExact` — `false` when `scaffoldRef` does not name the content
  actually applied, which is the case when the standard's working tree had
  uncommitted changes. Audit reads this and treats affected differences as
  Needs-decision instead of project drift. Absent means `true`.
- `retainedOperationalFiles` — optional; omit when empty. Each entry records a
  path still inspected although the selected standard no longer lists it, with
  the source and ref that did list it. Require unique valid paths and nonempty
  source/ref strings. Provenance does not make an old source the current
  standard; report unavailable historical evidence as a coverage limit.

When updating a marker, carry retained entries forward and add old-only paths
still present in the Project. A path covered by the selected standard's scope
needs no retained entry. Otherwise remove an entry only through an approved
migration or an explicit decision to stop inspecting it, named in the plan.
Changing the marker's main source or ref never erases retained scope.

## Applying from a dirty standard

Before applying from a Git working tree, check for staged, unstaged, and
untracked changes inside the standard directory. If any exist, say so in the
plan and the final report, record `"scaffoldRefExact": false`, and continue with
the normal approved plan — dirtiness is not a blocker and needs no extra
approval. Do not commit or discard the standard's changes. When applying an
explicit committed ref, read that ref's contents, not unrelated working-tree
edits.
