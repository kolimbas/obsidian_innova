---
tags:
  - n8n
  - tasks
  - innova
  - nivel-3
client: innova
flow: obsidian-pending-telegram-bot
status: live
updated: 2026-06-10
---

# Tasks — Obsidian pending → Telegram bot

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/innova/flows/obsidian-pending-telegram-bot/plan|Plan]]

---

## Setup

- [x] Bot creado con @BotFather → `@innovaok_bot`.
- [x] Credencial `telegram-innova-bot` en n8n (id `4jMI1PFVR4RkqE3Y`).
- [x] `chat_id` obtenido de la primera ejecución on-demand: `5719368566`.

## Build

- [x] WF diario `c4sCjnMYbPV7TcH3`: Schedule → Scan vault (Code) → Send Telegram.
- [x] WF on-demand `PIrXkj0OwdDLRRwH`: Telegram Trigger → Scan vault (Code) → Reply.
- [x] Motor de scan curado (section-aware, sin LLM), probado localmente (dry-run).
- [x] `n8n_validate_workflow` OK en ambos.

## Validate

- [x] On-demand `/pendientes` ejecutó OK (56 items, 8 proyectos).
- [x] Fix de parseo HTML (escapado `< > &`) → sin errores intermitentes.
- [x] Test de envío directo al chat confirmado.
- [x] Ambos workflows `active: true`.

## Harden / follow-ups

- [ ] Auto-sync del vault → GitHub (SSH + systemd timer) para reflejar ediciones locales.
- [ ] `sudo loginctl enable-linger ventas4` para que el timer sobreviva al logout (lo corre el usuario).
- [ ] Opcional: comando `/hoy` o `/resueltos`; refactor a sub-workflow compartido (hoy el Code está duplicado en los 2 WF).

## Document

- [x] Bundle Spec Kit (este).
- [x] Fila en `n8n/clients/_index.md` (workspace `innova`).
- [ ] Promover a pattern: `n8n/patterns/github-repo-scan-telegram.md` + node note `n8n/nodes/telegram.md` (HTML escaping).
