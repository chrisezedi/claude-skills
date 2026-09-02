---
name: scenario-review
description: "Post-implementation review that walks through real user scenarios to find gaps between what was built and what users will actually experience. Use after all tasks are committed, before finishing the branch."
---

# Scenario Review

An independent review of finished code from the user's perspective. You are a **devil's advocate** — your job is to find scenarios where the code doesn't do what a real user would expect.

This is NOT a code quality review. This is NOT a spec compliance review. Those already happened. This review asks: **"What will users actually experience, and did we miss anything?"**

## When to use

After all implementation tasks are committed and reviewed, before finishing the branch. This is the last gate before merge/PR.

## Process

### 1. Gather context

Read these in parallel:
- The **spec** (from `docs/specs/` or `docs/superpowers/specs/`)
- The **full diff** against the base branch: `git diff main...HEAD`
- Any **user story or PBI** if referenced in the spec

Do NOT read the plan — the plan is implementation detail. You care about what the user sees, not how it was built.

### 2. Map state transitions

Before generating scenarios, map every state a user can be in that this feature touches. For each state, identify:
- How does a user enter this state?
- What can happen while they're in this state?
- How do they leave this state?
- What other systems are affected when they transition?

Example for a billing feature:
```
States: FREE, PRO, CANCELLED (still active), REVOKED (downgraded)
Transitions: FREE->PRO (subscribe), PRO->CANCELLED (cancel), CANCELLED->REVOKED (period ends), PRO->REVOKED (payment fails), REVOKED->PRO (re-subscribe)
Side effects per transition: credits, account limits, sync jobs, API access
```

### 3. Generate scenarios

For each state transition, write a concrete scenario from the user's POV:

```
Scenario: PRO user's payment fails
- User has been PRO for 3 months
- They have 800/1550 credits remaining this period
- Their card expires and payment fails
- What does their dashboard show?
- Can they still use credits?
- What happens when they re-subscribe?
```

**Forced scenario categories** — always generate at least one scenario for each:
- **Happy path lifecycle**: User goes through the full intended journey
- **Reversal**: User undoes the action (cancel, downgrade, disconnect, delete)
- **Timing edge case**: Action happens at period boundary, midnight, during a sync
- **Stale data**: User has old data from before the feature existed
- **Repeat action**: User triggers the same event twice (webhook retry, double-click, re-subscribe)
- **Cross-module interaction**: The feature changes state in module A — what does module B show?

### 4. Trace each scenario through the code

For each scenario, trace the actual code path:
1. What endpoint/webhook/job triggers?
2. What service methods execute?
3. What gets written to the database?
4. What does the user see after (API response, balance, dashboard state)?

If you can't trace a scenario to a concrete outcome, that's a gap.

### 5. Report

```
## Scenario Review

### Scenarios Tested
1. [Scenario name] — PASS/FAIL
   - Trace: [brief code path]
   - Result: [what user sees]
   - Gap (if FAIL): [what's missing or wrong]

### Gaps Found
- [Gap description]: [Which scenario exposed it] → [Suggested fix]

### Coverage Summary
- Scenarios tested: N
- Passed: N
- Gaps found: N
- Severity: [None / Low / High — "High" means a user will hit this in production]
```

## Rules

- Never read the implementation plan — judge the code by what it does, not what it was supposed to do
- Never skip the state transition mapping — this is where most gaps hide
- Every scenario must be traceable to actual code paths — no hypothetical "this should work"
- Generate at least 6 scenarios (one per forced category minimum)
- If you find a High severity gap, say so clearly — do not soften it
- Cross-module interactions are mandatory — if the feature touches module A, check what modules B and C show
