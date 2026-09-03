# Bootstrap from existing assets

Bootstrap builds an Org Scaffold from what an organization already has, instead of
from the Built-in Starter. It runs once; afterwards the Org Scaffold is the source
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

For every scanned project inspect only:

- root and shallow directory layout (depth ~2)
- root `AGENTS.md` / `agents.md`
- root `README.md`
- root `.gitignore`
- a root package or workspace manifest only to detect an intentional monorepo shape
- `docs/README.md`, `docs/AGENTS.md`, and shallow documentation-directory names
- reusable documents under `docs/00_templates/`

Do not read application source, deep file trees, dependencies, lockfiles,
generated artifacts, data files, secrets, or detailed product specifications.

## Method

Use the analysis method in
[../../project-scaffold-audit/references/analysis.md](../../project-scaffold-audit/references/analysis.md):
scan each source, diff against the Built-in Starter, and classify. For Bootstrap,
treat conventions seen across multiple sources, or explicitly endorsed by the
user, as the scaffold baseline; keep single-source domain rules out unless asked.

Present the synthesized scaffold — structure, AGENTS.md rules, `.gitignore`,
templates — for approval before writing `scaffold/`. Then follow the git
follow-up in [org-scaffold.md](org-scaffold.md).

Requires the `project-scaffold-audit` skill installed. If it is absent, tell the
user and offer pattern 1 (Built-in Starter) instead.
