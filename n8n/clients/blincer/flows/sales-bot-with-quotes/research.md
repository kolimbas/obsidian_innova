---
tags:
  - n8n
  - research
  - blincer
  - nivel-3
client: blincer
flow: sales-bot-with-quotes
updated: 2026-05-29
---

# Research — Bot de ventas + cotizaciones/facturas automáticas

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/flows/sales-bot-with-quotes/spec|Spec]]

---

## 1. Prior flows of this client

| Flow | Relevance | Why / why not |
| --- | --- | --- |
| `credit-limit-invoice-block` | 🟡 Media — comparte credencial HubSpot, Tango, mapeo `tango_customer_id`, canal alerta. Acoplamiento de negocio: si el cliente confirma compra acá, debería pasar por la regla de bloqueo. | Llamar al check de credit limit como sub-step antes de emitir factura (reusar lógica, no duplicar). |
| `whatsapp-overdue-debt-reminder` | 🟡 Media — comparte credencial WhatsApp, conocimiento del provider y rate-limits. | Reusar credencial y nodo de envío. **Importante:** el bot conversa por WhatsApp inbound; el dunning envía outbound — pueden compartir instance pero hay que confirmar que el provider permite ambos modos. |
| `email-remarketing` | 🟢 Baja — solo comparten HubSpot Contact / Deal. | Aprovechar que este flow crea Deals para que el de remarketing los segmente. |

## 2. Cross-client patterns

| Source | Pattern | How it applies here |
| --- | --- | --- |
| `n8n/patterns/_index.md` | (vacío) | Promover `chat-bot-with-tool-use` y `quote-pdf-generation` cuando esté `live`. |
| Innova externo: `project_innova_pre_contratacion.md` (memoria) | Pipeline OAuth Drive + DOCX render | Reusar mecánica de render → PDF para cotización (template `.docx` + variables + export `.pdf` con LibreOffice o cloud-print). |
| Innova externo: `project_auditoria_presupuestos.md` (memoria) | Groq + nativo + OCR pipeline | Si OQ-1 termina en Groq, ya hay experiencia con su throughput. |

## 3. n8n nodes considered

| Node | Role | Key params | Gotchas / link to `n8n/nodes/` |
| --- | --- | --- | --- |
| `webhook` | Recibir messages.upsert del provider | path, method=POST, response=immediate | Acknowledgment rápido obligatorio (provider reintenta si > 5s). |
| `n8n-nodes-base.respondToWebhook` | ACK al provider | response code 200 con body vacío | Importante para que el provider no reintente. |
| `n8n-nodes-langchain.agent` o equivalente | LLM con tool-use | model, system prompt, tools | Si vamos Anthropic, validar que el nodo soporte tool_use bien. Alternativa: HTTP a la API directa. |
| `n8n-nodes-base.httpRequest` | Llamar Anthropic/OpenAI/Groq + Tango + WhatsApp send | URL, body, auth | Manejar tool_use loop manualmente si el nodo agent no alcanza. |
| `n8n-nodes-base.postgres` | Storage de conversaciones (OQ-8) | connection, query | Schema simple: `conversaciones(session_id, role, text, ts, deal_id)`. |
| `n8n-nodes-base.hubspot` | CRUD contact / deal / note | resource, operation | Para Deal stage move ver pipeline ids. |
| `n8n-nodes-base.googleDrive` | Upload PDF | folderId, fileName | Permisos: Innova service account con acceso al folder Blincer. |
| Render PDF — `httpRequest` a servicio externo (DocsAPI, Carbone, Gotenberg) o nodo community | Generar PDF de cotización | template + variables | DOCX → PDF self-hosted con Gotenberg es predecible y barato. |
| `splitInBatches` / `wait` | Throttle de turnos | n/a | El bot procesa por mensaje, no por batch, pero queue management puede requerirlo. |

## 4. External systems

| System | Auth | Rate limits | Idempotency support | Docs |
| --- | --- | --- | --- | --- |
| LLM provider (TBC) | API key | depende — Anthropic 50 RPM tier 1; OpenAI dinámico; Groq alto | client-side via request_id | depende |
| WhatsApp (TBC) | depende OQ-G2 | ya cubierto en research del flow 2 | message_id | ídem |
| HubSpot | Private App | 100/10s | upsert | ídem flow 1 |
| Tango Nexo (si aplica) | OAuth/API key | desconocido | depende | nexo docs |
| Google Drive (PDF storage) | service account | 1000 req/100s | upload con `uploadType=multipart` | https://developers.google.com/drive/api |
| Gotenberg (si se elige) | self-hosted | n/a | n/a | https://gotenberg.dev |

## 5. Base flow decision

- **Base flow chosen:** `none` directamente; tomamos pedazos de:
  - Flow 1 (idempotency pattern, HubSpot credentials, internal alert).
  - Flow 2 (WhatsApp credential, throttle pattern).
- **Coverage estimate:** ~20% (composable, pero arquitectura conversacional es nueva en este vault).
- **What we copy / what we change:** copy de credenciales y patrones cross-cutting; new build de la lógica conversacional, RAG/catalog injection, tool-use loop, PDF render.

## 6. Reuse summary

- ✅ Reusing:
  - Credenciales HubSpot, Tango, WhatsApp, Sheets, alerta interna.
  - Mapeo `tango_customer_id`.
  - Sub-call lógico al check del credit limit antes de emitir factura.
  - Lección "no decidir si lookup externo está caído" (memoria auditoría-facturas).
- 🆕 New work:
  - Credencial LLM provider.
  - Storage de conversaciones (Postgres si OQ-8 lo permite; sino Sheets como MVP frágil).
  - Pipeline de render PDF (Gotenberg + template DOCX).
  - Tool definitions para el LLM (`get_catalog`, `check_stock`, `create_quote`, `emit_invoice`, `handoff_human`, `lookup_customer`).
  - Pattern de **handoff a humano** (tarea HubSpot + alerta interna con resumen del LLM).
  - Custom timeline event "Conversación bot" en HubSpot.
- ⚠️ Risks identified:
  - **Alucinación de precios / stock:** crítico para confianza del cliente. Mitigación: prohibir al LLM responder precios/stock sin haber llamado primero el tool correspondiente; validar en code que la respuesta no incluya números no devueltos por tools.
  - **Costos LLM crecen con volumen:** sin cap, riesgo de gasto descontrolado. Mitigación: budget cap en spec + alerta cuando 80% del cap.
  - **Latencia compuesta:** WhatsApp + LLM + Tango + PDF + WhatsApp send → riesgo de superar el SLA de 30s para cotización. Mitigación: streaming de respuestas parciales (mensaje "te paso la cotización en unos segundos…") + paralelización de steps independientes.
  - **Deal duplicado:** si cliente conversa en distintos hilos, podemos crear N Deals para la misma intención. Mitigación: dedup por `(contact_id, last_24h, status=Cotización enviada)`.
  - **Provider WhatsApp simultáneo (inbound + outbound dunning):** mismo número, dos workflows distintos. Confirmar que no hay race conditions o token contention.
  - **Tango local sin API:** factura automática **no es viable**; degrade a "bot prepara, Sandra emite manual" sin perder el flujo conversacional.
