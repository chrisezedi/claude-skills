---
name: research
description: "Investigate external libraries, APIs, or services using primary sources before writing integration code. Use when a task involves an external dependency you need to verify — not for internal codebase questions."
---

# /research

Investigate a question about an external library, API, or service using primary sources. Save findings to a markdown file so they can be referenced during implementation.

## When to Use

- Before integrating with an external API or library
- When you're unsure how a dependency behaves (retry logic, error formats, rate limits, etc.)
- When training data might be outdated for a library's current version

## Process

### 1. Identify what needs researching

State the specific question. Not "how does BullMQ work" but "how does BullMQ handle job retries when the processor throws?"

### 2. Find the documentation

Use this order:
1. Known docs URL (e.g., `https://docs.nestjs.com/`) — fetch directly with WebFetch
2. If unsure of the URL, check `package.json` for the library version, then find the repo README for a docs link
3. For open-source libraries, read the source code on GitHub via raw URLs or `gh api`
4. **If you are not confident you have the correct or current documentation URL, ask the user to confirm before fetching.** Do not guess.

### 3. Investigate using primary sources only

- Official documentation
- Source code (GitHub)
- API reference pages
- Changelogs for version-specific behaviour

**Do not use:** Blog posts, Stack Overflow, tutorials, or training data as a source. These are secondary and may be outdated.

### 4. Save findings

Write a markdown file with:
- The question that was investigated
- Findings with citations (link to the specific doc page or source file)
- Any version-specific notes
- Save to: `docs/research/YYYY-MM-DD-<topic>.md`

### 5. Report back

Summarise the key findings. Flag anything that contradicts assumptions made in the spec or plan.

## Constraints

- Direct quotes from docs limited to 125 characters — paraphrase the rest
- Every claim must link to its source
- If a fetch fails or returns unexpected content, do not guess — ask the user for the correct URL
- Do not reproduce proprietary API response schemas in full — summarise the relevant fields

## Example

```
/research how does OpenAI's gpt-image-1 model handle multiple input images in the edit endpoint?
```

Output: `docs/research/2026-08-18-openai-image-edit-multi-image.md`
