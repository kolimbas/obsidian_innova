---
tags:
  - n8n
  - lessons-learned
  - nivel-2
updated: 2026-06-02
---

# Lessons Learned

> One-line takeaways from each flow retro. Most recent first. Link to the retro for the full story.

← Back to [[n8n/README|n8n KB]]

---

## Log

| Date | Client / flow | Takeaway |
| --- | --- | --- |
| 2026-06-02 | blincer · credit-limit (HubSpot wiring) | El HubSpot Trigger de n8n exige developer app + App ID (la Developer API Key sola no alcanza); para un disparo simple conviene HubSpot Workflow → Webhook + `Normalize`. Ver [[n8n/patterns/hubspot-workflow-webhook-trigger\|pattern]]. Además: los tokens de HubSpot que dan error "OAuth token expired / expiresAt: 0" suelen estar **truncados o revocados**, no realmente expirados. |
| 2026-06-02 | blincer · 3 flows Sheets (build) | googleSheets v4.5 no tiene op `lookup`; la idempotencia por Sheet necesita `read`+filtro **y** un nodo de write-back (no viene en el esqueleto). Ver [[n8n/nodes/google-sheets\|node note]]. |

---

## How to add an entry

At the end of every `retro.md`:

1. Pick the single most surprising or reusable insight.
2. Append a row above with the date, a link to the retro, and the takeaway in one sentence.
3. If the insight is generic, also promote it to `n8n/patterns/` or `n8n/nodes/`.
