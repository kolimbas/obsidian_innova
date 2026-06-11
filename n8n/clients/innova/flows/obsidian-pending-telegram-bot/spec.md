---
tags:
  - n8n
  - spec
  - innova
  - nivel-3
client: innova
flow: obsidian-pending-telegram-bot
status: live
updated: 2026-06-11
---

# Spec — Obsidian pending → Telegram bot

> A Telegram bot that reads this Obsidian vault and reports pending work (grouped into confirmed / unconfirmed projects / other tasks), **captures** new tasks & notes back into the vault, and is fully interactive (slash-menu + inline buttons). No LLM — deterministic, zero cost.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/innova/README|Innova Internal]]

---

## Goal

Give Francisco a daily push **and** an interactive on-demand assistant over Telegram that (a) reports what's pending across the whole vault, (b) lets him **capture** tasks/notes straight into the vault, and (c) **tick tasks done** — all without an LLM (reads the vault, parses curated markers, commits via the GitHub API).

## Trigger

- **Daily push:** cron `0 8 * * *` (tz America/Argentina/Buenos_Aires) → workflow `c4sCjnMYbPV7TcH3`.
- **On-demand:** Telegram Trigger (webhook, updates `message` + `callback_query`) → workflow `PIrXkj0OwdDLRRwH`.

## Commands

**📊 Lectura** (devuelven un mensaje, no tocan el vault):
- `/pendientes` (= `/start`, `/hola`, vacío) — todo lo pendiente, agrupado.
- `/clientes` — roster: cantidad + lista por tipo (web/automatización) / estado / fase (GAAB como prospecto n8n).
- `/hoy` — ítems nuevos / resueltos desde el último `/pendientes` (diff sobre snapshot).
- `/bloqueantes` — solo los ítems con `🚦` / `blocked` / `bloquea` / `esperando oqs`.
- `/buscar <texto>` — full-text sobre todas las notas; devuelve las que matchean.
- `/resumen` — conteos por bucket y por proyecto.
- `/help` (= `/menu`, `/ayuda`) — lista de comandos.
- `<cliente>`/keyword (`blincer`, `gpt-landings`, …) — filtra los pendientes de ese cliente.

**✍️ Captura** (escriben en el vault → commit a GitHub):
- `/tarea <cliente> <texto>` — agrega `- [ ] <texto> (bot <fecha>)` a `clientes/<slug>.md` (bajo `## Capturado por bot`). Si no reconoce el cliente → `inbox.md`.
- `/nota <texto>` (= `/idea`) — agrega a `inbox.md`.
- `/hecho <cliente|inbox> <texto>` — busca la primera línea `- [ ]` que contenga `<texto>` y la tilda `- [x]`.

**Interacción:**
- **Menú `/`** (Telegram `setMyCommands`) — los comandos aparecen al tipear `/`.
- **Botones inline** (`reply_markup.inline_keyboard`) bajo cada respuesta y bajo el digest diario: 🗂️ Pendientes · 🔁 Hoy · 🚦 Bloqueantes · 👥 Clientes · 📊 Resumen. El tap manda el comando (vía `callback_query`); un nodo `answerQuery` corta el spinner.

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| vault | repo | GitHub `kolimbas/obsidian_innova` (público) | yes | lectura por tree+raw, sin auth; **cache 60s** en static data |
| comando | string | `message.text` o `callback_query.data` | no | router lo parsea en `cmd` + `arg` |
| GitHub PAT | secret | credencial n8n `github-innova-vault` | sí (escritura) | fine-grained, Contents R/W solo sobre el repo |

## Outputs / Side effects

- Telegram `sendMessage` (HTML) al `chat_id` `5719368566` (daily) o al chat que escribió/tocó (on-demand).
- **Commits al vault** (comandos de captura) vía nodo GitHub `file:edit` con la credencial `github-innova-vault`.
- Snapshot del set de pendientes en n8n static data (`$getWorkflowStaticData('global').pending`) para el diff "desde ayer / desde tu último /pendientes".

## What counts as "pending" (deterministic rules)

- Checkboxes `- [ ]` **solo** bajo callouts `[!todo]` o secciones tituladas *Pendientes / Próximos pasos / Open dependencies / Implementación pendiente / Pre-build / **Capturado por bot***. (Se ignoran success-criteria de specs y build-steps de tasks → señal vs ruido.)
- `inbox.md`: **todo** `- [ ]` cuenta (bucket "🔧 Otras", proyecto "📥 Sin clasificar").
- Discovery: cuenta de `> Respuesta:` sin responder.
- Flows n8n: resumen por cliente (`N flows, M blocked-by-oqs`), no tarea por tarea.

## Rendering

- Mensaje en **HTML**: título y secciones en **negrita**, y cada proyecto en una **cita plegable** (`<blockquote expandable>`) — colapsado por default, se expande con un toque (adiós al choclo de texto).
- El contenido dinámico se escapa (`& < >`); las tags HTML no. Ítems cortados a 100 chars; cap 6 ítems/proyecto; truncado seguro a 4096.

## Success criteria

- [x] Mensaje diario 08:00 ART con los 3 grupos + diff "desde ayer".
- [x] On-demand responde lectura (`/pendientes`, `/clientes`, `/hoy`, `/bloqueantes`, `/buscar`, `/resumen`, `/help`, filtro por cliente).
- [x] Captura `/tarea` y `/nota` → commit al vault (cliente reconocido o `inbox.md`).
- [x] `/hecho` tilda la tarea y commitea.
- [x] Menú `/` (setMyCommands) + botones inline (en respuestas y en el digest diario).
- [x] Render HTML lindo (negritas + citas plegables).
- [x] Estable bajo uso interactivo: **cache de 60s** del vault (la escritura lee el archivo fresco) → sin storm de reintentos del webhook.
- [x] Costo nulo (sin LLM, repo público para lectura).

## Out of scope

- Resumen "inteligente"/priorizado por LLM (descartado por costo; ver retro).
- Multi-usuario / control de acceso (es un bot personal).

## Open questions / follow-ups

- [x] Auto-sync del vault → GitHub (SSH + systemd timer cada 10 min). **Hecho.**
- [ ] `sudo loginctl enable-linger ventas4` para que el timer sobreviva al logout (lo corre el usuario).
- [ ] Refactor a sub-workflow compartido (hoy el Code router está duplicado en los 2 WF).

## Stakeholders

| Role | Person |
| --- | --- |
| Requester / End user | Francisco (Innova) |
| Builder | Innova |
