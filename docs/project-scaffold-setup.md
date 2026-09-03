# project-scaffold setup guide

`project-scaffold` does not overwrite a project with a template. First create an
**Org Scaffold**, the standard shared by a team or organization. Apply it to a
project only with approval, then use `project-scaffold-audit` to feed approved
improvements back into the Org Scaffold.

The first-time flow is:

```text
Install the skills with gh
    ↓
Run project-scaffold once to create an Org Scaffold
    ↓
Use it on a project: create a new structure, or apply to an existing repository
    ↓
Run project-scaffold-audit repeatedly across projects to improve the Org Scaffold
    ↓
Use project-scaffold-maintain to feed suitable generic improvements into this repository
```

## Response behavior

When an agent is asked to read this guide and perform setup, it should use the
guide as a runbook rather than repeat it in chat. Include the following output
expectation in the setup request, or treat it as the default for this guide:

```text
Do not restate this guide. Start with the recommended next action in at most
five bullets. Ask for the smallest decision needed. Show the change plan only
immediately before an approved write.
```

The change plan is still required before a write: it lists the affected paths,
source conventions, compatibility effects, and intentional exceptions. The rule
above keeps the initial response concise; it does not remove the approval
boundary.

## 1. Install the skills

Install `project-scaffold` to create and apply an Org Scaffold, and
`project-scaffold-audit` to learn from applied projects and improve that Org
Scaffold. Both are required when you create an Org Scaffold by bootstrapping from
existing projects.

```sh
gh skill install bracelabs/agent-skills project-scaffold
gh skill install bracelabs/agent-skills project-scaffold-audit
```

Maintainers of this `agent-skills` repository should also install
`project-scaffold-maintain`. It is the feedback path from an audit's generic
improvements into this repository's bundled Built-in Starter; it does not update
an organization's own scaffold.

```sh
gh skill install bracelabs/agent-skills project-scaffold-maintain
```

## 2. Create the first Org Scaffold

In Codex or Claude Code, make this request before applying anything to a project:

```text
Use $project-scaffold to create an Org Scaffold. Start from the Built-in Starter,
show me the proposed contents before writing them, then create it at
~/.config/agent-skills/project-scaffold/scaffold/. I want to decide about Git
management and a remote repository only after reviewing the contents.
```

To derive a standard from reference projects instead, explicitly ask for a
Bootstrap. Name the sources and retain a narrow inspection boundary; do not ask
the agent to read application code or secrets.

```text
Use $project-scaffold to bootstrap an Org Scaffold from the reference projects
below. Inspect only each root AGENTS.md, README.md, .gitignore, docs/README.md,
docs/AGENTS.md, docs/00_templates/, and shallow directory structure. Propose
only technology-neutral conventions shared by the sources. Do not change
~/.config/agent-skills/project-scaffold/scaffold/ until I approve the proposal.

Reference projects:
- <absolute path>
- <absolute path>
```

The scaffold content lives in `scaffold/` under `PROJECT_SCAFFOLD_HOME` (default
`~/.config/agent-skills/project-scaffold/`); set `PROJECT_SCAFFOLD_HOME` to use
another location. After the contents are approved, Git management, creating a
remote, and pushing are separate choices that require explicit approval.

A Bootstrapped Org Scaffold is a starting hypothesis, not a finished standard.
Apply it to one or more representative repositories, then run audits over time.
Promote only the conventions supported by repeated evidence; this prevents a
single project's stack or domain rules from becoming organization-wide defaults.

## 3. Use the scaffold in a project

Once the Org Scaffold is approved, `project-scaffold` supports two project
modes. In either mode, it proposes a plan first and changes only what you
approve.

### Create a new project structure

For an empty or newly requested target, use the Org Scaffold to create the
approved repository structure and operational files. It does not authorize the
agent to select a product stack, copy application code, or make unrequested
technology decisions.

```text
Use $project-scaffold to create a new project structure at <absolute path> from
the current Org Scaffold. First show the change plan, including every file and
directory to create. Do not create or change anything until I approve the plan.
Do not add application code, dependencies, product rules, or technology choices
unless I explicitly request them.
```

### Apply to an existing repository

For an existing repository, first inspect only the permitted operational metadata
and preserve its local rules. Apply only the conventions that the approved plan
lists; an Org Scaffold is not permission to rewrite every difference.

```text
Use $project-scaffold to apply the current Org Scaffold to this project. First
inspect only the root AGENTS.md, README.md, .gitignore, docs/README.md,
docs/AGENTS.md, docs/00_templates/, and shallow directory structure. Show the
change plan and do not modify files until I approve it. Preserve existing local
conventions unless I explicitly ask to change them.
```

For either mode, the plan should identify every file and directory to create or
change, the convention and source behind it, compatibility effects, and
intentional local exceptions. Only after approval should the agent make minimal
changes and record the scaffold source and reference in `.project-scaffold.json`
at the project root.

## 4. Feed durable improvements back into the Org Scaffold

After the scaffold has been applied, run `project-scaffold-audit` whenever a
project reveals a convention that might help other projects. Repeat this with
representative repositories as they evolve; an Org Scaffold should grow from
several observed uses, not from one initial Bootstrap alone.

```text
Use $project-scaffold-audit to compare this project with its applied Org
Scaffold. Do not change anything. Inspect only the root AGENTS.md, README.md,
.gitignore, docs/README.md, docs/AGENTS.md, docs/00_templates/, and shallow
directory structure. Classify every difference as Local, Promote, Remove-Migrate,
or Needs-decision. For each item, report evidence, affected scope, benefit,
compatibility risk, and the smallest proposed change.
```

| Classification | Meaning |
| --- | --- |
| `Local` | Keep it in the project: it is justified by the stack, domain, or delivery model. |
| `Promote` | A durable, technology-neutral convention that belongs in the Org Scaffold. |
| `Remove-Migrate` | An outdated, duplicate, or conflicting convention that should be cleaned up. |
| `Needs-decision` | There is not enough evidence to establish intent; identify what would settle it. |

An audit is read-only by default. Approve individual `Promote` or
`Remove-Migrate` items before changing the Org Scaffold. For a Git-managed Org
Scaffold, make the minimal change on a branch and open a PR. For a non-Git-managed
scaffold, make the minimal direct edit and re-read it to verify the result.

## 5. Improve the Built-in Starter only when appropriate

If a promoted convention benefits most new projects and is independent of a
specific technology, organization, or domain, flag it in the audit report as a
**Global Promote candidate**. Do not edit the Built-in Starter during the audit.

Maintainers of the `agent-skills` repository can then request:

```text
Use $project-scaffold-maintain to integrate only the Global Promote candidates
from this audit report into the Built-in Starter. Reject technology-,
organization-, and domain-specific candidates. Run gh skill publish --dry-run,
then create a branch and PR. Do not merge or release.
```

## Decision guide

- Start from the Built-in Starter for the fastest path; you can evolve the Org
  Scaffold later.
- Bootstrap when you already have several mature projects that demonstrate the
  conventions you want to share.
- Treat a convention seen in only one project as `Local` unless the user endorses
  it for the organization.
- Keep every application and update flow as: **change plan → approval → minimal
  change**.
