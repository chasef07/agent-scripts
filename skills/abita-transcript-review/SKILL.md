---
name: abita-transcript-review
description: "Review recent or unreviewed Abita AgentCall transcripts from the acuity_site portal database for tool-call anomalies, failed reviews, booking/reschedule/cancel/insurance/transfer issues, and call smoothness opportunities. Use when asked to audit Abita calls, review transcripts, find production call failures, inspect tool errors, or prepare worker lanes from call evidence."
---

# Abita Transcript Review

Read-only diagnosis skill. Use portal `AgentCall` rows as source of truth. Do not edit code, create issues, push, or open PRs from this skill unless Chase explicitly asks.

## Source

- Primary repo: `/Users/chasefagen/acuity_site`
- Fix owners:
  - `/Users/chasefagen/abita_agent`: prompt, runtime, state, tool boundary, voice behavior
  - `/Users/chasefagen/abita_middleware`: AMD/API contracts, availability, booking, insurance source of truth
  - `/Users/chasefagen/acuity_site`: ingestion, normalization, DB, portal, admin analytics
- Re-check current schema/helpers before present-tense claims:
  - `prisma/schema.prisma`
  - `lib/call-types.ts`
  - `lib/admin-analytics.ts`
  - `lib/call-normalization.ts`
  - `lib/portal-overview.ts`

## Access

- Start with `git -C /Users/chasefagen/acuity_site status -sb`.
- Never print `DATABASE_URL`, tokens, phone numbers, DOBs, member IDs, or raw transcripts.
- If shell env lacks DB config, check exact key names only:

```bash
cd /Users/chasefagen/acuity_site
perl -ne 'print "$1\n" if /^\s*([A-Za-z_][A-Za-z0-9_]*)\s*=/' .env.local | sort
```

- Load `.env.local` without echoing values. Run read-only SQL only.
- If DB access is missing, stop with the exact missing piece; do not invent findings.

## Flow

1. Choose window:
   - default: last 24h
   - use 48h when Chase asks for broader transcript review
   - use unreviewed/recent when an orchestrator automation asks for new work
2. Aggregate first. Do not read transcripts linearly.
3. Select high-signal calls.
4. Inspect selected calls only, comparing transcript, tools, and backend-confirmed state.
5. Classify each finding as:
   - `Autonomous`: clear bug or deterministic improvement with focused validation path
   - `Needs Chase`: product/office policy/privacy decision
   - `Monitor`: real but not worth a code change yet
   - `Data gap`: evidence missing or ingestion incomplete
   - `No action`: behavior correct after inspection
6. For `Autonomous` findings, prepare a worker handoff. Do not fix inside this review thread.

## Aggregate Signals

Query metadata before full JSON payloads:

```sql
SELECT id, name FROM practice WHERE lower(name) LIKE '%abita%';

SELECT
  "callId",
  "startedAt",
  "durationSec",
  status,
  "toolCalls",
  "toolErrors",
  "needsReview",
  "reviewStatus",
  "reviewAverageScore",
  "bookedAppointment",
  "confirmedAppointment",
  "cancelledAppointment",
  transferred,
  "outcomeSummary"
FROM agent_call
WHERE "practiceId" = '<verified-abita-practice-id>'
  AND "startedAt" >= now() - interval '24 hours'
ORDER BY "toolErrors" DESC, "needsReview" DESC, "durationSec" DESC
LIMIT 100;
```

Rank first:

- `toolErrors > 1`
- `toolErrors = 1` on booking, reschedule, cancel, insurance, transfer, patient identity
- `reviewStatus = 'failed'`
- `reviewResult.passed = false`
- `needsReview = true`
- `status IN ('FAILED', 'ABANDONED', 'ESCALATED')`
- long calls with no booking, confirmation, cancellation, transfer, or clear resolution
- repeated same-tool loops, especially `get_availability`, `check_insurance`, `verify_patient`, `book_appt`, `reschedule_appt`
- high interruption, language switching, runtime errors, or latency spikes when present in `data.sessionEvents`, `data.language`, or `data.turns`

