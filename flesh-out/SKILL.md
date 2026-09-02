---
name: flesh-out
description: Explore a feature idea or bug through structured questioning until reaching an approved design. Use when starting any new feature, enhancement, or non-trivial bug fix.
---

# Flesh Out

Turn an idea into an approved design through collaborative dialogue. You explore, question, and propose — the user decides.

## Process

### 1. Explore project context

Before asking any questions, understand what exists:

- Check graphify (`graphify-out/graph.json`) for cross-module relationships and impact analysis
- Read relevant files, docs, recent commits
- Understand the current state of the area you're about to change

### 2. Assess scope

Before asking detailed questions, assess scale. If the request describes multiple independent subsystems (e.g., "build a platform with chat, file storage, billing, and analytics"), flag this immediately. Don't spend questions refining details of a project that needs to be decomposed first.

If the project is too large for a single spec, help the user decompose into sub-projects. Each sub-project gets its own flesh-out → grill-me → spec → plan cycle.

### 3. Ask clarifying questions

- One question per message — never batch questions
- Prefer multiple choice when possible
- Focus on understanding: purpose, constraints, success criteria
- **Verify from code before asking.** If a question can be answered by reading the codebase, read the code first, present what you find, and ask the user to confirm — never ask "how does X work?" when the answer is in the code.
- **Only ask about behaviour, never about implementation.** PBIs describe what the user sees and experiences. Technical decisions (caching, endpoint structure, API call patterns) are ours to make during spec and planning — not questions for the user during flesh-out. If a PBI contains technical suggestions from AI-generated content, ignore them and focus on the behavioural intent.

### 4. Propose approaches

Once you understand the problem:

- Propose 2-3 different approaches with trade-offs
- **Always recommend the proper fix, not the easy one.** Lead with the right engineering decision. If a simpler option is better for now, explain why — but always name the proper fix and what trade-off you're making by not doing it.
- Have a strong opinion — push back on suboptimal approaches

### 5. Present the design

Once the user picks an approach:

- Present the design section by section
- Scale each section to its complexity: a few sentences if straightforward, more if nuanced
- After each section, ask: "Does this look right?"
- Be ready to go back and revise if something doesn't make sense
- Always consider production impact: "What happens to existing records?"

### 6. Stop

When the user approves the full design, say:

> "Design approved. Run `/grill-me` to stress-test it before writing the spec."

**Do NOT write a spec.** Do NOT invoke any other skill. Your job is done.

## Rules

- Never invoke another skill — return control to the user
- Never write files — the design lives in conversation until `/spec` writes it
- Never skip the approach comparison — always propose 2-3 options even if one is obvious
- Always explore existing code before proposing changes
- Always ask about production impact before presenting the design as complete
