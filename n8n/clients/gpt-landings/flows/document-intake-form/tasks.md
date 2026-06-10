---
tags:
  - n8n
  - tasks
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: document-intake-form
updated: 2026-06-10
status: blocked-by-oqs
---

# Tasks — D · Document intake (backend)

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/document-intake-form/plan|Plan]]

> Ordered, verifiable. El front-end (web) se trackea aparte en la nota de negocio. Acá solo el backend n8n.

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] OQ-D-1: lista exacta de documentos por tipo de préstamo (def #4 🚦).
- [ ] OQ-D-2: ¿la lista cambia por partner/tipo?
- [ ] OQ-D-3: storage (Drive vs Supabase Storage) + tamaños máximos.
- [ ] OQ-D-4: dónde corre el front + auth del borrower (link único por préstamo).
- [ ] M0: DB con `documentos_requeridos`/`estado_checklist` + storage.

## Setup

- [ ] Credenciales `gptlandings-drive`/`gptlandings-db`.
- [ ] Cargar la definición de slots por `loan_type` en `documentos_requeridos`.
- [ ] Definir el contrato con el front (payload + pre-signed URL).
- [ ] Test payloads en `./test-payloads/`: `intake_single_doc.json`, `intake_complete_set.json`, `intake_resubmit_same_slot.json`, `intake_invalid_slot.json`.

## Build

- [ ] Crear workflow `gpt-landings / document-intake-form`.
- [ ] Node 1: Webhook `gptlandings-intake`.
- [ ] Node 2: Postgres — leer slots requeridos por `loan_type`.
- [ ] Node 3: Code — map doc→slot + validar slot.
- [ ] Node 4: persistir archivo (Drive/Supabase, idealmente pre-signed URL).
- [ ] Node 5: Postgres — upsert `estado_checklist` (status `received`).
- [ ] Node 6: Code — calcular completitud.
- [ ] Node 7: emitir evento a E (faltantes).
- [ ] Node 8: emitir evento a F (doc nuevo).
- [ ] Node 9: aviso al equipo si el checklist quedó completo.
- [ ] `mcp__n8n-mcp__validate_node` + `n8n_validate_workflow`.

## Validate

- [ ] `intake_single_doc.json` → slot pasa a `received`, checklist refleja faltantes.
- [ ] `intake_complete_set.json` → checklist completo → aviso al equipo.
- [ ] `intake_resubmit_same_slot.json` → upsert pisa, no duplica fila.
- [ ] `intake_invalid_slot.json` → error branch, sin corromper estado.
- [ ] Upload pesado (simulado) → no se trunca.
- [ ] Assert cada success criterion del spec.

## Harden

- [ ] Pre-signed URL para no pasar archivos pesados por n8n.
- [ ] Retries en storage/DB.
- [ ] Validar auth del borrower en el webhook (token del link único).

## Document

- [ ] Exportar `workflow.json`.
- [ ] `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/gpt-landings/README.md`.
- [ ] Cross-link con la sección web de `clientes/gpt-landings.md`.

## Handoff

- [ ] Confirmar a E y F que reciben los eventos.
- [ ] Escribir `retro.md`.
- [ ] Promover `n8n/patterns/checklist-intake-state.md` si generaliza.
- [ ] Follow-up: crear `n8n/nodes/google-drive.md` con los gotchas de upload (desde este retro).
- [ ] Append one-liner a `n8n/lessons-learned.md`.
