# Org Standard

The **Org Standard** is the standard a user or organization actually uses and
grows over time. It is distinct from the Built-in Starter (a generic starting
point bundled with the skill) and from any single Project.

## Location

- `$PROJECT_SCAFFOLD_HOME` if set, else `~/.config/agent-skills/project-scaffold/`.
- The Org Standard's content lives in `scaffold/` under that directory.
- Optional companion file in the same directory:
  - `bootstrap-sources.md` — a record of what a Bootstrap run scanned.

This path is outside every skill install directory, so `gh skill install` and
`gh skill update` never overwrite it.

## Contents

`scaffold/` mirrors what gets applied to a project: an `AGENTS.md`, a `README`
pattern, a `.gitignore` pattern, a `docs/` structure, and any `docs/00_templates/`
documents the org standardizes on. Keep it technology-neutral; stack-specific
starters belong in clearly labelled subdirectories only when repeatedly needed.

## Rename map (applies to every scaffold source)

Whether it comes from the Built-in Starter, a Bootstrap, or an existing
Org Standard, `project-scaffold` does not copy the source verbatim:

| In the scaffold source | Written into the project |
| --- | --- |
| `AGENTS.md.tmpl` | `AGENTS.md` |
| `README.md.tmpl` | `README.md` |
| `gitignore` (no dot) | `.gitignore` |
| everything else | same relative path |

**Not copied** — these are scaffold-repo metadata, never payload: `.git/`, the
root `scaffold.config.json`, the scaffold's root `README.md`, and its root
`.gitignore` (dotted). Nested README.md and .gitignore files remain payload. A
git-managed Org Standard keeps a dotted `.gitignore` for its own hygiene; the
project's `.gitignore` is built from the dotless `gitignore` payload.

Keep source filenames inside the Org Standard; rename only when applying to a
Project or normalizing a comparison. Reject duplicate destination paths (for
example, both `AGENTS.md` and `AGENTS.md.tmpl`) and unsupported `*.tmpl` names.
If an older standard has already renamed its root README or ignore payload,
ask which files are payload before proposing a migration; do not silently omit them.

## Operational inspection scope

An optional root `scaffold.config.json` declares additional operational files,
using exact project-relative paths (not source template names):

```json
{
  "operationalFiles": [
    "handbook/AGENTS.md",
    "handbook/README.md",
    ".github/PULL_REQUEST_TEMPLATE.md",
    ".github/workflows/review.yml",
    ".agents/skills/review/SKILL.md"
  ]
}
```

Bootstrap, apply, and audit add these paths to their default inspection scope.
No config means the documented default scope, not unrestricted scanning. During
Bootstrap the user may also name exact operational files; include the approved
resulting paths in the synthesized config. Record missing listed files as missing.
Unlisted files remain uninspected, including files linked from a listed file.

Require a JSON object with an `operationalFiles` array of unique, nonempty relative
file paths. Reject absolute paths, `..` segments, globs, directory entries, and
symlinks escaping the target root. Listing a path never authorizes reading secrets,
application code, dependencies, generated output, or product specifications, or
executing workflows/skills. Report invalid entries before scanning them. Listed
files may be inspected even when there is no matching payload in the standard.
This is a read scope, not permission to apply every listed file.

### Recovering the applied scope

For a previously scaffolded project, read `.project-scaffold.json` and resolve
`scaffoldSource` before scanning additional files. A local source points to the
standard directory; for a remote, use a matching clone or fetch/clone into the
Org Standard home's `.cache/` directory (use the default home if unset).
Locate the payload at the repository root or its `scaffold/` subdirectory; if
ambiguous, ask which is intended. Read its `scaffold.config.json` at
`scaffoldRef` with `git show <SHA>:<payload-prefix>scaffold.config.json` when
resolvable. A config absent at a successfully resolved version means default
scope, not a failed lookup. For a timestamp or unavailable version, use the
source's current config and report the fallback. If the source is unreachable,
report the coverage gap and use only known scope; do not infer missing paths.

For Bootstrap, combine this recovered scope with any source-local config and
exact user-named files. Read the files in the reference project, not the standard's
payload, as evidence for Bootstrap. For re-application, combine the recovered
scope with the selected standard's config, even when changing standard sources.
Validate every list using the contract above and retain each path's provenance.
Old-only entries are inspected for compatibility and reported as retained or
proposed migrations; their disappearance from a config is not deletion approval.
If scope recovery is incomplete, do not claim a complete migration review.

Keep the organization's temporary-artifact and durable-documentation conventions
in its operational rules. `tmp/` and `docs/` are Starter defaults, not required
directory names for every Org Standard.

## Creating it — three patterns

### 1. Built-in Starter

Copy `starter/` from the skill into `scaffold/`, preserving source filenames.
Adjust only what the user asks for. Do not carry over any product's domain rules,
names, or services.

### 2. Bootstrap from existing assets

Scan the user's chosen sources once (see [bootstrap.md](bootstrap.md)), synthesize
an Org Standard, and present it for approval before writing `scaffold/`. Existing
organizational rules and structure take priority over the Built-in Starter shape.

### 3. Use existing Standard

`scaffold/` already exists, or the user points at a local path or git remote.
Validate that it is readable and technology-neutral enough to apply; otherwise
report what is missing rather than guessing.

## Git management follow-up (patterns 1 and 2)

After the `scaffold/` content is approved, offer — do not assume:

1. `git init` in `$PROJECT_SCAFFOLD_HOME/scaffold/` (or the parent directory, if
   the user wants the companion files tracked too), an initial commit, and a
   `.gitignore` if needed.
2. If the user wants a remote: create it with
   `gh repo create <name> --private --source=<scaffold dir> --remote=origin --push`.
   Ask for the name and visibility; default to `--private`. Never create a remote
   or push without explicit approval.

"Not git-managed" is valid — in that case audit compares against the on-disk
`scaffold/` directly.

## Project marker

When `project-scaffold` applies an Org Standard to a project it writes
`.project-scaffold.json` at the project root:

```json
{
  "scaffoldSource": "<path or git remote URL of the Org Standard>",
  "scaffoldRef": "<git commit SHA when git-managed, else ISO 8601 timestamp>",
  "appliedAt": "<ISO 8601 timestamp>"
}
```

`project-scaffold-audit` reads this to pick its comparison baseline. If the file
is absent, audit compares against the current `scaffold/` and says so in the
report.

Before applying a Git working tree, check for staged, unstaged, and untracked
changes within the standard directory. If present, notify the user in the plan
and final report that the Standard has uncommitted changes and the recorded HEAD
SHA does not include them; a later audit may report them as differences. Continue
with the usual approved application plan: dirtiness alone is not a blocker or
an extra approval step. Do not commit or discard the Standard's changes. If there
is no commit yet, record a timestamp instead of inventing a SHA. When applying an
explicit committed ref, read that ref's contents, not unrelated working-tree edits.
