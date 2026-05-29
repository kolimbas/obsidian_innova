---
tags:
  - n8n
  - retro
  - blincer
  - nivel-3
client: blincer
flow: credit-limit-invoice-block
updated: 2026-05-29
status: not-yet-live
---

# Retro — Bloqueo automático de facturación por deuda

← Volver a [[n8n/METHODOLOGY|Methodology]]

> Mandatory. Se completa **después** de que el flow esté `live`. Por ahora queda como placeholder.

---

## Outcome

- Status at close: _pendiente — flow todavía no construido_
- Date live: _pendiente_
- Owner now: _pendiente_

## What worked

-

## What didn't

-

## Surprises (non-obvious things worth keeping)

-

## Reusable bits

| Bit | Promoted to | Notes |
| --- | --- | --- |
| HubSpot webhook → external lookup → flag back | `n8n/patterns/hubspot-external-flag.md` | candidato fuerte (ver `tasks.md` § Handoff) |
| Pattern de alerta interna | `n8n/patterns/internal-alert.md` | compartido con los otros 3 flows de Blincer |
| Idempotency por `eventId + occurredAt` con TTL | `n8n/patterns/webhook-idempotency.md` | reusable en todo flow con HubSpot Trigger |

## Node gotchas

- _pendiente_

## Time

- Estimated: _por estimar al cerrar OQs_
- Actual: _pendiente_
- Delta drivers: _pendiente_

## Follow-ups

- [ ] _pendiente_

## One-liner for lessons-learned

> _pendiente — completar al cerrar el flow_
