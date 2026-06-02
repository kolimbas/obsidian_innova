---
tags: [n8n, pattern]
problem: Deduplicar eventos/ítems para no repetir efectos externos, usando un Google Sheet como store
updated: 2026-06-02
---

# Pattern — Sheet-based idempotency (dedup)

← Volver a [[n8n/patterns/_index|Patterns]]

## Problem

Un flow con efectos hacia afuera (mandar WhatsApp/email, bloquear un deal, emitir factura) puede dispararse o re-ejecutarse con la misma entrada (HubSpot reentrega webhooks, un cron corre dos veces, un retry manual). Sin dedup se duplican los efectos. Hace falta garantizar **idempotencia**: procesar cada clave una sola vez.

## When to use

- Hay un efecto externo no-idempotente por sí mismo.
- Existe una **clave natural** que identifica "este evento ya lo hice" (ej. `eventId`, `(invoice_id, cadence_day)`, `(campaign_id, contact_id)`).
- El volumen es bajo/medio (cron diario, webhooks de a uno) — leer la tab entera por corrida es barato.

## When NOT to use

- Alto volumen / alta concurrencia → usar Postgres con `UNIQUE` + `INSERT ON CONFLICT`, o Redis con TTL, en vez de un Sheet (race conditions y límite de filas).
- El efecto ya es idempotente de por sí (upsert por id) → no hace falta store aparte.

## Implementation (n8n, Google Sheets v4.5)

Tres piezas. Clave: googleSheets v4.5 **no tiene op `lookup`** (ver [[n8n/nodes/google-sheets|node note]]).

1. **Read (claves vistas)** — nodo `googleSheets` `operation: read` sobre la tab de idempotencia, **`alwaysOutputData: true`** (para que el filtro corra aunque la tab esté vacía). No usa filtro: trae todas las claves.
2. **Dedup filter** — nodo `code`, `mode: runOnceForAllItems`. Arma un `Set` de las claves del read (`$('<read>').all()`), toma los ítems reales del nodo **upstream por nombre** (`$('<upstream>').all()` — el read pisa el item, por eso se referencia el upstream), y devuelve solo los que **no** están en el set, cargando la clave computada (`_idem_key`) para el write-back.
3. **Write-back** — tras el efecto, `append` de la clave a la tab. Va en rama **terminal** o fan-out (nunca inline en el camino principal: el append pisa el item). En flows que ya loguean la clave (ej. `campaigns_log` con `campaign_id`+`contact_id`), ese log **es** el write-back.

> El write-back va **después** del efecto, no antes: si se marca como visto y el envío falla, se perdería un envío legítimo.

## Example flows using this

- [[n8n/clients/blincer/flows/credit-limit-invoice-block/spec|credit-limit]] — clave `eventId` (o `objectId_occurredAt`) en `idempotency_credit`; write-back desde ambas ramas terminales.
- [[n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/spec|whatsapp-overdue]] — clave `(invoice_id, cadence_day)` → `dedup_key`; write-back fan-out desde `Log sent`.
- [[n8n/clients/blincer/flows/email-remarketing/spec|email-remarketing]] — clave `(campaign_id, contact_id)`; usa `campaigns_log` como store y como write-back.

## Variants / known issues

- **`read` filtrado** (en vez de read-all + Code) devuelve 0 ítems si no matchea → corta la rama. Por eso se usa read-all + filtro en Code.
- **Concurrencia:** dos corridas simultáneas pueden ambas leer "no visto" antes de escribir. Para cron diario no pasa; si hubiera solapamiento, serializar (`maxConcurrency: 1`) o migrar a Postgres.
- **Crecimiento de la tab:** sin TTL crece indefinido. Para dunning conviene podar por fecha; para campañas el `(campaign_id, contact_id)` es acotado.
