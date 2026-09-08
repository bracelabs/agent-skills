# Bootstrap from existing assets

Bootstrap builds an Org Standard from what an organization already has, instead of
from the Built-in Starter. It runs once; afterwards the Org Standard is the source
of truth and the scanned sources are no longer consulted on every run.

## Sources

Ask the user which to scan. Typical:

- one or more existing projects held up as good examples
- standalone `AGENTS.md` / `agents.md` files
- a `docs/` tree from a reference project
- an engineering handbook or development guidelines document

Record the resolved list, and the absolute workspace path they sit under, in
`$PROJECT_SCAFFOLD_HOME/bootstrap-sources.md` so a later re-bootstrap is
reproducible.

## Inspection boundary

Read only what [scope.md](scope.md) permits. For Bootstrap that is the default
scope, plus a root package or workspace manifest to detect an intentional
monorepo shape, plus exact operational files the user names, plus any
`operationalFiles` config in the sources.

For a source project carrying `.project-scaffold.json`, also recover the applied
standard's scope and its `retainedOperationalFiles` using
[the resolution table](scope.md#resolving-a-standard-version). That config
normally lives in the standard, not in the project. Record each recovered path's
source, ref, and any fallback in the Bootstrap report. The comparison baseline
remains the Built-in Starter.

Read the files in the reference project itself as evidence — not the payload of
whatever standard it was applied from.

## Method

Use the analysis method in
[../../project-scaffold-audit/references/analysis.md](../../project-scaffold-audit/references/analysis.md):
scan each source, diff against the Built-in Starter, and classify. For Bootstrap,
treat conventions seen across multiple sources, or explicitly endorsed by the
user, as the Org Standard baseline; keep single-source domain rules out unless asked.

Present the synthesized Org Standard — structure, operational rules, ignore
payload, templates, and any `scaffold.config.json` scope — for approval before
writing `scaffold/`. Preserve template source names according to the rename map;
do not pre-render them as project files. Then follow the git follow-up in
[org-standard.md](org-standard.md).

Requires the `project-scaffold-audit` skill installed. If it is absent, tell the
user and offer pattern 1 (Built-in Starter) instead.
