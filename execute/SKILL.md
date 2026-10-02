---
name: execute
description: Use when executing an approved implementation plan with vertical slice tasks. Use after /plan has produced an approved plan document.
---

# Execute

Execute an approved plan task-by-task using TDD within each vertical slice. Dispatch subagents for implementation, pause for user review between tasks.

## Process

### 1. Load the plan

Read the plan file passed as an argument (or the most recent plan in `docs/plans/`). Identify the next unfinished task.

### 2. Execute one task at a time

For each task, dispatch a subagent with a prompt structured around the TDD cycle. The subagent prompt MUST follow this template:

```
You are implementing Task N: <Title>.

## TDD Cycle — follow this exactly

### Step 1: Write failing tests
Write the test file(s) first. Test the BEHAVIOUR described in "What it delivers", not implementation details. Mock dependencies following the project's existing test patterns.

### Step 2: Run tests — verify RED
Run: `npx jest <test-file>`
Every new test MUST fail. If a test passes immediately, it proves nothing — delete it and write one that actually tests new behaviour.

### Step 3: Write minimal implementation
Write only the code needed to make the failing tests pass. No more.

### Step 4: Run tests — verify GREEN
Run: `npx jest <test-file>`
All tests must pass.

### Step 5: Run lint
Run: `pnpm run lint`
Fix any lint errors in files you created or modified.

## Rules
- Do NOT write implementation before tests
- Do NOT write tests and implementation in the same step
- If you catch yourself writing implementation first — stop, delete it, write the test first
- Read existing files before modifying them
- Do NOT run git commands, prisma migrations, or modify .env files

## Context
<paste the task details from the plan here — files, steps, what it delivers, blocked by>

## Existing patterns to follow
<paste relevant findings from codebase exploration — test patterns, service patterns, etc.>
```

### 3. Review subagent output

After the subagent completes:

1. Run tests yourself to verify they pass: `npx jest <test-file>`
2. Run lint: `pnpm run lint`
3. Read the key files the subagent created or modified
4. Flag any issues to the user

### 4. Pause for user review

Present a summary to the user:
- What was built
- Test results
- Any issues or deviations from the plan
- The exact `git add` and `git commit` commands

**STOP. Wait for the user to review, ask questions, and commit before moving to the next task.**

### 5. Repeat

Move to the next task only after the user confirms the previous one is committed.

## Red Flags — STOP if you catch yourself doing these

| Thought | Reality |
|---------|---------|
| "I'll have the subagent build everything at once" | That's not TDD. Structure the prompt around RED-GREEN. |
| "Tests alongside implementation is fine" | Tests-after prove "what does this do?" not "what should this do?" |
| "TDD doesn't work for greenfield code" | It does. The test fails with import errors first — that's valid RED. |
| "Schema/migration tasks can't be TDD'd" | Schema tasks are the exception. Service/controller/processor code is not. |
| "It's more efficient to skip TDD" | Efficiency without correctness is waste. The user explicitly asked for TDD. |
| "The subagent will figure it out" | No. The prompt must explicitly enforce TDD. Subagents follow instructions. |

## Manual testing

After each task, assess whether the task is manually testable before prompting the user:

1. **Check feasibility first.** Can the user hit an endpoint, query the DB, or observe a side effect right now? Consider whether the server needs to be running, whether seed data exists, whether the task is internal plumbing with no observable output.

2. **If testable:** Give the user exact steps:
   - Endpoint to hit (method, URL, headers, body)
   - DB state to set up or check (`SELECT` query, Prisma Studio instructions)
   - Expected result (response shape, status code, DB row values)
   - Ask the user to run the test and share results before moving on.

3. **If not testable:** Explain why (e.g., "This task adds internal plumbing — no endpoint or observable side effect yet. It becomes testable after Task N adds the controller.").

After all tasks are complete, run `/scenario-review` to catch cross-module interaction bugs.

## Rules

- Never skip TDD for tasks that include service, controller, or processor code
- Never batch-execute multiple tasks without review checkpoints
- Never invoke another skill — return control to the user
- Schema-only tasks (migrations, constants) are exempt from TDD — there's no behaviour to test
- Always read the plan before starting — don't work from memory
- **External data nullability:** Fields sourced from external APIs (Meta, ScrapeCreators, etc.) must be nullable (`String?`, `Int?`) in the Prisma schema — we cannot guarantee what third parties return. Never use `!` non-null assertions on external data; if TypeScript warns about a possible null, make the schema field nullable or provide an explicit default.
