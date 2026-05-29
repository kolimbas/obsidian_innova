---
tags:
  - n8n
  - tasks
  - blincer
  - nivel-3
client: blincer
flow: sales-bot-with-quotes
updated: 2026-05-29
status: blocked-by-oqs
---

# Tasks — Bot de ventas + cotizaciones/facturas automáticas

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/flows/sales-bot-with-quotes/plan|Plan]]

> **No empezar Build hasta cerrar OQs.** Por la magnitud de este flow, recomendación: armar un MVP "solo conversación" (sin cotización ni factura) primero, validar 1 semana, después sumar cotizaciones, después facturas.

---

## Pre-build (resolución de OQs y prerequisitos)

- [ ] Cerrar OQ-1 a OQ-9 del spec + OQ-G1, OQ-G2, OQ-G7.
- [ ] Definir budget LLM mensual (USD/mes) y modelo default + escalación.
- [ ] Diseñar system prompt + `handoff_rules` con Guillermo (sesión dedicada).
- [ ] Diseñar template DOCX de cotización (branding Blincer + variables).
- [ ] Setup Gotenberg (decisión: VPS Innova vs cloud) + endpoint accesible para n8n.
- [ ] Confirmar Postgres disponible para conversaciones (Supabase de Innova o nuevo schema).
- [ ] Si Cloud API: aprobar templates con Meta (no aplica al inbound libre, sí a outbound fuera de ventana 24h).
- [ ] Confirmar coexistencia provider WhatsApp con flow 2 (sin race conditions).
- [ ] HubSpot Products poblado con SKU, descripción, precio, descripción larga (knowledge base del bot).
- [ ] Crear custom props HubSpot Deal: `quote_pdf_url`, `quote_items_json`, `valid_until`, `invoice_id`, `last_bot_interaction_at`.
- [ ] Backfill `tango_customer_id` ya hecho en flow 1 — verificar cobertura.

## Setup

- [ ] Credenciales en n8n (nuevas):
  - [ ] `anthropic-blincer-bot` (API key)
  - [ ] `gdrive-blincer-quotes` (service account)
  - [ ] `pg-innova-shared` (connection string)
- [ ] Postgres schema (script en `./db/migration.sql`):
  - [ ] `conversaciones (id, from_phone, role, text, ts, deal_id, tokens_in, tokens_out)`
  - [ ] `processed_messages (message_id, from_phone, processed_at)` con index único.
  - [ ] `bot_errors (id, message_id, contact_id, step, error, payload, ts)`
- [ ] Sheets:
  - [ ] `bot_config` (cols: `bot_enabled`, `default_model`, `escalation_model`, `system_prompt`, `handoff_rules_json`, `monthly_budget_usd`, `approval_required`)
  - [ ] `bot_metrics` (daily row)
- [ ] Template DOCX subido a Drive (folder `INNOVA_MASTER/blincer/templates/`).
- [ ] Test phone configurado en provider WhatsApp.
- [ ] HubSpot pipeline review: stages "Cotización enviada", "Ganado", "Cerrado-perdido" presentes con slugs anotados acá:
  - `__pendiente__`

## Build — Fase 1 (MVP "solo conversación")

- [ ] Workflow `blincer / sales-bot-with-quotes` creado en n8n.
- [ ] Node 1: Webhook `/blincer-sales-bot` (POST, respond=immediate).
- [ ] Node 2: respondToWebhook 200.
- [ ] Node 3: Postgres lookup `processed_messages` por `message_id`. IF seen → end.
- [ ] Node 4: Postgres insert en `processed_messages`.
- [ ] Node 5: Postgres select últimos 20 turns de `conversaciones` por `from_phone`.
- [ ] Node 6: HubSpot search Contact by phone; if not → create.
- [ ] Node 7: Function build prompt (system + history + user msg).
- [ ] Node 8: httpRequest Anthropic messages endpoint (Haiku default), sin tools en esta fase.
- [ ] Node 9: WhatsApp send response.
- [ ] Node 10: Postgres insert user msg + assistant msg en `conversaciones`.
- [ ] Node 11: Sheets append `bot_metrics`.
- [ ] Validate Fase 1: 20 conversaciones de prueba sin tools (solo charla).

## Build — Fase 2 (sumar tools de lectura)

