---
tags:
  - n8n
  - spec
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: whatsapp-doc-reminder
status: draft
updated: 2026-06-10
---

# Spec — E · WhatsApp document reminders

> While a loan's document checklist is incomplete, send the borrower contextual WhatsApp reminders ("you've uploaded 9 of 10, missing X") on a cadence, and notify the team when it's complete.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/README|GPT Landings]]

> [!note] Reusa el flow de Blincer
> Base flow ≈ `whatsapp-overdue-debt-reminder` de Blincer (~70%): cron + per-item + WhatsApp + templates editables + sheet-idempotency. Cambia el **predicado** (checklist incompleto, no factura vencida) y las **variables** del template.

---

## Goal

Reduce manual chasing of documentation. Each day, for loans whose `estado_checklist` is incomplete, send the borrower a contextual WhatsApp reminder listing what's missing, on a configurable cadence and with editable templates. When a loan's checklist becomes complete, notify the team.

## Trigger

- Type: cron
- Schedule: diario (ej. `0 9 * * *`, tz a confirmar — South Florida / ET)
- Expected frequency: 1×/día; cada corrida procesa todos los préstamos con checklist incompleto

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `incomplete_loans` | list | DB `estado_checklist` (de D) | yes | préstamos con ≥1 slot pendiente |
| `borrower_contact` | object | DB `prestamos` | yes | número WhatsApp + nombre |
| `templates` | object | Sheet/DB (editable) | yes | un template por día de cadencia |
| `cadence_days` | list[int] | config (editable) | yes | default ej. cada 2 días (máx 1/día) |
| `opt_in` | bool | DB | yes | consentimiento del borrower |

Sample payload (interno, tras leer el checklist):

```json
{
  "loan_id": "L-2026-0042",
  "borrower_name": "Acme Holdings",
  "phone": "+13055551234",
  "completed": 9,
  "total": 10,
  "missing": ["bank_statements"],
  "days_since_last_reminder": 2
}
```

## Outputs / Side effects

- WhatsApp: mensaje al borrower (template del día de cadencia + variables `{{nombre}}`, `{{completados}}`, `{{total}}`, `{{faltan}}`).
- DB/Sheet `reminders_log`: una fila por intento (sent/skipped/error).
- Team alert: cuando un checklist se completa → aviso al canal interno (OQ-0.4).

## Success criteria

- [ ] Sólo préstamos con checklist **incompleto** reciben recordatorio.
- [ ] Un mensaje por préstamo por día de cadencia (idempotente por `(loan_id, cadence_day, date)`).
- [ ] Templates editables sin tocar n8n (Sheet/DB).
- [ ] Al completarse el checklist → aviso al equipo + se corta el recordatorio.
- [ ] Mensaje dentro de horario razonable (el cron lo garantiza).

## Out of scope

- El intake en sí (Módulo D).
- La validación de contenido (Módulo F).
- Recordatorios por otros canales (email fallback es V2 — ver OQ-E-5).

## Open questions

- [ ] **OQ-E-1** — **Número/cuenta WhatsApp Business + proveedor** (Cloud API oficial vs Evolution vs BSP). 🚦 *(def #5, arrancar YA por lead time)*
- [ ] **OQ-E-2** — **Templates HSM aprobados por Meta** (lead time de aprobación). 🚦
- [ ] **OQ-E-3** — Opt-in del borrower (¿lo da al iniciar el form?).
- [ ] **OQ-E-4** — Cadencia (default propuesto: cada 2 días, máx 1/día).
- [ ] **OQ-E-5** — Fallback si no hay WhatsApp (email vs solo log).

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Manager/owner GPT Landings |
| Approver | Manager/owner (templates y cadencia) |
| End user | Borrower (recibe) + equipo (recibe el "completo") |
