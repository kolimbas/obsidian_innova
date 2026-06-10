---
tags:
  - n8n
  - research
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: approval-and-esign
updated: 2026-06-10
---

# Research — C · Approval & e-sign

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/approval-and-esign/spec|Spec]]

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/gpt-landings/flows/` (2026-06-10).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `term-sheet-generation` (B) | 🟢 Alta | Provee el PDF a firmar. |
| `drive-folder-structure` (G) | 🟢 Alta | Se dispara al pasar el préstamo a `processing`. |
| `partner-matching-engine` (A) | 🟡 Media | Contexto del préstamo. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, other `n8n/clients/*/flows/` (2026-06-10).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/patterns/hubspot-workflow-webhook-trigger.md` | Webhook externo + `Normalize` defensivo + dedup por `event_id` | **Aplica igual** al webhook de DocuSign/PandaDoc: recibir "firmado", normalizar, deduplicar. |
| `n8n/patterns/sheet-idempotency.md` | Dedup lookup-then-insert | Idempotencia del evento "firmado" (no pasar a `processing` dos veces). |
| `n8n/clients/blincer/flows/credit-limit-invoice-block/` | Webhook → lookup → side-effect + Note + alerta interna | Shape general (webhook → cambiar estado + notificar). |

## 3. n8n nodes considered

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.httpRequest` | Enviar a firma vía e-sign API | endpoint, auth, envelope | depende OQ-C-1 (DocuSign vs PandaDoc) |
| `n8n-nodes-base.webhook` + `code` (Normalize) | Recibir "firmado" | path `gptlandings-esign` | normalizar payload del provider (ver pattern) |
| `n8n-nodes-base.postgres` | Cambiar estado a `processing` | update por `loan_id` | idempotente por `event_id` |
| email / canal interno | Notificar a `processing@` | — | canal OQ-0.4 |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| DocuSign | OAuth/JWT (si aplica) | por plan | webhook con event id | https://developers.docusign.com |
| PandaDoc | API key | por plan | webhook con event id | https://developers.pandadoc.com |
| Supabase/Postgres | conn string | n/a | upsert + dedup | https://supabase.com/docs |

## 5. Base flow decision

- **Base flow chosen:** parcial — `credit-limit-invoice-block` (Blincer) por el shape "webhook → cambiar estado + notificar" y `hubspot-workflow-webhook-trigger` por la dedup.
- **Coverage estimate:** ~35% (estructura webhook+dedup+estado; cambia el sistema externo a e-sign).
- **What we copy / what we change:** copiamos webhook idempotente + cambio de estado + alerta; cambiamos HubSpot por e-sign.

## 6. Reuse summary

- ✅ Reusing: pattern `hubspot-workflow-webhook-trigger` (webhook + Normalize + dedup), `sheet-idempotency`, shape de `credit-limit-invoice-block`.
- 🆕 New work: integración con la e-sign elegida (envío + parsing del webhook); estado `processing` + disparo de G.
- ⚠️ Risks identified: provider sin decidir (OQ-C-1) bloquea el envío; canal de aprobación sin definir (OQ-C-2); webhook sin `event_id` → idempotencia colapsa a `(loan_id, status)`.
