---
tags:
  - n8n
  - spec
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: document-intake-form
status: draft
updated: 2026-06-10
---

# Spec — D · Document intake (backend)

> Receive the documents uploaded from the web intake form, map them to per-loan-type slots, store them, and keep a per-item checklist that drives the reminders (E) and validation (F).

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/README|GPT Landings]]

> [!note] Backend only
> This bundle is the **n8n backend** of Module D. The **front-end intake form** (Next.js/React) is a web deliverable documented in `clientes/gpt-landings.md` § "Entregable web — Formulario de intake".

---

## Goal

Back the smart intake form: accept uploaded documents per loan, map each to its required slot (PFS, Track Record, credit report, bank statements, …) by loan type, persist files to Drive/Supabase, maintain real-time per-item state in `estado_checklist`, and emit events that trigger Module E (reminders) and Module F (validation).

## Trigger

- Type: webhook
- Schedule / endpoint / event: `POST /webhook/gptlandings-intake` ← web form (Next.js)
- Expected frequency: por subida del borrower (varias por préstamo)

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `loan_id` | string | form | yes | préstamo al que pertenece |
| `files` | multipart | form | yes | uploads (multipart/resumable, archivos pesados) |
| `doc_slot` | string | form | yes | a qué slot mapea cada archivo (PFS/track-record/…) |
| `loan_type` | string | DB `prestamos` | yes | determina el checklist de slots requeridos |
| `borrower_meta` | object | form | no | datos del borrower |

Sample payload:

```json
{
  "loan_id": "L-2026-0042",
  "loan_type": "fix-and-flip",
  "uploads": [
    {"doc_slot": "pfs", "filename": "pfs.pdf", "size": 2480000},
    {"doc_slot": "bank_statements", "filename": "bank.pdf", "size": 5120000}
  ]
}
```

## Outputs / Side effects

- Writes to: Google Drive (carpeta del préstamo) o Supabase Storage — los archivos subidos.
- Writes to: `documentos_requeridos` / `estado_checklist` — una fila por slot con `status` (`pending` → `received` → `validated`).
- Emits: evento a [[n8n/clients/gpt-landings/flows/whatsapp-doc-reminder/spec|E]] (estado del checklist) y a [[n8n/clients/gpt-landings/flows/track-record-validation/spec|F]] (doc nuevo a validar).

## Success criteria

- [ ] Cada doc subido aterriza en su slot correcto en Drive/Supabase **y** en DB.
- [ ] El checklist por préstamo refleja completo/incompleto en tiempo real.
- [ ] Uploads pesados (multipart/resumable) no se truncan.
- [ ] Idempotencia: re-subir el mismo archivo al mismo slot no duplica la fila de estado.

## Out of scope

- **El front-end del formulario** (entregable web — ver `clientes/gpt-landings.md`).
- La validación del **contenido** del documento (Módulo F).
- Los recordatorios al borrower (Módulo E).

## Open questions

- [ ] **OQ-D-1** — **Lista exacta de documentos por tipo** de préstamo. 🚦 *(def #4)*
- [ ] **OQ-D-2** — ¿La lista cambia por partner/tipo de loan?
- [ ] **OQ-D-3** — Tamaños máximos / storage (Google Drive vs Supabase Storage).
- [ ] **OQ-D-4** — Dónde corre el front (Vercel) y cómo autentica al borrower (link único por préstamo vs login). ⚠️

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Manager/owner GPT Landings |
| Approver | Manager/owner (define la lista de docs) |
| End user | Borrower (sube docs) + equipo (ve el checklist) |
