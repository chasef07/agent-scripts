---
name: cold-outbound-coordinator
description: "Coordinate Acuity Health cold outbound from the local outbound_ops ledger: prospect triage, enrichment, email drafts, approval/send gates, Gmail reply triage, suppression, follow-up, and reports. Use for growth-orchestrator heartbeats, worker handoffs, outbound audits, and safe send loops."
---

# Cold Outbound Coordinator

Run outbound from evidence, not volume. Ledger first. Keep the root orchestrator light; use workers for bounded work.

## Sources

- Project: `/Users/chasefagen/Projects/outbound_ops`
- Ledger: `/Users/chasefagen/Projects/outbound_ops/data/outbound.sqlite`
- Policy: `/Users/chasefagen/Projects/outbound_ops/config/outbound_policy.json`
- CRM: Airtable base `appJlbWspuuc34LvZ`, Companies table `tblhBmRj4vir0UF2z`
- Gmail read: `scripts/gog_acuity_readonly.sh`
- Gmail send: `scripts/gog_acuity.sh` only when all gates pass

## Modes

### Triage

Use for heartbeats and root orchestrator threads, including `Big Autistic Daddy 2.0`.

- Run report and policy check.
- Inspect due counts, reply counts, suppression status, and blockers.
- Do not research the web, draft many emails, or send.
- Reuse an existing worker for the same lane.
- Create a visible worker only for bounded work.

### Worker

Use for enrichment, drafting, reply triage, follow-up planning, or reporting.

- Work one lane at a time.
- Use the ledger before Airtable, Gmail, or web research.
- Record or propose ledger changes with exact proof.
- Stop when evidence, policy, suppression, or approval is missing.

### Send

Use only when explicitly granted.

- Run `policy-check` immediately before sending.
- Send only approved, unsuppressed touches within run/day caps.
- Record provider message/thread IDs.
- Stop on reply, bounce, unsubscribe, not-interested, or angry/legal response.

## Hard Rules

- Never print secrets, raw inbox dumps, patient data, private customer context, credentials, or full raw email bodies.
- Check suppression before prospecting, drafting, follow-up, or send.
- Never invent personalization.
- Do not email existing customers, competitors, suppressed contacts, weak-fit targets, or non-ophthalmology practices.
- Every commercial email must include sender identity, physical address, and opt-out text from policy.
- If policy blocks send, draft/log/report only.
- Prefer fewer high-fit prospects over volume.

## Commands

```bash
cd /Users/chasefagen/Projects/outbound_ops
python3 scripts/outbound.py report
python3 scripts/outbound.py policy-check
python3 scripts/outbound.py due --date YYYY-MM-DD
python3 scripts/outbound.py touches --approval-state needs_approval
```

Read Gmail safely:

```bash
scripts/gog_acuity_readonly.sh gmail search 'newer_than:1d' --max 20 --json
scripts/gog_acuity_readonly.sh gmail get <messageId> --sanitize-content --json
```

## Email Standard

- Goal: get a meeting.
- One email = one reason, one pain, one outcome, one ask.
- Keep it 5-8 short lines before signature.
- Structure: relevant signal -> Acuity outcome -> optional proof -> simple CTA.
- Default CTA: `Open to a quick 15-minute workflow review next week?`
- Avoid fake personalization, hype, long feature lists, vague AI language, and unsupported ROI claims.

Approved proof:

- Acuity is live in a six-location ophthalmology practice that had been dropping about 200 calls a week.
- Acuity now helps them capture and book roughly 200 appointments a week.
- Optional commercial line: over `$100k` in weekly captured revenue.
- Use anonymously unless Chase explicitly approves naming the practice.
- Never imply identical results are guaranteed.

## Classify

Prospect:

- `Ready`: good fit, reachable, unsuppressed, evidence exists.
- `Needs enrichment`: missing contact/email or weak evidence.
- `Suppress`: unsubscribe, bad fit, competitor, existing customer, invalid domain, do-not-contact.
- `Needs Chase`: strategic account, risky claim, unusual send decision.
- `No action`: duplicate, stale, or not worth pursuing.

Reply:

- `Interested`
- `Meeting requested`
- `Objection`
- `Not now`
- `Not interested`
- `Wrong person`
- `Referral`
- `Unsubscribe`
- `Bounce`
- `Angry/legal`
- `Unknown`

## Worker Prompt

```text
Goal: Acuity outbound growth loop
Thread: outbound_ops: <short lane>
Repo: /Users/chasefagen/Projects/outbound_ops
Task: <reply triage|prospect enrichment|email drafts|approved sends|report>
Boundary: <triage|local-edit|draft-only|send-approved>; no secrets, raw inbox dumps, patient data, or private customer context.

Use $cold-outbound-coordinator.
Ledger first.
Respect suppression and policy.
Do not send unless boundary is send-approved and gates pass.

Return:
- status
- rows reviewed
- changes made or proposed
- evidence used
- drafts/sends/replies/suppressions
- blockers
- decisions needed
- recommended next step
```

## Output

Report only meaningful changes:

- prospects added or enriched
- drafts created
- emails sent
- replies classified
- suppressions added
- blockers
- Chase decisions needed
