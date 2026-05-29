---
tags:
  - n8n
  - tasks
  - blincer
  - nivel-3
client: blincer
flow: whatsapp-overdue-debt-reminder
updated: 2026-05-29
status: blocked-by-oqs
---

# Tasks — Avisos automáticos de deuda vencida por WhatsApp

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/plan|Plan]]

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] Cerrar OQ-1 a OQ-7 del spec + OQ-G1, OQ-G2, OQ-G7.
- [ ] **Opt-in:** confirmar por escrito que los clientes activos firmaron consentimiento WhatsApp. Sin esto, NO se enciende el flow.
- [ ] Decidir provider (Evolution vs Cloud API) → si Cloud:
  - [ ] Crear WhatsApp Business Account + número verificado.
  - [ ] Submission de templates (6, uno por cadence_day default) a Meta para aprobación.
- [ ] Decidir source de facturas (Nexo API vs CSV vs ODBC).
- [ ] Confirmar tz n8n = ART (`echo $TZ` desde el container o config del workflow).
- [ ] Decidir fallback (A=email / B=Sheet / C=llamada) → MVP default = B.

## Setup

- [ ] Crear credenciales en n8n (las que faltan; reusar las del flow 1 donde aplique):
  - [ ] `whatsapp-blincer` (token del provider)
- [ ] Crear Sheets en `gsheets-blincer-ops`:
  - [ ] `cobranzas_config` (cols: `cadence_days`, `business_hours_window`, `excluded_customers`, `error_threshold_pct`, `fallback_threshold_count`)
  - [ ] `cobranzas_templates` (cols: `cadence_day`, `template_text`, `last_updated_by`)
  - [ ] `cobranzas_log` (cols: `timestamp`, `invoice_id`, `tango_customer_id`, `contact_id`, `cadence_day`, `status`, `provider_message_id`, `error`)
  - [ ] `cobranzas_fallback` (cols: `timestamp`, `invoice_id`, `customer`, `reason`)
  - [ ] `cobranzas_errors` (cols: `timestamp`, `invoice_id`, `node`, `error`, `payload`)
  - [ ] `cobranzas_metrics` (cols: `run_date`, `# evaluadas`, `# en cadencia`, `# enviados`, `# fallback`, `# ya pagó`, `# errores`, `p95 ms`)
- [ ] Sandra carga 6 templates default en `cobranzas_templates` (validar variables `{{nombre}}`, `{{invoice_id}}`, `{{amount_due}}`, `{{due_date}}`, `{{days_overdue}}`).
- [ ] Crear custom property HubSpot Company `last_dunning_at` (datetime).
- [ ] Crear timeline event HubSpot "Recordatorio cobranza" (CRM Cards API).
- [ ] Si Cloud API: registrar nombres de templates aprobados en `cobranzas_templates` (col adicional `cloud_api_template_name`).

## Build

- [ ] Crear workflow vacío en n8n: `blincer / whatsapp-overdue-debt-reminder`.
- [ ] Node 1: ScheduleTrigger cron `0 9 * * *`, tz `America/Argentina/Buenos_Aires`.
- [ ] Node 2: Sheets read `cobranzas_config!A:Z` → variables al contexto.
- [ ] Node 3: Sheets read `cobranzas_templates!A:D` → map por cadence_day.
- [ ] Node 4: HTTP / connector fetch overdue invoices (filter `vencimiento < today AND saldo > 0`).
- [ ] Node 5: Function calcular `days_overdue` y filtrar por `cadence_days`.
- [ ] Node 6: Function filter `excluded_customers`.
- [ ] Node 7: SplitInBatches batchSize=10, wait=2s.
- [ ] Node 8: Sheets lookup en `cobranzas_log` por `(invoice_id, cadence_day, today)` → IF dedupe.
- [ ] Node 9: HubSpot search Contact por `tango_customer_id`.
- [ ] Node 10: IF — `phone` matchea regex E.164.
- [ ] Node 11: HTTP re-fetch saldo de la factura específica.
- [ ] Node 12: Function compose message (template + vars).
- [ ] Node 13: Node WhatsApp (provider según OQ-1).
- [ ] Node 14: Sheets append `cobranzas_log` con `status=sent` + `provider_message_id`.
- [ ] Node 15: HubSpot update Company + create timeline event.
- [ ] Node 16: Sheets append `cobranzas_fallback` (rama no-WhatsApp).
- [ ] Node 17: Function summary + alerta interna si umbrales.
- [ ] Run `mcp__n8n-mcp__validate_node` en cada nodo.
- [ ] Run `mcp__n8n-mcp__n8n_validate_workflow`.

## Validate

- [ ] Datos sintéticos: 5 facturas vencidas con days_overdue ∈ {1, 5, 7, 15} → assert: 1 → sent (cadence 1), 5 → ignorada (no en cadence), 7 → sent (cadence 7), 15 → sent (cadence 15).
- [ ] Cliente excluido → ignorado, log skip.
- [ ] Cliente sin WhatsApp → fallback Sheet, sin envío.
- [ ] Cliente que pagó entre fetch y send (mockear saldo=0) → skip "ya pagó".
- [ ] Re-correr el flow el mismo día → 0 envíos nuevos (idempotency).
- [ ] Forzar Tango timeout → abort + alerta crítica.
- [ ] Modificar template a uno con placeholder inválido → flow lo detecta y aborta antes del envío (no manda mensaje vacío).
- [ ] Asegurar timezone — comparar timestamp del primer envío vs `date(ART)`.

## Harden

- [ ] Retry policy confirmada en cada nodo.
- [ ] Throttle (`splitInBatches` wait) ajustado según rate del provider real.
- [ ] Alerta wired a canal OQ-G7.
- [ ] Cron diario de revisión de `cobranzas_errors` (un workflow aparte mini).
- [ ] Dashboard básico: gráfico simple de `cobranzas_metrics` por mes (Sheets chart o tableau interno).

## Document

- [ ] Exportar workflow como `workflow.json`.
- [ ] Actualizar `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/blincer/README.md`.

## Handoff

- [ ] Notificar a Sandra con link a sheets `cobranzas_*` y cómo editar templates.
- [ ] Capacitación corta (15 min) a Sandra sobre cómo cambiar cadencias y templates.
- [ ] Escribir `retro.md`.
- [ ] Promover patterns:
  - `n8n/patterns/cron-bulk-whatsapp.md`
  - `n8n/patterns/sheets-config-driven-templates.md`
- [ ] Append one-liner a `n8n/lessons-learned.md`.
