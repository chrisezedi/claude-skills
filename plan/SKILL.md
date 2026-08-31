---
name: plan
description: Break an approved spec into vertical slice tasks with blocking dependencies. Use after /spec has produced an approved spec document.
---

# Plan

Break an approved spec into implementation tasks. Each task is a **vertical slice** — a narrow but complete path through every layer.

## Process

### 1. Read the spec

Read the spec file passed as an argument (or the most recent spec in conversation context). Understand every requirement.

### 2. Explore the codebase

Understand the current state of the files you'll be touching:

- Check graphify (`graphify-out/graph.json`) for cross-module relationships
- Read the actual source files to understand existing patterns, imports, and test conventions
- Identify which files need creating vs modifying

### 3. Map the file structure

Before defining tasks, list which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

### 4. Draft vertical slices

Break the work into tasks following these rules:

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, service, controller, tests): vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first
- "Make the change easy, then make the easy change"

</vertical-slice-rules>

Give each task its **blocking dependencies**: the other tasks that must complete before it can start. A task with no blockers can start immediately.

### 5. Handle wide refactors

A **wide refactor** is one mechanical change (rename a column, retype a shared symbol, convert an implicit join to explicit) whose blast radius fans across the whole codebase — a single edit breaks many call sites and no vertical slice can land green.

Don't force it into a vertical slice. Sequence it as **expand-contract**:

1. **Expand**: add the new form beside the old so nothing breaks
2. **Migrate**: change call sites in batches sized by blast radius, each batch its own task, keeping tests green batch to batch because the old form still exists
3. **Contract**: remove the old form once no caller remains, in a task blocked by every migrate batch

When even the batches can't stay green alone, keep the sequence but let them share an integration branch; green is promised only at the final integrate-and-verify task.

### 6. Write each task

Each task gets this structure:

```markdown
### Task N: <Title>

**What it delivers:** The end-to-end behaviour this task makes work.
**Blocked by:** Task numbers that must complete first, or "None".
**Testable?:** Yes/No — if yes, describe the exact manual test (endpoint to hit, DB state to set up, expected result). If no, explain why (e.g. "internal plumbing with no user-facing change yet").

**Files:**
- Create: `exact/path/to/file.ts`
- Modify: `exact/path/to/existing.ts`
- Test: `exact/path/to/test.ts`

**Steps:**

- [ ] Write the failing test
  (include actual test code)

- [ ] Run test to verify it fails
  Run: `<exact command>`
  Expected: FAIL with "<specific error>"

- [ ] Write minimal implementation
  (include actual implementation code)

- [ ] Run test to verify it passes
  Run: `<exact command>`
  Expected: PASS

- [ ] Run lint
  Run: `pnpm run lint`
  Expected: No errors

- [ ] Commit
  `git add <files>`
  `git commit -m "<message>"`
```

### 7. Self-review

After writing the complete plan, check it:

- **Spec coverage:** Skim each section of the spec. Can you point to a task that implements it? List any gaps.
- **Placeholder scan:** Any "TBD", "TODO", "similar to Task N", or vague steps? Fix them. Every step must have actual code.
- **Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks?
- **Vertical slice check:** Is each task independently testable? Does any task only touch one layer? If so, merge it with another task to form a complete slice.
- **Dependency check:** Are blocking edges correct? Does each task only depend on tasks that genuinely gate it?

Fix any issues inline.

### 8. Save and stop

Save to `docs/plans/YYYY-MM-DD-<feature>.md`.

Say:

> "Plan written to `<path>`. Review it, then execute when ready."

## Plan document header

Every plan must start with:

```markdown
# <Feature Name> Implementation Plan

**Goal:** One sentence describing what this builds.
**Architecture:** 2-3 sentences about the approach.
**Spec:** Link to the spec document.

---
```

## Rules

- Never invoke another skill — return control to the user
- Every task MUST be a vertical slice — never group all guard changes or all service changes into one task
- Every step that changes code MUST include the actual code — no "implement similar to above"
- Every task MUST include a lint step — tests passing does NOT mean code is clean
- Never use placeholders — if you don't know the exact code, explore the codebase until you do
- Avoid file paths in the spec doc, but use exact file paths in the plan — the plan is implementation-specific
- TDD within each task: failing test first, then minimal code to pass
