---
tags:
  - n8n
  - blincer
  - discovery
  - nivel-3
client: blincer
created: 2026-05-29
status: draft-para-enviar
---

# Cuestionario de Discovery — Blincer (tanda 2026-05-29)

> Necesitamos estas respuestas **antes** de empezar a construir cualquier flujo en n8n. Algunas son técnicas (hay que pasárselas a quien administra Tango / HubSpot / IT), otras son de negocio (Sandra para administración, Guillermo para comercial).
>
> Cuando puedan ir respondiendo, completen abajo de cada pregunta. No hace falta que sea todo de una.

← [[n8n/clients/blincer/README|Volver al README de Blincer]]

---

## Cómo leer este documento

- Cada bloque es un flujo del alcance acordado.
- Cada pregunta tiene un **interlocutor sugerido** (quién la responde mejor).
- Las que dicen 🚦 son **bloqueantes** — sin esa respuesta, no podemos arrancar el build.
- Las que dicen ⚠️ son críticas pero no bloqueantes (podemos hacer assumptions y validar después).

---

## 0. Transversales (afectan más de un flujo)

### 0.1 — Tango: ¿Qué versión usan? 🚦

**Interlocutor:** IT / proveedor Tango.

¿Es **Tango Nexo** (versión cloud con API REST) o **Tango local** (instalación en servidor propio)? Si es local, ¿qué motor de base de datos usa por debajo (SQL Server, Firebird, otro)? ¿Permiten acceso ODBC o solo export CSV?

**Por qué importa:** define si podemos integrar directo con Tango (Nexo) o si hay que pasar por CSV / RPA (local). Cambia 3 flujos.

> Respuesta:
>

---

### 0.2 — WhatsApp: ¿Qué proveedor usan o quieren usar? 🚦

**Interlocutor:** Guillermo + IT.

Opciones:
- **Cloud API oficial de Meta** (más caro, más estable, requiere número verificado y aprobación de templates).
- **Evolution API** (autohospedado, más barato, riesgo de baneo si hay mucho volumen).
- **360dialog** u otro BSP.

¿Tienen ya un número de WhatsApp Business verificado? ¿Lo vamos a usar para inbound (atender clientes) y outbound (mandar cobranzas) al mismo tiempo?

**Por qué importa:** dos flujos (cobranzas + bot de ventas) dependen de esto. Cambia costos, latencia, opt-in legal.

> Respuesta:
>

---

### 0.3 — HubSpot: ¿Qué plan exacto tienen y quién es admin? 🚦

**Interlocutor:** Guillermo o quien administre HubSpot.

¿Confirmamos que es **HubSpot Pro o superior**? ¿Quién es el admin que puede emitir un Private App Token? ¿Tienen también **Marketing Hub** contratado o solo el CRM?

**Por qué importa:** Los 4 flujos hablan con HubSpot. El plan determina qué scopes API tenemos disponibles.

> Respuesta:
>

---

### 0.4 — Canal de alerta interno: ¿Cómo prefieren que Sandra/Guillermo reciban avisos del sistema? 🚦

**Interlocutor:** Sandra + Guillermo.

Cuando un flujo necesita avisar algo a la operación interna (ej. "factura bloqueada por deuda, revisar"), ¿prefieren que llegue por:
- WhatsApp a un grupo interno
- Email a un buzón compartido
- Slack o Teams (si lo usan)
- Otro

**Por qué importa:** los 4 flujos generan alertas internas. Unificamos el canal para no fragmentar.

> Respuesta:
>

---

### 0.5 — Mapeo Tango ↔ HubSpot: ¿Cómo se relaciona un cliente entre los dos sistemas? ⚠️

**Interlocutor:** Sandra.

¿Hay un código de cliente Tango (`tango_customer_id`) cargado en cada Company de HubSpot? Si no, ¿de dónde sacamos el cruce (CSV export de Tango, criterio por CUIT, etc.)? ¿Alguien tiene la lista de equivalencias?

**Por qué importa:** sin este puente, ningún flujo puede ir de un sistema al otro.

> Respuesta:
>

---

### 0.6 — Stages del pipeline de HubSpot: ¿Cuáles existen y con qué nombre exacto? ⚠️

**Interlocutor:** Guillermo o quien administre HubSpot.

