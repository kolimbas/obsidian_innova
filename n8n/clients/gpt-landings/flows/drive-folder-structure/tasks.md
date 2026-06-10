---
tags:
  - n8n
  - tasks
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: drive-folder-structure
updated: 2026-06-10
status: blocked-by-oqs
---

# Tasks — G · Drive folder structure

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/drive-folder-structure/plan|Plan]]

> Ordered, verifiable. Bajo riesgo. No empezar Build sin la service account de Drive (M0).

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] OQ-G-1: confirmar nomenclatura de carpetas/archivos (default `/property` `/closing` `/borrower`).
- [ ] OQ-G-2: ¿la estructura cambia por tipo de préstamo?
- [ ] OQ-G-3: permisos/sharing de las carpetas.
- [ ] M0: service account Drive con permisos de crear/mover.
- [ ] C disparando `processing`; D entregando docs.

## Setup

- [ ] Credencial `gptlandings-drive` + `gptlandings-db`.
- [ ] Definir el mapa tipo-de-doc → subcarpeta.
- [ ] Test payloads en `./test-payloads/`: `loan_new.json`, `loan_existing.json`, `doc_classify.json`, `doc_unknown_type.json`.

## Build

- [ ] Crear workflow `gpt-landings / drive-folder-structure`.
- [ ] Node 1: trigger por `loan → processing`.
- [ ] Node 2: Google Drive — buscar carpeta del préstamo (IF existe).
- [ ] Node 3: Google Drive — crear `/loan_id` + `/property` `/closing` `/borrower`.
- [ ] Node 4: Postgres — guardar `folder_id`.
- [ ] Node 5: trigger por doc entrante (de D).
- [ ] Node 6: Code — clasificar tipo → subcarpeta (default `/borrower` si desconocido).
- [ ] Node 7: Google Drive — mover/renombrar a la subcarpeta.
- [ ] `mcp__n8n-mcp__validate_node` + `n8n_validate_workflow`.

## Validate

- [ ] `loan_new.json` → árbol creado, `folder_id` guardado.
- [ ] `loan_existing.json` → no recrea (idempotente).
- [ ] `doc_classify.json` → archivo en la subcarpeta correcta con nombre estándar.
- [ ] `doc_unknown_type.json` → va a default + flag.
- [ ] Re-disparar G → no duplica carpetas ni mueve dos veces.
- [ ] Assert cada success criterion del spec.

## Harden

- [ ] Chequear-antes-de-crear en todas las carpetas.
- [ ] Permisos de carpeta privados por defecto + sharing explícito.
- [ ] Retries en Drive/DB.

## Document

- [ ] Exportar `workflow.json`.
- [ ] `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/gpt-landings/README.md`.

## Handoff

- [ ] Mostrar al equipo cómo quedan organizadas las carpetas.
- [ ] Escribir `retro.md`.
- [ ] Promover `n8n/patterns/drive-folder-per-entity.md` (con el subagente pre-contratación como precedente) + follow-up `n8n/nodes/google-drive.md`.
- [ ] Append one-liner a `n8n/lessons-learned.md`.
