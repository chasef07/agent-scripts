# Agent Scripts

Shared agent instructions, skills, and small portable helpers for Chase's local workspaces.

This repo is canonical for:
- `AGENTS.md`: shared hard rules for Codex agents
- `skills/`: reusable workflow skills
- `scripts/`: dependency-light helpers used across projects
- `hooks/`: local guardrails such as skill validation

## Skills

Skills are the main routing layer. Each `skills/<name>/SKILL.md` has YAML front matter:

```yaml
---
name: skill-name
description: "Short generic trigger phrase."
---
```

Rules:
- Keep descriptions short and generic; optimize for routing, not documentation.
- Keep skill bodies terse and operational.
- Prefer helper scripts under `skills/<name>/scripts/` when a workflow has repeatable commands.
- Validate after edits: `scripts/validate-skills`.
- Quote `description` in front matter.

Global discovery points here with individual symlinks:
- `~/.codex/AGENTS.md -> ~/agent-scripts/AGENTS.md`
- `~/.codex/skills/<name> -> ~/agent-scripts/skills/<name>`

## Helpers

`scripts/validate-skills`
- Checks every `skills/*/SKILL.md`.
- Verifies YAML front matter plus required `name` and `description`.

Enable local hooks:

```bash
git config core.hooksPath hooks
```

## Syncing

Treat this repo as canonical for shared agent rules and portable helper scripts.
Repo-specific rules stay in each repo's own `AGENTS.md`.
