---
tags:
  - n8n
  - spec
  - blincer
  - nivel-3
client: blincer
flow: whatsapp-overdue-debt-reminder
status: draft
updated: 2026-05-29
---

# Spec — Avisos automáticos de deuda vencida por WhatsApp

> Los clientes con facturas vencidas reciben recordatorios por WhatsApp según una cadencia configurable, con mensaje personalizado, sin que Sandra tenga que disparar nada manualmente.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/README|Blincer]]

---

## Goal

Reducir el trabajo manual de cobranza de Sandra/Guillermo y acelerar el ciclo de cobro de Blincer mediante recordatorios automáticos de WhatsApp a clientes con facturas vencidas, en una cadencia parametrizable y con templates editables sin tocar n8n.

## Trigger

- **Type:** cron
- **Schedule:** diario a las 09:00 ART (`0 9 * * *`)
- **Expected frequency:** 1× / día; cada corrida procesa la lista completa de facturas vencidas que matcheen alguno de los días de la cadencia.

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `overdue_invoices` | list | Tango (cuenta corriente / facturas no cobradas) | yes | filter: `vencimiento < today AND saldo > 0` |
| `customer` | object | HubSpot Contact / Company por mapeo `tango_customer_id` | yes | de acá sacamos número WhatsApp y nombre |
| `template_messages` | object | HubSpot Company custom property o Sheet `cobranzas_templates` | yes | un template por cada día de la cadencia |
| `cadence_days` | list[int] | Sheet de config `cobranzas_config` | yes | default `[1, 7, 15, 30, 45, 60]` post-vencimiento |
| `excluded_customers` | list | HubSpot List `cobranza_excluida` o Sheet | no | clientes con plan especial / litigio / etc. |
| `business_hours_window` | string | Sheet config | no | default "09:00–18:00 ART"; el cron es 09:00, no debería pisar fuera de banda |

Sample payload (interno del flow, después del fetch de Tango):

```json
{
  "invoice_id": "FC-A-00012345",
  "tango_customer_id": "C-9421",
  "amount_due": 187540.50,
  "currency": "ARS",
  "due_date": "2026-05-15",
  "days_overdue": 14,
  "matches_cadence_day": 15
}
```

## Outputs / Side effects

- **WhatsApp:** mensaje al contacto principal del cliente, vía provider TBC (OQ-G2). El texto se compone del template del día de cadencia + variables (`{{nombre}}`, `{{invoice_id}}`, `{{amount_due}}`, `{{due_date}}`, `{{days_overdue}}`, link de pago si aplica).
- **HubSpot Contact timeline:** evento custom "Recordatorio cobranza enviado" con factura, día de cadencia, status (sent/failed).
- **HubSpot Company:** sumar `last_dunning_at` (custom property datetime).
- **Sheet `cobranzas_log`:** una row por intento (sent/skipped/error) con timestamp, factura, contacto, día de cadencia, status.
- **Sandra alert:** si > N facturas sin contacto disponible (no WhatsApp) o si tasa de error > 20% en una corrida → alerta resumen.

## Success criteria

- [ ] Cada día a las 09:00 ART, el flow lista facturas vencidas y matchea contra `cadence_days`. Si una factura está en `days_overdue ∈ cadence_days`, dispara WhatsApp.
- [ ] Cada cliente recibe **a lo sumo un mensaje por factura por día** (idempotencia por `invoice_id + cadence_day + date`).
- [ ] Los templates son editables por Sandra sin redeploy ni intervención de Innova (Sheet o HubSpot prop).
- [ ] Mensaje enviado dentro de business hours (09:00–18:00 ART) — el cron 09:00 garantiza esto, pero si por algún motivo el flow tarda y supera 18:00, no envía (deja en cola para mañana 09:00).
- [ ] Cobertura ≥ 95% de los clientes con WhatsApp registrado (los sin WhatsApp caen a Sheet de fallback para que Sandra revise).
- [ ] Log completo en `cobranzas_log` que permita auditar qué se envió a quién y cuándo.

## Out of scope

- Cobro / link de pago automático (puede sumarse vía variable de template, pero el motor de pagos no es parte del flow).
- Recordatorios por email (es un fallback potencial — ver OQ-3 — pero por ahora WhatsApp-only).
- Reconciliación bancaria automática de pagos recibidos (eso es otro flow del roadmap original).
- Bloqueo de facturación nueva por deuda (ese es el `credit-limit-invoice-block`).
- Detección de "ya pagó" entre el momento del fetch y el envío (mitigación: re-validar saldo en el último step antes del envío).

## Open questions

- [ ] **OQ-1 — Provider WhatsApp final** (referencia OQ-G2). Evolution API o Cloud API oficial. Impacta opt-in, costo, nodo n8n.
- [ ] **OQ-2 — Opt-in regulatorio.** ¿Los clientes de Blincer dieron consentimiento explícito para recibir notificaciones WhatsApp? WhatsApp Business policy lo exige. Sin opt-in documentado → riesgo de baneo (Cloud API) o usuario reporta spam (Evolution).
- [ ] **OQ-3 — Fallback si no hay WhatsApp.** Tres opciones:
  - **A:** Email automático (requiere Gmail / HubSpot Marketing Email).
  - **B:** Solo log en Sheet → Sandra los revisa manualmente.
  - **C:** Llamada saliente vía proveedor (out-of-scope inicial).
  Default propuesto: B en MVP, A en V2.
- [ ] **OQ-4 — Source de facturas vencidas.** Depende de OQ-G1 (Tango versión):
  - Nexo → API REST con filter por vencimiento.
  - Local sin API → CSV export programado (Sandra exporta diario o flow lee de Drive una carpeta watch).
- [ ] **OQ-5 — Cadencia configurable: ¿desde dónde?** Propuesta: Sheet `cobranzas_config` con columna `cadence_days` (JSON array). Alternativa: HubSpot custom property a nivel Company para cadencias por cliente.
- [ ] **OQ-6 — Templates: ¿uno único o por cliente?** Propuesta MVP: un set único de 6 templates (uno por día de cadencia default). Por-cliente en V2 si lo piden.
- [ ] **OQ-7 — Manejo de moneda extranjera.** ¿Hay facturas en USD? Si sí, el template debe reflejarlo.

*(OQ-G1, OQ-G2, OQ-G7 globales — ver [[n8n/clients/blincer/README|Blincer README]]).*

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Sandra (operación) + Guillermo (impacto cobranza) |
| Approver | Sandra (templates y cadencia) + Guillermo (decisión de "encender" en clientes) |
| End user (operativo) | Sandra (revisa fallbacks, ajusta templates) |
| End user (afectado) | Clientes de Blincer con deuda vencida |
