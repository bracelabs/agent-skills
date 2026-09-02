---
name: project-scaffold-maintain
description: Integrate approved Global Promote candidates from a project-scaffold-audit report into a skills repository's Built-in Starter. Use when curating generic conventions into the bundled starter template; requires a working checkout of the skills repository.
license: MIT
---

# Project Scaffold Maintain

Promote generic conventions **into the Built-in Starter** — the bundled template a
skills repository ships and `project-scaffold` copies from. This is the top of the
promotion chain:

```
Project → project-scaffold-audit → Org Scaffold
Org Scaffold → Global Promote candidate → project-scaffold-maintain → Built-in Starter
```

Use this only when maintaining a skills repository (this one, or a fork). It edits
`skills/project-scaffold/starter/` in a git checkout — never an installed skill
copy, whose edits are lost on the next `gh skill update`.

## Preconditions

- A working git checkout of the skills repository is the current directory, or the
  user names its path. Confirm `skills/project-scaffold/starter/` exists.
- Input: one or more **Global Promote candidates**, normally from a
  `project-scaffold-audit` report. Given a full audit report, take only the items
  it flagged as "Global Promote candidate"; ignore Local / plain Promote /
  Remove-Migrate items — those belong to an Org Scaffold, not the starter.

## Filter

Keep a candidate only if every test holds:

- technology-neutral — no framework, hosting, database, or language specifics
- no product, organization, or domain naming
- benefits most new projects, not just the audited one
- not already present in the starter

Reject the rest and state why. A convention seen in a single project is not
generic.

## Apply

1. For each kept candidate, make the smallest change under
   `skills/project-scaffold/starter/`: a new template document, an added `docs/`
   directory with its `README.md`, an `AGENTS.md.tmpl` rule, a `gitignore` line.
2. If a starter filename changes, update the rename map in
   `skills/project-scaffold/references/org-scaffold.md`.
3. Change `skills/project-scaffold/SKILL.md` only if the way scaffolds are created
   or applied actually changes; prefer starter-only edits.
4. Keep every starter file generic — placeholders like `<project name>`, never
   real content.

## Verify and hand off

- Run `gh skill publish --dry-run` from the repo root; it must pass.
- Summarize: candidates applied, candidates rejected (with reason), files changed.
- Commit on a branch and open a PR (`gh pr create`) describing source evidence
  (which audit, which project), scope, and compatibility risk.
- Do not merge. Do not run `gh skill publish --tag` — releasing is a separate
  maintainer decision.

## Not this skill's job

- Editing an Org Scaffold — that is `project-scaffold-audit`.
- Applying a scaffold to a project — that is `project-scaffold`.
- Cutting a release.
