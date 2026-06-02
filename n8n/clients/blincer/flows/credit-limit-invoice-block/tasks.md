---
tags:
  - n8n
  - tasks
  - blincer
  - nivel-3
client: blincer
flow: credit-limit-invoice-block
updated: 2026-06-02
status: blocked-by-oqs
---

# Tasks — Bloqueo automático de facturación por deuda

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/flows/credit-limit-invoice-block/plan|Plan]]

> Ordered, verifiable. Update as work progresses; do not delete completed items. **No empezar Build hasta que Setup esté 100% verde.**

> [!success] Progreso 2026-06-02 (Sheets backing)
> Hecho vía API n8n: spreadsheet **Blincer - Credit Limit** (`1kjsp67c8eKVTqj6FoHpKI6mTtqoTFsBSUdEv9E0gPaI`) creado con `audit_credit_block` + `idempotency_credit` (**falta** `errors_credit_block`). `REPLACE_SHEET_ID` reemplazado + credencial mapeada a `Google Sheets account`. `Idempotency lookup` quedó **disabled** (v4.5 sin op `lookup` + falta write-back). Siguen pendientes los placeholders no-Sheets: `REPLACE_STAGE_ID` (OQ-6), Tango (`REPLACE_TANGO_BASE`), alerta (`REPLACE_INTERNAL_ALERT_WEBHOOK`), credencial HubSpot real.

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] Confirmar respuestas a OQ-1 a OQ-6 del spec (responsable: Innova + cliente).
- [ ] Confirmar OQ-G1 (Tango versión) y OQ-G7 (canal alerta interno) globales.
- [ ] Si Tango es local: alinear con cliente el método de extracción (CSV programado vs ODBC) y aceptar latencia en el success criterion.
- [ ] HubSpot Pro+ confirmado + Private App token emitido con scopes: `crm.objects.deals.read/write`, `crm.objects.companies.read/write`, `crm.schemas.deals.write` (para custom property), `tickets.read` (si Notes corren por allí).
- [ ] Crear custom property `credit_limit_ars` (number, ARS) en HubSpot Company.
- [ ] Crear custom property `block_invoice` (bool, default false) en HubSpot Deal.
- [ ] Crear custom property `tango_customer_id` (string) en HubSpot Company.
- [ ] Crear stages "Listo para facturar" y "Bloqueado por deuda" en pipeline correspondiente; anotar slugs reales acá:
  - `dealstage_listo_facturar`: `__pendiente__`
  - `dealstage_bloqueado_deuda`: `__pendiente__`
- [ ] Backfill `tango_customer_id` en Companies relevantes (one-shot, script aparte — fuera del flow).
- [ ] Si Tango Nexo: obtener API key + endpoint + sample response para `/cuentas-corrientes` (o equivalente).

## Setup

- [ ] Crear credenciales en n8n:
  - [ ] `hubspot-blincer-main` (Private App token)
  - [ ] `tango-nexo-blincer` o conector local (según OQ-G1)
  - [ ] `gsheets-blincer-ops`
  - [ ] `internal-alert-blincer` (según OQ-G7)
- [ ] Crear Sheets / tablas:
  - [ ] `idempotency_credit_block` con columnas `eventId`, `occurredAt`, `deal_id`, `inserted_at`
  - [ ] `audit_credit_block` con columnas `timestamp`, `deal_id`, `company_id`, `current_debt`, `credit_limit`, `deal_amount`, `decision`, `duration_ms`
  - [ ] `errors_credit_block` con columnas `timestamp`, `deal_id`, `node`, `error`, `payload`
- [ ] Confirmar sandbox HubSpot disponible; si no, definir Deal de prueba `__test_credit_block__` en producción.
- [ ] Preparar test payloads (`./test-payloads/`): `deal_below_limit.json`, `deal_above_limit.json`, `deal_no_mapping.json`, `tango_timeout.json`.

## Build

