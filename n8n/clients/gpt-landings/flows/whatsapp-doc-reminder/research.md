---
tags:
  - n8n
  - research
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: whatsapp-doc-reminder
updated: 2026-06-10
---

# Research — E · WhatsApp document reminders

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/whatsapp-doc-reminder/spec|Spec]]

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/gpt-landings/flows/` (2026-06-10).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `document-intake-form` (D) | 🟢 Alta | Provee `estado_checklist` (el predicado "incompleto") + el evento de completitud. |
| `m0-infra-setup` | 🟡 Media | DB + credenciales. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, other `n8n/clients/*/flows/` (2026-06-10).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| **`n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/`** | **BASE FLOW (~70%)**: cron → fetch incompletos → split per-item → send WhatsApp → log, con templates editables + idempotencia | Reusar casi entero. Cambia el predicado ("checklist incompleto" vs "factura vencida") y las variables del template. |
| `n8n/patterns/sheet-idempotency.md` | Dedup `(loan_id, cadence_day, date)` | Un mensaje por préstamo por día de cadencia. |

## 3. n8n nodes considered

(Calcados de `whatsapp-overdue-debt-reminder`.)

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.scheduleTrigger` | Cron diario | cron, timezone | Confirmar tz de la instancia (South Florida / ET) — calcar la lección de tz de Blincer. |
| `n8n-nodes-base.postgres` | Leer préstamos con checklist incompleto | query | predicado `pending > 0` |
| `n8n-nodes-base.googleSheets` o `postgres` | Config + templates editables | range | mantener editable por el cliente |
| `n8n-nodes-base.splitInBatches` | Loop per-loan, throttle | batchSize | throttle obligatorio (rate WhatsApp) |
| WhatsApp (Cloud API) / Evolution | Enviar mensaje | template/phone | **Cloud API exige template HSM aprobado por Meta** (lead time) — calcar lección de Blincer |
| `n8n-nodes-base.if` | Skip si ya enviado hoy | condition | idempotencia |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| WhatsApp Cloud API (Meta) | Bearer (system user) | 250 conv/24h Tier 1, sube con calidad | message_id parcial | https://developers.facebook.com/docs/whatsapp/cloud-api |
| WhatsApp Evolution API | API key local | ~60-100 msg/min | manual (clientMessageId) | https://doc.evolution-api.com |
| Supabase/Postgres | conn string | n/a | dedup `(loan_id, cadence_day, date)` | https://supabase.com/docs |

## 5. Base flow decision

- **Base flow chosen:** `n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/` (spec + plan + research).
- **Coverage estimate:** ~70%.
- **What we copy / what we change:** copiamos cron + per-item + WhatsApp + templates + idempotencia + manejo de riesgos (opt-in, templates Meta, tz, fallback). Cambiamos el predicado (checklist incompleto) y las variables del template (`{{completados}}/{{total}}/{{faltan}}`).

## 6. Reuse summary

- ✅ Reusing: estructura completa de `whatsapp-overdue-debt-reminder`; `sheet-idempotency`; lecciones de riesgo WhatsApp (baneo, templates Meta, tz, opt-in).
- 🆕 New work: predicado sobre `estado_checklist`; variables del template de documentación; corte del recordatorio al completar + aviso al equipo.
- ⚠️ Risks identified: **lead time de templates HSM de Meta** (arrancar verificación YA, def #5); opt-in del borrower; tz de la instancia (South Florida/ET, no ART); baneo si Evolution sin opt-in.
