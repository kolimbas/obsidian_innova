---
tags:
  - n8n
  - research
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: partner-matching-engine
updated: 2026-06-10
---

# Research — A · Capital-partner matching engine

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/partner-matching-engine/spec|Spec]]

> Always run, even if the answer is "nothing relevant." The point is to make reuse the default.

---

## 1. Prior flows of this client

Folder scanned: `n8n/clients/gpt-landings/flows/` (2026-06-10).

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `m0-infra-setup` | 🟢 Alta | Provee la DB (`capital_partners`, `prestamos`) y credenciales base. |
| `term-sheet-generation` (B) | 🟢 Alta | **Consume** el JSON de salida de A — contrato de interfaz. |

## 2. Cross-client patterns

Folders scanned: `n8n/patterns/`, other `n8n/clients/*/flows/` (2026-06-10).

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/clients/blincer/flows/sales-bot-with-quotes/research.md` | Input variable → schema único + tool-use con LLM, prohibir al LLM "inventar" datos del catálogo | Mismo principio: normalizar input multi-formato y **prohibir a Claude decidir un hard pass** sin regla. |
| `n8n/patterns/sheet-idempotency.md` | Config editable en Sheet/DB | Guidelines de partners como datos editables por el cliente sin tocar n8n. |
| skill `claude-api` | SDK/params de Claude | Referencia al construir el step de "casos borrosos". |

## 3. n8n nodes considered

A validar con `mcp__n8n-mcp__search_nodes` / `get_node` durante el build. Pre-selección:

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `n8n-nodes-base.webhook` | Recibir loan intake | path estable | normalizar `body` (ver patrón webhook de Blincer C) |
| `n8n-nodes-base.code` | Normalizar a schema único + motor de reglas duras | JS | mantener reglas como datos, no hardcode |
| `n8n-nodes-base.postgres` (o Supabase) | Leer `capital_partners` + escribir `partner_match_json` | query/upsert | upsert por `loan_id` |
| LLM (Claude) vía `httpRequest`/langchain | Resolver casos borrosos | model, tools | forzar salida estructurada; nunca decidir hard pass |
| `n8n-nodes-base.if`/`switch` | Branch hard-pass vs borroso | condition | — |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| Supabase/Postgres | service role / conn string | n/a | upsert `ON CONFLICT (loan_id)` | https://supabase.com/docs |
| Claude API | API key (`gptlandings-claude`) | por tier | n/a (idempotencia a nivel loan) | ver skill `claude-api` |

## 5. Base flow decision

- **Base flow chosen:** `none` (no hay motor de reglas en el vault).
- **Coverage estimate:** ~15% (solo el shape de normalización + LLM del sales-bot).
- **What we copy / what we change:** copiamos el principio "reglas duras primero, LLM acotado y sin alucinar"; el motor de matching es nuevo y queda como **base** del pattern `rules-engine-with-llm-fallback`.

## 6. Reuse summary

- ✅ Reusing: shape de normalización multi-formato + guardrails de LLM (`sales-bot-with-quotes/research.md`); guidelines como datos editables (`sheet-idempotency.md`); DB y creds de M0.
- 🆕 New work: el motor de reglas determinista; el schema canónico del préstamo; el contrato JSON hacia B.
- ⚠️ Risks identified: guidelines informales/dispersas (def #1) → modelar lleva tiempo; alucinación de Claude → mitigar con reglas duras + `llm_flag` + revisión; costo de Claude si hay muchos borrosos.