## Inspect

For each selected call, compare:

- caller request
- agent promise / explanation
- tool called
- tool result
- final `AgentCall` fields
- `data.toolExecutions`, `data.sessionEvents`, `data.language`
- existing `reviewResult` if present

Use short paraphrases and turn numbers. Avoid raw transcript dumps.

## Heuristics

Tool anomaly:

- agent says success but tool errored
- tool succeeds but caller is not clearly told
- final flags contradict tool output
- repeated tool calls indicate loop or missing state
- tool error recovered cleanly may be `Monitor`, not a bug

Workflow checks:

- Booking: slot offered, caller confirmed exact slot, referring doctor handled, `book_appt` success reflected in final state.
- Reschedule: old appointment confirmed, new slot confirmed, cancellation/booking receipt is structured, no accidental new booking.
- Cancel: appointment identity confirmed, explicit cancel confirmation, loaded pre-call appointment state considered.
- Confirm: patient identity and appointment target are clear before confirmation.
- Insurance: medical vs routine vision lane is right, plan ambiguity handled, no repeated `check_insurance` loop.
- Transfer: correct for emergency, caller insists on human, unsupported request, or office policy; suspicious if used as avoidable fallback.
- Staff task: non-live office work captured with actionable summary; emergency/human-now paths transfer instead.

Smoothness opportunities:

- one-date-at-a-time availability loops
- repeated openings/acknowledgments
- long pauses or high latency around tools
- caller confusion after tool output
- avoidable re-asking for known state
- Spanish/language switch friction
- STT confidence or transcript split problems

False-positive guards:

- Do not assume every valid cancellation needs same-call `confirm_appt`; preloaded appointment state plus explicit confirmation can be valid.
- Do not mark transfers bad when caller insisted, request was urgent/clinical, or policy requires live handoff.
- Do not treat failed review infrastructure as a failed call without transcript/tool evidence.
- Missing audio on long calls can be storage/cap behavior; verify the row and payload fields before blaming ingestion.
- No-change is valid after evidence review.

## Output

Keep it compact:

```text
Window: <range>
Calls scanned: <n>
Flagged: <n>
Autonomous: <n>
Needs Chase: <n>
Monitor/Data gap: <n>

Finding:
- Call: <callId or portal URL>
- Symptom: <one line>
- Evidence: <turn/tool/final-state proof, no raw PHI>
- Owner: <abita_agent|abita_middleware|acuity_site|Chase>
- Confidence: <high|medium|low>
- Recommendation: <fix|monitor|ask Chase|no action>
- Worker task: <exact task if autonomous>
- Permission: <triage|local-edit|push-pr>
```

For autonomous worker tasks, hand off one repo/task only and include:

- call IDs
- exact evidence
- suspected boundary
- expected regression test
- preferred smallest deterministic fix

## Delegate

If `Autonomous` and owner is `abita_agent`, route through the orchestrator to a worker:

```text
Title: abita_agent: <short issue>
Repo: /Users/chasefagen/abita_agent
Task: <call evidence + suspected runtime/state/tool/prompt boundary>
Permission: <triage|local-edit|push-pr>
```

Rules:

- Use `push-pr` only when Chase or the orchestrator has explicitly granted it.
- Otherwise use `local-edit` for fixes or `triage` for uncertain evidence.
- Worker must implement the smallest deterministic runtime/state/tool-boundary fix.
- Prompt-only fixes are acceptable only when the issue is truly wording/routing.
- Worker must add focused regression coverage for accepted behavior bugs.
- Worker must run focused checks and `$autoreview` before publishing.
- Worker opens a draft PR only with `push-pr` permission.

If owner is `abita_middleware` or `acuity_site`, use the same worker pattern with that repo path. If owner is `Chase`, ask with concrete options and consequences.
