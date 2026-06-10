---
tags:
  - n8n
  - research
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: term-sheet-generation
updated: 2026-06-10
---

# Research — B · Term-sheet generation

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/term-sheet-generation/spec|Spec]]

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/gpt-landings/flows/` (2026-06-10).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `partner-matching-engine` (A) | 🟢 Alta | **Produce** el input (`qualifying_partners`). Contrato de interfaz. |
| `approval-and-esign` (C) | 🟢 Alta | **Consume** el PDF generado. |
| `m0-infra-setup` | 🟡 Media | Drive credentials + DB para referenciar el PDF. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, other `n8n/clients/*/flows/` (2026-06-10).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/clients/blincer/flows/sales-bot-with-quotes/research.md` (sub-workflow `create_quote`) | Render DOCX/HTML→PDF con Gotenberg/Playwright + template + variables, upload a Drive | **Casi idéntico**: cotización↔term sheet. Reusar el pipeline de render + upload. |
| `agentes/subagente-pre-contratacion.md` | `python-docx` → documento → Drive | Mecánica de generar doc + subirlo a Drive con service account. |

## 3. n8n nodes considered

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.code` | Render HTML del template con variables | JS / template engine | escapar valores; locale de moneda USD |
| `n8n-nodes-base.httpRequest` (Gotenberg) o Playwright service | HTML → PDF | endpoint + token | self-hosted Gotenberg en el VPS (decidir host) |
| `n8n-nodes-base.googleDrive` | Subir PDF a la carpeta del préstamo | folderId, binary | service account `gptlandings-drive` |
| `n8n-nodes-base.postgres` | Referenciar PDF + `valid_until` | update | — |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| Gotenberg / Playwright (self-hosted) | token / interno | depende host | n/a | https://gotenberg.dev |
| Google Drive | service account (`gptlandings-drive`) | 1000 req/100s | n/a | https://developers.google.com/drive/api |
| e-sign (si template nativo) | depende OQ-C-1 | — | — | — |

## 5. Base flow decision

- **Base flow chosen:** parcial — el sub-workflow `create_quote` de `sales-bot-with-quotes` (Blincer) como referencia de render.
- **Coverage estimate:** ~40% (pipeline render+upload; cambia el template y el origen de datos).
- **What we copy / what we change:** copiamos render HTML→PDF + upload Drive; cambiamos template (term sheet) y origen (JSON de A).

## 6. Reuse summary

- ✅ Reusing: pipeline render PDF + upload Drive (`sales-bot-with-quotes/research.md`, `agentes/subagente-pre-contratacion.md`); creds Drive y DB de M0.
- 🆕 New work: el template del term sheet (def #2) + el binding de campos del préstamo/partner; manejo de tasa/monto editables.
- ⚠️ Risks identified: sin template/ejemplo real (def #2) no se parametriza; decisión HTML→PDF vs e-sign depende de C (OQ-C-1); host de Gotenberg/Playwright a definir en M0.
