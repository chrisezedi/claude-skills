---
name: spec
description: Write an approved and grilled design into a spec document, ADRs, and glossary entries. Use after /flesh-out and /grill-me have produced a fully resolved design.
---

# Spec

Write the approved design into formal documents. You are a scribe, not an explorer — the design decisions have already been made in `/flesh-out` and stress-tested in `/grill-me`. Your job is to capture them precisely.

## Process

### 1. Review the conversation

Read back through the flesh-out and grill-me conversation to collect:

- The approved design (from flesh-out)
- Every decision that was made and why (from grill-me)
- Any new domain terms that were introduced
- Any constraints or edge cases that were resolved

### 2. Write the spec

Save to `docs/specs/YYYY-MM-DD-<feature>-design.md`.

Use this structure, scaling each section to its complexity:

```markdown
# <Feature Name>

## Problem
What's wrong or missing today.

## Solution
What we're building and how it works, in plain language.

## Schema Changes
New or modified database models, with Prisma syntax.

## Implementation
Section by section — data flow, enforcement points, error handling.
Scale each section: a few sentences if simple, more if nuanced.

## Diagrams

### Component Diagram
Mermaid graph showing modules, services, queues, tables and how they connect.

### Sequence Diagrams
Mermaid sequence diagrams for key API operations.
Show the API boundary — requests, service calls, queue jobs — not UI clicks.
One diagram per distinct flow (e.g., downgrade flow, upgrade flow, access check).

### ER Diagram
Mermaid ER diagram for new or changed database tables and their relationships.
```

### 3. Write ADRs

For each significant decision made during flesh-out and grill-me, write an ADR.

Save to `docs/adrs/NNNN-<decision-slug>.md`. Number sequentially — check existing ADRs to find the next number. If no ADRs exist yet, start at `0001`.

Use this format:

```markdown
# NNNN: <Decision Title>

**Date:** YYYY-MM-DD
**Status:** Accepted
**Context:** What prompted this decision? What were the constraints?
**Decision:** What did we decide?
**Alternatives considered:** What else did we consider and why did we reject it?
**Consequences:** What follows from this decision — both good and bad?
```

Only create ADRs for decisions that a future developer would ask "why did they do it this way?" about. Don't ADR obvious choices.

### 4. Update the glossary

If new domain terms were introduced during flesh-out or grill-me, add them to `docs/glossary.md`.

If the file doesn't exist, create it with this header:

```markdown
# Glossary

Domain terms used in this project. Updated as new concepts are introduced.

| Term | Definition |
|------|-----------|
```

Add one row per new term. Keep definitions short — one sentence.

### 5. Self-review

Before asking the user to review, check your own work:

- **Placeholder scan:** Any "TBD", "TODO", or vague sections? Fix them.
- **Internal consistency:** Do sections contradict each other? Does the architecture match the feature descriptions?
- **Ambiguity check:** Could any requirement be interpreted two ways? Pick one and make it explicit.
- **Conversation coverage:** Did you capture every decision from grill-me? Check the frontier questions that were asked.
- **Diagram accuracy:** Do the diagrams match the written spec? Are all components represented?

Fix any issues inline.

### 6. Ask for review

Present the written artifacts to the user:

> "Spec written to `<path>`. ADRs written to `<paths>`. Glossary updated at `<path>`. Please review and let me know if anything needs changing. When approved, run `/plan` to break it into tasks."

## Rules

- Never invoke another skill — return control to the user
- Never make design decisions — if something is ambiguous in the conversation, ask the user to clarify rather than deciding yourself
- Never skip the diagrams — every spec gets all three Mermaid diagrams
- Never skip the self-review — catch your own mistakes before the user has to
- Avoid specific file paths or code snippets in the spec — they go stale fast. Exception: schema definitions and interface shapes that encode decisions more precisely than prose
