---
tags:
  - n8n
  - tasks
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: whatsapp-doc-reminder
updated: 2026-06-10
status: blocked-by-oqs
---

# Tasks — E · WhatsApp document reminders

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/whatsapp-doc-reminder/plan|Plan]]

> Ordered, verifiable. **Gate duro: no encender el envío sin opt-in documentado.** Base flow = `whatsapp-overdue-debt-reminder` de Blincer.

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] OQ-E-1: decidir provider WhatsApp (Cloud API vs Evolution vs BSP) (def #5 🚦) — **arrancar verificación del número YA por el lead time.**
- [ ] OQ-E-2: si Cloud API → submission de templates HSM a Meta (aprobación ~días) 🚦.
- [ ] OQ-E-3: confirmar opt-in del borrower (se captura al iniciar el form).
- [ ] OQ-E-4: definir cadencia (default cada 2 días, máx 1/día).
- [ ] OQ-E-5: fallback sin WhatsApp (email vs solo log).
- [ ] D entregando `estado_checklist` + evento de completitud; M0 (DB + tz).
- [ ] Confirmar tz de la instancia (South Florida / ET).

## Setup

- [ ] Credencial `gptlandings-whatsapp` + `gptlandings-db` + `gptlandings-internal-alert`.
- [ ] Config + templates editables (Sheet/DB): un template por día de cadencia con variables `{{nombre}}`, `{{completados}}`, `{{total}}`, `{{faltan}}`.
- [ ] Si Cloud API: registrar nombres de templates aprobados.
- [ ] Test payloads en `./test-payloads/`: `reminder_incomplete.json`, `reminder_complete_today.json`, `reminder_no_optin.json`, `reminder_rerun_same_day.json`.

## Build

- [ ] Crear workflow `gpt-landings / whatsapp-doc-reminder`.
- [ ] Node 1: ScheduleTrigger (cron + tz ET).
- [ ] Node 2-3: Read config + templates.
- [ ] Node 4: Postgres — préstamos con checklist incompleto (`pending > 0`).
- [ ] Node 5: Code — match de cadencia.
- [ ] Node 6: SplitInBatches (throttle).
- [ ] Node 7: Idempotency check `(loan_id, cadence_day, date)`.
- [ ] Node 8: IF — número E.164 + `opt_in`.
- [ ] Node 9: Code — compose (template + completados/total/faltan).
- [ ] Node 10: Send WhatsApp (provider OQ-E-1).
- [ ] Node 11: append `reminders_log` (write-back idempotencia).
- [ ] Node 12: fallback (log/email).
- [ ] Node 13: IF checklist completo → aviso equipo + stop.
- [ ] `mcp__n8n-mcp__validate_node` + `n8n_validate_workflow`.

## Validate

- [ ] `reminder_incomplete.json` → mensaje con faltantes correctos.
- [ ] `reminder_complete_today.json` → no recuerda; avisa al equipo.
- [ ] `reminder_no_optin.json` → no envía (gate opt-in); va a fallback.
- [ ] `reminder_rerun_same_day.json` → 0 envíos nuevos (idempotencia).
- [ ] tz: timestamp del primer envío vs `date(ET)`.
- [ ] Assert cada success criterion del spec.

## Harden

- [ ] Retry 2× en el send; 3× en I/O.
- [ ] Throttle ajustado al rate real del provider.
- [ ] Alerta wired al canal interno.
- [ ] Cron de revisión de `reminders_errors`.

## Document

- [ ] Exportar `workflow.json`.
- [ ] `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/gpt-landings/README.md`.

## Handoff

- [ ] Capacitación corta al cliente sobre cómo editar templates/cadencia.
- [ ] Escribir `retro.md`.
- [ ] Promover/actualizar `n8n/patterns/cron-bulk-whatsapp.md` (ya candidato desde Blincer).
- [ ] Append one-liner a `n8n/lessons-learned.md`.
