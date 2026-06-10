---
tags:
  - n8n
  - research
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: track-record-validation
updated: 2026-06-10
---

# Research — F · Track-record validation & cross-check

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/track-record-validation/spec|Spec]]

> [!warning] CARVE-OUT — feasibility-gated
> No es solo "el flow más complejo": su **viabilidad** depende de OQ-F-1 (Elementix API). Este research prepara el terreno pero no se planifica build real hasta confirmar acceso a datos.

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/gpt-landings/flows/` (2026-06-10).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `document-intake-form` (D) | 🟢 Alta | Dispara F con cada doc nuevo + provee el archivo. |
| `m0-infra-setup` | 🟡 Media | DB + creds + budget Claude. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, other `n8n/clients/*/flows/` (2026-06-10).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/clients/blincer/flows/sales-bot-with-quotes/research.md` | Pipeline OCR + LLM + validación post-LLM (no alucinar datos) | Reusar el enfoque de extracción estructurada + guardrails. |
| `n8n/clients/blincer/flows/credit-limit-invoice-block/research.md` (+ memoria auditoría-facturas) | "No decidir si el lookup externo está caído" (fail-safe ante fuente externa) | **Aplica a Elementix**: si la fuente no responde, no afirmar match — flag "no verificado" + humano. |
| `n8n/patterns/sheet-idempotency.md` | Dedup por doc | Validar cada doc una sola vez. |

## 3. n8n nodes considered

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| OCR (servicio/Claude vision) | Extraer texto/campos del doc | model | budget (OQ-F-3) |
| LLM (Claude) vía `httpRequest`/langchain | Estructurar campos + comparar | model, tools | salida estructurada; nunca afirmar match sin fuente |
| `n8n-nodes-base.httpRequest` (Elementix/public records) | Cross-check | **depende OQ-F-1 (API vs UI)** | si solo UI → no hay node directo (scraping/manual) |
| `n8n-nodes-base.postgres` | Guardar validación + flags | update | idempotente por doc |
| alert | Flags al equipo | — | canal OQ-0.4 |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| **Elementix** | **¿API? ¿solo UI?** (OQ-F-1) | desconocido | n/a | **a confirmar — bloqueante** |
| Public records (county/FL) | varía por fuente | varía | n/a | a relevar |
| Claude API / OCR | API key | por tier | n/a | skill `claude-api` |
| Supabase/Postgres | conn string | n/a | dedup por doc | https://supabase.com/docs |

## 5. Base flow decision

- **Base flow chosen:** `none` (es el módulo más nuevo y de mayor incertidumbre).
- **Coverage estimate:** ~10% (solo el shape OCR+LLM y el principio fail-safe ante fuente externa).
- **What we copy / what we change:** copiamos extracción OCR+LLM con guardrails y el fail-safe de fuente externa; todo el cross-check contra Elementix es nuevo y **condicional**.

## 6. Reuse summary

- ✅ Reusing: OCR+LLM con guardrails (`sales-bot-with-quotes/research.md`), fail-safe ante lookup externo caído (`credit-limit-invoice-block/research.md`), `sheet-idempotency`.
- 🆕 New work: integración con Elementix/public records (condicional a OQ-F-1); criterios de match entidad↔dirección; revisión humana de flags.
- ⚠️ Risks identified: **OQ-F-1 define la viabilidad**; si Elementix es solo UI → scraping frágil/baneo o entrada manual; costo OCR+Claude por volumen; falsos positivos/negativos en el match → por eso validación humana final obligatoria.
