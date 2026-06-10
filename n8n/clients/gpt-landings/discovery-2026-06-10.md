---
tags:
  - n8n
  - gpt-landings
  - discovery
  - nivel-3
client: gpt-landings
created: 2026-06-10
status: draft-para-enviar
---

# Cuestionario de Discovery — GPT Landings (Fase 1, 2026-06-10)

> Necesitamos estas respuestas **antes** de empezar a construir cualquier módulo de la Fase 1. Algunas son técnicas (van para quien administre el VPS / Google Workspace / la e-sign / WhatsApp), otras son de negocio (el manager/owner que define guidelines, aprobaciones y documentación).
>
> Cuando puedan ir respondiendo, completen abajo de cada pregunta. No hace falta que sea todo de una.

← [[n8n/clients/gpt-landings/README|Volver al README de GPT Landings]]

---

## Cómo leer este documento

- Cada bloque es un módulo del alcance de Fase 1 (M0 + A…G).
- Cada pregunta tiene un **interlocutor sugerido** (quién la responde mejor): **Manager/owner** (negocio), **IT/DevOps** (infra y accesos del cliente), **Innova** (definición técnica nuestra).
- Las que dicen 🚦 son **bloqueantes** — sin esa respuesta no podemos arrancar el build de ese módulo.
- Las que dicen ⚠️ son críticas pero no bloqueantes (podemos asumir y validar después).

---

## 0. Transversales (afectan más de un módulo)

### 0.1 — Acceso al VPS 🚦

**Interlocutor:** IT/DevOps del cliente.

¿Nos dan acceso SSH/root al VPS propio? ¿Qué specs tiene (CPU/RAM/disco) y qué OS corre? ¿Hay algo ya instalado que tengamos que respetar?

**Por qué importa:** M0 (infra base) no arranca sin esto; define si n8n + Supabase/Postgres entran cómodos.

> Respuesta:
>

---

### 0.2 — Google Workspace del cliente 🚦

**Interlocutor:** IT/DevOps del cliente.

¿Tienen Google Workspace? ¿Podemos crear una **service account** (o configurar OAuth) con permisos para crear carpetas y subir archivos en su Drive? ¿Quién es el admin?

**Por qué importa:** los Módulos D (intake) y G (folders) escriben en Drive. Sin acceso no hay storage de documentos.

> Respuesta:
>

---

### 0.3 — Volumen esperado ⚠️

**Interlocutor:** Manager/owner.

¿Confirmamos ~10-15 préstamos/mes? ¿Hay estacionalidad o picos? ¿Cuántos documentos promedio por préstamo?

**Por qué importa:** dimensiona la infra de M0 (DB, storage, rate de WhatsApp, costo de Claude en F).

> Respuesta:
>

---

### 0.4 — Canal de alerta interno / `processing@` ⚠️

**Interlocutor:** Manager/owner.

Cuando un módulo necesita avisar algo al equipo interno (ej. "préstamo firmado, arrancar processing"; "doc no matchea el track record"), ¿prefieren que llegue por email a `processing@`, Slack, Teams, o WhatsApp interno?

**Por qué importa:** varios módulos (C, E, F) generan avisos internos. Unificamos el canal.

> Respuesta:
>

---

## 1. M0 — Infraestructura base

### 1.1 — Vault de secrets 🚦

**Interlocutor:** Innova + IT.

¿Doppler o Bitwarden Secrets para guardar credenciales (bancos/e-sign/WhatsApp/Claude)? ¿Tienen preferencia o cuenta ya?

> Respuesta:
>

### 1.2 — Supabase managed vs Postgres en el VPS ⚠️

**Interlocutor:** Innova + IT.

¿Corremos la DB como Supabase managed (más simple, backups incluidos) o Postgres en el mismo VPS (todo self-hosted, sin costo extra)?

> Respuesta:
>

### 1.3 — Dominio para n8n 🚦

**Interlocutor:** IT.

¿Qué dominio/subdominio usamos para la instancia n8n (para HTTPS y los webhooks)? ¿Quién maneja el DNS?

> Respuesta:
>

### 1.4 — Backups ⚠️

**Interlocutor:** IT.

¿Política de backups de la DB y del n8n (frecuencia, retención, dónde)?

> Respuesta:
>

---

## 2. Módulo A — Motor de matching con capital partners

### 2.1 — Formato y cantidad de guidelines 🚦

**Interlocutor:** Manager/owner.

¿Cuántos capital partners hay hoy? ¿Dónde viven sus guidelines (PDF, Excel, planilla, en la cabeza de alguien)? ¿El formato es homogéneo entre partners o cada uno tiene el suyo?

**Por qué importa:** es la materia prima del motor de reglas. Sin esto no modelamos el matching.

> Respuesta:
>

### 2.2 — Estabilidad de los parámetros ⚠️

**Interlocutor:** Manager/owner.

