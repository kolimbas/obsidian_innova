---
tags:
  - n8n
  - research
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: drive-folder-structure
updated: 2026-06-10
---

# Research — G · Drive folder structure

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/drive-folder-structure/spec|Spec]]

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/gpt-landings/flows/` (2026-06-10).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `approval-and-esign` (C) | 🟢 Alta | Dispara G al pasar el préstamo a `processing`. |
| `document-intake-form` (D) | 🟢 Alta | Los docs que G clasifica vienen de D. |
| `m0-infra-setup` | 🟡 Media | Service account Drive + DB. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, other `n8n/clients/*/flows/` (2026-06-10).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `agentes/subagente-pre-contratacion.md` | Estructura de carpetas por entidad en Drive + nomenclatura + idempotencia por marker (`LISTO`/`PROCESADO`) | **Análogo directo**: crear árbol por préstamo, nombrar de forma estándar, no recrear si existe. |
| `n8n/patterns/sheet-idempotency.md` | No recrear lo ya hecho | Idempotencia de la creación de carpetas (chequear antes de crear). |

## 3. n8n nodes considered

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.googleDrive` | Crear carpetas + mover/renombrar archivos | parent folderId, name | chequear existencia antes de crear (idempotencia) |
| `n8n-nodes-base.code` | Resolver nombre estándar + ruta destino por tipo | JS | nomenclatura OQ-G-1 |
| `n8n-nodes-base.postgres` | Guardar `folder_id` del préstamo | update | — |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| Google Drive | service account (`gptlandings-drive`) | 1000 req/100s/user | n/a (chequear-antes-de-crear) | https://developers.google.com/drive/api |
| Supabase/Postgres | conn string | n/a | update por loan | https://supabase.com/docs |

## 5. Base flow decision

- **Base flow chosen:** parcial — la mecánica de folders del `subagente-pre-contratacion` (Drive + nomenclatura + idempotencia).
- **Coverage estimate:** ~50% (estructura + idempotencia; cambia el árbol y el origen del trigger).
- **What we copy / what we change:** copiamos crear-si-no-existe + nomenclatura estándar; cambiamos el árbol (`/property`,`/closing`,`/borrower`) y el trigger (loan → processing).

## 6. Reuse summary

- ✅ Reusing: mecánica Drive del subagente pre-contratación; `sheet-idempotency` (no recrear); creds Drive y DB de M0.
- 🆕 New work: el árbol específico del préstamo; la clasificación de docs por tipo a subcarpeta; permisos/sharing.
- ⚠️ Risks identified: nomenclatura sin cerrar (OQ-G-1); permisos mal seteados exponen datos sensibles (OQ-G-3); clasificación errónea de un doc (mitigable, bajo impacto).