- [ ] Definir tool schemas: `get_catalog`, `check_stock`, `lookup_customer`.
- [ ] Node 8 (modificado): tool definitions en request a Anthropic.
- [ ] Node 7b: Tool router (switch).
- [ ] Node 7a (`get_catalog`): HubSpot products search.
- [ ] Node 7b (`check_stock`): HTTP Tango GET stock by sku.
- [ ] Node 7c (`lookup_customer`): HubSpot Company search.
- [ ] Node 8b: re-call Anthropic con tool_result (loop hasta `stop_reason=end_turn`, max 5 iter).
- [ ] Validate Fase 2: 10 conversaciones que requieren consultar catálogo/stock; assert no-hallucination.

## Build — Fase 3 (cotización)

- [ ] Sub-workflow `create_quote` con sus 7 nodos.
- [ ] Tool `create_quote` definido con schema (items array, customer_id, descuento opcional).
- [ ] Gotenberg integrado (HTTP request a `/forms/libreoffice/convert`).
- [ ] Drive upload + share link público restringido.
- [ ] HubSpot Deal create con stage "Cotización enviada".
- [ ] WhatsApp send media (PDF link).
- [ ] Validate Fase 3: 5 cotizaciones reales (cliente sintético) y review manual del PDF.

## Build — Fase 4 (facturación)

- [ ] Solo si OQ-G1 = Nexo. Si local, skip a "factura manual con asistencia bot" (out of scope para Fase 4 auto).
- [ ] Sub-workflow `emit_invoice`.
- [ ] Reusar lógica del flow 1 para credit-limit check.
- [ ] Tool `emit_invoice` con schema (deal_id).
- [ ] Si `approval_required=true` (default MVP): crear HubSpot Task asignada a Sandra, responder al cliente "facturando".
- [ ] On approval (manual o auto): Tango POST `/comprobantes`, get PDF, send.
- [ ] Update Deal stage → "Ganado".
- [ ] Validate Fase 4: 3 facturas en sandbox + 1 real con review.

## Build — Fase 5 (handoff)

- [ ] Sub-workflow `handoff_human`.
- [ ] Tool `handoff_human` con schema (reason, summary).
- [ ] Crear HubSpot Task con resumen + asignada al vendedor de turno.
- [ ] Internal alert al canal OQ-G7.
- [ ] Mensaje al cliente: "te paso con un asesor".
- [ ] Validate Fase 5: trigger explícito y por umbrales.

## Validate (integral)

- [ ] Ejecutar todo el set `./test-payloads/*.json`.
- [ ] Cero hallucinations sobre precios y stock (auditoría de 50 conversaciones).
- [ ] p95 latencia < 30s para cotizaciones, < 5s para charla simple.
- [ ] Costo LLM por conversación medido y dentro de budget.
- [ ] Idempotencia: reenviar mismo webhook 2× → 0 efectos extra.
- [ ] Credit-limit block dispara handoff (no emite factura).

## Harden

- [ ] Kill switch `bot_enabled=false` en `bot_config` corta el flow en nodo 1.
- [ ] Budget alert a Guillermo cuando 80% del cap mensual.
- [ ] Rate limit en el webhook (n8n max concurrent executions).
- [ ] Anti-loop validado: forzar conversación que confunde al LLM y verificar hard stop.

## Document

- [ ] Exportar workflow + sub-workflows como `workflow.json` y `workflow.<sub>.json`.
- [ ] Actualizar `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/blincer/README.md`.
- [ ] Documentar system prompt final en `bot_config.system_prompt` (versionado por timestamp).

## Handoff

- [ ] Capacitación a Guillermo + vendedores: cómo monitorear, cómo intervenir, cómo apagar.
- [ ] Capacitación a Sandra: aprobación de facturas pendientes en HubSpot Tasks.
- [ ] Escribir `retro.md` con énfasis en sorpresas del LLM y patterns de prompt.
- [ ] Promover patterns:
  - `n8n/patterns/chat-bot-with-tool-use.md`
  - `n8n/patterns/quote-pdf-generation.md`
  - `n8n/patterns/handoff-human.md`
  - `n8n/patterns/llm-budget-cap.md`
- [ ] Append one-liner a `n8n/lessons-learned.md`.
