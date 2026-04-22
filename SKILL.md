---
name: plan-feature-from-youtrack
description: "Generate a spec and implementation plan from a YouTrack issue, grounded in your codebase. Use when a YouTrack issue ID (e.g. INTOP-1486, ABC-123) appears in a message AND the user wants a spec, plan, or implementation breakdown. Triggers on: 'create a spec from INTOP-1486', 'plan ABC-123', 'DEV-4421 — give me a plan'. Does NOT trigger for: status checks, adding comments, PR reviews, debugging, or writing tests."
license: MIT
compatibility: "Requires the yt CLI (https://github.com/nickvdyck/yt) installed and authenticated with a YouTrack instance."
---

# Plan Feature from YouTrack

Generate a **spec** and optionally an **implementation plan** from a YouTrack card, grounded in the actual codebase.

**Announce at start:** "I'm using the plan-feature-from-youtrack skill to generate the [spec / spec and plan] for <card-id>."

---

## Step 1: Determine mode and card ID

From the user's message, identify:

- **Card ID** — e.g. `INTOP-1486`, `ABC-123`. If none is given, ask for it before proceeding.
- **Mode** — infer from the user's intent:
  - `spec-only` → user says "spec", "describe", "write a spec", or gives no indication they want tasks/plan
  - `spec-and-plan` → user says "plan", "implement", "create a plan", "give me tasks", "full plan"
  - When ambiguous, default to `spec-and-plan` and mention what you're doing.

---

## Step 2: Fetch the YouTrack card data

Run all three commands and collect the results:

```bash
yt issues show <card-id>
yt issues comments list <card-id>
yt issues related <card-id>
```

The `show` command gives you the description, type, state, assignees, and custom fields.
The `comments` provide additional context, decisions, and implementation notes added over time.
The `related` command reveals dependencies, sub-tasks, and linked issues.

If a related issue looks relevant (e.g. a parent epic, a blocking dependency), fetch its details too:
```bash
yt issues show <related-id>
```

---

## Step 3: Understand the codebase context

Before writing anything, explore the repository to understand the landscape. You're looking for:

1. **Agent/AI instructions** — read every `AGENTS.md`, `CLAUDE.md`, `.claude/CLAUDE.md` you can find (root and subdirectories). These define how to behave in this codebase.
2. **Project overview** — read `README.md` (and any other top-level `.md` files like `CONTRIBUTING.md`, `ARCHITECTURE.md`, `DEVELOPMENT.md`).
3. **Relevant code areas** — based on the card's subject matter, find the code that would be touched:
   - Search for keywords from the card title/description (domain terms, entity names, event names)
   - Understand the existing patterns (e.g. how events are currently handled, how similar features are structured)
   - Note the tech stack, module structure, and naming conventions
4. **Tests** — look at how existing tests are structured for similar features, so the plan can follow suit.

The goal is to write a spec and plan that feel native to this codebase — not generic.

---

## Step 4: Ask clarifying questions before writing the spec

Before putting anything to paper, take a moment to surface ambiguities — things that the card and codebase together don't answer clearly enough to write a solid spec. These questions help the user articulate implicit assumptions and hidden expectations they may not have thought to write down.

**How to do it:**

1. Based on what you found in Steps 2–3, identify the genuine uncertainties. Examples of things worth asking about:
   - Behaviour in edge cases not covered by the card (e.g. "what happens if the event arrives before the order is created?")
   - Data semantics that have multiple valid interpretations (e.g. "should `DamageTypesUpdated` merge with existing types or replace them entirely?")
   - Scope decisions that could go either way (e.g. "should each event ship as its own PR, or all together?")
   - Integration points or dependencies that aren't addressed (e.g. "does this need to publish a downstream event after updating?")

2. **Ask one question at a time.** Don't dump a list — present the most important question first, wait for the answer, then ask the next if still needed. This keeps the conversation focused and ensures each answer can inform the next question.

3. Aim for **2–4 questions** total before writing the spec. If the card is well-specified and the codebase makes the intent clear, fewer questions (or none) is fine. Don't ask questions you could answer yourself by reading the code.

4. Once you have enough clarity, tell the user: "Got it — I'll now write the spec." Then proceed to Step 5.

---

## Step 5: Write the spec file

Save to: **`<card-id>-spec.md`** in the current working directory (or wherever the user says).

Use this structure:

