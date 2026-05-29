---
tags:
  - n8n
  - spec
  - blincer
  - nivel-3
client: blincer
flow: sales-bot-with-quotes
status: draft
updated: 2026-05-29
---

# Spec — Bot de ventas + cotizaciones/facturas automáticas

> Un asistente conversacional por WhatsApp atiende al cliente final, asesora sobre productos, genera cotizaciones formales y, si el cliente confirma, emite la factura en Tango — todo sin intervención humana en el camino feliz.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/blincer/README|Blincer]]

> **Acopla flujos 3 y 4 de la tanda oficial** (decisión del usuario el 2026-05-29). Un solo bundle, un workflow.json. Posible split futuro si la complejidad lo justifica.

---

## Goal

Dar a Blincer un canal de ventas asíncrono que:
1. Recibe consultas vía WhatsApp.
2. Responde con asesoramiento usando el catálogo de productos como base de conocimiento (RAG mínimo o catálogo estructurado).
3. Genera cotización formal en PDF y la envía cuando el cliente lo pide.
4. Emite factura electrónica en Tango cuando el cliente confirma compra.
5. Hace handoff a humano cuando supera umbrales (monto, stock, intent).

Resultado de negocio: Guillermo libera horas comerciales; clientes recibidos fuera de horario son atendidos; trazabilidad completa en HubSpot.

## Trigger

- **Type:** webhook
- **Schedule / endpoint / event:** webhook del provider WhatsApp (inbound message) → endpoint n8n.
- **Expected frequency:** desconocido en MVP; estimar 50–500 mensajes/día post-rollout en estado estable.

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
| `from_phone` | string (E.164) | webhook WhatsApp | yes | identificador del cliente |
| `message_text` | string | webhook WhatsApp | yes | texto recibido |
| `message_media` | array | webhook WhatsApp | no | imagen/audio (V2 — out of scope MVP) |
| `conversation_history` | array | n8n storage (Postgres / Sheets) | yes | últimos N mensajes para contexto LLM |
| `customer_profile` | object | HubSpot Contact (lookup por phone) | no (opcional si es nuevo cliente) | nombre, empresa, segmento |
| `catalog` | array de productos | source TBC (OQ-G5) | yes | sku, descripción, precio, stock |
| `stock_snapshot` | map sku→qty | Tango o cache | yes | para verificar disponibilidad antes de cotizar |
| `system_prompt` | string | HubSpot custom prop o Sheet | yes | el persona del bot + reglas de negocio |
| `handoff_rules` | object | Sheet config | yes | umbrales monto / lista de intents |
| `approval_required` | bool | Sheet config | yes | si true, cotización requiere review antes de enviar (OQ-6) |

Sample webhook payload (Evolution API, abreviado):

```json
{
  "event": "messages.upsert",
  "instance": "blincer",
  "data": {
    "key": {"remoteJid": "5491155556666@s.whatsapp.net", "fromMe": false, "id": "ABCD1234"},
    "message": {"conversation": "Hola, necesito 50 cerraduras modelo X"},
    "messageTimestamp": 1748528400
  }
}
```

## Outputs / Side effects

- **WhatsApp:** respuesta conversacional al cliente (text).
- **HubSpot Contact:** create si no existe (con `phone`, `lifecyclestage=lead`). Si existe, actualizar `last_bot_interaction_at`.
- **HubSpot Deal:** crear cuando el cliente pide cotización formal, stage "Cotización enviada"; mover a "Ganado" al confirmar compra; o a "Cerrado-perdido" si rechaza.
- **PDF cotización:** generar y subir a Google Drive (o S3) + adjuntar link en HubSpot Deal + mandar al cliente.
- **Tango invoice:** emisión cuando el cliente confirma, vía Nexo API (si aplica). Stock de Tango se decrementa por la factura.
- **Conversation log:** Postgres / Sheets `conversaciones_bot` con cada turn (role, text, timestamp, deal_id).
- **Handoff event:** si umbral disparado, push a "Vendedor de turno" (canal interno + HubSpot task asignada).

## Success criteria

