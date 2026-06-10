---
tags:
  - n8n
  - plan
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: drive-folder-structure
updated: 2026-06-10
status: blocked-by-oqs
---

# Plan — G · Drive folder structure

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/drive-folder-structure/spec|Spec]] · [[n8n/clients/gpt-landings/flows/drive-folder-structure/research|Research]]

> ⚠️ **BLOQUEADO** — no ejecutar hasta resolver OQ-G-1..3 + M0 (service account Drive) + C/D. Bajo riesgo. Arquitectura propuesta: crear-si-no-existe + clasificación por tipo.

---

## Architecture

```mermaid
flowchart LR
  A[Evento: loan → processing] --> B{Carpeta del préstamo existe?}
  B -->|no| C[Crear /loan_id + subcarpetas]
  B -->|sí| D[Reusar folder_id]
  C --> E[Guardar folder_id en DB]
  D --> E
  F[Doc entrante de D] --> G[Detectar tipo → subcarpeta]
  G --> H[Mover/renombrar a /property|/closing|/borrower]
```

## Nodes

| # | Node | Type | Purpose | Key params | On error |
| --- | --- | --- | --- | --- | --- |
| 1 | `On processing` | trigger/exec | recibir `loan_id` al pasar a processing | — | n/a |
| 2 | `Folder exists?` | `googleDrive`+`if` | buscar carpeta del préstamo | query by name/parent | retry 3× |
| 3 | `Create tree` | `googleDrive` | crear `/loan_id` + `/property` `/closing` `/borrower` | parent, names | retry 3× |
| 4 | `Save folder_id` | `postgres` | guardar `folder_id` en `prestamos` | update | retry 3× |
| 5 | `On incoming doc` | trigger/exec | doc de D a clasificar | — | n/a |
| 6 | `Classify` | `code` | tipo → subcarpeta destino | JS, mapa tipo→folder | default `/borrower` + flag |
| 7 | `Move/rename` | `googleDrive` | mover a subcarpeta + nombre estándar | fileId, parent | retry 3× |

## Cross-cutting decisions

### Idempotency
- Dedup key: `loan_id` (carpeta) + `file_id` (clasificación).
- Strategy: chequear-antes-de-crear (no recrear carpeta si ya existe); mover es naturalmente idempotente.
- Why: re-disparar G no debe duplicar el árbol ni mover dos veces el mismo archivo.

### Error handling
- Retry policy: 3× backoff en Drive/DB.
- Dead-letter: `errors` con `{loan_id, file_id, node, error}`.
- Alerting: si no se puede crear el árbol → aviso interno (bloquea organización del préstamo).

### Credentials & secrets

| Credential | n8n credential name | Stored in | Owner |
| --- | --- | --- | --- |
| Google Drive | `gptlandings-drive` | n8n credentials | Innova |
| DB | `gptlandings-db` | n8n credentials | Innova |

### Observability
- Logs: creación de árbol (folder_id) + cada clasificación (archivo → subcarpeta).
- Métricas: `# árboles creados`, `# docs clasificados`, `# clasificaciones a default (sin tipo claro)`.

### Testing
- Test payloads: `loan_new.json` (crear árbol), `loan_existing.json` (no recrear), `doc_classify.json`, `doc_unknown_type.json` (→ default).
- Environment: carpeta de Drive de prueba (M0).
- Rollback: borrar carpeta de prueba; mover archivos de vuelta (script).

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| Nomenclatura sin cerrar | Media | Bajo | OQ-G-1; default razonable |
| Permisos exponen datos sensibles | Media | Alto | OQ-G-3; carpeta privada por defecto + sharing explícito |
| Doc clasificado mal | Media | Bajo | Default `/borrower` + flag para revisión |
| Carpeta duplicada por re-trigger | Baja | Bajo | Chequear-antes-de-crear |

## Open dependencies before build

- [ ] Resolver OQ-G-1..3.
- [ ] M0: service account Drive con permisos.
- [ ] C disparando el evento `processing`; D entregando docs.
