---
tags:
  - n8n
  - spec
  - innova
  - nivel-3
client: innova
flow: obsidian-pending-telegram-bot
status: live
updated: 2026-06-10
---

# Spec — Obsidian pending → Telegram bot

> A Telegram bot that reads this Obsidian vault daily and reports pending work, grouped into confirmed projects / unconfirmed projects / other tasks. No LLM — fully deterministic, zero cost.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/innova/README|Innova Internal]]

---

## Goal

Give Francisco a daily push (and on-demand answers) over Telegram about what's pending across the whole Innova vault, so nothing falls through the cracks. Built without an LLM (reads the vault, parses curated pending markers, formats a message).

## Trigger

- **Daily push:** cron `0 8 * * *` (tz America/Argentina/Buenos_Aires) → workflow `c4sCjnMYbPV7TcH3`.
- **On-demand:** Telegram Trigger (webhook) → workflow `PIrXkj0OwdDLRRwH`. Commands: `/pendientes` (todo) · `clientes` (roster: cantidad + lista de clientes por tipo/estado/fase, con GAAB como prospecto n8n) · un cliente/keyword (`blincer`, `gpt-landings`) para filtrar pendientes.

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| vault | repo | GitHub `kolimbas/obsidian_innova` (public) | yes | bajado por ZIP/tree+raw, sin auth |
| command | string | mensaje de Telegram (on-demand) | no | filtro por cliente/keyword |

## Outputs / Side effects

- Telegram `sendMessage` al `chat_id` `5719368566` (daily) o al chat que escribió (on-demand).
- Snapshot del set de pendientes en n8n static data (`$getWorkflowStaticData('global').pending`) para el diff "desde ayer".

## What counts as "pending" (deterministic rules)

- Checkboxes `- [ ]` **solo** bajo callouts `[!todo]` o secciones tituladas *Pendientes / Próximos pasos / Open dependencies / Implementación pendiente / Pre-build*. (Se ignoran success-criteria de specs y los build-steps de tasks → señal vs ruido.)
- Discovery: cuenta de `> Respuesta:` sin responder.
- Flows n8n: resumen por cliente (`N flows, M blocked-by-oqs`), no tarea por tarea.

## Classification (buckets)

- **✅ Confirmados / en curso:** `clientes/*.md` con `estado ∈ {en-curso, en-correcciones, listo-para-deploy, entregado, activo}`.
- **🟡 No confirmados:** `estado = pre-venta` (u otro no-confirmado).
- **🔧 Otras tareas:** `agentes/*`, "Pendientes globales" de `INNOVA_MASTER`, clientes sin nota de negocio (gaab).

## Success criteria

- [x] Mensaje diario 08:00 ART con los 3 grupos + diff "desde ayer".
- [x] On-demand responde `/pendientes` y filtros por cliente.
- [x] Curado (no vuelca cada checkbox): ~56 items / ~2.5k chars, no 800.
- [x] Render robusto (HTML + escapado de `< > &`), sin errores de parseo.
- [x] Costo nulo (sin LLM, repo público sin auth).

## Out of scope

- Editar el vault desde Telegram (solo lectura).
- Resumen "inteligente"/priorizado por LLM (se descartó por costo; ver retro).

## Open questions / follow-ups

- [ ] Auto-sync del vault → GitHub (para que el bot vea ediciones locales). En curso vía SSH + systemd timer.
- [ ] ¿Sumar comando `/resueltos` o `/hoy`?

## Stakeholders

| Role | Person |
| --- | --- |
| Requester / End user | Francisco (Innova) |
| Builder | Innova |
