# plan-feature-from-youtrack

A Claude Code agent skill that generates a **spec** and an **implementation plan** from a YouTrack issue, grounded in your actual codebase.

## What it does

When you reference a YouTrack issue ID (e.g. `INTOP-1486`, `ABC-123`) and ask for a spec or plan, this skill:

1. **Fetches the YouTrack card** — description, comments, and related issues via the `yt` CLI
2. **Explores your codebase** — reads agent instructions, project docs, and relevant code to understand context
3. **Asks clarifying questions** — surfaces ambiguities one at a time before committing to writing
4. **Generates a spec** — `<card-id>-spec.md` with background, acceptance criteria, and open questions
5. **Generates a plan** — `<card-id>-plan.md` with phased, actionable tasks sized for focused sessions

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- [yt CLI](https://github.com/nickvdyck/yt) installed and authenticated with your YouTrack instance

## Install

Copy (or symlink) the `plan-feature-from-youtrack` folder into your skills directory:

```bash
# Option 1: Copy
cp -r plan-feature-from-youtrack ~/.claude/skills/

# Option 2: Symlink (keeps it linked to the repo)
ln -s "$(pwd)/plan-feature-from-youtrack" ~/.claude/skills/plan-feature-from-youtrack
```

## Usage

In Claude Code, just mention a YouTrack issue ID and ask for a spec or plan:

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
