---
name: dev-team
description: Run a software-development task as a task-scoped team where the model you selected for this session is the Task Owner and delegates bounded work — implementation, investigation, QA, review, consultation — to same-CLI subagents. Works from Claude Code or Codex. Reach for an agmsg cross-CLI peer only to borrow a model the other CLI has. Not for a permanent coordinator or a single trivial edit.
license: MIT
---

# dev-team

Form a task-scoped team for one development task. The **Task Owner is the model
you selected for this session** (Claude Code or Codex). It owns the task
end-to-end and delegates only the bounded work that benefits from a separate
context or a different model.

**The default transport is your CLI's own subagent mechanism** — Claude Code's
Task/Agent tool, Codex's equivalent. It is light (no terminal, no watcher, no
permission-prompt hang) and results return inline. Use it for essentially all
delegation.

**agmsg is the exception, not the norm.** Start an agmsg peer only when the work
needs a model that lives on the *other* CLI — see
[Cross-CLI model access](#cross-cli-model-access-agmsg). A single trivial edit
needs no team at all; just do it.

## Roles

Roles are about responsibility and are independent of transport. Start only the
ones a task actually needs.

- **Task Owner (your session's model):** owns scope, decisions, integration, tests, and final checks. Never delegated.
- **Engineer / Investigator:** receives a bounded implementation or research ticket and reports directly to the owner.
- **Product Manager / PdM:** use when the product problem or requirements need definition before design or implementation. Clarify the user outcome, scope, priority, business rules, and acceptance criteria; do not choose screen structure, UI copy, or implementation, and surface unresolved product decisions to the owner.
- **Product Designer:** use when product requirements need a screen/UX specification before implementation. Define information hierarchy, flows, non-happy-path states, and concise consistent UI copy at an Engineer-ready level; do not change requirements, business rules, API/data model, or implementation approach, and report requirement gaps or contradictions to the owner.
- **Accessibility Reviewer:** use for meaningful UI or user-flow changes when accessibility review is needed. Check keyboard operation, focus, labels, error and status feedback, contrast, and instructions that rely only on colour; report barriers with evidence and a practical remediation, and normally do not edit code or change requirements.
- **QA:** use for behaviour changes, bug fixes, multi-file changes, regression risk, multiple acceptance criteria, or meaningful owner blind spots. QA normally reports findings and does not edit code.
- **Security Reviewer:** use only for auth/authz, sessions/cookies, APIs/external input, uploads, secrets, DB/RLS, infrastructure, dependency updates, or sensitive data. Report severity, evidence, and remediation; do not normally edit code.
- **Consultant:** gives a focused judgment only. The owner supplies Problem, Current state, Options, Trade-offs, Recommendation, Relevant files, and one Exact question. Do not send it to re-investigate the entire repository.
- **Specialist:** handles one narrowly defined hard problem after routing by difficulty, uncertainty, failure cost, and prior attempts. Do not select a model solely from this role title.

Every worker returns its result to the Task Owner. Workers may talk to each other
when it avoids a round-trip, but they never reassign work or choose the next
role/model — the owner is the routing and alignment authority.

Do not have two workers edit the same file concurrently; partition files or use
separate Git worktrees. Delegate read-only research, QA, and security review
freely when independent. The owner is accountable for outcomes, not for
personally doing every step.

Use a ticket-shaped message: send only the context the recipient needs, and cite
files, commits, diffs, or prior findings instead of making another model
rediscover them. Read [message templates](references/message-templates.md) when
delegating or escalating.

## Model routing

Use the cheapest model that can complete the ticket reliably. Role defines
responsibility; model defines the reasoning depth the current ticket needs — do
not pick an expensive model just because a role is called Consultant or
Specialist.

The Task Owner picks each worker's model and controls escalation. Escalate on
ambiguity, required reasoning depth, context size, failure cost, or prior failed
attempts — not task importance alone. When a worker judges its model
insufficient, it reports what is established, what is unresolved, why the current
approach falls short, and whether escalation is recommended; the owner decides
whether to retry, switch within the tier, escalate, or request an independent
opinion.

The user selects the Owner model **before** invoking this skill; the skill does
not change a running session's model. If the chosen model is unclear, ask the
user rather than spawning a second Owner to decide.

### Same-CLI subagents (default)

Map the tiers to subagent models on your current CLI:

| Tier | Claude Code owner | Codex owner |
| --- | --- | --- |
| **General** — routine implementation, mechanical edits, straightforward refactoring, simple docs | Haiku | Luna |
| **Consultant** — planning, requirements/product analysis, reviews, straightforward design, moderate debugging, trade-off analysis | Sonnet | Terra |
| **Specialist** — complex architecture or UX, difficult debugging, deep review, cross-domain reasoning, high-impact decisions | Opus | Sol |
| **Escalation** — exceptionally difficult or high-risk reasoning, unresolved system-wide problems, final independent review | *(cross-CLI: Astra)* | Astra |

The normal route is General → Consultant → Specialist → Escalation; the owner may
skip a tier when the ticket clearly justifies it. A Claude Code owner has no
same-CLI Escalation model — its Escalation tier is a cross-CLI Astra peer.

### Cross-CLI model access (agmsg)

Start an agmsg peer only to borrow a model the other CLI has:

- **Codex owner → Claude model.** Sonnet or Opus for a ticket that needs that
  model's strengths, or an independent second opinion from a different model
  family.
- **Claude Code owner → GPT model.** Luna, Terra, Sol, or Astra — most often
  Astra for Escalation-tier work (Claude has no same-CLI equivalent), or a
  cross-family second opinion at any tier.

Everything else stays on same-CLI subagents. The only other reason to use agmsg
is a peer that must persist and resume across many owner turns or be messaged
directly by other peers — rare in practice.

## Using agmsg

Only when [Cross-CLI model access](#cross-cli-model-access-agmsg) applies.

Requires the **agmsg** skill (<https://github.com/fujibee/agmsg>) installed and
the other CLI (`claude` or `codex`) available. Read the installed `agmsg` skill
before calling its scripts; use only its provided Bash scripts, never its
database, team files, or config directly. Resolve its `scripts/` directory from
wherever it is installed in the current environment.

### Resolve a team

Do this the first time a task needs an agmsg peer. Use the CLI name of the
current session (`claude-code` or `codex`) wherever `<cli>` appears.

1. If the invocation names a team, use that exact name — join it with `join.sh` if it is absent locally, or inspect its roster with `team.sh` and work in it if it exists. Do not create a local namesake when the request is to pull a remote team; follow agmsg's remote-pull flow.
2. Otherwise run `whoami.sh "$(pwd)" <cli>` and `team-list.sh --scope project "$(pwd)"` and resume the sole team associated with this project. If there is none, create a local team named from a safe lower-case slug of the project directory name (for example, `payments-api`). If more than one applies, ask the user; never choose arbitrarily.
3. Register the owner with `join.sh <team> <agent> <cli> "$(pwd)"`. Choose an unused identity such as `<team>-owner` (optionally model-qualified, e.g. `<team>-sol-owner`); inspect `team.sh <team>` first and suffix a number on collision. Remember the resolved team and identity for this task.
4. Check the owner's inbox at the start of every resumed turn and before declaring completion: `inbox.sh <team> <owner>`.

Teams and agent identities are local agmsg state — not shared across machines
unless remote sync has been configured.

### Spawn, use, despawn

Treat every agmsg peer as on-demand: `spawn` it for one bounded ticket, wait for
its report, incorporate the result, then `despawn` it. Do not keep idle peers
around between work items.

```bash
bash <agmsg-skill-dir>/scripts/send.sh <team> <from> <to> "<structured ticket>"
bash <agmsg-skill-dir>/scripts/inbox.sh <team> <agent>
bash <agmsg-skill-dir>/scripts/team.sh <team>
bash <agmsg-skill-dir>/scripts/spawn.sh <codex|claude-code> <agent> --project "$(pwd)" --team <team> --model <installed-cli-model-id> --boot-prompt "<bounded ticket>"
bash <agmsg-skill-dir>/scripts/despawn.sh <team> <owner> <spawned-agent> [--force]
```

- A `claude-code` spawn blocks until its watcher attaches; a `codex` spawn returns immediately, so a codex peer must get its assignment via `--boot-prompt`, not a message sent after it is idle.
- Bring a peer back later in the same task by spawning the **same name again** without `--fresh` — agmsg resumes its recorded session, so send only the delta (status, relevant files/commit, one next question). Use `--fresh` only when prior context would mislead.
- `despawn.sh` only tears down a role this owner spawned. A graceful `despawn` of a `codex` peer often reports "no live lock" (codex has no watcher) — that is normal; add `--force` only if a terminal window is left open. Never despawn a hand-started peer.
- Use verified local model IDs only — Codex peers `gpt-5.6-luna`, `gpt-5.6-terra`, `gpt-5.6-sol`, `gpt-6-astra`; Claude peers `claude-sonnet-5`, `claude-opus-5`. For a hand-started peer, `send.sh` alone is enough; every `from` and `to` must be registered unless intentionally using agmsg's documented `--force` exception.

Spawn only when it creates useful parallel work. Prefer one narrowly-scoped peer
over multiple speculative ones; parallelize only independent tickets.

## Completion gate

The Task Owner closes the task only after requirements and acceptance criteria
are met, relevant tests pass, triggered QA and security reviews are resolved, no
blocker remains, and the diff contains no unrelated changes. If an
expensive-model or cross-CLI recommendation was used, validate and implement its
outcome before closure.
