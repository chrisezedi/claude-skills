---
name: grill-me
description: Stress-test a design by working through a decision tree round by round until every branch is resolved. Use when the user wants to stress-test their thinking, or after /flesh-out produces an approved design.
---

# Grill Me

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask **one question at a time** — never batch questions. Present the question with your recommendation, wait for the user's answer, then ask the next frontier question. Each answer reshapes the tree and may unlock new questions.

Format each question like so:

```
Q<N> - <question title>: <question body, might be multiple paragraphs, including multiple choices>

Recommendation: <your recommended answer>
```

Wait for the user's answer before asking the next question. A question whose answer depends on an unsettled decision must wait until that decision is resolved.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

## Forced questions

These must appear on the frontier at some point during the session — if the design doesn't naturally surface them, force them in:

- What happens when optional inputs are missing or empty?
- Are there two code paths that need to stay in sync? If so, what ties them together?
- What does the consumer (API caller, AI model, downstream service) actually see? Walk through a concrete example.
- What happens to existing data in production?
- If there's a gap between user action and processing (queues, async jobs), what can change in that gap?
- What's the blast radius if this fails? Who gets paged?
- Is there a migration path, or is this a clean deploy?
- What other modules read or depend on the state this feature changes? Walk through each one — what do they show before vs after?

## When to stop

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

When done, say:

> "All branches resolved. Run `/spec` to write it up."

## Rules

- Never invoke another skill — return control to the user
- Never write files — the grilling is purely conversational
- Never accept "we'll figure it out later" — that's an unresolved branch. Push for an answer or explicitly mark it as out of scope with the user's agreement
- Have strong opinions — your recommendations should be genuinely what you'd do, not safe middle-ground answers
