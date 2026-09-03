# agent-skills

**English** | [日本語](README.ja.md)

Agent skills published for install with [`gh skill`](https://cli.github.com/manual/gh_skill).

| Skill | Purpose |
| --- | --- |
| `agmsg-team` | In an environment with [agmsg](https://github.com/fujibee/agmsg) and both the `claude` / `codex` CLIs installed, form a peer team with the session's selected model as Task Owner and split implementation, investigation, QA, review, and consultation across roles. Startable from Claude Code or Codex. |
| `project-scaffold` | Create an Org Standard and apply it to projects. Supports starting from the Built-in Starter, bootstrapping from existing projects/docs, or using an existing Org Standard. |
| `project-scaffold-audit` | Diff a project against its Org Standard and classify each difference as `Local` / `Promote` / `Remove-Migrate` / `Needs-decision`. After review, reflect approved changes into the Org Standard. |
| `project-scaffold-maintain` | For maintainers of this repo: review the improvements an audit marked `Global Promote` and open a PR folding them into the Built-in Starter. |

## Install

```sh
gh skill install bracelabs/agent-skills <skill-name>
```

## How project-scaffold works

For AI agents, see the [project-scaffold setup guide](docs/project-scaffold-setup.md)
for the first-time flow.

### Three layers

- **Built-in Starter** — the generic template bundled with the skill.
- **Org Standard** — a team's or organization's own standard. Its content lives in
  `scaffold/` under `$PROJECT_SCAFFOLD_HOME` (default
  `~/.config/agent-skills/project-scaffold/`). Put it under Git to share it across
  a team or organization.
- **Project** — a repository the Org Standard has been applied to.

### project-scaffold — create an Org Standard and apply it

On first run, choose how to obtain the Org Standard:

1. **From the Built-in Starter** — base it on the bundled template. Fast, generic.
2. **Bootstrap from existing assets** — read reference projects, `AGENTS.md`, or
   development guidelines once and synthesize it (needs `project-scaffold-audit`).
   Run several audits against the reference projects afterwards to fill it out.
3. **Use an existing Org Standard** — point at one that already exists (local path
   or Git remote).

- For 1 and 2, `git init` and remote-repo creation are offered afterwards (skip if
  not wanted).
- Once ready, apply it to a project — always "change plan → approval → minimal
  change".

### project-scaffold-audit — grow the Org Standard

A first Org Standard is thin; it fills out as you run audits repeatedly across
several projects. Run an audit whenever a project has accumulated conventions
worth standardizing.

Each audit:

1. Diff the project against the Org Standard.
2. Classify each difference:
   - `Local` — project-specific; keep as-is.
   - `Promote` — a candidate to fold into the Org Standard.
   - `Remove-Migrate` — outdated, duplicate, or conflicting; a candidate to clean up.
   - `Needs-decision` — not enough evidence to decide.
3. Review and adjust, then reflect only approved items into the Org Standard
   (PR if Git-managed, otherwise a direct edit).

Items generic enough for the Built-in Starter are only flagged `Global Promote`;
folding them in is `project-scaffold-maintain` (maintainers only).

## Maintaining this repo

- `gh skill publish --dry-run` must pass; release with `gh skill publish --tag vX.Y.Z`
  (creates a GitHub release; consumers install from tags).
- Don't hand-edit `skills/project-scaffold/starter/` — that is
  `project-scaffold-maintain`'s job.
- For local iteration, symlink `skills/<name>` into `~/.claude/skills/` and
  `~/.codex/skills/`. Keep a symlinked skill and a `gh skill install`/`update` of
  the same name apart — remove one before using the other.
