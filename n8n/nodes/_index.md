---
tags:
  - n8n
  - nodes
  - nivel-2
updated: 2026-06-02
---

# Node Notes

> Quirks, gotchas, and proven configurations for specific n8n nodes. Populated from flow retros — only what's non-obvious from official docs.

← Back to [[n8n/README|n8n KB]]

---

## Index

| Node | Why we have notes | Last update |
| --- | --- | --- |
| [[n8n/nodes/google-sheets\|Google Sheets]] | v4.5 sin op `lookup`; `read` filtrado corta rama si no matchea; autoMap append | 2026-06-02 |

---

## When to add an entry

- The node behaved differently than the docs suggested.
- A specific param combination took >15 min to figure out.
- Auth setup has non-obvious steps.
- A rate limit / quota was hit and worked around.

## Entry shape

```markdown
---
tags: [n8n, node]
node: <official node name>
updated: YYYY-MM-DD
---

# <Node Name>

## Why this note exists
## Auth / setup
## Proven configurations
## Gotchas
## Flows using this node
```
