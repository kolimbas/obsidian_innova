---
tags:
  - n8n
  - tasks
  - innova
  - nivel-3
client: innova
flow: obsidian-pending-telegram-bot
status: live
updated: 2026-06-11
---

# Tasks — Obsidian pending → Telegram bot

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/innova/flows/obsidian-pending-telegram-bot/plan|Plan]]

---

## Setup

- [x] Bot creado con @BotFather → `@innovaok_bot`.
- [x] Credencial `telegram-innova-bot` en n8n (id `4jMI1PFVR4RkqE3Y`).
- [x] `chat_id` obtenido de la primera ejecución on-demand: `5719368566`.
- [x] Credencial `github-innova-vault` (githubApi, id `ahSijtt5sOQRFKhv`) con PAT fine-grained (Contents R/W sobre el repo) para la captura.

## Build

- [x] WF diario `c4sCjnMYbPV7TcH3`: Schedule → Scan vault (Code) → Send Telegram.
- [x] WF on-demand `PIrXkj0OwdDLRRwH`: Telegram Trigger → Scan vault (router) → IF escritura → GitHub commit / Reply.
- [x] Motor de scan curado (section-aware, sin LLM), probado localmente (dry-run).
- [x] **Fase 1 — Lectura + menú:** `/hoy`, `/bloqueantes`, `/buscar`, `/resumen`, `/help`, `/clientes`, filtro por cliente; `setMyCommands`.
- [x] **Fase 2 — Captura:** `/tarea` y `/nota` → GitHub `file:edit` (cliente reconocido o `inbox.md`); `inbox.md` creado.
- [x] **Fase 3 — `/hecho`:** toggle `- [ ]` → `- [x]` + commit.
- [x] **Fase 4 — Botones inline:** `callback_query` en el trigger, `reply_markup` en las respuestas y en el digest diario, `Answer callback` (answerQuery).
- [x] Render HTML lindo: títulos en negrita + cita plegable (`<blockquote expandable>`) por proyecto.
- [x] `n8n_validate_workflow` OK en ambos.

## Validate

- [x] On-demand `/pendientes` ejecutó OK.
- [x] Fix de parseo HTML (escapado `< > &`) → sin errores intermitentes.
- [x] Captura confirmada end-to-end (commits `bot: captura...` / `bot: hecho...` en el repo).
- [x] Botones responden; menú `/` aparece.
- [x] Render lindo confirmado (dry-run local: blockquotes/negritas balanceadas, contenido escapado).
- [x] Ambos workflows `active: true`.

## Harden / follow-ups

- [x] Auto-sync del vault → GitHub (SSH + systemd timer cada 10 min).
- [x] **Fix de storm:** cache de 60s del vault (la escritura lee fresco e invalida el cache) → ejecuciones rápidas, sin reintentos en cascada del webhook. Runbook si reaparece: `deleteWebhook?drop_pending_updates=true` + desactivar WF, diagnosticar duración del Code node, reactivar.
- [ ] `sudo loginctl enable-linger ventas4` para que el timer sobreviva al logout (lo corre el usuario).
- [ ] Refactor a sub-workflow compartido (hoy el Code router está duplicado en los 2 WF).

## Document

- [x] Bundle Spec Kit (este) actualizado con comandos de lectura/captura/`hecho`, menú, botones, cache y render HTML.
- [x] Fila en `n8n/clients/_index.md` (workspace `innova`).
- [ ] Promover a pattern: `n8n/patterns/github-repo-scan-telegram.md` + node note `n8n/nodes/telegram.md` (HTML escaping + blockquote expandable + storm/cache).