Necesitamos los nombres reales de estas stages en el pipeline de Deals:
- "Listo para facturar"
- "Bloqueado por deuda"
- "Cotización enviada"
- "Ganado"
- "Cerrado-perdido"

Si alguna no existe, hay que crearla antes del build.

**Por qué importa:** los flujos disparan acciones leyendo o moviendo Deals entre stages.

> Respuesta:
>

---

## 1. Flujo — Bloqueo automático de facturación por deuda

> Cuando un Deal está listo para facturar, el sistema chequea la deuda viva del cliente contra su límite de crédito; si lo supera, bloquea el Deal y avisa a Sandra.

### 1.1 — ¿Cómo obtenemos la deuda actual de Tango? 🚦

**Interlocutor:** IT + Sandra.

Depende de la 0.1. Si es Nexo, ¿hay un endpoint tipo `/cuentas-corrientes/cliente/{id}`? Si es local, ¿el export CSV de cuenta corriente lo hace Sandra diario? ¿Cada cuánto?

> Respuesta:
>

### 1.2 — ¿Dónde vive el límite de crédito de cada cliente hoy? 🚦

**Interlocutor:** Sandra.

¿El límite está en Tango (ficha del cliente), en HubSpot (custom field en la Company), en una planilla aparte, o en la cabeza de Sandra? ¿Quién lo edita y cada cuánto?

**Recomendación nuestra:** llevarlo a HubSpot Company para que Sandra lo edite ahí y los flujos lo lean siempre actualizado.

> Respuesta:
>

### 1.3 — ¿Qué cuenta como "deuda" para el bloqueo? 🚦

**Interlocutor:** Sandra + Estudio Castro (contable).

Opciones (pueden combinarse):
- Saldo de cuenta corriente Tango
- Facturas emitidas no cobradas
- Cheques en cartera no acreditados
- Algún otro concepto

¿Es la suma de todo o cada concepto tiene su propio tope?

> Respuesta:
>

### 1.4 — ¿Qué tipo de bloqueo quieren? 🚦

**Interlocutor:** Guillermo + Sandra.

Tres opciones:
- **A — Solo alerta:** Sandra recibe el aviso, el Deal sigue su curso; ella decide si facturar o no.
- **B — Flag + cambio de stage:** el Deal se marca como bloqueado y se mueve a una stage "Bloqueado por deuda"; nadie factura hasta que Sandra desbloquee.
- **C — Bloqueo duro:** el sistema directamente impide que la factura se emita en Tango.

**Recomendación nuestra:** B (Flag + stage). C requiere mucha más integración con Tango.

> Respuesta:
>

---

## 2. Flujo — Avisos automáticos de deuda vencida por WhatsApp

> Cada día el sistema mira las facturas vencidas y manda recordatorios por WhatsApp según una cadencia configurable.

### 2.1 — Opt-in de los clientes: ¿Tienen consentimiento por escrito? 🚦🚦

**Interlocutor:** Sandra + asesor legal.

WhatsApp Business exige que el cliente haya dado consentimiento explícito para recibir mensajes comerciales/de cobranza. Si arrancamos a mandar sin opt-in:
- Con Cloud API oficial → riesgo de baneo del número.
- Con Evolution API → usuario marca como spam, baja la entregabilidad.

**¿Tienen registro de quiénes firmaron opt-in?** Si no, hay que conseguirlo antes de encender el flujo (puede ser un mail simple al cliente con "¿podemos avisarte por WhatsApp?").

> Respuesta:
>

### 2.2 — ¿Cómo obtenemos la lista de facturas vencidas? 🚦

**Interlocutor:** IT + Sandra.

Depende de la 0.1. Si es Nexo, lo consultamos por API. Si es local, ¿podés exportar diario un CSV con las facturas vencidas? ¿O accedemos a la DB directo?

> Respuesta:
>

### 2.3 — Si un cliente no tiene WhatsApp, ¿qué hacemos? ⚠️

**Interlocutor:** Sandra.

Tres opciones:
- **A — Email automático** (requiere tener el email cargado en HubSpot y una plantilla aprobada).
- **B — Solo dejarlo en una planilla** para que Sandra los llame manualmente.
- **C — Otra** (cartas, llamadas automatizadas, etc.).

**Recomendación nuestra:** B para arrancar, A para una segunda versión.

> Respuesta:
>

