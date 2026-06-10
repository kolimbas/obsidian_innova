---
tags:
  - cliente
  - automatizacion
  - activo
  - nivel-2
estado: pre-venta
fase: 1
updated: 2026-06-10
---

# GPT LANDINGS

> Private lender · hipotecas de 1er grado para inversores · South Florida

---

## Contacto clave

| Persona | Rol | Área |
| --- | --- | --- |
| **[Manager / owner]** | Define guidelines, aprobaciones y documentación | Negocio |
| **[IT / DevOps del cliente]** | VPS, Google Workspace, accesos | Infra |

> [!warning] Cargar nombres reales
> Los contactos van como placeholders hasta tener los nombres reales del lado GPT Landings.

---

## Contexto

Prestamista privado de **hipotecas de primer grado para inversores** en el sur de Florida. Quiere automatizar el flujo de originación del préstamo —desde que entra hasta que arranca *processing*— sacándole al equipo el trabajo repetitivo (matching con capital partners, term sheets, seguimiento de documentación, armado de folders). **El criterio de underwriting queda en las personas.**

Estamos **cerrando el contrato**. El desglose técnico de Fase 1 vive en `~/Descargas/desglose-tareas-fase1.pdf` y el detalle por módulo en la KB de n8n.

---

## Stack del cliente

- **VPS propio del cliente** (instancia n8n **nueva**, separada de la de Blincer)
- **n8n** self-hosted (pm2 + HTTPS/dominio)
- **Supabase / Postgres** (estado y parámetros)
- **Claude API** (extracción/razonamiento)
- **Google Drive API** (storage de documentos)
- **e-sign**: DocuSign o PandaDoc (a definir)
- **WhatsApp Business API** (proveedor a definir)
- **Elementix / registros públicos** (cross-check de track record — Módulo F, carve-out)

> [!info] Infra propia
> A diferencia de Blincer, GPT Landings corre sobre **su propio VPS**. No se reusan credenciales de otros clientes — sólo patrones y estructura de la KB.

---

## Modelo de cobro

**Por deliverable, no por hora.** **USD 5.000 por fase**, **4 fases** (USD 20.000 total de roadmap). Cada fase se aprueba y cobra cerrada.

> [!warning] Módulo F — cotizado aparte / condicional
> La validación + cross-check de track record (Módulo F, "Elementix") queda **fuera del precio cerrado de Fase 1**. Entra como **adicional condicionado** a que Elementix exponga API (ver discovery, pregunta 7.1). Es el módulo de mayor incertidumbre — no comprometer alcance/precio hasta validar factibilidad.

---

## Fase 1 — Procesamiento y originación (alcance actual)

> [!info] 7 módulos. Detalle Spec Kit en `n8n/clients/gpt-landings/`
> Orden de ataque: **M0 → A+B → C → D → G → E → F (al final, condicional).**

| Módulo | Qué hace | Riesgo |
| --- | --- | --- |
| **M0 — Infraestructura** | VPS + n8n/pm2 + HTTPS, DB (4 tablas), vault de secrets, Drive, repo dev/prod | Bajo (fundacional) |
| **A — Matching** | Para qué capital partners califica el préstamo (reglas duras + Claude) | Medio |
| **B — Term sheets** | Genera term sheet por partner que califica (HTML→PDF) | Medio |
| **C — Aprobación + firma** | Manager aprueba → e-sign → webhook firmado → estado `processing` | Medio |
| **D — Formulario intake** | Form web de carga de documentación con checklist por tipo | Medio |
| **E — Bot WhatsApp** | Recuerda al borrower mientras el checklist esté incompleto | Medio (lead time templates Meta) |
| **F — Validación track record** | OCR+Claude + cross-check vs registros públicos/Elementix | 🔴 Alto — **carve-out condicional** |
| **G — Folders Drive** | Estructura estándar de carpetas + clasificación de docs | Bajo |

### Entregable web — Formulario de intake (Módulo D)

> [!info] Cruza la línea web de Innova
> El **front-end** del formulario de intake (Módulo D) es un entregable **web** (Next.js/React — Lovable / Claude Code / Supabase / Vercel), no solo n8n. El **backend** (ingest, persistencia en Drive/Supabase, estado del checklist, disparo de E y F) vive como bundle n8n en [[n8n/clients/gpt-landings/flows/document-intake-form/spec|document-intake-form]].
>
> Pendiente de definir: dónde se deploya el front (Vercel) y cómo autentica al borrower (link único por préstamo vs login). Ver discovery 5.4.

---

## Fases del proyecto

| Fase | Descripción | Estado |
| --- | --- | --- |
| **Fase 1** | Procesamiento y originación (M0 + A–G; F condicional) | 🔵 Cerrando contrato — discovery pendiente |
| **Fase 2** | Servicing: recordatorios de pago, control de vencimientos, alertas de mora | 🔲 Contexto (no en este sprint) |
| **Fase 3** | Adquisición: outreach sobre datos públicos + base propia, Meta lookalikes, schema/AEO | 🔲 Contexto |
| **Fase 4** | Trading de notas: approach a family offices / crowdfunding | 🔲 Contexto |

---

## Estado del proyecto

> [!todo] Próximos pasos
> - [ ] Cerrar contrato + 50% del primer deliverable (M0)
> - [ ] Mandar el [[n8n/clients/gpt-landings/discovery-2026-06-10|cuestionario de discovery]] al manager / IT
> - [ ] Resolver las 8 definiciones bloqueantes (especialmente 7.1 — Elementix, define el carve-out de F)
> - [ ] Conseguir acceso al VPS + Google Workspace del cliente
> - [ ] Crear carpeta del cliente en Drive (`INNOVA_MASTER/reuniones/gpt-landings/`)
> - [ ] WhatsApp: arrancar verificación de número + submission de templates a Meta (lead time)
> - [ ] Estimar tiempos por módulo y arrancar por M0

---

## Riesgos

| Riesgo | Impacto | Mitigación |
| --- | --- | --- |
| **Elementix solo UI (Módulo F)** | Alto | Carve-out: fuera del precio cerrado; validar API en discovery 7.1 antes de comprometer |
| Lead time de templates WhatsApp (Meta) | Medio | Arrancar verificación + submission apenas haya número (def #5) |
| Guidelines de partners dispersas / informales | Medio | Relevar formato y cantidad en discovery (def #1) antes de modelar el motor |
| Alucinación de Claude en matching/extracción | Medio | Reglas duras deterministas primero; Claude solo para borrosos; validación humana final |
| Sin acceso al VPS / Workspace a tiempo | Alto | Bloquea M0 → bloquea todo; pedir accesos como prioridad #1 |
| Costo de Claude descontrolado (A + F) | Medio | Cap mensual definido en discovery (2.4 / 7.3); modelo barato por default |
| Datos sensibles (financieros del borrower) | Alto | Secrets en vault; storage con permisos; cumplimiento según jurisdicción (US/FL) |
