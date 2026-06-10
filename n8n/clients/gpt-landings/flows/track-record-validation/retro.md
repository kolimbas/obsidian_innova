---
tags:
  - n8n
  - retro
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: track-record-validation
updated: 2026-06-10
status: not-yet-live
---

# Retro — F · Track-record validation & cross-check

← Volver a [[n8n/METHODOLOGY|Methodology]]

> Mandatory. Se completa **después** de que el flow esté `live`. Por ahora placeholder — **módulo en carve-out condicional, no construido**.

---

## Outcome

- Status at close: _pendiente — carve-out condicional, no construido (depende de OQ-F-1: Elementix API)_
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
| OCR + LLM con guardrails (no afirmar sin fuente) | `n8n/patterns/ocr-llm-extract-guardrails.md` | candidato |
| Fail-safe ante fuente externa caída | `n8n/patterns/external-source-failsafe.md` | candidato (compartido con credit-limit) |

## Node gotchas

- _pendiente_ (Elementix API/UI; OCR sobre PDFs financieros)

## Time

- Estimated: _por estimar (condicional)_
- Actual: _pendiente_
- Delta drivers: _pendiente_

## Follow-ups

- [ ] Documentar el resultado de OQ-F-1 y su impacto en alcance/precio.

## One-liner for lessons-learned

> _pendiente — clave: factibilidad de fuentes externas (API vs UI) define la viabilidad de un módulo entero._
