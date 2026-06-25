---
name: linkedin-marketing-loop
description: "Run Acuity Health LinkedIn company-page marketing ops: draft posts, approval-gated publish, sync engagement, comments, and campaign reports from the local linkedin_ops ledger."
---

# LinkedIn Marketing Loop

Use for Acuity Health company LinkedIn posts, engagement tracking, content drafts, campaign reports, or LinkedIn marketing automation.

## Source Of Truth

- Project: `/Users/chasefagen/Projects/linkedin_ops`
- Ledger: `/Users/chasefagen/Projects/linkedin_ops/data/linkedin_ops.sqlite`
- CLI: `/Users/chasefagen/Projects/linkedin_ops/scripts/linkedin_ops.py`
- Campaign seed: `/Users/chasefagen/Projects/linkedin_ops/config/campaigns/ophthalmology-ai-receptionist.json`

Never print access tokens, client secrets, raw private comments, or private customer context.

## Loop

```bash
cd /Users/chasefagen/Projects/linkedin_ops
python3 scripts/linkedin_ops.py auth-status
python3 scripts/linkedin_ops.py seed-campaign config/campaigns/ophthalmology-ai-receptionist.json
python3 scripts/linkedin_ops.py draft --campaign ophthalmology-ai-receptionist --count 5
python3 scripts/linkedin_ops.py list-posts --status DRAFT
python3 scripts/linkedin_ops.py sync-metrics --days 30
python3 scripts/linkedin_ops.py sync-comments
python3 scripts/linkedin_ops.py report --campaign ophthalmology-ai-receptionist
```

## Publishing

Publishing is always approval-gated.

1. Show the draft:
   ```bash
   python3 scripts/linkedin_ops.py show-post --post-id <id>
   ```
2. Get explicit Chase approval for that exact post.
3. Approve locally:
   ```bash
   python3 scripts/linkedin_ops.py approve --post-id <id> --notes "<approval>"
   ```
4. Publish only after explicit approval:
   ```bash
   python3 scripts/linkedin_ops.py publish --post-id <id> --yes
   ```

Do not auto-reply to comments. Surface comments that need Chase or sales follow-up.

## Report Shape

- Drafts created
- Posts published
- Top posts by impressions/clicks/comments
- Comments needing follow-up
- Content recommendation for next cycle
- Auth/API blocker, if any

