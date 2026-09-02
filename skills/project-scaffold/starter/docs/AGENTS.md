# Documentation guide

`docs/` is the versioned source of truth. Update it in the same change as the
code, not after.

## What each directory holds

- `01_product/` — vision, MVP, personas, concept.
- `02_requirements/` — what to achieve. Not how.
- `03_spec/` — how it is built: architecture, APIs, data, UI behaviour. Not why.
- `04_decisions/` — decision records. Add one for a technology choice, a
  dependency added or dropped, a data-model or API change, an architecture
  change, or an operational-rule change.
- `05_discussions/` — open questions only. When resolved, fold the outcome into
  requirements/spec and add a decision record if one is warranted.
- `00_templates/` — templates for new documents. Do not treat a template as
  authoritative content.

## Workflow for a new capability

1. Update `02_requirements/`.
2. Add a `05_discussions/` note if something is unresolved.
3. Write or update `03_spec/`.
4. Record the chosen design in `04_decisions/`.
5. Implement.

## Before finishing a docs change

- Does `01_product/` need updating?
- Are `02_requirements/` and `03_spec/` updated?
- Is a decision record needed?
- Can any `05_discussions/` item be closed?
- Does `docs/README.md` need a new entry under Key documents?
