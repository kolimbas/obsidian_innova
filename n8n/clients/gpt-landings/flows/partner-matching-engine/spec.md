---
tags:
  - n8n
  - spec
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: partner-matching-engine
status: draft
updated: 2026-06-10
---

# Spec — A · Capital-partner matching engine

> Given a loan application, return which capital partners it qualifies for, with the reason — hard deterministic rules first, Claude only for the fuzzy cases.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/README|GPT Landings]]

---

## Goal

When a loan is loaded (from Excel/CSV, email or the intake form), normalize it to a single schema and evaluate it against each capital partner's guidelines. Output a structured JSON listing the partners it qualifies for (with rationale and a flag for cases resolved by Claude rather than a hard rule). That JSON feeds Module B (term-sheet generation).

## Trigger

- Type: webhook / form (loan intake) — or a new row in `prestamos`
- Schedule / endpoint / event: `POST /webhook/gptlandings-loan-intake` (path estable) o evento de D
- Expected frequency: ~10-15 loans/month (def #8)

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `loan_application` | object | webhook/form/email | yes | monto, LTV, plaza/condado, tipo de propiedad, tipo de loan, borrower |
| `raw_format` | enum | source | yes | `excel\|csv\|email\|form` → normalizar a schema único |
| `capital_partners` | list | DB `capital_partners` | yes | cada uno con sus `guidelines` (reglas) |
| `guidelines` | object | DB / Sheet (editable) | yes | tipo de propiedad, condado/geografía, monto min-max, LTV, tipo de loan, etc. |

Sample payload:

```json
{
  "loan_id": "L-2026-0042",
  "raw_format": "form",
  "loan_application": {
    "amount_usd": 450000,
    "ltv": 0.65,
    "property_type": "SFR",
    "county": "Miami-Dade",
    "loan_type": "fix-and-flip",
    "borrower_entity": "Acme Holdings LLC"
  }
}
```

## Outputs / Side effects

- Writes to: `prestamos.partner_match_json` (upsert por `loan_id`) — `{loan_id, qualifying_partners:[{partner_id, rationale, hard_pass:bool, llm_flag:bool}]}`.
- Other state changes: `estado_checklist` puede inicializarse según el tipo de loan (slots requeridos).
- Hands off to: [[n8n/clients/gpt-landings/flows/term-sheet-generation/spec|term-sheet-generation]] (input contract).

## Success criteria

- [ ] Input en cualquiera de los 4 formatos → schema único sin pérdida de campos clave.
- [ ] Cada partner evaluado contra sus reglas duras de forma **determinista y auditable** (queda el porqué).
- [ ] Casos borrosos marcados `llm_flag: true` con explicación de Claude; **nunca** auto-aprobados como hard pass.
- [ ] El JSON de salida es válido y consumible por B (contrato estable).

## Out of scope

- Generar el term sheet (Módulo B).
- Aprobación humana / envío a firma (Módulo C).
- Mantener/editar las guidelines (las edita el cliente en su fuente; acá solo se leen).

## Open questions

- [ ] **OQ-A-1** — Formato y **cantidad** de guidelines por partner (¿PDF/Excel/planilla/cabeza?). 🚦 *(def #1)*
- [ ] **OQ-A-2** — Estabilidad de los parámetros (¿cambian seguido? ¿quién los edita?). ⚠️
- [ ] **OQ-A-3** — Dónde viven hoy las guidelines y de dónde las leemos (DB editable vs Sheet).
- [ ] **OQ-A-4** — Schema canónico del préstamo (campos obligatorios).
- [ ] **OQ-A-5** — Budget/modelo Claude para casos borrosos. ⚠️ *(ref def #2 de costo)*

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Manager/owner GPT Landings |
| Approver | Manager/owner |
| End user | Equipo de originación (ve a qué partners califica) + Módulo B (consume el JSON) |
