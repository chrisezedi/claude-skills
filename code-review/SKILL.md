---
name: code-review
description: "Two-axis code review: standards compliance (coding conventions + smell baseline) and spec compliance (does the code match the spec). Use during the Execute step after each subagent task, or for final PR review."
---

# /code-review

Review code changes on two axes: **Standards** (is the code well-written?) and **Spec** (does the code match the spec?). Report findings separately — one axis passing does not excuse the other failing.

## Process

### 1. Pin the diff

Review only what changed, not the entire file. Use:
```bash
git diff <fixed-point>...HEAD
git log <fixed-point>..HEAD --oneline
```

The fixed point is the commit before the current task started. This keeps the review focused on new changes and avoids flagging pre-existing issues.

### 2. Identify sources

**For standards:** Read `WORKFLOW.md` and `CLAUDE.md` for project conventions. Apply the smell baseline below as universal heuristics.

**For spec:** Find the spec in `docs/superpowers/specs/` or the plan in `docs/superpowers/plans/`. If reviewing a specific task, use the task description from the plan.

### 3. Review — Standards axis

Check the diff against project conventions and the smell baseline.

**Smell baseline** (from Fowler's refactoring catalogue):

| Smell | What it means | Fix |
|-------|--------------|-----|
| Mysterious Name | Unclear naming | Rename; if you can't name it, the design is unclear |
| Duplicated Code | Same logic repeated | Extract and share |
| Feature Envy | Method uses another object's data more than its own | Move the method to where the data lives |
| Data Clumps | Parameters that always travel together | Bundle into a type |
| Primitive Obsession | Using a string/number where a domain type belongs | Create a type |
| Repeated Switches | Same if/switch cascade in multiple places | Use a map or polymorphism |
| Shotgun Surgery | One change requires edits scattered across many files | Consolidate |
| Divergent Change | One file edited for unrelated reasons | Split by responsibility |
| Speculative Generality | Abstraction built for a future that hasn't arrived | Delete it; inline until needed |
| Message Chains | Long navigation chains (a.b.c.d.e) | Hide behind a single method |
| Middle Man | Class that mostly delegates to another | Remove it; call the target directly |
| Refused Bequest | Subclass ignoring inherited behaviour | Use composition instead |

**Rules:**
- Project conventions override the smell baseline
- Skip anything already enforced by tooling (ESLint, Prettier)
- Distinguish hard violations from judgment calls
- Limit: 400 words

### 4. Review — Spec axis

Check the diff against the spec or task description.

- **Missing:** Spec requirement not implemented
- **Partial:** Requirement implemented but incomplete
- **Extra:** Code added that the spec didn't ask for (scope creep)
- **Wrong:** Implementation contradicts the spec

Quote the specific spec line for each finding. Limit: 400 words.

Skip this axis if no spec is available.

### 5. Report

Present findings under separate headings:

```
## Standards
[findings]

## Spec
[findings]

## Summary
Standards: N findings (worst: ...)
Spec: N findings (worst: ...)
```

Do not merge or rerank findings across axes.
