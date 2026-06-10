---
tags:
  - n8n
  - research
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: m0-infra-setup
updated: 2026-06-10
---

# Research — M0 · Infrastructure base

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/m0-infra-setup/spec|Spec]]

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/gpt-landings/flows/` (2026-06-10).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| (none — first bundle of GPT Landings) | — | M0 is the "flow-zero" that enables A–G. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, other `n8n/clients/*/flows/` (2026-06-10).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/clients/blincer/README.md` § Credentials policy | All creds in n8n store, secrets vault TBD (Doppler/Bitwarden) | Same policy; **new instance → no Blincer creds reused.** |
| `n8n/clients/blincer/README.md` ("Error Workflow `T-000`") | Default Error Workflow assigned to every flow | Create the GPT Landings equivalent in M0. |
| `n8n/METHODOLOGY.md` + `n8n/templates/*` | Spec Kit scaffolding | M0 seeds the convention for the whole client. |

## 3. n8n nodes considered

(M0 is infra; the only n8n object is the base Error Workflow.)

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.errorTrigger` | Base Error Workflow entrypoint | — | Set as instance default error workflow. |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| VPS (client) | SSH/root | n/a | n/a | — |
| n8n self-hosted | pm2 + reverse proxy (HTTPS) | n/a | n/a | https://docs.n8n.io/hosting/ |
| Supabase / Postgres | service role / connection string | n/a | `UNIQUE` + `ON CONFLICT` | https://supabase.com/docs |
| Google Drive | service account / OAuth | 1000 req/100s/user | n/a | https://developers.google.com/drive/api |

## 5. Base flow decision

- **Base flow chosen:** `none` (infra setup, not a flow).
- **Coverage estimate:** n/a
- **What we copy / what we change:** copy Blincer's credentials policy + Error-Workflow convention; everything else is fresh on the client VPS.

## 6. Reuse summary

- ✅ Reusing: credentials policy + Error-Workflow convention (`n8n/clients/blincer/README.md`); Drive OAuth/service-account mechanics documented in `agentes/subagente-pre-contratacion.md` (Google Cloud project + Drive API + OAuth Desktop).
- 🆕 New work: VPS provisioning, n8n+pm2+HTTPS, 4-table schema, secrets vault choice, backups.
- ⚠️ Risks identified: no VPS/Workspace access on time blocks everything; managed-vs-self-hosted DB decision affects backups & cost.
