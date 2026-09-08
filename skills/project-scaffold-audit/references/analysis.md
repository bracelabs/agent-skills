# Org Standard scan and diff analysis

Shared method used by `project-scaffold-audit` (Project → Org Standard) and by
`project-scaffold` Bootstrap (existing assets → Org Standard).

## 1. Scan

Read only what
[scope.md](../../project-scaffold/references/scope.md) permits, and assemble the
run's path list from
[the scope table](../../project-scaffold/references/scope.md#assembling-the-scope-for-a-run)
there. That covers the default scope, `operationalFiles` configs, the marker's
`retainedOperationalFiles`, validation, and how to resolve an applied version.
Report uninspected areas as coverage limits.

From each in-scope file, collect what bears on conventions:

- shallow directory layout, noting intentional empty dirs and placeholders
- root `AGENTS.md` / `agents.md` — every operating rule, including the chosen
  temporary-artifact / durable-documentation boundary
- root `README.md` — structural claims only
- root `.gitignore` — rules supporting that boundary (`/tmp/` for the Starter)
- `docs/README.md` and shallow `docs/` directory names; the numbered lifecycle
  (`01_product` … `06_execution`) if used
- `docs/AGENTS.md` — how docs/ is used: directory roles, workflow, checklist
- reusable documents under `docs/00_templates/`

For a standard or Starter, normalize payload paths before scanning/comparing:
root `AGENTS.md.tmpl` → `AGENTS.md`, root `README.md.tmpl` → `README.md`, root
`gitignore` → `.gitignore`; all other payload paths stay unchanged. Exclude `.git/`
and root `README.md`, `.gitignore`, `scaffold.config.json` as standard metadata;
nested README.md and .gitignore files are payload. Reject duplicate destinations
and unsupported `*.tmpl` names. Never compare the standard's own README or ignore
file as project content. If an older standard has already renamed those payloads,
resolve their intent with the user rather than silently omitting them.

## 2. Diff

Compare the scanned metadata against the baseline — the Built-in Starter for
Bootstrap, the resolved Org Standard version for audit. Record, with exact file
paths:

- missing / renamed / materially different structural or documentation conventions
- AGENTS.md rules that diverge — separate local domain constraints from reusable
  operating conventions
- `.gitignore` divergence and the corresponding operational boundary rules
- additions in the target absent from the baseline that could be generic
  improvements

Compare conventions, not literal placeholder values: filling `<project name>`
or a template's example fields is expected customization, not a missing rule.
Do not read product specifications to infer those values.

For audit, use Project vs applied version to detect divergence, then check both
against the current standard. Also report current-standard changes that have not
reached the project, without treating them as mandatory migrations. An applied
ref records the source of a selective application, not proof every rule was
adopted. When the marker says `scaffoldRefExact: false`, the applied baseline
does not name what was applied — prefer Needs-decision over asserting drift. If
intent is unknown, use Needs-decision.

When a project improvement is already in the current standard, record it as
already incorporated with no change proposed, outside the actionable findings;
do not propose Promote again. If current and project changes conflict, explain
the conflict before recommending a change. Differences alone are not violations.

## 3. Classify

| Bucket | Test |
| --- | --- |
| **Local** | Explained by the target's stack, domain, or delivery model. |
| **Promote** | Durable, technology-neutral, and plausibly reusable in other projects; a candidate for the Org Standard. |
| **Remove / Migrate** | Outdated, duplicated, or contradicts the current standard. |
| **Needs-decision** | Cannot tell whether it is intentional; name the evidence that would settle it. |

For audit, a convention observed in one project may be a Promote candidate without
prior endorsement. Explain its reuse case and record the extent of supporting
evidence; observation count informs adoption, not eligibility for nomination.
Keep project-specific domain rules Local — a single-project domain rule is never
a common convention. Candidate classification is not approval.

For Bootstrap, "Promote" means "include in the initial Org Standard"; a convention
seen in only one source stays out unless the user endorses it.

## 4. Report

Follow [report.md](report.md): fixed header, one row per finding with its change
target, already-incorporated items listed separately, and Global Promote
candidates flagged. Bootstrap has no marker to record, but reports the same
scope, coverage limits, and per-bucket counts.
