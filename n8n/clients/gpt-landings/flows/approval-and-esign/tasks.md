---
tags:
  - n8n
  - tasks
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: approval-and-esign
updated: 2026-06-10
status: blocked-by-oqs
---

# Tasks — C · Approval & e-sign

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/approval-and-esign/plan|Plan]]

> Ordered, verifiable. No empezar Build sin provider de e-sign decidido y B entregando el PDF.

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] OQ-C-1: decidir DocuSign vs PandaDoc + confirmar cuenta con acceso a API/scopes (def #3 🚦).
- [ ] OQ-C-2: definir quién aprueba y por qué canal (def #7).
- [ ] OQ-C-3: confirmar si el webhook trae `event_id` único.
- [ ] B entregando el PDF del term sheet; M0 (DB + canal interno).
- [ ] Crear la app/integración en la e-sign + configurar el webhook a `/webhook/gptlandings-esign`.

## Setup

- [ ] Credenciales `gptlandings-esign`, `gptlandings-db`, `gptlandings-internal-alert`.
- [ ] Store de dedup (`idempotency_esign`).
- [ ] Test payloads en `./test-payloads/`: `esign_send.json`, `esign_signed_webhook.json`, `esign_duplicate_webhook.json`.

## Build

- [ ] Crear workflow `gpt-landings / approval-and-esign`.
- [ ] Node 1: intake de aprobación (canal OQ-C-2).
- [ ] Node 2: HTTP — crear envelope en la e-sign con el PDF de B.
- [ ] Node 3: Postgres — guardar `envelope_id` + `approver`.
- [ ] Node 4: Webhook `gptlandings-esign` (firmado).
- [ ] Node 5: Code — Normalize del payload del provider.
- [ ] Node 6: Dedup por `event_id` (patrón sheet-idempotency).
- [ ] Node 7: Postgres — `estado='processing'` + `signed_at`.
- [ ] Node 8: Notificar a `processing@` / canal interno.
- [ ] Node 9: disparar `drive-folder-structure` (G).
- [ ] `mcp__n8n-mcp__validate_node` + `n8n_validate_workflow`.

## Validate

- [ ] `esign_send.json` → envelope creado, `envelope_id` guardado.
- [ ] `esign_signed_webhook.json` → estado `processing`, notificación enviada, G disparado.
- [ ] `esign_duplicate_webhook.json` (reentrega) → no duplica estado ni notificación (idempotente).
- [ ] Aprobación ausente → no se manda a firma.
- [ ] Assert cada success criterion del spec.

## Harden

- [ ] Retries en e-sign/DB/alertas.
- [ ] Alerta crítica si firmado pero falla el cambio de estado.
- [ ] Validar firma del webhook (secret/HMAC del provider) si está disponible.

## Document

- [ ] Exportar `workflow.json`.
- [ ] `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/gpt-landings/README.md`.

## Handoff

- [ ] Notificar al equipo de processing cómo se ve el aviso.
- [ ] Escribir `retro.md`.
- [ ] Promover `n8n/patterns/esign-webhook-idempotent.md` (variante de hubspot-workflow-webhook-trigger).
- [ ] Append one-liner a `n8n/lessons-learned.md`.
