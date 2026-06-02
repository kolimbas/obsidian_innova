---
tags:
  - n8n
  - node
node: Google Sheets
updated: 2026-06-02
---

# Google Sheets

← Volver a [[n8n/nodes/_index|Node Notes]]

## Why this note exists

El set de operaciones del nodo cambió entre versiones y el comportamiento de `append` no es obvio. Estos puntos costaron tiempo al cablear el backing de Sheets de los flows de Blincer (typeVersion **4.5**).

## Auth / setup

- En la instancia de Blincer la credencial OAuth2 que funciona es **`Google Sheets account`** (id `NNpCFCk3F2rhlxUk`), la misma que usan todos los `BLINCER-T0xx`. No hace falta una credencial por flow.
- **Convención de sharing del cliente:** los spreadsheets se comparten como *"cualquiera con el link = editor"*. Así la credencial OAuth2 puede leer/escribir sin importar a qué cuenta Google esté atada (verificado en el sheet productivo de T006).

## Proven configurations

- **Append** con `columns.mappingMode = autoMapInputData`: escribe las claves del JSON entrante. Los headers de la fila 1 de la tab deben coincidir con esas claves para que queden columnas prolijas.
- **documentId** se referencia por `{ "__rl": true, "value": "<spreadsheetId>", "mode": "id" }`.

## Gotchas

- ⚠️ **v4.5 NO tiene operación `lookup`** (existía en v1/v2). Un nodo con `operation: "lookup"` en v4.5 es inválido. Para buscar/deduplicar por valor: `operation: "read"` + `filtersUI.values: [{ lookupColumn, lookupValue }]`.
- ⚠️ **`read` con filtro que no matchea devuelve 0 items** → corta la rama (no propaga el item original). Para idempotencia del tipo "seguir si NO existe" no alcanza el `read` solo: hace falta un Code que lea la tab y compare en JS, **y** un nodo de *write-back* que inserte la clave después de procesar. Patrón completo en [[n8n/patterns/sheet-idempotency|Sheet-based idempotency]] (implementado en los 3 flows de Blincer el 2026-06-02: read `alwaysOutputData` + Code `runOnceForAllItems` + write-back).
- **autoMap append**: si el JSON trae claves que no están como header, n8n las agrega como columnas nuevas al final (puede ensuciar la tab si los nombres varían).

## Flows using this node

- [[n8n/clients/blincer/flows/credit-limit-invoice-block/spec|Bloqueo facturación por deuda]] (audit + idempotencia)
- [[n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/spec|Avisos deuda vencida WhatsApp]] (config/templates/log/fallback)
- [[n8n/clients/blincer/flows/email-remarketing/spec|Remarketing email]] (config/log/queue/metrics/suppression)
