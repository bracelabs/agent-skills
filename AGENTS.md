# Agent instructions

## Repository scope

- This repository publishes reusable agent skills under `skills/<name>/`.
- Keep each skill's `SKILL.md`, supporting references, and metadata consistent with
  the skill's documented purpose.
- `README.md` and `README.ja.md` are project-specific public documentation. Update
  them when the published skill set or user-facing workflow changes.

## Validation and release

- Run `gh skill publish --dry-run` after changing a skill and before publishing.
- Publish skills with `gh skill publish`; do not publish from an installed copy.
- Release notes must include the matching `gh skill update` command so users can
  update the released skills.
- Use a small Conventional Commits-style commit for each verified change.
- Before committing, inspect `git status`, the staged diff, and run
  `git diff --check`.

## Documentation

- `docs/project-scaffold-setup.md` is the English setup guide for AI agents.
- Keep the setup guide concise and update the README links when its location or
  purpose changes. Do not add the full project-scaffold lifecycle to this small
  repository unless it becomes necessary.

## Temporary files and existing work

- Put uncommitted shared artifacts in `tmp/<work-item>/`. Do not use `docs/tmp/`
  or a system `/tmp/` path for artifacts shared in this repository.
- The root `.gitignore` keeps the anchored `/tmp/` rule; do not remove it.
- Preserve the user's uncommitted changes. Do not investigate, revert, or include
  unrelated files in a change.

## When uncertain

- Do not guess at a skill's contract, publishing behavior, or user-facing command.
- Inspect the relevant skill and project documentation first, then ask for the
  smallest missing decision.
