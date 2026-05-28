---
tags:
  - n8n
  - patterns
  - nivel-2
updated: 2026-05-28
---

# Patterns

> Reusable building blocks promoted from concrete flow retros. A pattern lives here only after it's been used in **at least one** real flow.

← Back to [[n8n/README|n8n KB]]

---

## Catalog

| Pattern | Problem it solves | First seen in |
| --- | --- | --- |
| _none yet_ |  |  |

---

## How to add a pattern

1. During a retro, identify a generic bit (idempotent upsert, webhook auth check, error-to-Slack branch, etc.).
2. Create `n8n/patterns/<pattern-slug>.md` with: problem, when to use, when NOT to use, n8n implementation (nodes + key params), example flow link.
3. Add a row above.
4. Next flow's `research.md` should cite it.

## Pattern file shape

```markdown
---
tags: [n8n, pattern]
problem: <one-liner>
updated: YYYY-MM-DD
---

# Pattern — <name>

## Problem
## When to use
## When NOT to use
## Implementation
## Example flows using this
## Variants / known issues
```
