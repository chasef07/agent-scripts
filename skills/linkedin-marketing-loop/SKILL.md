---
name: linkedin-marketing-loop
description: "Run Acuity Health LinkedIn company-page marketing from the local linkedin_ops ledger: autonomous Acuity company-page publishing, orchestrator triage, post drafts, metrics sync, comment triage, approval packets, and campaign reports."
---

# LinkedIn Marketing Loop

Run LinkedIn as a ledger-backed marketing loop. Acuity Health company-page publishing is now autonomous under the standing policy below; comments, replies, DMs, paid promotion, customer naming, and unusual claims still require explicit approval.

## Sources

- Project: `/Users/chasefagen/Projects/linkedin_ops`
- Ledger: `/Users/chasefagen/Projects/linkedin_ops/data/linkedin_ops.sqlite`
- CLI: `/Users/chasefagen/Projects/linkedin_ops/scripts/linkedin_ops.py`
- Campaign: `/Users/chasefagen/Projects/linkedin_ops/config/campaigns/ophthalmology-ai-receptionist.json`
- Policy: `/Users/chasefagen/Projects/linkedin_ops/config/policy.json`

Never print access tokens, client secrets, raw private comments, credentials, or private customer context.

## Modes

### Triage

Use for heartbeats and root orchestrator threads, including `Big Autistic Daddy 2.0`.

- Check auth health.
- Sync/report metrics when safe.
- List draft, approved, and published counts.
- Surface blockers and decision needs.
- Do not reply, create comments, send DMs, run paid promotion, name customers, or make unusual claims.
- Reuse existing workers for the same lane.

### Approval Packet

Use when Chase needs a concise decision packet.

- Show draft IDs and full post copy.
- Explain why each draft exists.
- Show approved-but-unpublished posts.
- Recommend approve, revise, publish, or hold.
- Include suggested graphic concepts only; do not create/upload graphics unless asked.

### Worker

Use for content drafts, metrics analysis, engagement triage, or campaign reports.

- Work one lane at a time.
- Use the ledger as source of truth.
- Keep writes local unless the boundary explicitly grants publishing.
- Do not auto-reply to comments.

### Autonomous Publish

Use for the recurring Acuity Health company-page lane. Chase approved this standing rule on 2026-06-29: post to Acuity Health once every three days without per-post approval, using content based on Acuity's own website and campaign facts.

- Run at most one post per automation run.
- Use `/Users/chasefagen/Projects/linkedin_ops/scripts/linkedin_ops.py autopublish --campaign ophthalmology-ai-receptionist --yes`.
- Keep content grounded in `https://acuityhealth.io`, the local campaign config, published metrics, and prior ledger facts.
- Prefer concrete operational themes: EMR booking, after-hours capture, call summaries, transfers, two-way texting, analytics, missed-call capture, and front-desk relief.
- Do not name a customer unless Chase explicitly approves that name.
- Do not promise guaranteed results or make clinical claims.
- Do not comment, reply, send DMs, upload paid media, or run paid promotion.
- Record the resulting LinkedIn URN through the CLI and report only the post id, URN, and any blocker.

### Manual Publish

Use when Chase approves an exact post or when publishing a specific ledger row outside the autonomous lane.

- Require local `APPROVED` status.
- Require `publish --yes`.
- Publish one exact post at a time.
- Record the resulting LinkedIn URN through the CLI.

## Commands

```bash
cd /Users/chasefagen/Projects/linkedin_ops
python3 scripts/linkedin_ops.py auth-status
python3 scripts/linkedin_ops.py seed-campaign config/campaigns/ophthalmology-ai-receptionist.json
python3 scripts/linkedin_ops.py sync-metrics --days 30
python3 scripts/linkedin_ops.py sync-comments
python3 scripts/linkedin_ops.py list-posts --status DRAFT
python3 scripts/linkedin_ops.py list-posts --status APPROVED
python3 scripts/linkedin_ops.py autopublish --campaign ophthalmology-ai-receptionist --yes
python3 scripts/linkedin_ops.py report --campaign ophthalmology-ai-receptionist
```

Approval and publish:

```bash
python3 scripts/linkedin_ops.py show-post --post-id <id>
python3 scripts/linkedin_ops.py approve --post-id <id> --notes "<approval>"
python3 scripts/linkedin_ops.py publish --post-id <id> --yes
```

## Content Standard

- Speak to ophthalmology owners, administrators, and office managers.
- Make one point per post.
- Prefer operational truth over generic AI commentary.
- Best topics: missed-call capture, front-desk relief, after-hours booking, transfer visibility, scheduling workflow, office-manager reporting.
- Use the six-location proof point when it sharpens the post.
- Do not name the practice unless Chase explicitly approves it.
- Do not promise identical results.

## Classify

Post:

- `Draft ready`: clear, on-message, approval-worthy.
- `Needs revision`: useful idea, weak copy.
- `Approved`: Chase approved exact copy.
- `Published`: live and recorded.
- `Hold`: not useful now.

Engagement:

- `Sales follow-up`
- `Comment reply needed`
- `Useful signal`
- `Ignore`
- `Data gap`
- `API blocker`

## Worker Prompt

```text
Goal: Acuity LinkedIn marketing loop
Thread: linkedin_ops: <short lane>
Repo: /Users/chasefagen/Projects/linkedin_ops
Task: <autonomous publish|approval packet|content drafts|metrics report|engagement triage|publish approved post>
Boundary: <triage|local-edit|autonomous-publish|approval-packet|publish-approved>; no tokens, raw private comments, credentials, or private customer context.

Use $linkedin-marketing-loop.
Ledger first.
Do not reply, comment, send DMs, upload media, run paid promotion, or name customers unless the boundary explicitly allows it. Autonomous Acuity company-page publishing is allowed only through the `autopublish --yes` lane.

Return:
- status
- posts reviewed
- drafts/recommendations
- metrics signal
- comments needing follow-up
- blockers
- decisions needed
- recommended next step
```

## Output

Report only meaningful changes:

- new drafts
- approval decisions needed
- approved-but-unpublished posts
- posts published
- metrics signal changed
- comments needing follow-up
- auth/API blockers
