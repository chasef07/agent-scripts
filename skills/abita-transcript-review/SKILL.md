---
name: abita-transcript-review
description: "Review Abita AgentCall evidence from the acuity_site portal database for real production issues in booking, reschedule, cancel, insurance, transfer, language, latency, and tool behavior. Use for Abita transcript audits, heartbeat triage, high-signal call investigation, worker handoffs, and transcript-backed PR evidence."
---

# Abita Transcript Review

Find real call failures. Route the fix to the right owner. Stay read-only unless a separate worker boundary explicitly grants implementation.

Agent OS routing rule: Abita transcript findings are autonomous engineering work.
Do not use `reviewStatus`, `reviewResult`, or `needsReview` annotations as
required routing evidence or blockers. Missing review annotations are not a Chase
decision. If a transcript-loop signal is worth capturing, route it to a ready
worker that can make the smallest repo fix or evidence-loop improvement and open
a PR when reasonable.

## Sources

- Portal repo: `/Users/chasefagen/Projects/acuity_site`
- Fix owners:
  - `abita_agent`: prompt, voice, runtime state, tool definitions, tool guards
  - `abita_middleware`: AMD/API truth, availability, booking, insurance contracts
  - `acuity_site`: ingestion, normalization, review jobs, portal analytics
  - `Chase`: office policy, clinical policy, privacy, money, customer decision

Re-check current code before present-tense claims:

- `prisma/schema.prisma`
- `lib/call-types.ts`
- `lib/call-normalization.ts`
- `lib/admin-analytics.ts`
- `lib/portal-overview.ts`

## Modes

### Triage

Use for heartbeats and orchestrator routing.

- Run aggregate metadata only.
- Do not inspect raw transcripts.
- Classify signal.
- Reuse an existing worker/PR for the same issue class.
- Create a visible worker only for bounded, high-signal work.

### Review

Use for worker threads, specific call IDs, or direct audits.

- Inspect only selected high-signal calls.
- Compare caller request, agent promise, loaded state, tool call/result, final flags, and backend-confirmed state when available.
- Use short paraphrases and turn numbers. Do not dump transcript text.

### Handoff

Use when a finding is `Autonomous`.

- One worker = one repo/task.
- Include call IDs, exact sanitized evidence, suspected boundary, expected regression test, and smallest deterministic fix.
- Grant `push-pr` only when explicitly allowed.

## Hard Rules

- Start with `git -C /Users/chasefagen/Projects/acuity_site status -sb`.
- Load `.env.local` without echoing values.
- Run read-only SQL only.
- Never print secrets, tokens, `DATABASE_URL`, phone numbers, DOBs, member IDs, patient names, private customer context, or raw transcripts.
- Aggregate first. Do not read calls linearly.
- Use structured fields and `data.toolExecutions`; do not search broad JSON text for tool names.
- Weak evidence means `No action`, `Monitor`, or a ready evidence-loop worker;
  do not block on Chase only because review annotations are missing.
- A recovered tool error can be `Monitor`, not automatically a bug.
- If DB access is missing, stop with the exact missing piece.

Safe env-key check:

```bash
cd /Users/chasefagen/Projects/acuity_site
perl -ne 'print "$1\n" if /^\s*([A-Za-z_][A-Za-z0-9_]*)\s*=/' .env.local | sort
```

## Signals

Rank by production risk:

1. Agent promised success but the tool/backend did not confirm it.
2. Final `AgentCall` flags contradict tool results.
3. Tool errors on `book_appointment`, `reschedule_appointment`, `cancel_appointment`, `transfer_call`, `check_insurance`, or patient identity.
4. Repeated tool loops after a clear request.
5. Transfer after avoidable tool/runtime failure.
6. Failed review with tool or final-state evidence.
7. Long latency, silence, interruption, or language-switch friction tied to concrete events.
8. Smoothness opportunities after correctness issues are handled.

Handle legacy tool aliases only when present in the data, such as `book_appt`, `reschedule_appt`, or `cancel_appt`.

## Checks

- Booking: exact slot offered, caller confirmed, tool succeeded, final state reflects it.
- Reschedule: old appointment identified, new slot confirmed, no accidental new booking, final state reflects it.
- Cancel: appointment identity and explicit cancel confirmation are clear.
- Confirm: patient and appointment target are clear before confirmation.
- Insurance: medical vs routine vision lane is right; ambiguity does not loop.
- Transfer: valid for emergency, caller insistence, unsupported request, or office policy; suspicious as avoidable fallback.
- Language: detected language, tool/runtime language, and TTS response language stay coherent.
- Latency: isolate STT, EOU, LLM, tool, TTS, and provider/runtime gaps when traces exist.

False-positive guards:

- Do not blame the model when the tool contract or runtime state is the real boundary.
- Do not mark policy-required transfers as failures.
- Do not treat missing `reviewStatus`, `reviewResult`, or `needsReview`
  annotations as a transcript-routing blocker.
- Do not treat review infrastructure failure as call failure without call evidence.
- Do not assume missing audio is ingestion failure without checking row/payload fields.

## Classify

- `Autonomous`: clear bug or deterministic improvement with focused validation path.
- `Needs Chase`: product, office policy, clinical, privacy, money, credential, or customer decision. Do not use this for Abita transcript engineering findings in Agent OS; make them ready worker/PR work instead.
- `Monitor`: real signal but not worth a change yet.
- `Data gap`: evidence missing or source of truth unavailable. In Agent OS, route this as a ready evidence-loop worker, not a human approval blocker.
- `No action`: behavior correct or evidence too weak.

## Output

Keep it compact.

```text
Window: <range>
Calls scanned: <n>
Flagged: <n>
Autonomous: <n>
Needs Chase: <n>
Monitor/Data gap: <n>

Finding:
- Call: <sanitized call id>
- Symptom: <one line>
- Evidence: <turn/tool/final-state proof, no raw PHI>
- Owner: <abita_agent|abita_middleware|acuity_site|Chase>
- Confidence: <high|medium|low>
- Action: <fix|worker|monitor|ask Chase|no action>
- Worker: <title + permission if needed>
```

## Worker Prompt

```text
Title: <owner>: <short issue>
Repo: <absolute repo path>
Task: <exact issue and sanitized call evidence>
Boundary: <triage|local-edit|push-pr>; no direct main push, merge, release, destructive git, secrets, raw transcripts, or PHI.
Expected proof: <focused test/check and review evidence>

Use $abita-transcript-review for the evidence pass.
Respect dirty worktrees.
Implement the smallest deterministic fix at the right boundary.
Ask Chase before $autoreview unless already approved.
Open a draft PR when the Agent OS worker boundary grants PR creation and checks pass.
```

## PR Evidence

For transcript-backed fixes, require:

- `Issue`: production symptom and aggregate count.
- `Evidence`: sanitized call IDs plus short tool/final-state proof.
- `Why it failed`: prompt, runtime state, tool contract, middleware/API, ingestion, or policy boundary.
- `Fix`: what changed and why that boundary owns it.
- `Benefit`: expected production improvement.
- `Validation`: focused checks, `$autoreview` status when approved, and CI/check status.

Never include raw transcripts, phone numbers, DOBs, member IDs, patient names, credentials, private URLs, or office-sensitive context in a public PR body.
