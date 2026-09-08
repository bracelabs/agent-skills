# Org Standard

The **Org Standard** is the standard a user or organization actually uses and
grows over time. It is distinct from the Built-in Starter (a generic starting
point bundled with the skill) and from any single Project.

What these skills may read, and the `.project-scaffold.json` marker contract,
are defined once in [scope.md](scope.md).

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

An optional root `scaffold.config.json` widens the read scope — see
[operationalFiles](scope.md#operationalfiles).

Keep the organization's temporary-artifact and durable-documentation conventions
in its operational rules. `tmp/` and `docs/` are Starter defaults, not required
directory names for every Org Standard.

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

### Payload that runs

Some payload does something the moment it lands: CI workflows, Git hooks, agent
skills and slash commands, and scripts an operational rule tells an agent to
run. Applying such a file is not the same as applying a README. List it as its
own category in the change plan, with what triggers it and what it does, and get
that approved separately from documentation changes. Never apply it silently as
part of "everything else" in the rename map.

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

A remote is also what lets a Project marker name the standard portably, so
prefer it for any standard shared beyond one machine. "Not git-managed" is still
valid — audit then compares against the on-disk `scaffold/` directly.
