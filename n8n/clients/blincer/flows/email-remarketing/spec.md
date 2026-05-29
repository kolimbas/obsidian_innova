---
tags:
  - n8n
  - spec
  - blincer
  - nivel-3
client: blincer
flow: email-remarketing
status: draft
updated: 2026-05-29
---

# Spec — Remarketing y difusiones por email

> Campañas segmentadas y difusiones masivas a la base de Blincer, configuradas y disparadas por el equipo comercial sin pasar por Innova.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/README|Blincer]]

---

## Goal

Habilitar a Guillermo a (a) lanzar campañas one-off (anuncios, promos, novedades) a segmentos de la base y (b) tener flujos recurrentes de nurturing por estado del lead/cliente, sin escribir HTML ni armar listas a mano. La salida es engagement medible (open rate, CTR, replies) en HubSpot.

## Trigger

- **Type:** doble trigger
  - **A — Manual:** disparo desde HubSpot (workflow nativo del CRM) al cambiar un Deal/Contact a un estado o tag de "send campaign". n8n recibe webhook.
  - **B — Cron:** nurturing programado (ej. "X días después de cotización sin compra → email follow-up").
- **Schedule:** webhook (A) on-demand; cron (B) diario 10:00 ART para evaluar reglas de nurturing.
- **Expected frequency:** A → 1–4× / mes; B → diario con volumen depende de funnel.

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `campaign_id` | string | Sheet `campaigns_config` o HubSpot custom property | yes (A) | identifica la campaña a enviar |
| `segment` | object | HubSpot List (estática o smart) | yes | criterios de filtrado |
| `template_id` | string | HubSpot Marketing Email o Mailchimp template | yes | el contenido HTML |
| `variables` | object | mix de campaign config + contact props | no | sustituciones tipo `{{first_name}}`, `{{last_purchase_date}}` |
| `send_window` | object | Sheet `marketing_config` | no | horas permitidas (default 09:00–18:00 ART) |
| `daily_cap` | int | Sheet `marketing_config` | yes | límite anti-spam (default 1500/día) |
| `suppression_list` | list | HubSpot List `do_not_email` + bounces históricos | yes | nadie en esta lista recibe nada |

Sample webhook payload (HubSpot workflow → n8n):

```json
{
  "campaign_id": "Q2-2026-nuevo-modelo-tango",
  "triggered_by_user": "guillermo@blincer.com",
  "list_id": "47",
  "scheduled_for": null
}
```

## Outputs / Side effects

- **Email send:** vía proveedor (TBC OQ-1) — HubSpot Marketing o Mailchimp.
- **HubSpot Contact:** custom property `last_campaign_received_at` + custom event "Campaign sent" en timeline.
- **HubSpot Company:** rollup de campañas recientes.
- **Sheet `campaigns_log`:** row por contact-campaign con status (sent, suppressed, bounced, opened, clicked).
- **Reportes:** Sheet `campaigns_metrics` con métricas agregadas por campaña.
- **Alerta a Guillermo:** end-of-campaign summary (sent, open rate, CTR, bounces) al cerrar la ventana de tracking (48h post-send).

## Success criteria

- [ ] Guillermo lanza una campaña desde HubSpot (cambiando estado/tag) y el flow toma el control y arma la lista filtrada respetando `suppression_list`.
- [ ] Send respeta `send_window` y `daily_cap` — si excede, agenda restantes para el día siguiente.
- [ ] Cada contact recibe la campaña **una sola vez** por `campaign_id` (idempotency).
- [ ] Tracking de aperturas y clicks queda registrado en HubSpot timeline del contact.
- [ ] Bounces y unsubscribes se suman automáticamente a `suppression_list` (cron nightly).
- [ ] Guillermo recibe el summary de la campaña a las 48h.
- [ ] Cero envíos a contacts en `do_not_email` o con bounce histórico.
- [ ] Métricas históricas de campañas accesibles para Guillermo (Sheet o HubSpot reports).

## Out of scope

- A/B testing automático (V2).
- Diseño visual de emails — los templates los arma Guillermo en HubSpot/Mailchimp UI.
- Multi-canal (sumar SMS o WhatsApp) — para WhatsApp ya existe `whatsapp-overdue-debt-reminder`; mezclar dunning con marketing en el mismo flow es un anti-pattern.
- Sequence/drip campaigns avanzadas (que dependen de comportamiento) — si Guillermo lo necesita, sale otro flow.
- Personalización por LLM del cuerpo del email — V2 (potencial bot de copywriting).
- Sync inverso (importar contacts desde Mailchimp a HubSpot) — proceso aparte si OQ-1 termina en Mailchimp.

## Open questions

- [ ] **OQ-1 — Marketing platform.** HubSpot Marketing Hub vs Mailchimp:
  - **HubSpot Marketing Hub Pro+**: ya tienen el CRM (Pro+ confirmado), tracking nativo y listas integradas — menos sincronización entre sistemas. Plan extra de Marketing Hub puede costar más que Mailchimp.
  - **Mailchimp (legacy del roadmap original)**: separado del CRM, requiere sync, pero infra ya existe.
  - Recomendación MVP: **HubSpot Marketing Hub** si está incluido/contratado; Mailchimp si no.
- [ ] **OQ-2 — Quién mantiene `suppression_list`.** Propuesta: HubSpot List `do_not_email` (con auto-add de unsubscribes vía webhook) + Sheet `manual_suppression` editada por Guillermo.
- [ ] **OQ-3 — `daily_cap` por reputación.** Empezar conservador (1500) y subir gradualmente según reputación de envío. ¿Hay un dominio dedicado de envío o usan el corporativo?
- [ ] **OQ-4 — Métricas de éxito (KPIs).** ¿Qué define éxito de campaña para Guillermo? Open rate ≥ X%, CTR ≥ Y%, reply rate ≥ Z%? Sin esto, el summary del flow es informativo pero no actionable.
- [ ] **OQ-5 — Window de tracking.** 48h propuesto. ¿Suficiente o quiere 7 días? Cambia cuándo cierra el summary.
- [ ] **OQ-6 — Compliance.** ¿Tienen política GDPR/LGPD/Ley 25.326 (Argentina)? Necesario para defaults de unsubscribe y data retention.
- [ ] **OQ-7 — Reportes a quién más.** Solo a Guillermo o también a Innova / Sandra? Cambia canal de salida del summary.

*(Cruza con OQ-G7 global — canal alerta interno).*

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Guillermo |
| Approver | Guillermo |
| End user (operativo) | Guillermo (lanza campañas) |
| End user (afectado) | Base de contacts de Blincer |
