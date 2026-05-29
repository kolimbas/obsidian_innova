---
tags:
  - n8n
  - research
  - blincer
  - nivel-3
client: blincer
flow: email-remarketing
updated: 2026-05-29
---

# Research — Remarketing y difusiones por email

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/flows/email-remarketing/spec|Spec]]

---

## 1. Prior flows of this client

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `credit-limit-invoice-block` | 🟢 Baja — solo comparte credencial HubSpot. | — |
| `whatsapp-overdue-debt-reminder` | 🟢 Baja — pattern de cadencia y throttle es reaplicable mentalmente. | Reusar concepto de `daily_cap` y `send_window`. |
| `sales-bot-with-quotes` | 🟡 Media — el bot crea Deals que este flow puede segmentar (ej. "cotización enviada hace 7 días sin compra"). | Coordinar custom props del Deal. |

## 2. Cross-client patterns

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/patterns/_index.md` | (vacío) | Promover `bulk-email-send-with-tracking` cuando esté `live`. |

## 3. n8n nodes considered

| Node | Role | Key params | Gotchas |
| --- | --- | --- | --- |
| `webhook` | Trigger A (manual desde HubSpot) | path, method=POST | HubSpot workflow puede mandar payload limitado; complementar con re-fetch del Contact List. |
| `scheduleTrigger` | Trigger B (nurturing cron) | cron `0 10 * * *`, tz ART | — |
| `hubspot` | Get List members, send marketing email, get analytics | resource, operation | Si OQ-1 = HubSpot Marketing Hub, hay nodos específicos para "Marketing Email" más allá del CRM. |
| `mailchimp` (community node) | Send campaign via Mailchimp API | api key, list id, template id | Si OQ-1 = Mailchimp. |
| `googleSheets` | Config + log + metrics | sheetId, range | Igual que flows previos. |
| `splitInBatches` | Respect daily_cap + send_window | batchSize, wait | Probablemente batchSize 100 con wait 30s. |
| `wait` | Delay nocturno si excede send_window | until cron expression | Mejor: encolar para mañana via Sheet `campaign_queue`. |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| HubSpot Marketing Hub | Private App + Marketing scope | 100/10s + email send limits según plan | per-contact dedup nativa | https://developers.hubspot.com/docs/api/marketing/marketing-emails |
| Mailchimp | API key | 10/s (límite documentado) | mensaje id | https://mailchimp.com/developer |
| Domain reputation (whoever sends) | n/a | warmup gradual | n/a | depende del provider |

## 5. Base flow decision

- **Base flow chosen:** `none`.
- **Coverage estimate:** 0%
- **What we copy / what we change:** comparte primitivas con `whatsapp-overdue-debt-reminder` (config-driven, daily_cap, suppression). Promover esos patterns en común.

## 6. Reuse summary

- ✅ Reusing:
  - Credencial HubSpot `hubspot-blincer-main` (con scope marketing si OQ-1 = HubSpot Marketing).
  - Credencial Sheets.
  - Pattern de internal alert.
  - Concepto de `daily_cap` y `send_window` del flow 2 (queue para mañana).
- 🆕 New work:
  - Credencial Mailchimp (si aplica).
  - Sheets `campaigns_config`, `campaigns_log`, `campaigns_metrics`, `manual_suppression`, `campaign_queue`.
  - HubSpot List `do_not_email` mantenida automáticamente.
  - Custom property HubSpot Contact `last_campaign_received_at` + custom event.
- ⚠️ Risks identified:
  - **Reputación del dominio:** envíos masivos sin warmup pueden mandar al spam y bajar deliverability futuro. Mitigación: `daily_cap` agresivo + monitoreo de bounce rate.
  - **Unsubscribe legal:** todo email comercial requiere link de unsubscribe operativo (cumplimiento). HubSpot/Mailchimp lo agregan auto; si se manda con un nodo HTTP custom, hay que asegurarse de incluirlo.
  - **Lista mal segmentada:** Guillermo dispara campaña a la audiencia equivocada → daño reputacional. Mitigación: preview / dry-run obligatorio que liste destinatarios sin enviar; require manual confirm.
  - **Doble envío** si HubSpot workflow y cron solapan: idempotency por `(campaign_id, contact_id)`.
