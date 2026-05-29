---
tags:
  - n8n
  - research
  - blincer
  - nivel-3
client: blincer
flow: whatsapp-overdue-debt-reminder
updated: 2026-05-29
---

# Research — Avisos automáticos de deuda vencida por WhatsApp

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/spec|Spec]]

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/blincer/flows/` (2026-05-29).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `credit-limit-invoice-block` | 🟡 Media — comparte HubSpot (Company + tango_customer_id), credencial Tango y canal de alerta interna. No comparte payload ni trigger. | Reusar credencial + mapeo Tango/HubSpot + pattern de internal-alert. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, otros `n8n/clients/*/flows/` (2026-05-29).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/patterns/_index.md` | (vacío) | Promover `cron → fetch → loop → send-message-with-idempotency` cuando esté `live`. |
| Travel agency externa (memoria: `project_auditoria_presupuestos.md`) | Loop unificado con OCR + clasificación; gestión de fallbacks por canal | Reusar el shape "loop con fallback por canal" si OQ-3 va por A (email fallback). |

## 3. n8n nodes considered

A validar con `mcp__n8n-mcp__search_nodes` / `get_node` durante el build. Pre-selección:

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.scheduleTrigger` | Cron 09:00 ART | cron expression, timezone | Confirmar timezone n8n instance = ART (America/Argentina/Buenos_Aires). |
| `n8n-nodes-base.httpRequest` (Tango) o connector local | Fetch facturas vencidas | URL, auth, query | Tango Nexo response shape a documentar en `n8n/nodes/tango-nexo.md` al promover. |
| `n8n-nodes-base.googleSheets` | Leer config + templates + log | Sheet ID, range | Mantener Sheets como source para que Sandra edite sin tocar n8n. |
| `n8n-nodes-base.hubspot` | Get Contact por `tango_customer_id`, sumar timeline event | resource, operation | Para timeline event custom hace falta CRM Card / Custom Events API — confirmar scopes. |
| `n8n-nodes-base.itemLists` + `splitInBatches` | Loop por factura, throttle de envíos | batchSize | Throttle obligatorio para no gatillar rate-limit de WhatsApp provider. |
| **WhatsApp (Evolution)** vía `httpRequest` o nodo community | Enviar mensaje | endpoint + token + message body | Evolution API: si no hay nodo nativo, usar HTTP. Riesgo de baneo si se hace spam. |
| **WhatsApp (Cloud API)** vía nodo oficial Meta | Enviar mensaje | phoneNumberId + token + template name | Cloud API exige `template_name` pre-aprobado por Meta para mensajes outbound fuera de la ventana de 24h. Templates requieren submission + aprobación. |
| `n8n-nodes-base.if` | Skip si idempotency match | condition | — |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| Tango Nexo / local | depende OQ-G1 | n/a / desconocido | n/a | depende |
| HubSpot Pro+ | Private App | 100/10s | upsert por contact id | https://developers.hubspot.com |
| WhatsApp Evolution API (self-hosted) | API key local | depende del host; típico 60–100 msg/min sin baneo | manual (vía clientMessageId del SDK) | https://doc.evolution-api.com |
| WhatsApp Cloud API (Meta) | Bearer (system user token) | 250 conversaciones/24h Tier 1; sube con quality | message_id idempotencia parcial | https://developers.facebook.com/docs/whatsapp/cloud-api |
| Google Sheets | OAuth / service account | 60 req/min/user | n/a | https://developers.google.com/sheets/api |

## 5. Base flow decision

- **Base flow chosen:** `none` (no hay flows previos con cron + WhatsApp en el vault).
- **Coverage estimate:** 0%
- **What we copy / what we change:** este flow va a quedar como **base** del pattern `cron-bulk-whatsapp` para futuros flows de cobranza/notificación de otros clientes.

## 6. Reuse summary

- ✅ Reusing:
  - Credencial HubSpot `hubspot-blincer-main` (creada en flow 1).
  - Credencial Tango `tango-nexo-blincer` o local (creada en flow 1).
  - Pattern `internal-alert` para resumen a Sandra (compartido con flow 1).
  - Mapeo `tango_customer_id` ya backfilleado por el setup de flow 1.
- 🆕 New work:
  - Credencial WhatsApp provider (depende OQ-1).
  - Sheet `cobranzas_config`, `cobranzas_templates`, `cobranzas_log`, `cobranzas_fallback`.
  - Custom property HubSpot Company `last_dunning_at`.
  - Custom timeline event HubSpot "Recordatorio cobranza" (requiere CRM Cards API).
- ⚠️ Risks identified:
  - **Baneo WhatsApp:** crítico si se usa Evolution sin opt-in claro. Mitigación: requerir confirmación de opt-in en spec antes de live.
  - **Cloud API templates:** si vamos por la oficial, hay un step previo de submission + aprobación de Meta (~1–2 días) por cada texto distinto. Eso choca con OQ-6 (templates editables por Sandra).
  - **Timezone mismatch:** si el n8n no está en ART, los envíos pueden caer fuera de banda. Validar antes de build.
  - **Pago entre fetch y envío:** cliente paga a las 09:01, el flow envía a las 09:03 → mensaje incorrecto. Mitigación: re-validar saldo justo antes del envío.
