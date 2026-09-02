---
name: agmsg-team
description: Run software-development tasks through an agmsg peer team where the model you selected for this session is the Task Owner and Luna is the default worker. Works from Claude Code or Codex. Use for peer-to-peer multi-agent development that needs selective QA, security review, or focused model escalation; not for a permanent coordinator or a single trivial edit.
license: MIT
---

# agmsg model-selected development

Use this skill to form a task-scoped **peer team**. There is no permanent
commander and no permanently resident worker pool: the **Task Owner is the model
you selected for this session** (in Claude Code or Codex). It owns the task
end-to-end, then starts only the peer roles needed for bounded work over agmsg.

Requires the **agmsg** skill (<https://github.com/fujibee/agmsg>) installed, and
both the `claude` and `codex` CLIs available — an agmsg team spawns peers on
both. Read the installed `agmsg` skill before calling its scripts. Use only its
provided Bash scripts; never access its database, team files, or config directly.

## Start or resume a team

Resolve the team before doing task work. Use the CLI name of the current session
(`claude-code` or `codex`) wherever `<cli>` appears below.

1. If the invocation specifies a team name, use that exact name. If it is absent locally, create/join it with `join.sh`; if it exists, inspect its roster with `team.sh` and work in it. Do not create a local namesake when the request is to pull a remote team—follow agmsg's remote-pull flow instead.
2. Without an explicit name, run `whoami.sh "$(pwd)" <cli>` and `team-list.sh --scope project "$(pwd)"`. Resume the sole team associated with this project. If there is none, create a local team named from a safe lower-case slug of the project directory name (for example, `payments-api`). If more than one applies, ask the user which existing team to use; never choose arbitrarily.
3. Register the Task Owner with `join.sh <team> <agent> <cli> "$(pwd)"` when needed. Choose an unused identity such as `<team>-owner` (optionally model-qualified, e.g. `<team>-sol-owner`); inspect `team.sh <team>` first and suffix a number on collision. Remember the resolved team and agent identity for this task.
4. Check the owner's inbox at the start of every resumed turn and before declaring completion: `inbox.sh <team> <owner>`.

The first invocation therefore creates the default project team when no usable project team exists; an explicitly supplied team always wins. Teams and agent identities are local agmsg state, so do not assume they are shared across machines unless remote sync has been configured.

## Owner model selection

The user selects the Owner model **before** invoking this skill; the skill does
not change a running session's model. Match the tier to the work:

- **Fast/small tier** (Codex Luna, Claude Haiku): simple, well-scoped issues.
- **Mid tier** (Codex Terra, Claude Sonnet): ordinary feature work, multi-file changes, moderate design judgment.
- **Top tier** (Codex Sol, Claude Opus): hard bugs, high-risk changes, or major architecture only.

If the chosen model is unclear, ask the user rather than spawning a second Owner
to decide.

The selected Task Owner must establish requirements and acceptance criteria,
inspect relevant code and documentation, split only independent work, implement
or integrate the result, run appropriate tests, and verify completion. Luna
remains the default model for Engineer, Investigator, QA, Security Reviewer, and
routine delegated work.

Delegate read-only research, QA, and security review freely when independent. Do
not have two peers edit the same file concurrently; partition files or use
separate Git worktrees. A Task Owner is accountable for outcomes, not for
personally doing every step.

Use a ticket-shaped message. Send only the context the recipient needs, and cite
files, commits, diffs, or prior findings instead of making another model
rediscover them. Read [message templates](references/message-templates.md) when
delegating or escalating.

## Roles and direct communication

- **Task Owner (your selected model):** owns scope, decisions, integration, tests, and final checks.
- **Engineer / Investigator (Luna):** receives a bounded implementation or research ticket and reports directly to the owner.
- **QA (Luna):** use for behaviour changes, bug fixes, multi-file changes, regression risk, multiple acceptance criteria, or meaningful owner blind spots. QA normally reports findings and does not edit code.
- **Security Reviewer (Luna):** use only for auth/authz, sessions/cookies, APIs/external input, uploads, secrets, DB/RLS, infrastructure, dependency updates, or sensitive data. Report severity, evidence, and remediation; do not normally edit code.
- **Consultant (a mid-tier model other than the owner):** gives a focused judgment only. The owner supplies Problem, Current state, Options, Trade-offs, Recommendation, Relevant files, and one Exact question. Do not send it to re-investigate the entire repository.
- **Specialist (top tier — Sol or Opus):** use only when the current Owner is not already top tier and the problem is a hard bug still unresolved after about two fixes, complex concurrency, deep DB/SQL, high-risk security, large architectural change, or a decision a mid-tier model cannot resolve. Prefer Opus for requirements/architecture comparisons and Sol for difficult code, debugging, or repo-wide investigation.

Peers should message each other directly (`Owner → QA → Owner`, `Owner → Security → Owner`). Do not route ordinary communication through a consultant or specialist.

## Escalation and spawning

Keep the selected Owner throughout the task. Escalate only at a concrete decision
or failure point; record the response back to that Owner, who makes the
implementation and completion decision.

### On-demand lifecycle and context reuse

Treat every non-owner role as an on-demand worker: `spawn` it for one bounded
ticket, wait for its report, incorporate the result, then `despawn` it. Do not
keep idle QA, security, consultant, or engineer processes around between work
items.

`spawn.sh <type> <name> --project "$(pwd)" [--team <team>] [--model <cli-model-id>] [--boot-prompt "<ticket>"] [--no-wait] [--fresh]`
pre-joins `<name>`, opens a tmux pane (inside tmux) or a new terminal, and starts
the CLI. A `claude-code` spawn blocks until its watcher attaches; a `codex` spawn
returns immediately, so a codex peer must get its assignment via `--boot-prompt`,
not a message sent after it is idle.

To bring a role back later in the same task, `spawn` the **same name again**
without `--fresh` — agmsg resumes its recorded session, so send only the delta
(status, relevant files/commit, one next question). Use `--fresh` only when prior
context would mislead.

`despawn.sh <team> <owner> <name> [--force]` is the inverse of `spawn` and only
tears down a role this owner spawned. A graceful `despawn` of a `codex` peer
often reports "no live lock" (codex has no watcher) — that is normal; add
`--force` only if a terminal window is left open. Never despawn a hand-started
peer.

### agmsg interfaces

Resolve the `agmsg` skill's `scripts/` directory from wherever it is installed in
the current environment (its skill directory) rather than assuming a fixed path,
then call:

```bash
bash <agmsg-skill-dir>/scripts/send.sh <team> <from> <to> "<structured ticket>"
bash <agmsg-skill-dir>/scripts/inbox.sh <team> <agent>
bash <agmsg-skill-dir>/scripts/team.sh <team>
bash <agmsg-skill-dir>/scripts/spawn.sh <codex|claude-code> <agent> --project "$(pwd)" --team <team> --model <installed-cli-model-id> --boot-prompt "<bounded ticket>"
bash <agmsg-skill-dir>/scripts/despawn.sh <team> <owner> <spawned-agent> [--force]
```

`--model` is passed through by agmsg to the installed CLI. Use verified local
model IDs only — for Codex peers `gpt-5.6-luna`, `gpt-5.6-terra`, `gpt-5.6-sol`;
for another CLI use its documented identifier rather than inventing one. A Codex
peer without monitor delivery must receive its initial assignment through
`--boot-prompt`, not a message sent after it is idle.

Spawn only when it creates useful parallel work. Prefer one narrowly-scoped worker over multiple speculative workers; parallelize only independent tickets. For a hand-started peer, `send.sh` is sufficient. Inspect the roster before assigning; every `from` and `to` must be registered unless intentionally using agmsg's documented `--force` exception.

## Completion gate

The selected Task Owner closes the task only after requirements and acceptance criteria are met, relevant tests pass, triggered QA and security reviews are resolved, no blocker remains, and the diff contains no unrelated changes. If an expensive-model recommendation was used, validate and implement its outcome before closure.
