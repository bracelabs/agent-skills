# agmsg ticket templates

Keep messages compact, factual, and addressed to one recipient. Use a commit hash, diff path, or exact files in place of repeated repository background.

## Delegation

```text
Task: <bounded outcome>
Status: requested
Relevant files: <paths / commit / diff>
Acceptance criteria: <checks>
Constraints: <ownership, no-edit rule, or deadline>
Reply with: <finding, patch, test result, or decision>
```

## QA / security finding

```text
Task: QA review | Security review
Status: complete
Relevant files: <paths / commit>
Findings:
- [severity] <evidence and impact>
Suggested fix: <specific remediation>
Validation performed: <tests / inspection>
Decision needed: <only if blocking>
```

## Consultant request

```text
Task: focused design consultation
Problem: <one concise statement>
Current state: <facts already investigated>
Options: A) ... B) ...
Trade-offs: <concrete costs / risks>
Recommendation: <owner's current leaning>
Relevant files: <paths, symbols, commit>
Exact question: <one decision to answer>
Reply with: decision, rationale, and constraints for implementation
```

## Specialist request and result

```text
Task: <hard bug / DB / concurrency / architecture question>
Status: escalated after <attempts and observed results>
Relevant files: <paths / reproduction / logs>
Question: <narrow technical question>
Reply with: diagnosis or decision, evidence, and a minimal safe next step
```

The Task Owner sends the final resolution to the originating peer only when it affects that peer's work. Avoid copying the same long transcript to every agent.
