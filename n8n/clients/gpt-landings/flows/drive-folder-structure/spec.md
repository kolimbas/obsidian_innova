---
tags:
  - n8n
  - spec
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: drive-folder-structure
status: draft
updated: 2026-06-10
---

# Spec — G · Drive folder structure

> When a loan is created (moves to `processing`), build the standard Google Drive folder tree and classify incoming documents into it.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/README|GPT Landings]]

---

## Goal

On loan creation (state `processing`, triggered by C), create the standard per-loan folder structure in Google Drive (`/property`, `/closing`, `/borrower`), apply a standardized naming convention, and classify each incoming document into the right subfolder by detected type.

## Trigger

- Type: event
- Schedule / endpoint / event: cuando `prestamos.estado` pasa a `processing` (evento de C); también por doc entrante (de D) para clasificar
- Expected frequency: 1× por préstamo (creación) + por doc entrante (clasificación)

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `loan_id` | string | C / DB | yes | préstamo |
| `loan_meta` | object | DB `prestamos` | yes | datos para nombrar la carpeta |
| `incoming_doc` | ref | D | no | doc a clasificar (modo clasificación) |
| `doc_type` | string | D / detección | no | a qué subcarpeta va |

## Outputs / Side effects

- Google Drive: árbol de carpetas creado bajo la carpeta del préstamo (`/property`, `/closing`, `/borrower`).
- Google Drive: docs entrantes movidos/renombrados a su subcarpeta según tipo.
- DB: referencia del `folder_id` del préstamo guardada en `prestamos`.

## Success criteria

- [ ] Estructura `/property`, `/closing`, `/borrower` creada **idempotentemente** (no duplica si ya existe).
- [ ] Cada doc clasificado en su subcarpeta con nombre estándar.
- [ ] Corre desde n8n/Python con la service account de M0.
- [ ] El `folder_id` del préstamo queda referenciado en DB.

## Out of scope

- Validación de contenido del doc (Módulo F).
- El upload inicial (Módulo D — G organiza lo que D persiste).

## Open questions

- [ ] **OQ-G-1** — Nomenclatura exacta de carpetas/archivos (default `/property` `/closing` `/borrower`).
- [ ] **OQ-G-2** — ¿La estructura cambia por tipo de préstamo?
- [ ] **OQ-G-3** — Permisos/sharing de las carpetas (equipo / borrower / partner).

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Manager/owner GPT Landings |
| Approver | Manager/owner (nomenclatura) |
| End user | Equipo (navega las carpetas del préstamo) |
