---
tags:
  - n8n
  - tasks
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: term-sheet-generation
updated: 2026-06-10
status: blocked-by-oqs
---

# Tasks — B · Term-sheet generation

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/term-sheet-generation/plan|Plan]]

> Ordered, verifiable. No empezar Build sin el template real (def #2) y A entregando el JSON.

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] OQ-B-1: conseguir template del term sheet + campos exactos + ejemplo real (def #2 🚦).
- [ ] OQ-B-2: decidir HTML→PDF propio vs template nativo de la e-sign (depende de OQ-C-1).
- [ ] OQ-B-3: definir branding/datos fiscales (GPT Landings / partner / ambos).
- [ ] A entregando `qualifying_partners` estable + M0 (Drive + Gotenberg/Playwright).

## Setup

- [ ] Credenciales `gptlandings-drive`, `gptlandings-db` + endpoint Gotenberg en env.
- [ ] Cargar el template HTML del term sheet (parametrizable).
- [ ] Test payloads en `./test-payloads/`: `tsheet_single_partner.json`, `tsheet_multi_partner.json`, `tsheet_edit_rate.json`.

## Build

- [ ] Crear workflow `gpt-landings / term-sheet-generation`.
- [ ] Node 1: recibir input de A (`loan_id`, `qualifying_partners`, defaults).
- [ ] Node 2: SplitInBatches por partner.
- [ ] Node 3: Code — validar que el partner venga del JSON de A.
- [ ] Node 4: Code — bind del template HTML con datos + tasa/monto.
- [ ] Node 5: HTTP Gotenberg — HTML→PDF.
- [ ] Node 6: Google Drive — upload a la carpeta del préstamo (nombre determinista).
- [ ] Node 7: Postgres — guardar `pdf_url` + `valid_until`.
- [ ] Node 8: marcar listo para C.
- [ ] `mcp__n8n-mcp__validate_node` + `n8n_validate_workflow`.

## Validate

- [ ] `tsheet_single_partner.json` → 1 PDF correcto en Drive + referencia en DB.
- [ ] `tsheet_multi_partner.json` → N PDFs, uno por partner.
- [ ] `tsheet_edit_rate.json` → re-render pisa el PDF anterior (no duplica).
- [ ] Mismo input → mismo PDF (reproducible).
- [ ] Assert cada success criterion del spec.

## Harden

- [ ] Retries en Gotenberg/Drive/DB.
- [ ] Permisos de la carpeta de Drive acotados.
- [ ] Manejo de fallo de render de un partner sin cortar el loop.

## Document

- [ ] Exportar `workflow.json`.
- [ ] `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/gpt-landings/README.md`.

## Handoff

- [ ] Confirmar a C que el PDF + `valid_until` están disponibles.
- [ ] Escribir `retro.md`.
- [ ] Promover `n8n/patterns/html-to-pdf-drive.md` si generaliza (compartible con futuros docs).
- [ ] Append one-liner a `n8n/lessons-learned.md`.