¿Los criterios (tipo de propiedad, condado/geografía, monto min-max, LTV, tipo de loan) cambian seguido o son estables? ¿Quién los actualiza?

> Respuesta:
>

### 2.3 — Schema canónico del préstamo ⚠️

**Interlocutor:** Manager/owner.

¿En qué formato entra hoy un préstamo (Excel/CSV, email, formulario)? ¿Qué campos trae siempre? Necesitamos definir el schema único al que normalizamos.

> Respuesta:
>

### 2.4 — Budget / modelo Claude para casos borrosos ⚠️

**Interlocutor:** Manager/owner (paga) + Innova.

Para los casos que no resuelve una regla dura, usamos Claude. ¿Hay un cap mensual de gasto de IA? (define modelo).

> Respuesta:
>

---

## 3. Módulo B — Generación de term sheets

### 3.1 — Template y campos exactos del term sheet + ejemplo real 🚦

**Interlocutor:** Manager/owner.

¿Tienen un template de term sheet actual? Necesitamos los **campos exactos** y un **ejemplo real** (con datos dummy) para parametrizarlo.

> Respuesta:
>

### 3.2 — HTML→PDF propio vs template en la e-sign ⚠️

**Interlocutor:** Innova.

¿Generamos el PDF nosotros (pipeline Playwright/Chromium) o usamos el template nativo de la plataforma de e-sign? (depende de 4.1).

> Respuesta:
>

### 3.3 — Branding / datos fiscales ⚠️

**Interlocutor:** Manager/owner.

¿El term sheet lleva branding/datos fiscales de GPT Landings, del capital partner, o de ambos?

> Respuesta:
>

---

## 4. Módulo C — Aprobación y firma digital

### 4.1 — DocuSign vs PandaDoc + API 🚦

**Interlocutor:** Manager/owner + IT.

¿Qué plataforma de e-sign usan o quieren usar? ¿Ya tienen cuenta y acceso a API (plan que habilite API)?

**Por qué importa:** define el módulo C entero (envío a firma + webhook de "firmado") y parte de B.

> Respuesta:
>

### 4.2 — Quién aprueba y por qué canal ⚠️

**Interlocutor:** Manager/owner.

¿Quién aprueba un term sheet antes de mandarlo a firma? ¿Cómo quiere recibir el pedido de aprobación (email, portal, Slack, WhatsApp)?

> Respuesta:
>

### 4.3 — Idempotencia del webhook ⚠️

**Interlocutor:** Innova.

La e-sign reentrega webhooks. ¿Su payload trae un `event_id` único? (lo usamos para no procesar dos veces el "firmado").

> Respuesta:
>

---

## 5. Módulo D — Formulario inteligente de intake

### 5.1 — Lista exacta de documentos por tipo de préstamo 🚦

**Interlocutor:** Manager/owner.

Necesitamos la **lista cerrada** de documentos requeridos por cada tipo de préstamo (ej. PFS, Track Record, credit report, bank statements, etc.).

> Respuesta:
>

### 5.2 — ¿La lista cambia por partner o por tipo de loan? ⚠️

**Interlocutor:** Manager/owner.

¿El checklist es único o varía según el capital partner / tipo de operación?

> Respuesta:
>

### 5.3 — Tamaños máximos y storage ⚠️

**Interlocutor:** Manager/owner + IT.

¿Tamaños máximos de archivo esperados? ¿Guardamos en Google Drive o en Supabase Storage?

> Respuesta:
>

### 5.4 — Dónde corre el front y cómo autentica al borrower ⚠️

**Interlocutor:** Innova + Manager.

El formulario es web propio (Next.js). ¿Lo deployamos en Vercel? ¿Cómo identifica/autentica al borrower que sube docs (link único por préstamo, login, etc.)?

> Respuesta:
>

---

## 6. Módulo E — Bot de seguimiento por WhatsApp

### 6.1 — Número/cuenta WhatsApp Business + proveedor 🚦 (arrancar YA)

**Interlocutor:** Manager/owner + IT.

¿Tienen un número de WhatsApp Business verificado? ¿Quieren Cloud API oficial de Meta, Evolution API (self-hosted), o un BSP (Twilio/360dialog)?

**Por qué importa:** define costo, estabilidad y opt-in. **Arrancar ya: la verificación + aprobación de templates de Meta tiene lead time de días.**

> Respuesta:
>

### 6.2 — Templates HSM aprobados por Meta 🚦

**Interlocutor:** Manager/owner + Innova.

Los mensajes proactivos (recordatorios) requieren **templates HSM aprobados por Meta**. ¿Avanzamos con la submission apenas tengamos el número? Definamos los textos base.

> Respuesta:
>

### 6.3 — Opt-in del borrower ⚠️

**Interlocutor:** Manager/owner + legal.

