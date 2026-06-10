---
tags:
  - n8n
  - spec
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: term-sheet-generation
status: draft
updated: 2026-06-10
---

# Spec — B · Term-sheet generation

> For each capital partner the loan qualifies for, generate a term sheet (HTML→PDF), with rate and amount editable before it goes out.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/README|GPT Landings]]

---

## Goal

Consume the `qualifying_partners` JSON produced by Module A and, for each qualifying partner, render a formal term sheet (PDF) populated with the loan and partner data. Rate and amount are editable defaults before emission. The output is ready for Module C (approval & e-sign).

## Trigger

- Type: called from A (sub-workflow / event)
- Schedule / endpoint / event: recibe el JSON `qualifying_partners` de `partner-matching-engine`
- Expected frequency: por préstamo que tenga ≥1 partner que califica (~10-15/mes × partners)

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `loan_id` | string | A | yes | clave del préstamo |
| `qualifying_partners` | list | A (`prestamos.partner_match_json`) | yes | uno o más partners |
| `term_sheet_template` | doc | DB/Drive | yes | template parametrizable (campos exactos, OQ-B-1) |
| `rate` / `amount` | number | default config + edición manual | yes | el lender pone el número final |

Sample payload:

```json
{
  "loan_id": "L-2026-0042",
  "qualifying_partners": [
    {"partner_id": "P-01", "rationale": "LTV 0.65 ≤ 0.70; Miami-Dade OK", "hard_pass": true}
  ],
  "defaults": {"rate": 0.115, "amount_usd": 450000}
}
```

## Outputs / Side effects

- Writes to: Google Drive (carpeta del préstamo) — un PDF por partner que califica.
- Other state changes: referencia del PDF + `valid_until` guardada en `prestamos` (o `documentos_requeridos`).
- Hands off to: [[n8n/clients/gpt-landings/flows/approval-and-esign/spec|approval-and-esign]] (term sheet listo para aprobar/enviar).

## Success criteria

- [ ] Un term sheet por partner que califica, con datos del préstamo y del partner correctos.
- [ ] Tasa y monto editables antes de emitir (defaults configurables).
- [ ] PDF reproducible: mismo input → mismo documento.
- [ ] El PDF queda guardado en Drive y referenciado en el préstamo.

## Out of scope

- El motor de matching (Módulo A).
- El envío a firma / aprobación (Módulo C).
- El branding/diseño visual fino más allá del template provisto.

## Open questions

- [ ] **OQ-B-1** — Template y **campos exactos** del term sheet + ejemplo real. 🚦 *(def #2)*
- [ ] **OQ-B-2** — HTML→PDF propio (Playwright/Gotenberg) vs template nativo de la e-sign. ⚠️ *(depende de OQ-C-1)*
- [ ] **OQ-B-3** — Branding/datos fiscales: ¿de GPT Landings, del partner, o ambos?

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Manager/owner GPT Landings |
| Approver | Manager/owner (define template) |
| End user | El que revisa/edita tasa y monto + Módulo C |
