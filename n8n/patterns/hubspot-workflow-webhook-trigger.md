---
tags: [n8n, pattern]
problem: Disparar un workflow n8n ante un cambio en HubSpot sin depender del HubSpot Trigger (developer app + App ID)
updated: 2026-06-02
---

# Pattern — HubSpot Workflow → n8n Webhook

## Problem

El **HubSpot Trigger** nativo de n8n (`n8n-nodes-base.hubspotTrigger`) usa la **Developer API**: requiere
una credencial `hubspotDeveloperApi` con **Developer API Key + App ID** y una **app de desarrollador** con la
suscripción de webhook registrada contra el portal de producción. Es frágil y burocrático cuando lo único que
querés es "cuando un Deal/Contact cambia, avisá a n8n". Para eso alcanza con un Private App token (acciones) +
un disparo simple.

## When to use

- Tenés un **Private App token** (acciones HubSpot) pero **no** querés montar la developer app sólo para el trigger.
- El disparo es un evento de automatización ("Deal entra a stage X", "Contact entra a lista Y") que HubSpot
  Workflows puede detectar.
- Querés que el contrato de payload sea explícito y versionable, no el shape opaco del webhook de la dev API.

## When NOT to use

- Necesitás eventos de bajo nivel que HubSpot Workflows no expone (ej. cambios de propiedad arbitrarios a alta
  frecuencia, o cuentas sin Operations/Marketing que habiliten webhooks en Workflows).
- Volumen muy alto donde el rate de Workflows-webhook sea un cuello (ahí sí dev API + colas).

## Implementation

**Lado n8n:**
1. Nodo **Webhook** (`n8n-nodes-base.webhook`, `httpMethod: POST`, `path` estable, ej. `blincer-credit-limit`).
   La URL de producción es `<N8N_BASE>/webhook/<path>`.
2. Nodo **`Normalize webhook`** (Code, `runOnceForAllItems`) justo después: aísla el shape de HubSpot del resto
   del flow. Lee de `it.json.body` (el Webhook node mete el cuerpo bajo `.body`) y emite los **campos canónicos**
   que el flow ya esperaba del trigger viejo (ej. `objectId`, `propertyValue`, `eventId`, `occurredAt`). Hacelo
   **defensivo**: acepta `body.dealId`/`body.objectId`/`body.properties.hs_object_id.value`, etc., así tolera
   tanto un payload custom como el estándar de HubSpot.
3. Para la **dedup**, derivá `eventId` del body si viene; si no, fallback determinístico (ej.
   `objectId + '_' + dealstage`) → idempotencia "una vez por (objeto, estado)". Documentá la semántica.

**Lado HubSpot (lo arma el cliente):**
- Crear un **Workflow nativo**: enrollment "Deal entra a stage X" → acción **Send webhook (POST)** apuntando a
  `<N8N_BASE>/webhook/<path>`, con payload que incluya al menos el `hs_object_id` y la propiedad relevante.

> Migrar de HubSpot Trigger a este patrón = borrar el nodo trigger, insertar `Webhook → Normalize webhook`
> antes del primer filtro, y reapuntar las expresiones `$('<Trigger>')` → `$('Normalize webhook')`.

## Example flows using this

- [[n8n/clients/blincer/flows/credit-limit-invoice-block/plan|Blincer · Credit Limit Invoice Block]] — migrado de
  `hubspotTrigger` a `Webhook (HubSpot)` + `Normalize webhook` el 2026-06-02 (path `blincer-credit-limit`).

## Variants / known issues

- **Shape del payload:** el "Send webhook" de HubSpot Workflows manda su propio JSON; por eso el `Normalize`
  debe ser defensivo. Si controlás el payload (custom code action), mandá `{dealId, dealstage, eventId}` y listo.
- **Idempotencia sin eventId:** sin un id de evento real, la dedup colapsa a "una vez por (objeto, estado)";
  re-entradas al mismo stage se deduplican. Si querés re-evaluar en cada entrada, incluí un `eventId`/timestamp
  único en el payload.
- **Seguridad:** el webhook queda público; agregá un header secreto (validado en `Normalize`) o IP allowlist si
  el evento es sensible.
