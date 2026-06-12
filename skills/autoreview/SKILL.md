---
name: autoreview
description: "Autoreview closeout: run Codex structured review on dirty local work, branch diffs, PRs, or commits before final, commit, push, or ship."
---

# Autoreview

Run Codex structured review as a closeout check. Review is advisory; code-path truth wins.

## Use

- User asks for autoreview, Codex review, second-model review, or review closeout.
- Non-trivial code changed before final, commit, push, PR, or ship.
- PR/branch was fixed and needs one clean final pass.

## Contract

- Verify every finding in real code before fixing.
- Read adjacent code; follow the runtime path, not just the diff.
- Check dependency docs/source/types when behavior depends on external contracts.
- Reject speculative, unrealistic, overbroad, or complexity-adding findings.
- If accepted finding shows a bug class, scan current PR scope for sibling cases.
- Prefer simple fixes at the right owner boundary.
- If a review fix changes code: rerun focused tests, then rerun review.
- Stop when helper/review exits clean with no accepted/actionable findings.
- Do not run extra reviews for nicer wording or a second clean line.
- Do not push just to review. Push only when user asked for push/ship/PR update.

## Target

Prefer helper:

```bash
AUTOREVIEW="${AUTOREVIEW:-/Users/chasefagen/agent-scripts/skills/autoreview/scripts/autoreview}"
"$AUTOREVIEW" --help
```

Dirty local work:

```bash
"$AUTOREVIEW" --mode local
```

Branch or PR:

```bash
"$AUTOREVIEW" --mode branch --base origin/main
```

Use actual PR base when available:

```bash
base=$(gh pr view --json baseRefName --jq .baseRefName)
"$AUTOREVIEW" --mode branch --base "origin/$base"
```

Single committed change:

```bash
codex review --commit HEAD
```

Rules:

- Use local mode only for real unstaged/staged/untracked work.
- Use branch mode for committed, pushed, or PR work.
- A clean local review only proves no local patch exists.
- Fetch failure is a warning; report stale-ref risk if relevant.

## Parallel Closeout

Format first if formatting can move lines.

```bash
"$AUTOREVIEW" --parallel-tests "<focused test command>"
```

If tests or review cause edits, rerun affected tests and review once more.

## Long Runs

- Be patient; large reviews can take many minutes.
- Do not kill quiet review work just because it paused briefly.
- If helper output reports progress or the process is alive, let it finish.

## Final

Report:

- review command
- tests/proof run
- accepted findings fixed
- rejected findings and reason
- final clean result or remaining conscious risk
