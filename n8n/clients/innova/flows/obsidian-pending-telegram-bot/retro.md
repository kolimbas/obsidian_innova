---
tags:
  - n8n
  - retro
  - innova
  - nivel-3
client: innova
flow: obsidian-pending-telegram-bot
status: live
updated: 2026-06-10
---

# Retro — Obsidian pending → Telegram bot

← Volver a [[n8n/METHODOLOGY|Methodology]]

---

## Outcome

- Status at close: **live** (ambos workflows activos el 2026-06-10).
- Owner now: Innova.

## What worked

- Validar el motor de parseo **localmente** (Node, contra el repo real) antes de tocar n8n → cero debugging a ciegas.
- Repo público → lectura sin auth ni costo; sin LLM, gratis.
- Reusar el mismo Code node en los 2 workflows.

## What didn't (y cómo se arregló)

- **Primera versión demasiado greedy:** contaba *todo* `- [ ]` (success-criteria, build-steps) → 813 "pendientes". Fix: extracción **section-aware** (solo bajo `[!todo]` o headings de pendientes) + resumir flows por `status`. Bajó a ~56.
- **Errores 400 intermitentes en Telegram:** el nodo parsea **HTML por default**; `<client>-<system>-<env>` se lee como tag → "can't parse entities". Fix: **escapar `< > &`** y fijar `parse_mode: HTML`.

## Surprises (non-obvious worth keeping)

- En el **Code node** de n8n el helper es **`$helpers.httpRequest`** (no `this.helpers`) y el static data es **`$getWorkflowStaticData`** (no `this.getWorkflowStaticData`). Dejé fallback a `this.*` por las dudas.
- El nodo Telegram **siempre** aplica un parse_mode (default HTML) → texto con `<`, `>`, `&`, o `_`/`*` desbalanceados rompe. Mandar plano = escapar para HTML.

## Reusable bits

| Bit | Promoted to | Notes |
| --- | --- | --- |
| Bajar un repo de GitHub y escanearlo en un Code node | `n8n/patterns/github-repo-scan.md` | candidato |
| Telegram: escapar HTML antes de enviar | `n8n/nodes/telegram.md` | candidato (node note) |
| Extracción de pendientes section-aware de Markdown | `n8n/patterns/markdown-pending-extract.md` | candidato |

## Time

- Estimated: —
- Actual: ~1 sesión.

## Follow-ups

- [ ] Auto-sync del vault (SSH + timer).
- [ ] Promover los patterns/node note de arriba.

## One-liner for lessons-learned

> El nodo Telegram de n8n parsea HTML por default: escapar `< > &` (o el texto con `<...>`/`_` rompe con 400 "can't parse entities"). En Code nodes usar `$helpers.httpRequest` / `$getWorkflowStaticData`, no `this.*`.
