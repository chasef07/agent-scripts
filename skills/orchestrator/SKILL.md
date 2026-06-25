---
name: orchestrator
description: "Turn goals, tasks, and scheduled wakeups into bounded visible worker-thread execution. Use for multi-step goals, thread delegation, worker monitoring, and synthesizing worker outputs."
---

# Orchestrator

Control plane. Keep the root thread light. Understand the goal, split the work, deploy workers, check progress, synthesize results, and report back.

The orchestrator reads enough to route correctly. Workers do deep execution.

## Loop

1. Define done.
   - What outcome, artifact, proof, deadline, or decision matters?
   - If done is unclear and guessing would waste work, ask first.

2. Split work.
   - Break the goal into independent workstreams.
   - Keep each workstream bounded by one repo, domain, artifact, or task.
   - Do not create workers for vague, weak, or purely speculative work.

3. Classify.
   - `No action`: not useful now.
   - `Monitor`: watch, but do not act yet.
   - `Autonomous`: clear scope, enough evidence, safe execution path.
   - `Needs owner`: product, money, privacy, legal, credential, external-send, or irreversible decision.
   - `Data gap`: missing evidence before action.
   - `Blocked`: exact external step required.
   - `Done`: complete with proof.

4. Choose state.
   - One-off task: no durable board unless useful.
   - Multi-step goal: create or update a goal board.
   - Recurring operation: update the recurring ledger.
   - Use the board path named by the user or prompt; otherwise default to `~/orchestrator.md`.

5. Deploy workers.
   - Reuse an active worker if it already owns the same workstream.
   - Create a visible thread only for bounded work.
   - Name workers `<domain>: <short task>`, for example `abita_agent: reschedule guard` or `webinar: email sequence`.
   - Give exact scope, relevant context, boundary, expected output, and stop condition.

6. Check workers.
   - Workers do not magically report back to the root thread.
   - The orchestrator checks worker threads, reads their latest result, and decides whether to steer, wait, ask, or close.
   - Let coherent workers continue. Intervene only when blocked, unsafe, stale, done, or drifting.

7. Synthesize.
   - Combine worker outputs into one answer or next action.
   - Resolve conflicts.
   - Bring owner questions back as concise decision briefs.

8. Close.
   - Report only meaningful changes: worker created/steered, artifact ready, PR/report/draft ready, blocker, decision needed, risk changed, or goal complete.
   - Update state if durable state is in use.

## Worker Rules

- One worker = one bounded workstream.
- Workers execute; they do not orchestrate.
- Workers do not spawn subworkers, manage threads, or edit orchestrator state.
- Prefer visible project/worktree threads for durable work.
- Use hidden subagents only for short read-heavy research when clearly useful or explicitly requested.
- Do not duplicate large worker context in the root thread.

## Worker Prompt

```text
Goal: <overall goal>
Thread: <domain>: <short task>
Task: <one exact workstream>
Boundary: <what this worker may do and must not do>
Context: <only the relevant evidence, links, paths, or constraints>

Use relevant local instructions and skills first.
Do not spawn subworkers or manage other threads.
Stay in scope.
Respect existing work and dirty state.
Run focused checks when changing files.
Stop if the boundary, missing evidence, or unsafe action blocks progress.

Return:
- status
- output/artifact
- proof
- files/links changed
- blocker
- decision needed
- recommended next step
```

## Boundary

Use plain-language boundaries instead of vague freedom.

Examples:

- `Inspect only; no edits.`
- `Draft only; do not send or publish.`
- `Edit locally and report proof; do not push.`
- `Open a draft PR if checks pass.`
- `Do not spend money, merge, release, delete, send externally, or make irreversible changes without explicit approval.`

## State

State should be active-board style, not a diary.

Goal board shape:

```md
# <Goal>

## Outcome
## Deadline
## Workstreams
## Workers
## Decisions Needed
## Blockers
## Artifacts
## Recently Completed
```

## Decision Briefs

Do not ask from vague status. Bring:

- decision needed
- evidence
- options
- recommendation
- consequence of each choice

## Stop Conditions

Stop and ask when:

- the goal is ambiguous enough that decomposition would be wrong
- the worker boundary would be crossed
- destructive or irreversible action is needed
- money, legal, privacy, credential, or external-send approval is needed
- evidence is weak and action would be speculative
