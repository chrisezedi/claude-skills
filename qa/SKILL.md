---
name: qa
description: "Generate and run through a manual QA checklist from the user's perspective. Use after implementation is complete and scenario review has passed, before finishing the branch."
---

# QA

Generate a manual testing checklist by thinking like the user, then walk through each item with the developer.

## Process

### 1. Read the spec and code

Read these in parallel:
- The **spec** (from `docs/specs/` or `docs/superpowers/specs/`)
- The **changed service files** — understand what endpoints, webhooks, or jobs are involved
- The **controller layer** — identify the exact endpoints to hit

Do NOT read the plan. Think from the user's perspective, not the implementation's.

### 2. Map user journeys

Before writing test cases, map every journey a real user could take through this feature:

- **Primary journey**: The happy path the feature was built for
- **Reversal journey**: User undoes or cancels the action
- **Edge journeys**: Boundary conditions, empty states, repeat actions
- **Existing user journey**: What does a user who existed before this feature experience?

For each journey, note:
- What state the user starts in (plan, balance, accounts, data)
- What action triggers the feature (API call, webhook, cron, background job)
- What the user should see after (response, dashboard state, balance, data)

### 3. Generate the checklist

Each item follows this format:

```
- [ ] **[Journey]: [What to verify]**
  Setup: [State to create — DB records, webhook payloads, etc.]
  Action: [Exact command — curl, Swagger, webhook trigger, etc.]
  Expected: [What the user should see — response body, DB state, balance]
```

**Forced categories** — always include at least one item for each:
- Happy path (feature works as designed)
- Idempotency (same action twice produces same result, not duplicate data)
- Reversal (undo the action — does the system return to a clean state?)
- Stale data (user existed before this feature — do they get correct behavior?)
- Error path (what happens when the action fails — bad input, missing data, service down?)
- Cross-module visibility (feature changes state in module A — what does the API in module B return?)

### 4. Provide exact commands

For each checklist item, provide the exact way to trigger it:

- **API endpoints**: Full curl command or Swagger path with example payload
- **Webhooks**: Example webhook payload to send (via Polar dashboard, curl to webhook endpoint, or test script)
- **Background jobs**: How to trigger the job (endpoint that enqueues, BullMQ dashboard, direct queue push)
- **Database state**: Prisma Studio instructions or SQL to verify the result (user runs these, not the agent)

If you don't know the exact endpoint or payload shape, read the controller and DTO files to find out. Never guess.

### 5. Walk through together

Present the full checklist. Then go item by item:
- User runs the action
- User reports what they see
- Mark pass or fail
- If fail: diagnose immediately — read the relevant code, check what went wrong, propose a fix

Do not move to the next item until the current one is resolved (pass or explicitly deferred).

### 6. Report

After all items:

```
## QA Results

Passed: N/N
Failed: N (list)
Deferred: N (list with reasons)

Ready to merge: Yes/No
```

If any item failed, the feature goes back to Execute for a fix, then re-test the failed items.

## Rules

- Never skip the forced categories — every QA session tests at least 6 scenarios
- Never guess endpoints or payloads — read the actual controller and DTO files
- Never mark an item as passed without the user confirming the result
- Never batch items — go one at a time, wait for the user's result
- Always include "existing user" scenarios — what happens to users who existed before this feature?
- Provide exact curl/Swagger commands — "hit the endpoint" is not a test step
- If a test requires specific DB state, tell the user exactly what to create in Prisma Studio