### 2.4 — Cadencia de recordatorios: ¿qué días post-vencimiento? ⚠️

**Interlocutor:** Sandra.

Default propuesto: día 1, 7, 15, 30, 45, 60 post-vencimiento. ¿Te queda bien? ¿Querés más agresivo, más suave?

> Respuesta:
>

### 2.5 — Templates de los mensajes: ¿uno único o uno por cliente? ⚠️

**Interlocutor:** Sandra.

Default propuesto: un set único de 6 templates (uno por día de cadencia), editables por vos en una planilla sin tocar n8n. En el futuro podríamos personalizar por cliente si lo necesitás.

¿Te alcanza con uno único para arrancar?

> Respuesta:
>

### 2.6 — ¿Hay facturas en moneda extranjera (USD)? ⚠️

**Interlocutor:** Sandra.

Si sí, los templates tienen que reflejarlo (no podemos hablar de "$X" sin clarificar moneda).

> Respuesta:
>

---

## 3. Flujo — Bot de ventas + cotizaciones/facturas automáticas

> Atiende al cliente final por WhatsApp, asesora con el catálogo, genera cotización formal, y al confirmar emite factura en Tango.

### 3.1 — Budget mensual del bot (LLM): ¿cuánto pueden destinar? 🚦

**Interlocutor:** Guillermo (paga el cliente).

El bot usa inteligencia artificial (LLM) que cobra por mensaje. Para 200 mensajes/día, estimamos USD 30–150/mes según modelo. ¿Hay un cap mensual definido? Sin este dato no elegimos modelo.

> Respuesta:
>

### 3.2 — Catálogo de productos: ¿dónde vive y quién lo mantiene? 🚦

**Interlocutor:** Guillermo.

Opciones:
- **HubSpot Products** (nativo, sincable con Tango por un job aparte) — recomendación nuestra.
- **Tango directo** (siempre actualizado, latencia mayor).
- **Planilla curada manualmente.**

¿Tienen catálogo cargado en algún lado ya o hay que armarlo desde cero? ¿Stock se chequea cuándo (en cada cotización o cada X horas)?

> Respuesta:
>

### 3.3 — ¿Cuándo el bot pasa a un humano? 🚦

**Interlocutor:** Guillermo.

Necesitamos reglas concretas. Default propuesto:
- Monto cotización > ARS X → handoff (¿qué X?).
- Cliente dice "quiero hablar con una persona" → handoff.
- Bot detecta queja, sin stock, o pedido especial → handoff.

¿Querés afinar estos umbrales o agregar otros?

> Respuesta:
>

### 3.4 — ¿Quién aprueba las cotizaciones y facturas antes de enviarlas? 🚦

**Interlocutor:** Guillermo + Sandra.

Tres modos:
- **A — Full auto:** el bot cotiza y factura sin pasar por humano. Máxima velocidad, máximo riesgo.
- **B — Review de cotización:** el bot prepara, Guillermo aprueba en HubSpot, el sistema envía.
- **C — Review de factura:** el bot envía cotización auto, pero antes de emitir factura espera OK de Sandra.

**Recomendación MVP:** C. Migrar a A cuando haya confianza.

> Respuesta:
>

### 3.5 — Si llega un mensaje de un número que no está en HubSpot, ¿qué hace el bot? ⚠️

**Interlocutor:** Guillermo.

Dos opciones:
- **A — Crea el contacto directamente** con el número y conversa.
- **B — Le pide datos primero** (nombre, empresa, CUIT) antes de cotizar.

¿Cuál prefieren?

> Respuesta:
>

### 3.6 — System prompt del bot: ¿qué personalidad / reglas le pegamos? ⚠️

**Interlocutor:** Guillermo.

El "system prompt" es la personalidad del bot. Ejemplos a definir:
- Tono (formal/coloquial, voseo/tuteo).
- Qué puede prometer (descuentos, plazos de entrega).
- Qué NO puede hacer (negociar precio sin pasar por humano, dar info técnica que no esté en catálogo).
- Política de descuentos si aplica.

Lo armamos juntos en una reunión de 30 min cuando estés disponible.

> Respuesta:
>

### 3.7 — Template visual de la cotización (PDF): ¿tienen uno actual? ⚠️

**Interlocutor:** Guillermo.

