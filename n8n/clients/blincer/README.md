---
tags:
  - n8n
  - client
client: blincer
status: pre-sales
updated: 2026-05-28
---

# Blincer — n8n Workspace

> Lock factory (Argentina). Same owners as GAAB. Business context lives in [[clientes/blincer]] (Spanish).

← [[n8n/clients/_index|Clients]]

---

## Scope (current understanding)

Automate Sandra's admin work (bank reconciliation, recurring invoicing, management report consolidation) and Guillermo's commercial workflows (order traceability, omnichannel inbox, stock alerts). See [[clientes/blincer]] for project phases and pricing model.

## Stack

| System | Use | Integration risk |
| --- | --- | --- |
| Tango Gestión (version TBC) | ERP | **High** if local-only (no API) → CSV / RPA |
| Galicia · BBVA · Cooperativa | Banking | Usually scrape or CSV import |
| Gmail · Google Drive · Google Sheets | Docs / ops | Native n8n nodes |
| Mailchimp · Metricool · Google Ads | Marketing | Native nodes / REST |

## Credentials policy

- All credentials live in n8n's credential store, never in node parameters or env vars in plaintext.
- Bank credentials owner: Sandra (until handoff).
- Secrets vault for non-n8n secrets: TBD (Doppler vs Bitwarden Secrets — decide before Phase 1).

## Flows

| Flow | Status | Trigger | Last update |
| --- | --- | --- | --- |
| _none yet_ |  |  |  |

## Open dependencies

- [ ] Confirm Tango version (local vs Nexo)
- [ ] Get list of accesses / credentials from client
- [ ] Decide secrets vault

## Links

- Business note: [[clientes/blincer]]
- Pre-contract subagent: [[agentes/subagente-pre-contratacion]]
