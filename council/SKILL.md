---
name: council
description: Three-agent sequential debate (optimist, realist, neutral) that reviews an approved spec before planning. Use after /spec produces an approved spec, before /plan.
---

# Agent Council

Run a structured debate over the approved spec using three subagents with distinct perspectives. Each agent builds on the previous agent's output, creating a real back-and-forth — not three isolated opinions.

## When to run

After the spec is approved (Step 3) and before design documentation (Step 4). This step is mandatory for every feature.

## Process

### 1. Locate the spec

Find the most recent spec in `docs/superpowers/specs/`. If multiple exist, ask the user which one to review.

### 2. Dispatch agents sequentially

Use the registered subagents in `~/.claude/agents/`. Run them **in sequence** — each receives the spec AND the output of all previous agents.

#### Agent 1: `council-optimist`

Dispatch with the spec content. The optimist makes the strongest case FOR the design.

#### Agent 2: `council-realist`

Dispatch with the spec content AND the optimist's full assessment. The realist finds holes and directly challenges the optimist's weak points.

#### Agent 3: `council-neutral`

Dispatch with the spec content, the optimist's assessment, AND the realist's assessment. The neutral synthesizes both sides and delivers a verdict.

### 3. Generate the council report

Save the combined output to `docs/superpowers/council/YYYY-MM-DD-<feature>-council-review.md` with this structure:

```markdown
# Council Review: <Feature Name>

**Spec reviewed:** `docs/superpowers/specs/<spec-file>`
**Date:** YYYY-MM-DD
**Verdict:** Proceed | Proceed with changes | Revisit spec

---

## Optimist's Case

<Optimist's full assessment>

---

## Realist's Challenges

<Realist's full assessment>

---

## Neutral's Synthesis

<Neutral's full assessment>

---

## Key Disagreements

| # | Topic | Optimist | Realist | Resolution |
|---|-------|----------|---------|------------|
| 1 | ...   | ...      | ...     | ...        |

---

## Final Recommendation

<Neutral's verdict with reasoning and any required changes before proceeding>
```

### 4. Walk the user through the report

After generating the report, present it section by section:
1. Summarize the Optimist's key points
2. Summarize the Realist's key challenges
3. Highlight the key disagreements and how they were resolved
4. Present the verdict

Wait for the user's response before proceeding.

### 5. Next step

Based on the verdict:
- **Proceed** → "Run `/archify` to generate design diagrams, then `/plan`."
- **Proceed with changes** → List the specific changes needed. User updates the spec, then re-run council or proceed.
- **Revisit spec** → "Fundamental concerns were raised. Run `/spec` to revise the design."

## Rules

- Never skip agents — all three must run, in order
- Never invoke another skill — return control to the user
- Agents must read the codebase, not assume — every claim needs a citation
- The council reviews the DESIGN, not the implementation plan — it runs before planning
- If the spec file cannot be found, ask the user to point to it — never guess
