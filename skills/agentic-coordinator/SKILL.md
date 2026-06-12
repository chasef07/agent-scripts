---
name: agentic-coordinator
description: "Coordinate Chase repo work with an orchestrator thread: triage queues, create or reuse worker threads, monitor every 5 minutes, maintain a local log, and prepare draft PRs or owner decisions."
---

# Agentic Coordinator

Control-plane skill. Keep root thread light: inspect, delegate, monitor, decide, report. Put repo investigation, edits, tests, PRs, and proof in worker threads.

## Scope

Default repos:

- `/Users/chasefagen/abita_agent`
- `/Users/chasefagen/abita_middleware`
- `/Users/chasefagen/acuity_site`
- `/Users/chasefagen/onlinedoctornote`

Do not broaden unless Chase names more repos or says all/everything.
Ignore archived/retired/suppressed repos after Chase says to.

## Persistent Log

- Root owns `~/chase-orchestrator.md`; workers never edit it.
- Append only meaningful changes: worker created/reassigned, queue decision, PR opened, blocker, owner decision, landed work.
- Include repo path, full GitHub URLs, worker title, phase, exact blocker.
- Never log secrets, tokens, PHI, raw transcripts, or routine polling.

## Permissions

Treat each permission separately:

- `triage`: inspect repo/GitHub/CI; no edits.
- `delegate`: create/reuse/rename/steer worker threads.
- `local-edit`: worker may edit and test locally.
- `push-pr`: worker may push branch and open/update draft PR.
- `direct-main`: direct `main` push; only when Chase explicitly asks.
- `merge-release`: merge, close, release, tag, publish; only with explicit current request.

Default when unclear: `triage` only. For this setup, prefer `push-pr` over `direct-main`.

## Operating Loop

1. Read `~/chase-orchestrator.md` if present.
2. Inspect active worker threads before steering them.
3. For each scoped repo, check only what matters: local status, open PRs/issues, CI, recent blockers.
4. Classify items:
   - `Autonomous`: bounded bug/CI/test/docs/internal fix, clear verification path.
   - `Needs owner`: product choice, privacy/security call, missing credential/access, unsafe live proof, destructive choice.
   - `Blocked`: tool/auth/CI/environment issue with exact next step.
   - `Done`: proof complete; worker can be archived or reassigned.
5. Delegate autonomous work only within granted permission.
6. Ask Chase only with concrete choices, proof, risk, and recommendation.

## Worker Threads

- One repo/task per worker.
- Reuse existing repo worker when continuing related work.
- Rename on assignment: `<Repo>: <short current task>`.
- Examples:
  - `abita_agent: daily transcript review`
  - `abita_middleware: availability CI fix`
  - `acuity_site: portal call review`
  - `onlinedoctornote: schema drift debug`
- Workers do not create subworkers, manage threads, or edit the orchestrator log.
- Do not interrupt active coherent work. Steer only if blocked, done, wrong repo/task, unsafe, or clearly drifting.

## Worker Prompt

Use this shape:

```text
Repo: /Users/chasefagen/<repo>
Task: <exact task or URL>
Permission: <triage|local-edit|push-pr>

Use repo AGENTS.md and relevant docs first.
Do not spawn subworkers or manage other threads.
Respect unrelated dirty changes.
Implement only within scope.
Run focused checks and the repo's normal validation when practical.
Use $autoreview before publish when code changed and the skill is available.

Return:
- status
- files changed
- commands/proof
- PR/commit/check URLs if any
- blockers
- exact next decision needed
```

## Repo Bias

- `abita_agent`: runtime/tool/state truth first; transcript-backed fixes only; no prompt-only safety.
- `abita_middleware`: middleware is source of truth for AMD/API contracts; verify handlers/domain tests.
- `acuity_site`: portal/call-review behavior from DB/runtime evidence; keep UX changes scoped.
- `onlinedoctornote`: schema/CI/admin changes need focused validation and generated-client awareness.

## Monitoring

- Five-minute heartbeat only when Chase asks to keep monitoring.
- Report meaningful changes only: new blocker, done worker, PR ready, CI failed/passed, owner decision needed.
- No routine "still checking" reports unless asked.
- On idle/completed worker: inspect repo queue, then reassign, ask decision, or archive.

## Owner Decision Brief

Never ask from a bare URL. Include:

- URL/title
- what changes and who benefits
- proof done
- risk/missing evidence
- recommendation
- exact choices and consequences

## Stop Conditions

Stop and ask when:

- permission boundary would be crossed
- destructive git needed
- direct main/merge/release requested implicitly but not explicit
- credential/live access missing
- worker has conflicting newer instructions
- evidence is weak and code change would be speculative
