---
tags:
  - n8n
  - tasks
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: track-record-validation
updated: 2026-06-10
status: blocked-by-oqs
---

# Tasks — F · Track-record validation & cross-check

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/track-record-validation/plan|Plan]]

> [!warning] CARVE-OUT — gate de factibilidad antes de todo
> Este módulo está **fuera del precio cerrado de Fase 1**. El primer task es un gate: sin confirmar Elementix, no se avanza.

---

## Pre-build (gate de carve-out + OQs)

- [ ] **GATE — OQ-F-1: confirmar si Elementix expone API.** (def #6 🚦🚦)
  - Si **API** → F puede cotizarse como adicional; seguir.
  - Si **solo UI** → decidir con el cliente: scraping (frágil) vs **entrada asistida manual**, o **DESCARTAR F de Fase 1**.
- [ ] OQ-F-2: si solo UI, decidir scraping vs manual.
- [ ] OQ-F-3: cap de gasto OCR + Claude.
- [ ] OQ-F-4: criterios exactos de match entidad↔dirección (qué dispara un flag).
- [ ] Decisión comercial: precio del adicional / inclusión.
- [ ] D entregando docs + M0 (DB + Claude).

## Setup

- [ ] Credenciales `gptlandings-claude` (+ `gptlandings-elementix` solo si hay API).
- [ ] Test payloads sintéticos en `./test-payloads/`: `valid_match.json`, `valid_mismatch.json`, `source_down.json`, `ocr_fail.json`.
- [ ] **No** usar fuentes reales hasta confirmar acceso legítimo (OQ-F-1).

## Build (condicional — solo si el gate habilita)

- [ ] Crear workflow `gpt-landings / track-record-validation`.
- [ ] Node 1: trigger por evento de D.
- [ ] Node 2: OCR + Claude — extraer campos del doc.
- [ ] Node 3: Code — estructurar track record declarado.
- [ ] Node 4: IF — Elementix API disponible.
- [ ] Node 5a: HTTP cross-check (API) con fail-safe ante fuente caída.
- [ ] Node 5b: preparar comparación para humano (rama manual).
- [ ] Node 6: Code — computar flags (criterios OQ-F-4).
- [ ] Node 7: Postgres — guardar `validation_status` + `flags`.
- [ ] Node 8: surface + alerta al equipo.
- [ ] `mcp__n8n-mcp__validate_node` + `n8n_validate_workflow`.

## Validate

- [ ] `valid_match.json` → sin flags, status validado (sujeto a revisión humana).
- [ ] `valid_mismatch.json` → flag trazable con el motivo.
- [ ] `source_down.json` → "no verificado" (fail-safe), **nunca** "match".
- [ ] `ocr_fail.json` → `needs_human`, sin corromper estado.
- [ ] Re-validar mismo doc → no duplica flags.
- [ ] Assert: toda auto-validación queda marcada para revisión humana.

## Harden

- [ ] Fail-safe ante fuente externa caída.
- [ ] Cap de tokens/costo + alerta.
- [ ] Retries DB/alertas.

## Document

- [ ] Exportar `workflow.json`.
- [ ] `spec.md` status → `live` (si se construye).
- [ ] Actualizar fila en `n8n/clients/gpt-landings/README.md` (sigue marcada carve-out hasta cierre comercial).

## Handoff

- [ ] Capacitar al analista sobre cómo revisar los flags.
- [ ] Escribir `retro.md` (clave: documentar el resultado de OQ-F-1 y su impacto en alcance/precio).
- [ ] Append one-liner a `n8n/lessons-learned.md` (factibilidad de fuentes externas tipo Elementix).
