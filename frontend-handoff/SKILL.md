---
name: frontend-handoff
description: Use when backend API changes need to be communicated to frontend developers. Trigger on /frontend-handoff or when user asks to summarize API changes for the frontend team.
---

# Frontend Handoff

Generate a concise summary of backend API changes for frontend developers. State what changed — not what they should do.

## Process

1. **Find the changes** — read the spec, diff, or recent commits on the current branch to identify API changes.

2. **For each affected endpoint**, state:
   - Endpoint method and path
   - What changed in plain language (fields added, removed, renamed, type changed)
   - Flag as **Breaking** or **Non-breaking**

3. **List any renamed keys or values** inline (e.g. `PACK_5` → `PACK_120`)

4. **Point to Swagger docs** for the full updated response shapes — do not include JSON examples.

## Rules

- Do NOT tell frontend devs what to do — they'll figure it out
- Do NOT include JSON response examples — refer to Swagger instead
- Do NOT include implementation details (service names, internal logic)
- Do NOT mention deploy coordination
