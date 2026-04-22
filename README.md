# Plan-feature-from-YouTrack Agent Skill

An agent skill that generates a **spec** and an **implementation plan** from a YouTrack issue, grounded in your actual codebase.

Works with any agentic harness that supports the [Agent Skills specification](https://agentskills.io/specification) — Claude Code, Codex, Cursor, Copilot, Gemini CLI, and others.

## What it does

When you reference a YouTrack issue ID (e.g. `INTOP-1486`, `ABC-123`) and ask for a spec or plan, this skill:

1. **Fetches the YouTrack card** — description, comments, and related issues via the `yt` CLI
2. **Explores your codebase** — reads agent instructions, project docs, and relevant code to understand context
3. **Asks clarifying questions** — surfaces ambiguities one at a time before committing to writing
4. **Generates a spec** — `<card-id>-spec.md` with background, acceptance criteria, and open questions
5. **Generates a plan** — `<card-id>-plan.md` with phased, actionable tasks sized for focused sessions

## Prerequisites

- The [yt CLI](https://github.com/nickvdyck/yt) installed and authenticated with your YouTrack instance

## Install

```bash
npx skills add xpepper/plan-feature-from-youtrack-agent-skill
```

## Usage

In your agent, mention a YouTrack issue ID and ask for a spec or plan:

```
> create a spec from INTOP-1486
> plan the implementation for ABC-123
> DEV-4421 — can you plan this for me?
```

The skill supports two modes:

| Mode | Trigger phrases | Output |
|------|----------------|--------|
| `spec-only` | "spec", "describe", "write a spec" | `<card-id>-spec.md` |
| `spec-and-plan` | "plan", "implement", "create a plan" | `<card-id>-spec.md` + `<card-id>-plan.md` |

## License

MIT
