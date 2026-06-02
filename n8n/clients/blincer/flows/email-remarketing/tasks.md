---
tags:
  - n8n
  - tasks
  - blincer
  - nivel-3
client: blincer
flow: email-remarketing
updated: 2026-06-02
status: blocked-by-oqs
---

# Tasks — Remarketing y difusiones por email

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/flows/email-remarketing/plan|Plan]]

> [!success] Progreso 2026-06-02 (Sheets backing)
> Hecho vía API n8n: spreadsheet **Blincer - Campaigns** (`1-T8VhM8B-u0RPvZfCoZO9c7B1DaFTC0Qu0ElrD9uv6c`) creado con `campaigns_config`, `manual_suppression`, `campaigns_log`, `campaign_queue`, `campaigns_metrics` (columnas según skeleton; **falta** `campaigns_errors` y columnas extra del § Setup). `REPLACE_SHEET_ID` reemplazado en los 10 nodos + credencial mapeada a `Google Sheets account`. `Idempotency lookup` quedó **disabled** (v4.5 sin op `lookup` + falta write-back). `Re-trigger main workflow` ya apunta a `https://n8n.srv1512692.hstgr.cloud/webhook/blincer-campaign` (`REPLACE_N8N_BASE` resuelto 2026-06-02).
>
> **Update (dedup, mismo día):** dedup real ya implementada (`read campaigns_log` → `Dedup filter` por `(campaign_id, contact_id)`; write-back vía el propio `Log sent`) — ver plan.md y [[n8n/patterns/sheet-idempotency|pattern]]. Error Workflow `T-000` asignado.
>
> **Update (HubSpot, mismo día):** credencial `hubspot-blincer-apptoken` (App Token, id `A3JekIL652cjutl4`) enganchada a `Resolve segment`, `Update HubSpot Contact`, `Get bounces & unsubs (24h)` y `Add to do_not_email` (`authentication: appToken`); `Send via platform` queda disabled sin credencial (OQ-1). ⚠️ token provisto **rechazado** por HubSpot → pegar un Private App token válido. Pendiente con token vivo: resolver `REPLACE_DO_NOT_EMAIL_LIST_ID`.

---

## Pre-build

- [ ] Cerrar OQ-1 a OQ-7 del spec + OQ-G7.
- [ ] Decidir HubSpot Marketing Hub vs Mailchimp + contratar/confirmar acceso.
- [ ] Validar compliance: política privacidad publicada, link unsub obligatorio.
- [ ] Warmup plan acordado con Guillermo (200 → 500 → 1500 daily_cap).

## Setup

- [ ] Credenciales en n8n:
  - [ ] HubSpot Private App con scope marketing adicional (extender `hubspot-blincer-main`).
  - [ ] `mailchimp-blincer` (si aplica).
- [ ] Sheets:
  - [ ] `campaigns_config` (cols: `campaign_id`, `template_id`, `list_id`, `daily_cap`, `send_window`, `mode`, `created_by`)
  - [ ] `campaigns_log` (cols: `timestamp`, `campaign_id`, `contact_id`, `status`, `error`, `provider_send_id`)
  - [ ] `campaigns_metrics` (cols: `campaign_id`, `closed_at`, `sent`, `opened`, `clicked`, `bounced`, `unsubscribed`, `open_rate`, `ctr`)
  - [ ] `manual_suppression` (cols: `email`, `added_by`, `reason`, `added_at`)
  - [ ] `campaign_queue` (cols: `campaign_id`, `contact_id`, `scheduled_for`, `processed`)
  - [ ] `campaigns_errors` (cols: `timestamp`, `campaign_id`, `node`, `error`, `payload`)
  - [ ] `marketing_config` (cols: `default_send_window`, `bounce_threshold_pct`, `kill_switch`)
- [ ] HubSpot Lists:
  - [ ] `do_not_email` (smart list: añade bounces + unsubscribes).
  - [ ] `__test_marketing__` (estática con 3 contacts internos).