```markdown
# Spec: <Card Summary>

**YouTrack**: <card-id> · **Type**: <type> · **State**: <state>

## Background

<2–4 sentences: why this work exists, what problem it solves, and what triggered it.
Draw from the card description, comments, and any linked issues.>

## What We're Building

<Clear description of the change or feature. Be concrete — name the specific
entities, events, endpoints, or behaviors involved. If the card mentions
multiple items (e.g. five events), list them explicitly.>

## Technical Context

<What exists today that's relevant. Which modules/services are involved.
What patterns or abstractions already exist that this change should follow.
Reference actual file paths where useful (e.g. `src/orders/handler.rs`).>

## Acceptance Criteria

<Numbered list of observable, testable outcomes. Each criterion should
be verifiable — avoid vague statements like "it works correctly".>

1. ...
2. ...

## Out of Scope

<Explicitly call out adjacent things that are NOT part of this change,
to prevent scope creep. Derive these from the card and codebase context.>

## Open Questions

<Any ambiguities that need resolution before or during implementation.
Include things that aren't clear from the card or the codebase.>

## Notes

<Optional: implementation hints, risks, links to relevant code, Notion pages,
or other resources mentioned in the card.>
```

Keep the spec honest: don't invent decisions that weren't made. Flag genuine uncertainty in "Open Questions" rather than papering over it.

---

## Step 6: Ask clarifying questions before writing the plan (if mode is `spec-and-plan`)

Before writing the plan, do one more round of targeted questions — this time focused on implementation decisions rather than requirements. The spec is now written, so these questions are about *how* to build it, not *what* to build.

Typical plan-level questions:
- Delivery strategy: "Should this be one PR or split into smaller ones? The card suggests one per event."
- Testing approach: "Are there existing integration test fixtures I should follow, or should I set up new ones?"
- Ordering constraints: "Is there a dependency between any of these changes that should affect sequencing?"
- Risk: "Is there anything in the current implementation that's fragile or likely to cause problems during this change?"

Again, **one question at a time**, and only ask what genuinely matters for structuring the plan. If everything is clear from the spec + codebase, skip this step entirely and proceed to writing the plan.

Once answered (or if no questions are needed), tell the user: "Great — writing the plan now." Then proceed to Step 7.

---

## Step 7: Write the plan file (if mode is `spec-and-plan`)

Save to: **`<card-id>-plan.md`** in the same directory as the spec.

The plan is an ordered, trackable list of implementation tasks. It should be actionable — a developer (including an AI agent) should be able to pick up a task and know exactly what to do.

Use this structure:

```markdown
# Implementation Plan: <Card Summary>

**Spec**: [<card-id>-spec.md](./<card-id>-spec.md)
**YouTrack**: <card-id>

## Overview

<1–2 sentences describing the overall approach and how the work is phased.>

## Files to Create / Modify

| File | Action | Purpose |
|------|--------|---------|
| `path/to/file.ext` | Create / Modify | What this file will do |

## Tasks

### Phase 1: <Meaningful name, e.g. "Data model" or "Event handling">

- [ ] **1.1** <Task title>
  - **What**: <What to implement — be specific about logic, not just "add a handler">
  - **Files**: `path/to/file.ext`
  - **Test**: <How to verify this works — unit test, integration test, manual check>

- [ ] **1.2** <Task title>
  ...

### Phase 2: <Next phase>

- [ ] **2.1** ...

## Testing Strategy

<How to verify the whole feature end-to-end once all tasks are done.>

## Definition of Done

- [ ] All tasks above completed
- [ ] Tests pass (specify which test suite)
- [ ] <Any other project-specific done criteria from AGENTS.md/CLAUDE.md>
```

**Guiding principles for the plan:**
- Phases should build on each other — each phase should leave the codebase in a working, committable state.
- Follow TDD if the project practices it (check AGENTS.md/CLAUDE.md): write failing tests first, then implementation.
- Tasks should be small enough to complete in one focused session (~1–3 hours).
- Don't plan for hypothetical requirements — only what the spec calls for.
- If the card has sub-tasks or split suggestions (e.g. "one PR per event"), reflect that in the phasing.

---

## Output summary (Step 8)

When done, tell the user:
- What files were generated and where
- Any open questions from the spec that need answers before implementation can begin
- If `spec-and-plan` mode: a one-line summary of the phases in the plan

If you couldn't fetch some data (e.g. `yt` not authenticated, issue not found), say so clearly and generate what you can from available information.
