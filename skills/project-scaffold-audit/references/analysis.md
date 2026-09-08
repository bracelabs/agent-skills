# Org Standard scan and diff analysis

Shared method used by `project-scaffold-audit` (Project → Org Standard) and by
`project-scaffold` Bootstrap (existing assets → Org Standard).

## 1. Scan

For each target (project, Org Standard, or asset) collect only permitted metadata:

- shallow directory layout (depth ~2), noting intentional empty dirs and placeholders
- root `AGENTS.md` / `agents.md` — every operating rule, including the chosen
  temporary-artifact / durable-documentation boundary
- root `README.md` — structural claims only
- root `.gitignore` — rules supporting that boundary (`/tmp/` for the Starter)
- `docs/README.md` and shallow `docs/` directory names; the numbered lifecycle
  (`01_product` … `06_execution`) if used
- `docs/AGENTS.md` — how docs/ is used: directory roles, workflow, checklist
- reusable documents under `docs/00_templates/`
- additional exact project-relative paths in the standard's root
  `scaffold.config.json` `operationalFiles` array; for audit use the union from
  the applied and current versions, identifying which version lists each path

The optional config is a JSON object, for example
`{"operationalFiles":["handbook/README.md",".github/workflows/review.yml"]}`.
Require unique, nonempty relative file paths; reject absolute paths, `..`
segments, globs, directory entries, and symlinks escaping the target root.
Report invalid entries before reading them. Missing listed files are findings.
The list cannot override the exclusions below or authorize execution. Do not
follow links to unlisted files. No config means the default scope above. Bootstrap
also accepts exact user-named operational files and includes approved resulting
paths in the new standard's config. Report uninspected areas as coverage limits.

For a standard or Starter, normalize payload paths before scanning/comparing:
root `AGENTS.md.tmpl` → `AGENTS.md`, root `README.md.tmpl` → `README.md`, root
`gitignore` → `.gitignore`; all other payload paths stay unchanged. Exclude `.git/`
and root `README.md`, `.gitignore`, `scaffold.config.json` as standard metadata;
nested README.md and .gitignore files are payload. Reject duplicate destinations
and unsupported `*.tmpl` names. Never compare the standard's own README or ignore
file as project content. If an older standard has already renamed those payloads,
resolve their intent with the user rather than silently omitting them.

Never read application source, dependency trees, lockfiles, generated output, data
files, secrets, or product specifications.

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
adopted. If intent is unknown, use Needs-decision.

When a project improvement is already in the current standard, record it as
already incorporated with no change proposed, outside the actionable findings;
do not propose Promote again. If current and project changes conflict, explain
the conflict before recommending a change. Differences alone are not violations.

## 3. Classify

| Bucket | Test |
| --- | --- |
| **Local** | Explained by the target's stack, domain, or delivery model. |
| **Promote** | Durable, technology-neutral, seen across multiple targets or user-endorsed; the Org Standard should adopt it. |
| **Remove / Migrate** | Outdated, duplicated, or contradicts the current standard. |
| **Needs-decision** | Cannot tell whether it is intentional; name the evidence that would settle it. |

For Bootstrap, "Promote" means "include in the initial Org Standard"; a convention
seen in only one source stays out unless the user endorses it.

## 4. Report

One finding per row: bucket, evidence (files), affected scope, benefit,
compatibility risk, smallest change. Flag any Promote item generic enough for the
Built-in Starter as a **Global Promote candidate**.

Include applied and current baseline identities, inspection scope, missing listed
files, coverage limits, and already-incorporated items. If only the current
baseline is available, state that the origin of a difference cannot be established.
