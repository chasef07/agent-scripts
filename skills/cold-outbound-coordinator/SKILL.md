---
name: cold-outbound-coordinator
description: "Coordinate cold outbound prospecting, drafting, approval, sending, reply triage, follow-up, suppression, and reporting from Chase's local outbound ledger plus Airtable/Gmail integrations."
---

# Cold Outbound Coordinator

Control-plane skill. Keep the main thread on queue decisions, proof, and owner asks. Use bounded workers for research, enrichment, drafting, reply triage, and reports when useful.

## Source

- Project: `/Users/chasefagen/Projects/outbound_ops`
- Ledger: `/Users/chasefagen/Projects/outbound_ops/data/outbound.sqlite`
- CRM: Airtable base `appJlbWspuuc34LvZ`, Companies table `tblhBmRj4vir0UF2z`
- Email: Gmail through `scripts/gog_acuity_readonly.sh` by default; send through `scripts/gog_acuity.sh` only when policy permits.

## Defaults

- Auto-send only when policy permits it and compliance gates pass.
- If policy blocks send, draft/log instead.
- Check suppression before every prospect, draft, follow-up, and send.
- Stop sequence on reply, bounce, unsubscribe, explicit not-interested, or legal/angry response.
- Never invent personalization. Use website, CRM, prior touches, inbox, or public-source evidence.
- Never print secrets, raw inbox dumps, patient data, private customer context, or credentials.
- Keep emails short, accurate, non-deceptive, and easy to opt out of.

## Cold Email Theory

- Goal: get a meeting, not explain the product.
- One email = one reason, one pain, one outcome, one ask.
- Best length: 5-8 short lines before signature.
- Structure: relevant signal -> Acuity outcome -> simple meeting CTA.
- Use concrete Acuity language: AI receptionist for ophthalmology; answer patient calls; book visits into the EMR; transfer staff-only requests with context; keep the front desk focused on patients in the office.
- Avoid fake personalization, hype, long feature lists, ROI claims without evidence, and vague AI language.
- CTA default: `Open to a quick 15-minute workflow review next week?`

## Proof Points

- Approved anonymous case study: Acuity is live in a six-location ophthalmology practice that had been dropping about 200 calls a week; Acuity now helps them capture and book roughly 200 appointments a week.
- Optional revenue line: this represents over $100k in weekly captured revenue. Use sparingly, only when the email would benefit from a commercial proof point.
- Do not name the practice unless Chase explicitly approves it.
- Do not promise identical results. Phrase as observed proof, not a guarantee.
- Best cold email use: one short proof sentence after the Acuity outcome, then the meeting CTA.

## Classify

Lead:

- `Ready to draft`: good fit, reachable, not suppressed, evidence exists.
- `Needs enrichment`: missing contact/email or weak evidence.
- `Needs Chase`: strategic/account decision or risky message.
- `Suppress`: unsubscribe, bad fit, competitor, existing customer, invalid domain, do-not-contact.
- `No action`: stale, duplicate, or not worth pursuing.

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

## Loop

1. Run report.
2. Pull due companies.
3. Check suppression.
4. Enrich only due or high-priority rows.
5. Draft evidence-backed touch.
6. Mark draft `needs_approval`.
7. Send only when policy permits, caps allow it, and compliance footer is present.
8. Read replies as sanitized summaries.
9. Classify and update next action/date.
10. Report meaningful changes only.

## Commands

```bash
cd /Users/chasefagen/Projects/outbound_ops
python3 scripts/outbound.py report
python3 scripts/outbound.py due --date YYYY-MM-DD
python3 scripts/outbound.py export-companies reports/companies.csv
```

Gmail read-only examples:

```bash
scripts/gog_acuity_readonly.sh gmail search 'newer_than:1d' --max 20 --json
scripts/gog_acuity_readonly.sh gmail get <messageId> --sanitize-content --json
```

## Worker Prompt

```text
Repo: /Users/chasefagen/Projects/outbound_ops
Task: <prospect enrichment|draft emails|reply triage|report>
Permission: <triage|local-edit|draft-only|send-approved>

Use the local ledger first.
Do not send email unless permission is send-approved and the touch is approved.
Respect suppression.
No raw secrets, raw inbox dumps, patient data, or private customer context.

Return:
- rows reviewed
- changes proposed or made
- evidence used
- drafts created
- owner decisions needed
- risks
```

## Go Live Gates

- `gog auth doctor --check` clean for the sending account.
- Sender account, signature, and physical address configured.
- Suppression import complete.
- Airtable sync tested read-only.
- First campaign copy approved.
- First 10 sends manually approved.
- Bounce and reply classification tested.
