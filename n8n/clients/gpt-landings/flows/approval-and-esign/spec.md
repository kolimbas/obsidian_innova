---
tags:
  - n8n
  - spec
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: approval-and-esign
status: draft
updated: 2026-06-10
---

# Spec — C · Approval & e-sign

> The manager approves a term sheet → it goes out for digital signature → on "signed" the loan moves to `processing` and the team is notified.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/README|GPT Landings]]

---

## Goal

Take an approved term sheet, send it for signature via the e-sign platform (DocuSign/PandaDoc) through their API, and react to the "envelope completed / signed" webhook by notifying `processing@` and creating the loan record in state `processing` (which in turn triggers Module G — folder structure).

## Trigger

- Type: webhook (two-stage)
- Schedule / endpoint / event: (1) manager approval (canal TBD) → send-to-sign; (2) `POST /webhook/gptlandings-esign` ← e-sign "envelope completed"
- Expected frequency: por término aprobado (~10-15/mes)

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `loan_id` | string | B / DB | yes | préstamo |
| `term_sheet_pdf` | ref | Drive (de B) | yes | documento a firmar |
| `approver` | string | approval action | yes | quién aprobó (trazabilidad) |
| `esign_event` | object | e-sign webhook | yes | `event_id`, `envelope_id`, `status` |

Sample payload (e-sign webhook, abreviado):

```json
{
  "event": "envelope-completed",
  "event_id": "evt_8f3a...",
  "envelope_id": "env_123",
  "loan_id": "L-2026-0042",
  "status": "completed",
  "completed_at": 1781100000
}
```

## Outputs / Side effects

- e-sign: envelope/documento enviado a firma (vía API).
- DB: al firmar → `prestamos.estado = 'processing'` (+ `signed_at`, `envelope_id`).
- Notifies: email a `processing@` (o canal OQ-0.4) — "préstamo {loan_id} firmado, arrancar processing".
- Hands off to: [[n8n/clients/gpt-landings/flows/drive-folder-structure/spec|drive-folder-structure]] (G) al pasar a `processing`.

## Success criteria

- [ ] Sólo term sheets **aprobados** se mandan a firma.
- [ ] El webhook "firmado" → estado `processing` + notificación, **idempotente** ante reentrega (`event_id`).
- [ ] Trazabilidad: queda registrado quién aprobó, cuándo, y el `envelope_id`.
- [ ] Al pasar a `processing`, se dispara G (folders).

## Out of scope

- Generación del term sheet (Módulo B).
- Intake de documentación (Módulo D).
- Lógica de límite de crédito / underwriting (humano).

## Open questions

- [ ] **OQ-C-1** — **DocuSign vs PandaDoc** + cuentas/API/scopes. 🚦 *(def #3)*
- [ ] **OQ-C-2** — **Quién aprueba** term sheets y por qué canal (email/portal/Slack/WhatsApp). ⚠️ *(def #7)*
- [ ] **OQ-C-3** — Idempotencia del webhook de la e-sign: ¿trae `event_id` único?

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Manager/owner GPT Landings |
| Approver | Manager/owner (aprueba el term sheet) |
| End user | Equipo de processing (recibe el aviso) + borrower (firma) |
