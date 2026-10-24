---
name: storm-research
description: "Use when deeply researching any topic from multiple expert perspectives before making decisions, designing features, or forming opinions. Trigger on /storm-research or when the user wants to understand a topic thoroughly — not for investigating a specific library (use /research) or reviewing a spec (use /council)."
---

# Storm Research

Multi-perspective research method based on Stanford's STORM (Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking). Produces a grounded briefing on any topic in 4 steps.

## When to Use

- Exploring a topic before designing a feature
- Understanding tradeoffs between competing approaches
- Researching a domain you're unfamiliar with
- Any time you need depth beyond a single-perspective answer

## When NOT to Use

- Investigating a specific library/API dependency — use `/research`
- Reviewing an approved spec — use `/council`
- Quick factual lookups — just answer directly

## Process

Run all 4 steps sequentially in one conversation. Each step builds on the previous.

### Step 1: Five-Perspective Scan

Ask the user for the topic if not provided, then generate 5 expert perspectives.

**The 5 experts:**

| Expert | Lens | Key question |
|--------|------|--------------|
| **Practitioner** | Day-to-day realities | What do they know that academics miss? |
| **Academic** | Peer-reviewed evidence | Where does evidence contradict popular belief? |
| **Skeptic** | Strongest counterargument | What evidence do proponents conveniently ignore? |
| **Economist** | Financial incentives | Who profits from the current narrative? |
| **Historian** | Historical patterns | What parallels exist and how did they play out? |

**Each expert provides:**
- Core position (2 sentences max)
- Strongest supporting evidence
- One insight only this perspective would surface

**Rule:** Experts must disagree where the evidence supports it. Do not smooth over contradictions.

### Step 2: Contradiction Map

Analyse the 5 perspectives and produce:

| # | Conflict | Claim A | Claim B | Importance |
|---|----------|---------|---------|------------|
| 1 | ...      | ...     | ...     | High/Med/Low |

Then answer:
1. Which perspective has the strongest evidence? Weakest? Why?
2. What one question would resolve the biggest contradiction?
3. **Universal agreement** — what does EVERY perspective agree on? (Likely true.)
4. **Blind spot** — what did NONE of the perspectives address? (Often the most valuable finding.)

### Step 3: Synthesis Briefing

Structure:

1. **One-paragraph summary** — brief a decision-maker who has 60 seconds and needs nuance, not just the headline
2. **5 key findings** — ranked by reliability; for each, note which perspectives support it and which challenge it
3. **Hidden connection** — one non-obvious link between findings that only emerges from looking at all 5 perspectives together
4. **Actionable insight** — based on all evidence, what should the user actually DO differently? Be specific.
5. **Frontier question** — the one question that, if answered, would change everything about how we understand this topic

### Step 4: Peer Review

Self-critique the briefing:

1. **Confidence scores** — rate each of the 5 key findings (1-10) with reasoning
2. **Weakest link** — which claim are you least confident in? What would verify it?
3. **Bias check** — which perspective dominated the synthesis? Was any voice underweighted?
4. **Missing perspective** — is there a 6th angle that would change the conclusions?
5. **Overall grade** — honest assessment of the briefing's quality and what would improve it

If the peer review surfaces material issues (confidence < 5 on key findings, dominant bias, missing perspective that changes conclusions), **revise the synthesis** and document what changed.

### Step 5: Save Reports

Save two versions of the report:

#### 5a. Markdown report

Save to `docs/research/storm/YYYY-MM-DD-<topic-slug>.md`:

```markdown
# Storm Research: <Topic>

**Date:** YYYY-MM-DD
**Requested by:** <user context if provided>

---

## Expert Perspectives

### The Practitioner
...

### The Academic
...

### The Skeptic
...

### The Economist
...

### The Historian
...

---

## Contradiction Map

| # | Conflict | Claim A | Claim B | Importance |
|---|----------|---------|---------|------------|

**Strongest evidence:** ...
**Weakest evidence:** ...
**Resolving question:** ...
**Universal agreement:** ...
**Blind spot:** ...

---

## Synthesis Briefing

### Summary
...

### Key Findings
1. ...

### Hidden Connection
...

### Actionable Insight
...

### Frontier Question
...

---

## Peer Review

### Confidence Scores
| # | Finding | Score | Reasoning |
|---|---------|-------|-----------|

### Weakest Link
...

### Bias Check
...

### Missing Perspective
...

### Overall Grade
...

### Revisions Made
(if any)
```

#### 5b. HTML report

Save to `docs/research/storm/YYYY-MM-DD-<topic-slug>.html` — a standalone, visually polished HTML file with:

- **No external dependencies** — all CSS inline, no CDN links
- **Dark/light theme toggle** — default to dark, persist choice in localStorage
- **Clean typography** — system font stack, comfortable reading width (max 720px), generous whitespace
- **Visual hierarchy** — each section clearly delineated with subtle dividers or card-style containers
- **Colour-coded expert cards** — each of the 5 experts gets a distinct accent colour for their section
- **Contradiction map as a styled table** — with importance badges (High = red, Medium = amber, Low = green)
- **Confidence scores as visual bars** — horizontal bars (1-10 scale) with colour gradient (red → yellow → green)
- **Collapsible sections** — each major section (Perspectives, Contradictions, Synthesis, Peer Review) can be collapsed/expanded; all expanded by default
- **Print-friendly** — `@media print` styles that hide the theme toggle and expand all sections
- **Responsive** — readable on mobile

Use semantic HTML (`<article>`, `<section>`, `<details>`, `<summary>`) and CSS custom properties for theming. Keep the JavaScript minimal — only theme toggle and section collapse.

Present a brief summary to the user after saving both files. Do not dump the entire report into chat — they can read the files.

## Rules

- Run all 4 analysis steps before saving — do not skip steps
- Use WebSearch to ground claims in real sources when possible — do not rely solely on training data
- If the topic is too broad, ask the user to narrow it before starting
- Never invoke another skill — return control to the user after the report
- Flag when a perspective is speculative vs evidence-backed
