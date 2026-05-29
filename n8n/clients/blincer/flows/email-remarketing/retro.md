---
tags:
  - n8n
  - retro
  - blincer
  - nivel-3
client: blincer
flow: email-remarketing
updated: 2026-05-29
status: not-yet-live
---

# Retro — Remarketing y difusiones por email

← Volver a [[n8n/METHODOLOGY|Methodology]]

> Mandatory. Se completa **después** de que el flow esté `live`.

---

## Outcome

- Status at close: _pendiente_
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
| Bulk email send con tracking | `n8n/patterns/bulk-email-with-tracking.md` | base para campañas de otros clientes |
| Suppression sweep nightly | `n8n/patterns/suppression-sweep.md` | aplica a cualquier flow de email outbound |
| Queue overflow para respetar daily_cap | `n8n/patterns/queue-overflow.md` | aplica a WhatsApp y Email |
| Dry-run gate como salvavidas | `n8n/patterns/dry-run-gate.md` | aplica a todo flow con efectos masivos |

## Node gotchas

- _pendiente_

## Time

- Estimated: _por estimar_
- Actual: _pendiente_
- Delta drivers: _pendiente_

## Follow-ups

- [ ] A/B testing automático (V2).
- [ ] Personalización via LLM (V2).
- [ ] Sequence campaigns por comportamiento (V2).

## One-liner for lessons-learned

> _pendiente_
