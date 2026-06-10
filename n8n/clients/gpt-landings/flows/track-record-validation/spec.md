---
tags:
  - n8n
  - spec
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: track-record-validation
status: draft
updated: 2026-06-10
---

# Spec — F · Track-record validation & cross-check

> [!warning] CARVE-OUT CONDICIONAL
> **Este módulo NO está incluido en el precio cerrado de Fase 1.** Entra como **adicional condicionado** a que **Elementix exponga API** (OQ-F-1 / discovery def #6 — la pregunta más importante). Documentado para scoping; **no comprometer alcance ni precio** hasta validar factibilidad técnica.

> Validate each document as it arrives (OCR + Claude) and cross-check the declared track record against public records / Elementix, flagging anything that doesn't match. Final validation stays human.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/README|GPT Landings]]

---

## Goal

As documents arrive (from D), extract their data (OCR + Claude over PFS, track record, bank statements) and cross-check the declared track record against public records / Elementix — verifying the declared entity actually owned and signed at the addresses the borrower claims. Raise automatic flags on mismatches. A human makes the final call; the system only surfaces and prepares.

## Trigger

- Type: event (per incoming document)
- Schedule / endpoint / event: evento de D (`document-intake-form`) por cada doc recibido
- Expected frequency: por doc subido (varios por préstamo)

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `loan_id` | string | D | yes | préstamo |
| `document` | ref | Drive/Supabase (de D) | yes | el doc a validar |
| `doc_slot` | string | D | yes | PFS / track-record / bank-statements / … |
| `declared_track_record` | object | form / DB | yes | entidades + direcciones declaradas |
| `public_records_source` | api/ui | Elementix / otros | yes | **OQ-F-1: API o solo UI** |

## Outputs / Side effects

- Writes to: `estado_checklist` (o tabla de validación) — por doc: `{validation_status, flags[], extracted_fields}`.
- Surfaces: resumen para revisión humana (qué matchea / qué no).
- Notifies: flags automáticos al equipo cuando algo no coincide (canal OQ-0.4).

## Success criteria

- [ ] Cada doc validado al llegar con OCR + Claude, con campos extraídos.
- [ ] Cross-check del track record produce flags **trazables** (qué se comparó, contra qué fuente, resultado).
- [ ] **Toda** auto-validación pasa por **revisión humana final** antes de afectar el préstamo.
- [ ] (Condicional a OQ-F-1) el cross-check contra Elementix/public records funciona end-to-end.

## Out of scope

- La decisión final de aprobación (humana).
- El intake en sí (Módulo D).
- Cualquier compromiso de alcance/precio antes de validar OQ-F-1 (carve-out).

## Open questions

- [ ] **OQ-F-1** — **¿Elementix expone API o es solo UI?** + qué fuentes de public records y si son accesibles por API. 🚦🚦 *(def #6, la más importante — define si F entra como adicional o queda fuera)*
- [ ] **OQ-F-2** — Si Elementix es solo UI: ¿scraping (frágil) o entrada asistida manual?
- [ ] **OQ-F-3** — Modelo OCR + budget Claude para el volumen de docs.
- [ ] **OQ-F-4** — Criterios exactos de "match" entidad↔dirección (qué dispara un flag).

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Manager/owner GPT Landings |
| Approver | Manager/owner (decide si F entra y a qué precio) |
| End user | Analista que revisa los flags (validación humana final) |
