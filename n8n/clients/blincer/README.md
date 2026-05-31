---
tags:
  - n8n
  - cliente
  - nivel-3
client: blincer
status: pre-sales
updated: 2026-05-29
---

# Blincer — n8n Workspace

> Lock factory (Argentina). Same owners as GAAB. Business context lives in `clientes/blincer.md` (Spanish).

← [[n8n/clients/_index|Clients]]

---

## Scope (current understanding)

Automate Sandra's admin work (bank reconciliation, recurring invoicing, management report consolidation) and Guillermo's commercial workflows (order traceability, omnichannel inbox, stock alerts). See `clientes/blincer.md` for project phases and pricing model.

**Canonical stack confirmado por el cliente (2026-05-29):** Tango (ERP) + HubSpot (CRM / comercial). Bancos, Google Workspace y herramientas de marketing del roadmap original se conservan como periféricos.

## Stack

| System | Use | Integration risk |
| --- | --- | --- |
| **HubSpot (Pro+)** | CRM, Deals, Marketing Email, Workflows API | Native node + REST · webhooks habilitados |
| Tango Gestión (version TBC) | ERP, facturación, cuenta corriente | **High** if local-only (no API) → CSV / RPA |
| Galicia · BBVA · Cooperativa | Banking | Usually scrape or CSV import |
| Gmail · Google Drive · Google Sheets | Docs / ops | Native n8n nodes |
| Mailchimp · Metricool · Google Ads | Marketing (legacy — evaluar reemplazo por HubSpot Marketing) | Native nodes / REST |
| WhatsApp (provider TBC) | Canal cliente final + cobranzas | Evolution API vs Cloud API oficial |

## Credentials policy

- All credentials live in n8n's credential store, never in node parameters or env vars in plaintext.
- Bank credentials owner: Sandra (until handoff).
- Secrets vault for non-n8n secrets: TBD (Doppler vs Bitwarden Secrets — decide before Phase 1).

## Flows

| Flow | Status | Trigger | Last update |
| --- | --- | --- | --- |
| [[n8n/clients/blincer/flows/credit-limit-invoice-block/spec\|Bloqueo facturación por deuda]] | skeleton built (OQ nodes disabled) | HubSpot Deal stage change | 2026-05-31 |
| [[n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/spec\|Avisos deuda vencida WhatsApp]] | skeleton built (OQ nodes disabled) | Cron diario 09:00 ART | 2026-05-31 |
| [[n8n/clients/blincer/flows/sales-bot-with-quotes/spec\|Bot ventas + cotizaciones/facturas]] | skeleton built (OQ nodes disabled) | WhatsApp inbound webhook | 2026-05-31 |
| [[n8n/clients/blincer/flows/email-remarketing/spec\|Remarketing y difusiones email]] | skeleton built (OQ nodes disabled) | Manual HubSpot + cron nurturing | 2026-05-31 |

> [!note] Build status 2026-05-31 — importable skeletons
> Cada flow tiene un `workflow.json` importable en n8n (Import from File). Construidos a mano desde los `plan.md` (el MCP de n8n no estaba conectado). Los nodos que dependen de open questions quedan **`disabled: true`** (Tango, provider WhatsApp, LLM, canal de alerta interno OQ-G7, plataforma de email). Todos los workflows están `active: false`. Antes de activar: mapear credenciales (`hubspot-blincer-main`, `gsheets-blincer-ops`, etc.), reemplazar los placeholders `REPLACE_*`, resolver las OQs y habilitar los nodos correspondientes.

## Open dependencies

- [ ] Confirm Tango version (local vs Nexo) — bloquea plan de 3 flows
- [ ] Confirm WhatsApp provider final (Evolution API vs Cloud API oficial) — bloquea plan de 2 flows
- [ ] Confirm HubSpot edition exacto + scopes API disponibles
- [ ] Decidir LLM provider y budget para sales-bot
- [ ] Decidir HubSpot Marketing vs Mailchimp para email-remarketing
- [ ] Source of truth del catálogo + stock para sales-bot
- [ ] Política de aprobación humana antes de emitir cotización/factura
- [ ] Canal de alerta interno (Sandra/Guillermo) — WhatsApp interno, email o Slack/Teams
- [ ] Get list of accesses / credentials from client
- [ ] Decide secrets vault

## Links

- [[n8n/clients/blincer/discovery-2026-05-29|Cuestionario de discovery — tanda 2026-05-29]] (para mandarle a Sandra/Guillermo/IT)
- Business note: `clientes/blincer.md`
- Pre-contract subagent: `agentes/subagente-pre-contratacion.md`
