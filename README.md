# agent-skills

**English** | [日本語](README.ja.md)

Agent skills published for install with [`gh skill`](https://cli.github.com/manual/gh_skill).
One directory per skill under `skills/<name>/`.

| Skill | Purpose |
| --- | --- |
| `agmsg-team` | In an environment with [agmsg](#agmsg) and both the `claude` / `codex` CLIs installed, form a peer team with the session's selected model as Task Owner and split implementation, investigation, QA, review, and consultation across roles. Startable from Claude Code or Codex. |
| `project-scaffold` | Create and update an Org Scaffold and apply it to projects. Supports starting from the Built-in Starter, bootstrapping from existing projects/docs, or using an existing Org Scaffold. |
| `project-scaffold-audit` | Diff a project against its Org Scaffold and classify each difference as `Local` / `Promote` / `Remove-Migrate` / `Needs-decision`. After review, reflect approved changes into the Org Scaffold. |
| `project-scaffold-maintain` | For maintainers of this repo: review the improvements an audit marked `Global Promote` and open a PR folding them into the Built-in Starter. |

## Install

```sh
gh skill install bracelabs/agent-skills <skill-name>
```

## How project-scaffold works

### Three layers

- **Built-in Starter** — the generic template bundled with the skill. A starting
  point, never mandatory.
- **Org Scaffold** — a team's or organization's own standard, grown over time.
  Lives at `$PROJECT_SCAFFOLD_HOME` (default
  `~/.config/agent-skills/project-scaffold/`). Put it under git to distribute it
  across a team or organization.
- **Project** — a repository the Org Scaffold has been applied to.

Improvements flow bottom-up, and every step is reflected only after explicit
approval.

### project-scaffold — create an Org Scaffold and apply it

On first run, choose how to obtain the Org Scaffold:

1. **From the Built-in Starter** — base it on the bundled template. Fast, generic.
2. **Bootstrap from existing assets** — read example projects, `AGENTS.md`, or
   development guidelines once and synthesize it (needs `project-scaffold-audit`).
3. **Use an existing Org Scaffold** — point at one that already exists (local path
   or git repo).

- For 1 and 2, `git init` and remote-repo creation are offered afterwards (skip if
  not wanted).
- Once ready, apply it to a project — always "propose a change plan → approve →
  minimal edits".

### project-scaffold-audit — grow the Org Scaffold

When something belongs in the standard, audit the project:

1. Diff the project against the Org Scaffold.
2. Classify each difference:
   - `Local` — project-specific; keep as-is.
   - `Promote` — a candidate to fold into the Org Scaffold.
   - `Remove-Migrate` — outdated, duplicated, or conflicting; a candidate to clean up.
   - `Needs-decision` — not enough evidence to decide.
3. Review and adjust, then reflect only approved items into the Org Scaffold
   (PR if git-managed, otherwise a direct edit).

Items generic enough for the Built-in Starter are only flagged `Global Promote`;
folding them in is `project-scaffold-maintain` (maintainers only).

<a id="agmsg"></a>
## agmsg

`agmsg-team` builds on **agmsg** — a separate skill for peer-to-peer messaging and
lifecycle control between agent sessions (`send`/`inbox`/`team`/`spawn`/`despawn`).
It requires agmsg (<https://github.com/fujibee/agmsg>) installed, and both the
`claude` and `codex` CLIs available — a team spawns peers on both.

## Maintaining this repo

- `gh skill publish --dry-run` must pass; release with `gh skill publish --tag vX.Y.Z`
  (creates a GitHub release; consumers install from tags).
- Don't hand-edit `skills/project-scaffold/starter/` — that is
  `project-scaffold-maintain`'s job.
- For local iteration, symlink `skills/<name>` into `~/.claude/skills/` and
  `~/.codex/skills/`. Keep a symlinked skill and a `gh skill install`/`update` of
  the same name apart — remove one before using the other.