- [ ] HubSpot custom property `last_campaign_received_at` (datetime, Contact).
- [ ] Timeline event "Campaign sent" via CRM Cards API.
- [ ] HubSpot workflow nativo "Lanzar campaña" que dispara webhook a n8n (URL del webhook).

## Build (workflow principal)

- [ ] Workflow `blincer / email-remarketing` creado.
- [ ] Node 1: Webhook `/blincer-email-campaign`.
- [ ] Node 1': ScheduleTrigger `0 10 * * *` ART (nurturing).
- [ ] Node 2: Sheets read `campaigns_config` filter `campaign_id`.
- [ ] Node 3: IF mode=dry-run → enviar preview a Guillermo y end.
- [ ] Node 4: HubSpot get list members.
- [ ] Node 5: HubSpot list `do_not_email` + Sheets `manual_suppression` merge.
- [ ] Node 6: Function subtract suppression.
- [ ] Node 7: Sheets lookup idempotency.
- [ ] Node 8: Function split send/queue según `daily_cap` y `send_window`.
- [ ] Node 9: HubSpot Marketing send (o Mailchimp).
- [ ] Node 10: Sheets append `campaigns_log`.
- [ ] Node 11: HubSpot update contact + timeline event.
- [ ] Node 12: Sheets append `campaign_queue` para overflow.
- [ ] Node 13: Internal alert a Guillermo (start summary).
- [ ] Validate cada nodo con `mcp__n8n-mcp__validate_node`.

## Build (workflows secundarios)

- [ ] `email-remarketing-analytics`:
  - [ ] Cron 09:00 ART
  - [ ] Find campaigns ≥48h old en `campaigns_log`
  - [ ] Get analytics
  - [ ] Compute + update `campaigns_metrics`
  - [ ] Send 48h summary a Guillermo
- [ ] `email-remarketing-suppression-sweep`:
  - [ ] Cron 02:00 ART
  - [ ] Get bounces/unsubs últimas 24h
  - [ ] Add to `do_not_email` HubSpot List
- [ ] `email-remarketing-queue-flush`:
  - [ ] Cron 09:00 ART (después del flush de la suppression)
  - [ ] Read `campaign_queue` con `scheduled_for ≤ today`
  - [ ] Re-trigger main workflow
  - [ ] Mark processed

## Validate

- [ ] Dry-run de una campaña de 50 contacts → preview entregado, 0 sends.
- [ ] Campaña real a `__test_marketing__` (3 contacts) → 3 sends, 0 queue.
- [ ] Lista mayor a `daily_cap` → split correcto.
- [ ] Reenviar misma campaña → 0 sends (idempotency).
- [ ] Bounce sintético → entra a `do_not_email` en próxima sweep.
- [ ] Suppression manual añadida → no recibe en campaña siguiente.
- [ ] Kill switch (`kill_switch=true`) → main workflow no envía.

## Harden

- [ ] Monitoreo bounce rate → pausar campaña si > 5%.
- [ ] Warmup gradual `daily_cap` (200 → 500 → 1500).
- [ ] Alertas wiring confirmadas.

## Document

- [ ] Exportar workflows como `workflow.json`, `workflow.analytics.json`, `workflow.suppression.json`, `workflow.queue-flush.json`.
- [ ] Actualizar `spec.md` status → `live`.
- [ ] Actualizar fila en `n8n/clients/blincer/README.md`.

## Handoff

- [ ] Capacitar a Guillermo: cómo armar campaña en HubSpot, cómo disparar via tag/workflow, cómo leer summary.
- [ ] Documentar SLA: campañas con > 10k destinatarios requieren aviso de 48h.
- [ ] Escribir `retro.md`.
- [ ] Promover patterns: `n8n/patterns/bulk-email-with-tracking.md`, `n8n/patterns/suppression-sweep.md`, `n8n/patterns/queue-overflow.md`.
- [ ] Append one-liner a `n8n/lessons-learned.md`.