¿El borrower da consentimiento para recibir avisos por WhatsApp en algún punto del proceso (ej. al iniciar el form)?

> Respuesta:
>

### 6.4 — Cadencia de recordatorios ⚠️

**Interlocutor:** Manager/owner.

Default propuesto: recordatorio cada X días mientras el checklist esté incompleto (ej. cada 2 días, máx 1/día). ¿Les sirve? ¿Más agresivo, más suave?

> Respuesta:
>

### 6.5 — Fallback si no hay WhatsApp ⚠️

**Interlocutor:** Manager/owner.

Si el borrower no tiene WhatsApp: ¿email automático, o solo dejarlo en una lista para seguimiento manual?

> Respuesta:
>

---

## 7. Módulo F — Validación incremental + cross-check de track record

> [!warning] Módulo F está FUERA del precio cerrado de Fase 1
> Su inclusión depende de la respuesta a 7.1. Lo documentamos para scoping; **no lo comprometemos** hasta validar factibilidad técnica con Elementix.

### 7.1 — ¿Elementix expone API o es solo UI? + fuentes de public records 🚦🚦

**Interlocutor:** Manager/owner + IT.

**Esta es la pregunta más importante de todo el discovery.** ¿Elementix tiene API? Si sí, es directo. Si es solo UI, hay que automatizar/scrapear el front (más frágil) o dejar entrada asistida manual. Además: ¿qué fuentes de registros públicos usan hoy y son accesibles por API?

**Por qué importa:** define si F entra como adicional cotizado o queda fuera de alcance. Es el módulo con más incertidumbre.

> Respuesta:
>

### 7.2 — Si Elementix es solo UI ⚠️

**Interlocutor:** Manager/owner + Innova.

En ese caso, ¿preferís que automaticemos el front (riesgo de fragilidad/baneo) o una entrada asistida manual donde el sistema prepara y un humano confirma?

> Respuesta:
>

### 7.3 — OCR + budget Claude ⚠️

**Interlocutor:** Manager/owner.

La extracción sobre PFS / track record / bank statements usa OCR + Claude (cobra por documento). ¿Hay cap de gasto para esto?

> Respuesta:
>

### 7.4 — Criterios de match entidad↔dirección ⚠️

**Interlocutor:** Manager/owner.

¿Qué cuenta como "match"? (ej. la entidad declarada figura como dueña y firmante en la dirección X). ¿Qué tolerancia/criterio define un flag?

> Respuesta:
>

---

## 8. Módulo G — Estructura automática de folders en Drive

### 8.1 — Nomenclatura exacta ⚠️

**Interlocutor:** Manager/owner.

¿Cómo quieren que se llamen las carpetas y los archivos? Default propuesto: por préstamo, `/property` · `/closing` · `/borrower`. ¿Va?

> Respuesta:
>

### 8.2 — ¿La estructura cambia por tipo de préstamo? ⚠️

**Interlocutor:** Manager/owner.

¿El árbol de carpetas es único o varía según el tipo de operación?

> Respuesta:
>

### 8.3 — Permisos / sharing ⚠️

**Interlocutor:** Manager/owner + IT.

¿Quién accede a las carpetas del préstamo (equipo interno, el borrower, el capital partner)? ¿Qué nivel de permiso?

> Respuesta:
>

---

## Después de que respondan

Cuando tengamos las respuestas (aunque sean parciales), del lado nuestro:

1. Actualizamos cada `plan.md` sacando el `blocked-by-oqs` y ajustando arquitectura según las decisiones.
2. Damos vuelta los assumptions a hechos.
3. Estimamos tiempos por módulo.
4. Decidimos orden de build: **M0 → A+B → C → D → G → E (en paralelo apenas esté el form) → F al final** (solo si 7.1 lo habilita).
5. Arrancamos por **M0 (infraestructura)**, que es prerequisito de todo lo demás.

---

## Links útiles

- [[n8n/clients/gpt-landings/README|README GPT Landings]]
- [[n8n/clients/gpt-landings/flows/m0-infra-setup/spec|Spec — M0 Infraestructura]]
- [[n8n/clients/gpt-landings/flows/partner-matching-engine/spec|Spec — A Matching]]
- [[n8n/clients/gpt-landings/flows/term-sheet-generation/spec|Spec — B Term sheets]]
- [[n8n/clients/gpt-landings/flows/approval-and-esign/spec|Spec — C Aprobación + firma]]
- [[n8n/clients/gpt-landings/flows/document-intake-form/spec|Spec — D Intake form]]
- [[n8n/clients/gpt-landings/flows/whatsapp-doc-reminder/spec|Spec — E WhatsApp reminders]]
- [[n8n/clients/gpt-landings/flows/track-record-validation/spec|Spec — F Track record (carve-out)]]
- [[n8n/clients/gpt-landings/flows/drive-folder-structure/spec|Spec — G Folders Drive]]
