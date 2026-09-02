# Org Scaffold

The **Org Scaffold** is the standard a user or organization actually uses and
grows over time. It is distinct from the Built-in Starter (a generic starting
point bundled with the skill) and from any single Project.

## Location

- `$PROJECT_SCAFFOLD_HOME` if set, else `~/.config/agent-skills/project-scaffold/`.
- Scaffold content lives in `scaffold/` under that directory.
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

Whether the scaffold comes from the Built-in Starter, a Bootstrap, or an existing
Org Scaffold, `project-scaffold` does not copy it verbatim:

| In the scaffold source | Written into the project |
| --- | --- |
| `AGENTS.md.tmpl` | `AGENTS.md` |
| `README.md.tmpl` | `README.md` |
| `gitignore` (no dot) | `.gitignore` |
| everything else | same relative path |

**Not copied** — these are scaffold-repo metadata, never payload: `.git/`, the
scaffold's own `README.md`, and the scaffold's own `.gitignore` (dotted). A
git-managed Org Scaffold keeps a dotted `.gitignore` for its own hygiene; the
project's `.gitignore` is built from the dotless `gitignore` payload.

## Creating it — three patterns

### 1. Built-in Starter

Copy `starter/` from the skill into `scaffold/`, applying the rename map above.
Adjust only what the user asks for. Do not carry over any product's domain rules,
names, or services.

### 2. Bootstrap from existing assets

Scan the user's chosen sources once (see [bootstrap.md](bootstrap.md)), synthesize
a scaffold, and present it for approval before writing `scaffold/`. Existing
organizational rules and structure take priority over the Built-in Starter shape.

### 3. Use existing Scaffold

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

When `project-scaffold` applies a scaffold to a project it writes
`.project-scaffold.json` at the project root:

```json
{
  "scaffoldSource": "<path or git remote URL of the Org Scaffold>",
  "scaffoldRef": "<git commit SHA when git-managed, else ISO 8601 timestamp>",
  "appliedAt": "<ISO 8601 timestamp>"
}
```

`project-scaffold-audit` reads this to pick its comparison baseline. If the file
is absent, audit compares against the current `scaffold/` and says so in the
report.
