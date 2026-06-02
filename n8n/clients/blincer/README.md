---
tags:
  - n8n
  - cliente
  - nivel-3
client: blincer
status: pre-sales
updated: 2026-06-02
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
| [[n8n/clients/blincer/flows/credit-limit-invoice-block/spec\|Bloqueo facturación por deuda]] | skeleton + Sheets backing wired (OQ nodes disabled) | HubSpot Deal stage change | 2026-06-02 |
| [[n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/spec\|Avisos deuda vencida WhatsApp]] | skeleton + Sheets backing wired (OQ nodes disabled) | Cron diario 09:00 ART | 2026-06-02 |
| [[n8n/clients/blincer/flows/sales-bot-with-quotes/spec\|Bot ventas + cotizaciones/facturas]] | skeleton built (OQ nodes disabled) · usa Postgres, no Sheets | WhatsApp inbound webhook | 2026-05-31 |
| [[n8n/clients/blincer/flows/email-remarketing/spec\|Remarketing y difusiones email]] | skeleton + Sheets backing wired (OQ nodes disabled) | Manual HubSpot + cron nurturing | 2026-06-02 |

> [!note] Build status 2026-05-31 — importable skeletons
> Cada flow tiene un `workflow.json` importable en n8n (Import from File). Construidos a mano desde los `plan.md` (el MCP de n8n no estaba conectado). Los nodos que dependen de open questions quedan **`disabled: true`** (Tango, provider WhatsApp, LLM, canal de alerta interno OQ-G7, plataforma de email). Todos los workflows están `active: false`. Antes de activar: mapear credenciales (`hubspot-blincer-main`, `gsheets-blincer-ops`, etc.), reemplazar los placeholders `REPLACE_*`, resolver las OQs y habilitar los nodos correspondientes.

> [!success] Progreso 2026-06-02 — Sheets backing cableado en n8n
> Los 4 skeletons ya están importados en la instancia n8n del cliente (`n8n.srv1512692.hstgr.cloud`). En los 3 flows que usan Sheets (credit-limit, whatsapp-overdue, email-remarketing) se hizo vía API:
> - Creados los **Google Sheets backing** (ver sección siguiente) y reemplazados todos los `REPLACE_SHEET_ID` por los IDs reales.
> - **Credencial Sheets:** se mapeó a la credencial OAuth2 existente **`Google Sheets account`** (id `NNpCFCk3F2rhlxUk`), la misma de los `BLINCER-T0xx`. El nombre `gsheets-blincer-ops` de los planes nunca llegó a existir como credencial → docs corregidos.
> - **Idempotencia:** los `Idempotency lookup` se reemplazaron por la dedup real (`read` + `Dedup filter` + write-back) en los 3 flows — patrón [[n8n/patterns/sheet-idempotency|Sheet-based idempotency]]. Detalle en cada `plan.md`.
> - **Error Workflow** `T-000` (`9zlznI4wuzz6MNSX`) asignado a los 3 flows.
> - Siguen `active: false`. HubSpot y Postgres siguen como placeholders (no hay credencial real en la instancia todavía).

## Backing stores — Google Sheets (2026-06-02)

Un spreadsheet por flow (dueño `borgonifrancisco@gmail.com`, compartidos *anyone-with-link = editor* según convención del cliente). Las columnas creadas son las que el **skeleton realmente usa** — más magras que los `plan.md` (las tabs `*_errors` y `*_metrics` y varias columnas extra del plan **todavía no se crearon**).

| Flow | Spreadsheet | ID | Tabs creadas |
| --- | --- | --- | --- |
| credit-limit | Blincer - Credit Limit | `1kjsp67c8eKVTqj6FoHpKI6mTtqoTFsBSUdEv9E0gPaI` | `audit_credit_block`, `idempotency_credit` |
| whatsapp-overdue | Blincer - Cobranzas | `12-VpWiZ2iw0QiHVast7-NOrAwASEAxOJuH4jer-Jb40` | `cobranzas_config`, `cobranzas_templates`, `cobranzas_log`, `cobranzas_fallback`, `idempotency_cobranzas` |
| email-remarketing | Blincer - Campaigns | `1-T8VhM8B-u0RPvZfCoZO9c7B1DaFTC0Qu0ElrD9uv6c` | `campaigns_config`, `manual_suppression`, `campaigns_log`, `campaign_queue`, `campaigns_metrics` |

> [!check] Cobranzas corregida (2026-06-02)
> El mix-up inicial entre `cobranzas_config` y `cobranzas_templates` se arregló: `cobranzas_config` quedó con `cadence_days | excluded_customers` + fila `[1,7,15,30,45,60]` / `[]`, y `cobranzas_templates` con sus 4 filas de plantilla. Verificado.

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
