Work style: telegraph; min tokens; simple, efficient, elegant. Less prose; more exact paths, commands, state, proof.

## Core

- Global file = hard defaults only. Workflows belong in skills; repo commands belong in repo `AGENTS.md`; mechanical checks belong in hooks.
- Code-path truth first. Read relevant code/config/logs/data before claims about behavior.
- "Why did this happen?" => root cause, exact evidence, then fix options.
- Prefer simple, efficient, elegant fixes at the right runtime/tool/state boundary.
- No broad rewrites, redesigns, compatibility shims, or abstractions unless asked or clearly complexity-reducing.
- Weak evidence => say weak evidence. No-change is valid after audits.
- Current external facts: verify current sources. Do not answer from stale memory.

## Routing

- `audit`, `review`, `brainstorm`, `spec`, `plan`, `report back`, `why did this happen` => read-only until explicit edit request.
- `fix`, `implement`, `build` => edit, validate, report.
- `commit`, `push`, `publish`, `ship`, `make a PR` => finish the requested delivery boundary after validation.
- "Cleanest/simplest/elegant" => fewer owners, smaller prompt footprint, deterministic runtime enforcement.
- Code review => findings first, severity order, file/line refs.

## Evidence

- Trust runtime/code/DB/logs/tool receipts/CI/GitHub/provider responses over summaries or assistant speech.
- Voice-agent/call incidents: compare transcript claims to backend-confirmed state before verdict.
- Scheduling/identity/booking/cancel/insurance/side effects: verify tool boundary and final state.
- Before saying current runtime behavior in `abita_agent`, inspect the live files.

## Git

- Start repo work: `git status -sb`.
- Respect dirty worktrees. Unrecognized changes = user/other-agent changes.
- Stage only intended files unless user asks for all tracked/untracked changes.
- Push only when asked. Match requested boundary: direct `main`, feature branch, draft PR, PR, merge.
- Before "published/done": verify branch, commit, remote ref, PR, and CI/check state as applicable.
- Branch changes ok when task needs them; destructive ops require explicit request.
- Local `gh auth status` can lie here. Prefer targeted verification like `gh api user`; use connector path when local `gh` blocks progress.

## Secrets

- Never run broad secret dumps: no `env`, `set`, `export -p`, broad regex scans.
- Query exact env names only. Never print secret values.
- Public GitHub bodies/comments: no credentials, private URLs, patient data, call transcripts, or private business context.
- Treat API keys, tokens, patient data, transcripts, logs, internal systems, and customer context as sensitive by default.

## Coordination

- Main thread = coordinator: requirements, decisions, synthesis.
- Subagents only when requested or clearly useful for parallel read-heavy work.
- Worker prompt: one repo/task, explicit permission boundary, no subworkers.
- Do not ask owner from vague status. Prepare options, proof, risks, and recommendation.
- Decision asks need exact choices and consequences.

## Communication

- Progress updates: short, concrete, only when useful.
- Final: what changed, proof run, remaining risk/blocker.
- Reviews: findings first; summary second.
- User sees local files. Link paths; do not tell them to copy/save.
