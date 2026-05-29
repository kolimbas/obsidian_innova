---
tags:
  - n8n
  - retro
  - blincer
  - nivel-3
client: blincer
flow: sales-bot-with-quotes
updated: 2026-05-29
status: not-yet-live
---

# Retro — Bot de ventas + cotizaciones/facturas automáticas

← Volver a [[n8n/METHODOLOGY|Methodology]]

> Mandatory. Por la complejidad, recomendación: una retro al cierre de **cada fase** (1 a 5), no solo al final.

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
| Chat bot con tool-use | `n8n/patterns/chat-bot-with-tool-use.md` | base para futuros bots de otros clientes |
| Render PDF (Gotenberg + DOCX template) | `n8n/patterns/quote-pdf-generation.md` | reusable para cualquier cotización formal |
| Handoff a humano (Task + alert + transición) | `n8n/patterns/handoff-human.md` | patrón estándar |
| Budget cap LLM con alertas escalonadas | `n8n/patterns/llm-budget-cap.md` | aplica a cualquier flow LLM |
| Anti-hallucination: validación post-LLM de números devueltos | `n8n/patterns/llm-grounding-check.md` | crítico para cualquier bot con precios |

## Node gotchas

- _pendiente_

## Time

- Estimated: _por estimar (probablemente 80–120 hs sumando 5 fases)_
- Actual: _pendiente_
- Delta drivers: _pendiente_

## Follow-ups

- [ ] Multi-channel (sumar web chat) — V2
- [ ] Media inbound (cliente manda foto) — V2
- [ ] Link de pago integrado (MercadoPago) — V2
- [ ] Multi-idioma — V3

## One-liner for lessons-learned

> _pendiente_
