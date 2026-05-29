---
tags:
  - n8n
  - spec
  - blincer
  - nivel-3
client: blincer
flow: credit-limit-invoice-block
status: draft
updated: 2026-05-29
---

# Spec — Bloqueo automático de facturación por deuda

> Ningún cliente con deuda viva por encima de su límite de crédito recibe una nueva factura sin revisión humana.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/README|Blincer]]

---

## Goal

Cuando un Deal de Blincer está listo para facturarse, el flujo compara la deuda viva del cliente contra su límite de crédito y, si lo supera, **interrumpe la facturación**: marca el Deal con un flag de bloqueo, deja nota explicando por qué, y avisa a Sandra para que decida (aprobar, ajustar límite, gestionar cobranza). Si no lo supera, el Deal queda libre para continuar (manualmente o disparando el flujo de facturación — fuera de scope acá).

## Trigger

- **Type:** webhook (HubSpot)
- **Schedule / endpoint / event:** `deal.propertyChange` sobre `dealstage`, filtrado a la stage "Listo para facturar" (slug exacto a confirmar con el cliente)
- **Expected frequency:** estimado 5–30 deals/día en estado estable (dato a calibrar después de 2 semanas en producción)

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `deal_id` | string | HubSpot webhook | yes | clave para fetchear el deal completo |
| `deal_amount` | number (ARS) | HubSpot Deal `amount` | yes | monto a facturar |
| `company_id` / `contact_id` | string | HubSpot Deal associations | yes | resolver el cliente cuya deuda hay que mirar |
| `tango_customer_id` | string | HubSpot Company custom property | yes | mapeo HubSpot ↔ Tango (sin esto no hay query de deuda) |
| `credit_limit` | number (ARS) | HubSpot Company custom property `credit_limit_ars` | yes | tope vigente, editado por Sandra |
| `current_debt` | number (ARS) | Tango cuenta corriente del cliente | yes | suma de saldo pendiente — fuente concreta a definir según OQ-1 |

Sample payload (HubSpot webhook):

```json
{
  "subscriptionType": "deal.propertyChange",
  "objectId": 38501094821,
  "propertyName": "dealstage",
  "propertyValue": "12345678",
  "occurredAt": 1748528400000,
  "sourceId": "portal-99999999",
  "portalId": 99999999
}
```

## Outputs / Side effects

- **HubSpot Deal:** setea `block_invoice = true` (custom property booleano, a crear) y agrega Note con texto `"Facturación bloqueada: deuda ARS X supera límite ARS Y al {timestamp}"`. Mueve el deal a stage "Bloqueado por deuda" (slug a confirmar).
- **HubSpot Company:** agrega Note con la misma info (trazabilidad por cliente).
- **Notificación a Sandra:** mensaje al canal de alerta interno (TBC — OQ-G7) con `deal_url`, monto, deuda actual, límite, link al cliente en Tango si aplica.
- **Tango:** ninguno (no escribimos en Tango; el bloqueo es declarativo del lado HubSpot/operativo).
- **Log:** registro de cada decisión (allow/block) en Sheet `n8n_audit_credit_block` o en HubSpot Deal timeline — formato a decidir en plan.md.

## Success criteria

- [ ] Un deal con `deal_amount` que llevaría la deuda total > `credit_limit` queda con `block_invoice = true` y en stage "Bloqueado por deuda" **en menos de 60 s** desde el cambio de stage en HubSpot.
- [ ] Un deal con deuda + monto ≤ `credit_limit` **no es modificado** por el flujo (sólo deja log "allowed").
- [ ] Sandra recibe alerta en el canal definido con todos los datos requeridos (deal, cliente, montos).
- [ ] Re-ejecución del mismo webhook (HubSpot a veces reentrega) **no duplica** Notes ni alertas (idempotencia por `deal_id + propertyChange occurredAt`).
- [ ] Si Tango está caído / no responde dentro de 10 s → flujo cae a rama de error: deja Note "Bloqueo no verificado — Tango sin respuesta" y notifica a Sandra (fail-safe: bloquear ante duda).

## Out of scope

- Emisión real de la factura en Tango (eso es trabajo del flujo `sales-bot-with-quotes` o de un workflow de facturación masiva separado).
- Gestión de cobranza (recordatorios) — eso vive en `whatsapp-overdue-debt-reminder`.
- Cambio del límite de crédito por parte de Sandra (lo edita manualmente en HubSpot Company).
- Bloqueo retroactivo de facturas ya emitidas.
- Manejo de moneda extranjera / conversiones (se asume ARS — confirmar).

## Open questions

- [ ] **OQ-1 — Cómo obtenemos `current_debt` de Tango.** Depende de OQ-G1 (versión Tango):
  - Si Nexo → endpoint REST `/cuentas-corrientes/cliente/{id}` o equivalente.
  - Si local → CSV export programado o consulta directa a la DB (con la DB típica de Tango local, vía ODBC).
- [ ] **OQ-2 — Ubicación canónica del `credit_limit`.** Propuesta: HubSpot Company custom property `credit_limit_ars` editable por Sandra. Alternativa: Tango (más cerca de Sandra pero más friction para n8n). Pregunta clave: ¿quién y desde dónde lo edita hoy?
- [ ] **OQ-3 — Qué cuenta como "deuda".** ¿Saldo de cuenta corriente Tango? ¿Facturas emitidas no cobradas? ¿Cheques en cartera? ¿Suma o cada uno con su tope?
- [ ] **OQ-4 — Bloqueo preventivo (alerta) o efectivo.** Variantes:
  - **A — Sólo alerta:** mensaje a Sandra, no se mueve el deal. La emisión sigue siendo manual.
  - **B — Flag + stage:** lo descripto arriba. Bloqueo es declarativo del lado HubSpot.
  - **C — Bloqueo duro:** requeriría modo "approval gate" antes del flow de facturación (interlock real). Asume que la emisión está integrada con n8n. Más invasivo.
- [ ] **OQ-5 — Mapeo HubSpot ↔ Tango.** ¿Existe ya un `tango_customer_id` en HubSpot Company? Si no, hay un step previo de backfill que no entra en este flow.
- [ ] **OQ-6 — Stage exacta en HubSpot.** ¿Cuál es el slug real de "Listo para facturar" y "Bloqueado por deuda"? Si no existen, alguien las crea en HubSpot antes del build.

*(OQ-G1, OQ-G7 son globales — ver [[n8n/clients/blincer/README|Blincer README]] · "Open dependencies").*

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Guillermo (decisión comercial) + Sandra (operación contable) |
| Approver | Guillermo |
| End user (operativo) | Sandra (recibe la alerta y decide) |
| End user (afectado) | Vendedor/a que intentó facturar (queda informado vía HubSpot Deal Note) |
