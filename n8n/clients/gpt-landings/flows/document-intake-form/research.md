---
tags:
  - n8n
  - research
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: document-intake-form
updated: 2026-06-10
---

# Research — D · Document intake (backend)

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/document-intake-form/spec|Spec]]

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/gpt-landings/flows/` (2026-06-10).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `m0-infra-setup` | 🟢 Alta | DB (`documentos_requeridos`, `estado_checklist`) + Drive creds. |
| `whatsapp-doc-reminder` (E) | 🟢 Alta | Consume el estado del checklist para recordar. |
| `track-record-validation` (F) | 🟢 Alta | Se dispara con cada doc nuevo. |
| `drive-folder-structure` (G) | 🟡 Media | Los archivos aterrizan en la estructura que arma G. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, other `n8n/clients/*/flows/` (2026-06-10).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/patterns/sheet-idempotency.md` | Estado por ítem sin duplicar | `estado_checklist`: una fila por slot, re-subida pisa, no duplica. |
| `n8n/nodes/google-sheets.md` | Gotcha v4.5 sin `lookup` | Si el checklist se respaldara en Sheets (en vez de Postgres). |
| `agentes/subagente-pre-contratacion.md` | Subir archivos a Drive con service account | Mecánica de upload + carpeta por entidad. |

## 3. n8n nodes considered

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.webhook` | Recibir el POST del form | path `gptlandings-intake`, binary | uploads grandes → considerar resumable / pre-signed URLs (no pasar todo por n8n) |
| `n8n-nodes-base.code` | Mapear doc→slot por `loan_type` | JS | lista de slots desde DB (def #4) |
| `n8n-nodes-base.googleDrive` / Supabase Storage | Guardar archivos | folderId / bucket | OQ-D-3 |
| `n8n-nodes-base.postgres` | Upsert `estado_checklist` | onConflict (loan_id, slot) | — |
| event/`executeWorkflow` | Disparar E y F | — | — |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| Web form (Next.js, Vercel) | borrower auth (OQ-D-4) | n/a | n/a | Innova web line |
| Google Drive / Supabase Storage | service account / service role | 1000 req/100s | n/a | docs respectivas |
| Supabase/Postgres | conn string | n/a | upsert (loan_id, slot) | https://supabase.com/docs |

## 5. Base flow decision

- **Base flow chosen:** `none` (no hay intake form en el vault).
- **Coverage estimate:** ~10% (idempotencia + Drive upload).
- **What we copy / what we change:** copiamos idempotencia por ítem + upload Drive; el backend de intake con checklist es nuevo y queda como base de un futuro pattern `checklist-intake-state`.

## 6. Reuse summary

- ✅ Reusing: `sheet-idempotency` (estado por ítem), upload Drive del subagente pre-contratación, DB de M0.
- 🆕 New work: el front-end (web, fuera de este bundle), el mapeo doc→slot por tipo, manejo de uploads pesados (resumable / pre-signed URLs).
- ⚠️ Risks identified: archivos pesados por n8n (mejor pre-signed URL directo a storage); lista de docs sin cerrar (def #4); auth del borrower sin definir (OQ-D-4).
