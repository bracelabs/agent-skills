# Scaffold scan and diff analysis

Shared method used by `project-scaffold-audit` (Project → Org Scaffold) and by
`project-scaffold` Bootstrap (existing assets → Org Scaffold).

## 1. Scan

For each target (project, scaffold, or asset) collect only permitted metadata:

- shallow directory layout (depth ~2), noting intentional empty dirs and placeholders
- root `AGENTS.md` / `agents.md` — every operating rule, especially the
  `tmp/<work-item>/` vs `docs/` boundary
- root `README.md` — structural claims only
- root `.gitignore` — presence of an anchored `/tmp/` rule and other
  shared-artifact rules
- `docs/README.md` and shallow `docs/` directory names; the numbered lifecycle
  (`01_product` … `06_execution`) if used
- reusable documents under `docs/00_templates/`

Never read application source, dependency trees, lockfiles, generated output, data
files, secrets, or product specifications.

## 2. Diff

Compare the scanned metadata against the baseline — the Built-in Starter for
Bootstrap, the resolved Org Scaffold version for audit. Record, with exact file
paths:

- missing / renamed / materially different structural or documentation conventions
- AGENTS.md rules that diverge — separate local domain constraints from reusable
  operating conventions
- `.gitignore` divergence, especially the `/tmp/` rule and its AGENTS.md counterpart
- additions in the target absent from the baseline that could be generic
  improvements

## 3. Classify

| Bucket | Test |
| --- | --- |
| **Local** | Explained by the target's stack, domain, or delivery model. |
| **Promote** | Durable, technology-neutral, seen across multiple targets or user-endorsed; the Org Scaffold should adopt it. |
| **Remove / Migrate** | Outdated, duplicated, or contradicts the current standard. |
| **Needs-decision** | Cannot tell whether it is intentional; name the evidence that would settle it. |

For Bootstrap, "Promote" means "include in the initial scaffold"; a convention
seen in only one source stays out unless the user endorses it.

## 4. Report

One finding per row: bucket, evidence (files), affected scope, benefit,
compatibility risk, smallest change. Flag any Promote item generic enough for the
Built-in Starter as a **Global Promote candidate**.
