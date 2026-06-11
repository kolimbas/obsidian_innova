---
tags:
  - n8n
  - retro
  - innova
  - nivel-3
client: innova
flow: obsidian-pending-telegram-bot
status: live
updated: 2026-06-11
---

# Retro — Obsidian pending → Telegram bot

← Volver a [[n8n/METHODOLOGY|Methodology]]

---

## Outcome

- Status at close: **live** (ambos workflows activos).
- 2026-06-11: ampliado de "solo lectura" a **asistente interactivo** — comandos de lectura (`/hoy`, `/bloqueantes`, `/buscar`, `/resumen`, `/help`), **captura** al vault (`/tarea`, `/nota`, `/hecho` vía GitHub PAT), menú `/`, botones inline, y render HTML lindo (negritas + citas plegables).
- Owner now: Innova.

## What worked

- Validar el motor de parseo **localmente** (Node, contra el repo real) antes de tocar n8n → cero debugging a ciegas.
- Repo público → lectura sin auth ni costo; sin LLM, gratis.
- Reusar el mismo Code node en los 2 workflows.

## What didn't (y cómo se arregló)

- **Primera versión demasiado greedy:** contaba *todo* `- [ ]` (success-criteria, build-steps) → 813 "pendientes". Fix: extracción **section-aware** (solo bajo `[!todo]` o headings de pendientes) + resumir flows por `status`. Bajó a ~56.
- **Errores 400 intermitentes en Telegram:** el nodo parsea **HTML por default**; `<client>-<system>-<env>` se lee como tag → "can't parse entities". Fix: **escapar `< > &`** y fijar `parse_mode: HTML`.
- **Storm de mensajes (2026-06-11):** al tocar varios botones seguidos, el Code node bajaba ~100 archivos en *cada* ejecución → ejecuciones de ~2 min → Telegram reintrega el webhook (timeout) → bucle de respuestas. Fix de raíz: **cache de 60s** del vault en static data (botones seguidos reusan el cache, ejecución ~1-2s). Para cortar el incendio en vivo: `deleteWebhook?drop_pending_updates=true` + desactivar el WF; reactivar tras el fix. ⚠️ La **escritura no usa el cache**: lee el archivo fresco e invalida el cache (`SD._fcTs=0`), si no un `/tarea` pisaría una captura previa con contenido viejo.
- **Captura sin pisar la sucursal de auto-sync:** el commit del bot (GitHub API) y el push local (SSH timer) conviven porque tocan archivos dedicados (`inbox.md`, sección `## Capturado por bot`) y el GitHub node toma sha fresco; el pull --rebase del timer los integra.

## Surprises (non-obvious worth keeping)

- En el **Code node** de n8n el helper es **`$helpers.httpRequest`** (no `this.helpers`) y el static data es **`$getWorkflowStaticData`** (no `this.getWorkflowStaticData`). Dejé fallback a `this.*` por las dudas.
- El nodo Telegram **siempre** aplica un parse_mode (default HTML) → texto con `<`, `>`, `&`, o `_`/`*` desbalanceados rompe. Mandar plano = escapar para HTML.
- **`<blockquote expandable>`** (Bot API 7.0+) es la mejor arma contra el "choclo de texto": cada proyecto colapsado, se expande con un toque. Truco: escapar **solo el contenido dinámico** y dejar las tags sin escapar (un flag `out(msg, html=true)` que saltea el escapeo global). Ojo: el contenido colapsado **igual cuenta** para el límite de 4096 → cap por ítem/proyecto + truncado que cierra `</blockquote>`.
- **`callback_query` (botones):** sin `answerCallbackQuery` el botón queda con el spinner girando; un nodo Telegram `answerQuery` aparte (con `onError: continueRegularOutput` para tolerar los updates `message` sin `callback_query.id`) lo corta. El `chatId` del reply debe contemplar `message.chat.id` **o** `callback_query.message.chat.id`.
- Un **reply keyboard / inline keyboard en el digest diario** dispara taps que caen en el webhook del on-demand (un bot = un webhook) → el diario queda interactivo sin webhook propio.

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

- [x] Auto-sync del vault (SSH + timer).
- [ ] Promover los patterns/node note de arriba (sumar: cache anti-storm, blockquote expandable, captura write-back con GitHub node).

## One-liner for lessons-learned

> Un bot de Telegram sobre webhook que tarda > el timeout (acá: bajar ~100 archivos por ejecución) entra en **storm de reintentos**; la cura es hacer la ejecución rápida (cache de 60s) — y para escrituras, leer el archivo **fresco** (no del cache) para no pisar datos. Bonus: `<blockquote expandable>` mata el choclo de texto, escapando solo el contenido y no las tags.
