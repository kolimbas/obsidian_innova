---
tags:
  - n8n
  - cliente
  - nivel-3
client: innova
status: internal
updated: 2026-06-10
---

# Innova — Internal Tooling (n8n)

> n8n workspace for **Innova's own** internal automations (not a client). Instance: `vora2-n8n.6ewc8s.easypanel.host`.

← [[n8n/clients/_index|Clients]]

---

## Scope

Automations that serve Innova itself rather than a client. First one: a Telegram bot that reads this Obsidian vault and reports pending work.

## Flows

| Flow | Status | Trigger | Last update |
| --- | --- | --- | --- |
| [[n8n/clients/innova/flows/obsidian-pending-telegram-bot/spec\|Obsidian pending → Telegram bot]] | 🟢 live | Cron diario 08:00 ART + Telegram on-demand | 2026-06-10 |

## Conventions

- Internal tools live here so they don't pollute the client roster, but follow the same Spec Kit.
- Credentials in the n8n credential store (e.g. `telegram-innova-bot`).