Necesitamos un template DOCX/PDF con branding de Blincer (logo, datos fiscales, condiciones de venta, etc.) que vamos a usar como base para las cotizaciones generadas por el bot. ¿Tenés uno actual? ¿Lo podés compartir?

> Respuesta:
>

---

## 4. Flujo — Remarketing y difusiones por email

> Campañas masivas a la base de contacts de Blincer, segmentadas, con tracking de aperturas/clicks.

### 4.1 — HubSpot Marketing Hub vs Mailchimp: ¿cuál usan o quieren contratar? 🚦

**Interlocutor:** Guillermo (decisión de costo).

Si ya tienen Marketing Hub contratado o lo quieren contratar (~USD 50–890/mes según volumen), todo queda dentro de HubSpot — más simple. Si no, usamos Mailchimp (~USD 13–350/mes según volumen) que es más barato pero requiere sincronización entre sistemas.

¿Qué prefieren?

> Respuesta:
>

### 4.2 — Suppression list ("no enviarle a estos clientes"): ¿quién la mantiene y desde dónde? ⚠️

**Interlocutor:** Guillermo.

Default propuesto: dos fuentes:
1. **HubSpot smart list** que se llena sola con bounces y unsubscribes.
2. **Planilla manual** donde Guillermo agrega contactos a excluir (litigios, clientes premium, etc.).

¿Te queda bien o querés otro flujo?

> Respuesta:
>

### 4.3 — Volumen y warmup: ¿cuántos contacts tiene la base y cuánto envían hoy? ⚠️

**Interlocutor:** Guillermo + IT.

Necesitamos saber el tamaño de la base y si tienen dominio dedicado de envío. Si arrancamos a mandar 1500 emails/día sin warmup gradual, podemos terminar en spam y bajar la entregabilidad de todo. Plan propuesto: arrancar 200/día semana 1, 500/día semana 2, 1500/día semana 3.

> Respuesta:
>

### 4.4 — KPIs de éxito: ¿qué número los hace decir "la campaña funcionó"? ⚠️

**Interlocutor:** Guillermo.

Ejemplos: open rate ≥ 25%, CTR ≥ 3%, reply rate ≥ 1%. Sin esto, los reportes del sistema son informativos pero no nos dicen si vale la pena seguir invirtiendo.

> Respuesta:
>

### 4.5 — Ventana de tracking: ¿cuántas horas/días después del envío cerramos el reporte? ⚠️

**Interlocutor:** Guillermo.

Default propuesto: 48 horas. ¿Suficiente o quieren 7 días?

> Respuesta:
>

### 4.6 — Compliance / privacidad: ¿tienen política publicada? 🚦

**Interlocutor:** Guillermo + legal.

Para mandar emails comerciales en Argentina (Ley 25.326) necesitamos:
- Política de privacidad publicada en el sitio.
- Link de "darse de baja" en todos los emails (lo ponen HubSpot/Mailchimp automático).
- Retención de datos definida (cuánto tiempo guardamos info de un contact).

¿Está todo eso ya? Si no, hay que armarlo antes de encender el flujo.

> Respuesta:
>

### 4.7 — ¿A quién más le mandamos los reportes de campaña? ⚠️

**Interlocutor:** Guillermo.

Default: solo a Guillermo. ¿Querés que Sandra o alguien de Innova también lo reciba?

> Respuesta:
>

---

## Después de que respondan

Cuando tengamos las respuestas (aunque sea parciales), del lado nuestro:

1. Actualizamos cada `plan.md` sacando el `blocked-by-oqs` y ajustando arquitectura según las decisiones.
2. Damos vuelta los assumptions a hechos.
3. Estimamos tiempos por flujo.
4. Decidimos orden de build (priorizando lo que esté menos bloqueado).
5. Arrancamos por **Bundle 1 (Bloqueo facturación)** que es el que menos OQs tiene.

---

## Links útiles

- [[n8n/clients/blincer/README|README Blincer]]
- [[n8n/clients/blincer/flows/credit-limit-invoice-block/spec|Spec — Bloqueo facturación]]
- [[n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/spec|Spec — Avisos WhatsApp]]
- [[n8n/clients/blincer/flows/sales-bot-with-quotes/spec|Spec — Bot ventas + cotizaciones]]
- [[n8n/clients/blincer/flows/email-remarketing/spec|Spec — Remarketing email]]