- [ ] Crear workflow vacío en n8n: nombre `blincer / credit-limit-invoice-block`.
- [ ] Node 1: HubSpot Deal Trigger — event `deal.propertyChange`, propertyName `dealstage`.
- [ ] Node 2: IF filter sobre `propertyValue === '<dealstage_listo_facturar>'`.
- [ ] Node 3: Idempotency check (lookup en `idempotency_credit_block`). Branch: seen → end; new → insert + continue.
- [ ] Node 4: HubSpot — Get Deal con `includeAssociations=true`.
- [ ] Node 5: HubSpot — Get Company por `associations.company.id`.
- [ ] Node 6: IF validate mapping (`tango_customer_id` no vacío). Error branch si vacío.
- [ ] Node 7: HTTP Request a Tango — GET deuda. Timeout 10s, retry 1×.
- [ ] Node 8: Function — calcular `proyectado` y `block`.
- [ ] Node 9: IF block / allow.
- [ ] Node 10a (allow): Sheets append en `audit_credit_block` con `decision='allowed'`.
- [ ] Node 10b (block): HubSpot Update Deal — `block_invoice=true`, mover stage a `__dealstage_bloqueado_deuda__`.
- [ ] Node 11: HubSpot Create Note en Deal con texto plantilla.
- [ ] Node 12: HubSpot Create Note en Company.
- [ ] Node 13: Send internal alert (canal según OQ-G7).
- [ ] Node 14: Sheets append en `audit_credit_block` con `decision='blocked'`.
- [ ] Error branch E1 (no mapping): Note "Falta mapeo" + alerta + append a `errors_credit_block`.
- [ ] Error branch E2 (Tango down): tratar como block + alerta crítica + append a `errors_credit_block`.
- [ ] Configurar retry policy 3× backoff exponencial en todos los nodos HubSpot/HTTP.
- [ ] Run `mcp__n8n-mcp__validate_node` en cada nodo.
- [ ] Run `mcp__n8n-mcp__n8n_validate_workflow` en el workflow completo.

## Validate

- [ ] Ejecutar con `deal_below_limit.json` → assert: `audit_credit_block` tiene row `allowed`, deal sin cambios.
- [ ] Ejecutar con `deal_above_limit.json` → assert: deal con `block_invoice=true`, stage cambiado, 2 Notes creadas, alerta recibida, audit row `blocked`.
- [ ] Ejecutar con `deal_no_mapping.json` → assert: cae a E1, alerta enviada, `errors_credit_block` con row, deal sin cambios.
- [ ] Ejecutar con Tango mockeado timeout → assert: E2 dispara, alerta crítica, deal marcado bloqueado por fail-safe, audit row con flag `failsafe=true`.
- [ ] Reentregar el mismo webhook 2× → assert: la segunda no duplica Notes ni alertas (idempotency).
- [ ] Cambiar `credit_limit_ars` en Company y reentregar → assert: usa el nuevo valor (no cachea).
- [ ] p95 duración < 60s sobre 50 ejecuciones de prueba.

## Harden

- [ ] Retries configurados en HubSpot (3×), HTTP Tango (1× a 5s + fallback fail-safe), alertas (3× con fallback email).
- [ ] Alert wiring confirmado al canal OQ-G7.
- [ ] Rate-limit revisado: HubSpot 100/10s burst — concurrency del workflow ≤ 5 paralelas.
- [ ] Cron diario 09:00 ART que lee `errors_credit_block` y notifica si hay rows nuevos en últimas 24h.

## Document

- [ ] Exportar workflow como `workflow.json` en esta carpeta.
- [ ] Actualizar `spec.md` status → `live`.
- [ ] Actualizar fila correspondiente en `n8n/clients/blincer/README.md` (status + Last update).
- [ ] Anotar slugs reales de stages en `tasks.md` § Pre-build.

## Handoff

- [ ] Notificar a Sandra + Guillermo con execution URL + cómo monitorear (link a HubSpot view filtered por `block_invoice=true`).
- [ ] Escribir `retro.md` con: tiempos, sorpresas, gotchas de HubSpot/Tango, qué se promueve a pattern.
- [ ] Promover el shape genérico "HubSpot webhook → external lookup → flag back" a `n8n/patterns/hubspot-external-flag.md`.
- [ ] Promover el pattern de alerta interna a `n8n/patterns/internal-alert.md` (compartido con los otros 3 flows de Blincer).
- [ ] Append one-liner a `n8n/lessons-learned.md`.
