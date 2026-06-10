---
tags:
  - n8n
  - tasks
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: partner-matching-engine
updated: 2026-06-10
status: blocked-by-oqs
---

# Tasks — A · Capital-partner matching engine

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/partner-matching-engine/plan|Plan]]

> Ordered, verifiable. No empezar Build sin M0 listo y las guidelines reales relevadas.

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] OQ-A-1: relevar formato y **cantidad** de guidelines por partner (def #1 🚦).
- [ ] OQ-A-3: definir dónde se leen las guidelines (DB editable vs Sheet).
- [ ] OQ-A-4: definir el schema canónico del préstamo (campos obligatorios).
- [ ] OQ-A-2: confirmar estabilidad de parámetros y quién los edita.
- [ ] OQ-A-5: cap de gasto Claude + modelo (Haiku default).
- [ ] M0 listo (DB con `capital_partners` + creds).

## Setup

- [ ] Credenciales `gptlandings-db` y `gptlandings-claude` en n8n.
- [ ] Cargar guidelines reales de 1-2 partners como seed en `capital_partners`.
- [ ] Preparar test payloads en `./test-payloads/`: `loan_clean_pass.json`, `loan_borderline.json`, `loan_no_match.json`, `loan_bad_format.json`.

## Build

- [ ] Crear workflow `gpt-landings / partner-matching-engine`.
- [ ] Node 1: Webhook `gptlandings-loan-intake`.
- [ ] Node 2: Code — normalizar `raw_format` (excel/csv/email/form) a schema único.
- [ ] Node 3: Postgres — leer `capital_partners` activos + guidelines.
- [ ] Node 4: Code — motor de reglas duras por partner (reglas como datos).
- [ ] Node 5: IF — detectar caso borroso.
- [ ] Node 6: Claude — evaluar borroso con salida estructurada (`llm_flag=true`).
- [ ] Node 7: Code — armar `qualifying_partners[]` con rationale + flags.
- [ ] Node 8: Postgres — upsert `prestamos.partner_match_json` por `loan_id`.
- [ ] Node 9: disparar `term-sheet-generation` (Execute Workflow / evento).
- [ ] Wire error branches (formato no soportado, Claude falla → `needs_human`).
- [ ] `mcp__n8n-mcp__validate_node` en cada nodo + `n8n_validate_workflow`.

## Validate

- [ ] `loan_clean_pass.json` → qualifying_partners con `hard_pass=true`, sin LLM.
- [ ] `loan_borderline.json` → marca `llm_flag=true` con explicación; no auto-aprueba hard pass.
- [ ] `loan_no_match.json` → qualifying_partners vacío, rationale del rechazo.
- [ ] `loan_bad_format.json` → error branch + `needs_human`, sin upsert corrupto.
- [ ] Re-correr mismo `loan_id` → upsert pisa, no duplica.
- [ ] Assert cada success criterion del spec.

## Harden

- [ ] Retries DB 3×; Claude 1×.
- [ ] Cap de tokens/llamada + alerta si gasto > umbral.
- [ ] Validación del JSON de salida contra el contrato de B.

## Document

- [ ] Exportar `workflow.json`.
- [ ] `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/gpt-landings/README.md`.

## Handoff

- [ ] Notificar a Innova/equipo que B puede consumir el JSON.
- [ ] Escribir `retro.md`.
- [ ] Promover `n8n/patterns/rules-engine-with-llm-fallback.md` si generaliza.
- [ ] Append one-liner a `n8n/lessons-learned.md`.