- [ ] Cliente nuevo escribe → recibe respuesta < 5 s, en línea con el `system_prompt` (saludo, oferta de ayuda).
- [ ] Cliente que pide cotización ("dame precio por 50 cerraduras X") → recibe PDF de cotización < 30 s **si** stock OK y monto bajo umbral, o **handoff** si supera.
- [ ] Cliente confirma compra → factura emitida en Tango y enviada al cliente (PDF) en < 60 s.
- [ ] HubSpot Deal refleja el ciclo completo con stages correctos y notes auditables.
- [ ] Conversación retiene contexto: si el cliente menciona "ese mismo modelo" dos turns más tarde, el bot resuelve correctamente.
- [ ] Handoff funciona: cuando monto > umbral, el bot pasa al humano con resumen + historial completo.
- [ ] Cero alucinaciones de precios / stock — el LLM nunca inventa data del catálogo (forzar via tool-use o RAG estricto).

## Out of scope

- **MVP:** mensajes con media (imagen/audio del cliente) — V2.
- Pagos integrados (link de pago, MercadoPago checkout) — V2.
- Multi-idioma (asume español rioplatense).
- Reportes BI/dashboard de performance del bot — un flow paralelo de reporting.
- Detección de fraude / verificación de identidad del cliente — humano valida en handoff.
- Templates marketing/nurturing post-compra — eso es `email-remarketing`.
- Recordatorios de cobranza si el cliente no paga la factura emitida — eso es `whatsapp-overdue-debt-reminder`.

## Open questions

- [ ] **OQ-1 — LLM provider + modelo + budget.** Opciones (recomendación entre paréntesis):
  - Anthropic Claude Sonnet 4.6 / Haiku 4.5 (calidad alta, tool use sólido, costos medios)
  - OpenAI GPT (familiar, ecosistema grande)
  - Groq (latencia ultra-baja, modelos open source)
  - Budget estimado mensual: definir cap (USD/mes) — sin esto no decidimos modelo.
- [ ] **OQ-2 — Provider WhatsApp** (= OQ-G2 global). Cloud API permite buttons/lista → mejor UX para cotizar. Evolution es más barato.
- [ ] **OQ-3 — Tango versión** (= OQ-G1). Bifurca el step "Emitir factura":
  - Nexo → API REST `/comprobantes`.
  - Local → no automatizable directo → V2 requiere RPA o el bot solo genera cotización y la factura la hace Sandra manualmente.
- [ ] **OQ-4 — Source of truth del catálogo + stock.** Opciones:
  - HubSpot Products (nativo, sincable con Tango por job aparte).
  - Tango directo (siempre actualizado, latencia mayor).
  - Sheet curada por Guillermo (manual, no escala).
  - Recomendación: HubSpot Products como SoT presentado al LLM + lookup justo-en-tiempo a Tango para verificar stock antes de cotizar.
- [ ] **OQ-5 — Umbral de handoff a humano.** Tres ejes:
  - Monto: > ARS X → handoff.
  - Intent: "hablar con persona", "queja", "compra grande" → handoff.
  - Confianza del bot: si el LLM no puede resolver (señal interna) → handoff.
  - Valores específicos a definir con Guillermo.
- [ ] **OQ-6 — Política de aprobación humana antes de emitir cotización/factura.** Opciones:
  - **A — Full auto:** bot decide y emite. Solo handoff si umbrales disparan.
  - **B — Review cotización:** bot prepara, Guillermo aprueba en HubSpot, sistema envía.
  - **C — Review factura:** bot envía cotización auto, factura requiere confirmación humana.
  - Recomendación MVP: C (cotización auto, factura con review). Migrar a A cuando haya confianza.
- [ ] **OQ-7 — Catálogo cómo se presenta al LLM.** ¿RAG (embeddings + retrieval) o `catalog_summary` inyectado en system prompt (si < 8k tokens)? Si catálogo > 200 items → RAG.
- [ ] **OQ-8 — Storage de conversaciones.** Sheet escala mal; Postgres escala bien. ¿Hay Postgres disponible en la infra de Blincer / Innova?
- [ ] **OQ-9 — Identidad del cliente.** Si número WhatsApp no matchea HubSpot Contact, ¿el bot crea el Contact directo o pide datos primero (nombre, CUIT)?

*(Cruza con OQ-G1, OQ-G2, OQ-G7 globales).*

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Guillermo (estratégico — más ventas, menos fricción) |
| Approver | Guillermo + Sandra (lado contable de la emisión de factura) |
| End user (operativo) | Vendedor/a de turno (recibe handoffs) |
| End user (afectado) | Cliente final de Blincer (conversa con el bot) |
