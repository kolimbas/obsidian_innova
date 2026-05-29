---
tags:
  - n8n
  - research
  - blincer
  - nivel-3
client: blincer
flow: credit-limit-invoice-block
updated: 2026-05-29
---

# Research — Bloqueo automático de facturación por deuda

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/flows/credit-limit-invoice-block/spec|Spec]]

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/blincer/flows/` (2026-05-29).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| _ninguno_ | — | Primer bundle de Blincer. Sin historial de implementaciones reutilizables todavía. |

> Nota: este flow forma parte de la **primera tanda** de Blincer. Los 3 hermanos (`whatsapp-overdue-debt-reminder`, `sales-bot-with-quotes`, `email-remarketing`) van en paralelo — chequear specs de ellos antes de plan.md por si comparten infra (credencial HubSpot, mapeo Tango, canal de alerta a Sandra).

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, otros `n8n/clients/*/flows/` (2026-05-29).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/patterns/_index.md` | (sin patterns aún) | Promover el shape "HubSpot webhook → lookup en sistema externo → flag back en HubSpot" como pattern cuando este flow se promueva a `live`. |
| Proyectos Innova fuera del vault | `project_auditoria_facturas.md` (memoria) — auditoría de facturas en n8n para travel agency, bloqueado esperando `lead_id` de byref | Estructura similar: webhook → lookup cruzado → decisión. Reusar la lección de "no decidir si el lookup externo está caído" → en este flow ya está reflejado como fail-safe en el spec. |

## 3. n8n nodes considered

A validar con `mcp__n8n-mcp__search_nodes` / `get_node` durante el build. Pre-selección basada en el catálogo conocido:

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.hubspotTrigger` | Capturar `deal.propertyChange` | event, propertyName, dealStageFilter | HubSpot reentrega webhooks (idempotencia obligatoria). Crear `n8n/nodes/hubspot-trigger.md` al promover. |
| `n8n-nodes-base.hubspot` | Get Deal + Get Company + Update Deal + Create Note | objectType, associations, properties | El `associations` para Deal→Company puede venir en payload o requerir GET adicional. |
| `n8n-nodes-base.httpRequest` | Llamar a Tango (Nexo REST) o a un endpoint intermedio que exponga la DB local | URL, auth (Bearer / API key), timeout | Si Tango es local sin API → se reemplaza por nodo lector de CSV o `Postgres`/`MySQL`/`Microsoft SQL` según el motor que use Tango local. |
| `n8n-nodes-base.if` | Decisión deuda+monto vs límite | condition | — |
| `n8n-nodes-base.set` | Armar payload de Note y de alerta | values | — |
| `n8n-nodes-base.googleSheets` o HubSpot Note | Audit log | sheet/range o deal id | Decidir en plan.md: Sheet es más visible para Sandra; HubSpot Note es self-contained. |
| Notificación interna (TBC OQ-G7) | Email / WhatsApp / Slack node | depende del provider | Mismo nodo se va a reusar en los 4 flows de Blincer → candidato directo a pattern `n8n/patterns/internal-alert.md`. |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| HubSpot (Pro+) | OAuth 2.0 (Private App token recomendado) | 100 req / 10 s burst, 250k / día (Pro) | Webhook reentregas existen → usar `eventId` + `occurredAt`; PATCH de propiedades es idempotente por sí. | https://developers.hubspot.com/docs/api/overview |
| Tango Nexo (si aplica) | OAuth / API key (a confirmar con cuenta del cliente) | desconocido — Nexo tiene throttling no documentado público | No tiene primitivas de idempotencia → manejarla del lado n8n. | https://nexo.tangosoft.com/ (acceso del cliente) |
| Tango local (si aplica) | Acceso DB directo (ODBC/SQL Server más común) | n/a (DB local) | n/a | n/a — depende de la versión y schema |
| Canal de alerta (TBC) | depende | depende | n/a | — |

## 5. Base flow decision

- **Base flow chosen:** `none` (primer flow del cliente, no hay base).
- **Coverage estimate:** 0%
- **What we copy / what we change:** se arranca desde el template + decisiones tomadas en el spec. Cuando este flow llegue a `live`, su `workflow.json` queda como **base candidata** para futuros flows que sean "HubSpot webhook → lookup externo → flag back".

## 6. Reuse summary

- ✅ Reusing:
  - Estructura template Spec Kit (`n8n/templates/`).
  - Credencial HubSpot OAuth — se va a crear una sola para Blincer y la comparten los 4 flows de la tanda.
  - Lección "no decidir si el lookup externo está caído" (memoria del proyecto auditoría-facturas).
- 🆕 New work:
  - Custom properties HubSpot (`credit_limit_ars`, `block_invoice`, `tango_customer_id`) — alta en HubSpot antes del build.
  - Stages HubSpot ("Listo para facturar", "Bloqueado por deuda") — alta en el pipeline correspondiente.
  - Conector Tango (forma depende de OQ-G1).
  - Pattern `internal-alert` (compartido con los otros 3 flows).
- ⚠️ Risks identified:
  - Si Tango es local sin API, el flow real depende de exportes CSV programados → latencia ≥ 1 día. Eso choca con el success criterion de "menos de 60 s". Mitigación: bloqueo preventivo con la última snapshot disponible + alerta a Sandra con timestamp del último sync.
  - Si el mapeo HubSpot↔Tango (`tango_customer_id`) no existe, todos los lookups fallan en silencio. Mitigación: validar en el primer nodo y caer a rama de error con mensaje específico.
  - HubSpot webhook reentrega — riesgo de Notes duplicadas si idempotencia no se implementa antes del build (no después).
