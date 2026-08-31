---
name: user-story
description: Write a ClickUp user story ticket from a feature description or idea. Outputs a fully structured ticket with User Story, Feature Description, and Acceptance Criteria in the standard Wilow format.
---

# Writing a User Story

When invoked, gather context from the user about what feature or behaviour they want to describe, then produce a complete ClickUp ticket in the format below.

## How to Use

Ask the user for:
1. **Who** is the user persona (e.g. "Lara", "an admin", "a new user")
2. **What** they want to do / see / understand
3. **Why** — the underlying goal or motivation
4. **Feature scope** — what UI, data, or behaviour is involved

If the user has already provided enough context in their message, skip straight to writing the ticket.

---

## Output Format

Produce the ticket in this exact structure. Use clear, plain English. No jargon. No padding.

---

### User Story

[Persona] wants to [high-level goal]. [One or two sentences explaining their motivation and what they're trying to understand or accomplish.]

---

### Feature Description

[Name the feature or UI element being built.]

List what it does:
- What is shown by default
- How it behaves (interactions, filters, chart types, etc.)
- Any key UI controls (slicers, toggles, tabs)
- Data sources or API values involved
- Edge cases baked into the design

Use bullet points and sub-bullets. Be specific about defaults, axis labels, time ranges, and data values where relevant.

---

### Acceptance Criteria

Write one AC per distinct scenario. Use the Gherkin format:

**AC[N] — [Short descriptive title]**

Given [context / precondition],
When [trigger or action],
Then [observable outcome].

---

## Acceptance Criteria Categories to Cover

Always consider these scenarios and include an AC for each that applies:

| Category | What to cover |
|---|---|
| **Happy path** | Default load with real data |
| **Slicer / filter interaction** | What changes when a control is used |
| **Empty state** | No data at all (new account, no connection, not yet synced) |
| **API / data failure** | What the user sees when the fetch fails |
| **Loading state** | If relevant — skeleton, spinner, etc. |
| **Permissions** | If the feature is role-gated |

---

## Style Rules

- Write everything from a **product manager's perspective** — focus on user-facing outcomes and business behaviour, not technical implementation
- Never mention implementation details anywhere in the ticket: no API endpoints, error codes, database fields, service names, queue jobs, or code-level concepts — even in the Feature Description
- Describe what the system *does* from the user's point of view, not *how* it does it
- Feature Description is for the builder — be precise about observable behaviour, not the mechanism behind it
- ACs are pass/fail — each one should be independently testable by a QA engineer without reading any code
- Avoid vague words like "correctly", "properly", "as expected"
- empty states, and error states are NOT optional — always include them if the feature involves data or charts

---

## Example

### User Story

Lara wants to see overall performance on Meta at a macro level. She wants to understand how much has been spent on the platform and how that spend breaks down — by ad format (e.g., image, video, carousel) and by placement (e.g., Instagram, Facebook, Audience Network).

---

### Feature Description

On a new **Overview** tab, display a line chart showing total Meta ad spend over time.

- **X-axis:** Month
- **Y-axis:** Cost (Amount Spent)
- **Default view:** Line chart showing aggregate spend for the last 11 complete months plus the current partial month
- **Slicers:** Two slicer controls allow the user to break down spend by:
  - **Format** — values returned by the Meta API (e.g., image, video, carousel, collection)
  - **Placement** — values returned by the Meta API (e.g., Facebook, Instagram, Audience Network, Messenger)
- **Slicer behaviour:** When a slicer is active, the chart switches from a line chart to a stacked area chart. Each sub-category is rendered in a distinct colour so the user can see both the total spend and the proportional breakdown at a glance.

---

### Acceptance Criteria

**AC1 — Default chart load**

Given a user navigates to the Overview tab,
When the tab loads,
Then a line chart displays showing total monthly spend for the last 12 months (11 complete calendar months + the current partial month). The current month is visually labelled to indicate it is partial data.

**AC2 — Format slicer**

Given the user selects the Format slicer,
When the slicer is applied,
Then the chart switches to a stacked area chart with each ad format (as returned by the Meta API) shown as a distinct coloured area. The total spend remains readable from the top of the stacked area.

**AC3 — Placement slicer**

Given the user selects the Placement slicer,
When the slicer is applied,
Then the chart switches to a stacked area chart with each placement (as returned by the Meta API) shown as a distinct coloured area. The total spend remains readable from the top of the stacked area.

**AC4 — Fewer than 12 months of data**

Given the user's account has fewer than 12 months of historical data,
When the chart loads,
Then only the available months are displayed (left-aligned). No empty gaps or placeholder months are shown.

**AC5 — No data available**

Given the user has no spend data (new account, no Meta connection, or data not yet synced),
When the chart loads,
Then a clean empty state is displayed within the chart container with a message such as "No spend data available yet" and a prompt to check their Meta connection. No broken axes or zero-line charts.

**AC6 — API failure**

Given the Meta API call fails on load,
When the chart attempts to render,
Then an inline error message is shown within the chart container (e.g., "Couldn't load spend data — try refreshing"). No modal or toast interruption.
