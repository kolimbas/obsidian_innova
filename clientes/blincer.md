---
tags:
  - cliente
  - automatizacion
  - activo
  - nivel-2
estado: pre-venta
fase: 0
updated: 2026-05-11
---

# BLINCER

> Fábrica de cerraduras · Argentina · Mismos dueños que GAAB

---

## Contacto clave

| Persona | Rol | Área |
| --- | --- | --- |
| **Sandra Cerezo** | Gerente Administrativa | Administración / Contabilidad |
| **Guillermo Berardi** | Coordinador Comercial | Comercial / Marketing |

---

## Contexto

Sandra se jubila este año — hay que capturar su conocimiento y automatizar sus procesos **antes de que se vaya**. Guillermo quiere transformación digital y trazabilidad completa del pedido.

> [!warning] Confirmar urgente
> ¿Cuándo exactamente se jubila Sandra? Julio o diciembre cambia todo el timeline.

---

## Stack del cliente

- **Tango Gestión LOCAL** (servidor on-prem `BLI-SVRV-TANGO`, SQL Server 2019, base `BLINCER_SRL`) — confirmado 2026-06-12
- Bancos: Galicia · BBVA · Cooperativa
- Gmail · Google Drive · Google Sheets
- Mailchimp · Metricool · Google Ads

> [!success] Riesgo Tango RESUELTO (2026-06-12)
> Tango es local **pero** sobre SQL Server → se integra por **consultas SQL de solo-lectura** (directo, robusto), NO por CSV/RPA como se temía. No hace falta migrar a Nexo. Ya hay 2 vistas read-only + usuario de solo-lectura creados. Detalle: [[n8n/clients/blincer/tango-integration|Tango integration]] · accesos en `accesos.md`.

---

## Modelo de cobro

**Por deliverable, no por hora.** Cada fase tiene precio cerrado. Cuanto más eficiente la ejecución, mejor el margen.

---

## Áreas y proyectos

> [!info] Tanda oficial 2026-05-29 — 5 flujos confirmados por el cliente
> El cliente bajó alcance concreto sobre Tango + HubSpot. Specs en `n8n/clients/blincer/flows/`:
> 1. [[n8n/clients/blincer/flows/credit-limit-invoice-block/spec|Bloqueo de facturación por deuda]]
> 2. [[n8n/clients/blincer/flows/whatsapp-overdue-debt-reminder/spec|Avisos de deuda vencida por WhatsApp]]
> 3. [[n8n/clients/blincer/flows/sales-bot-with-quotes/spec|Bot de ventas + cotizaciones/facturas]] (acopla flujos 3 y 4 de la tanda)
> 4. [[n8n/clients/blincer/flows/email-remarketing/spec|Remarketing y difusiones email]]
>
> Estos 5 ítems no reemplazan el roadmap original de Sandra/Guillermo (más abajo) — son la primera tanda en producción.

### Área Sandra — Administración

> [!tip] Quick wins (mayor impacto, arrancar por acá)
> 1. **Conciliación bancaria** (3 bancos + VISA) — ~8-12 hs/mes ahorradas
> 2. **Consolidador Info Gerencia** (quincenal + mensual) — ~6 hs/mes
> 3. **Facturación recurrente Blincer + reparto socios** — ~3 hs/mes
> 4. **Calendario DDJJ y vencimientos** — preventivo, evita multas
> 5. **Seguimiento marcas y patentes** (Jaque, GAAB, ZIAC) — repositorio centralizado
> 6. **Asientos contables recurrentes** — ~4 hs/mes

**No automatizable — reasignar:**
Coordinación inter-áreas · Criterio contable · Relación con socios y Estudio Castro · Arqueo de caja físico

### Área Guillermo — Comercial

| Proyecto | Descripción | Prioridad |
| --- | --- | --- |
| **C1 — Trazabilidad integral** | Dashboard pedido → stock → producción → despacho | ⭐ Más estratégico |
| **C2 — Omnicanal** | WhatsApp + Email + Web en una bandeja (Chatwoot o SaaS) | Alta |
| **C3 — Alertas stock crítico** | Notificación automática cuando SKU baja de umbral | Media |
| **C4 — Marketing automation** | Reportes Mailchimp + Google Ads + Metricool unificados | Media |
| **C5 — Separación cobranzas** | Sacar cobranzas del área comercial | Media |

> [!warning] C1 depende de la visita
> Si producción no tiene sistema digital, Trazabilidad se vuelve mucho más complejo. Ver en la visita qué hay del lado producción antes de dar un precio.

---

## Fases del proyecto

| Fase | Descripción | Estado |
| --- | --- | --- |
| **Fase 0** | Discovery — visita fábrica, entrevistas Sandra/Guillermo, confirmar Tango, lista de accesos | 🔲 Pendiente |
| **Fase 1** | Infraestructura — VPS, n8n, Postgres, conexiones base | 🔲 Pendiente |
| **Fase 2** | Quick wins Sandra (todos los 🟢 del área contable) | 🔲 Pendiente |
| **Fase 3** | Proyectos comerciales Guillermo (C1 al C5, a confirmar prioridades) | 🔲 Pendiente |
| **Fase 4** | Transferencia, documentación y capacitación del equipo | 🔲 Pendiente |
| **Retainer** | Soporte mensual + mejoras continuas desde mes 1 | 🔲 Pendiente |

---

## Estado del proyecto

> [!todo] Próximos pasos
> - [ ] Confirmar fecha de jubilación de Sandra
> - [ ] Definir precio por fase (deliverable, no por hora)
> - [ ] Armar estimate en QuickBooks — Opción A (fases aprobables independientemente)
> - [ ] Preparar agenda para la visita a la fábrica
> - [x] Confirmar versión de Tango (local vs Nexo) → **LOCAL/SQL Server**, integración por SQL read-only (sin migrar a Nexo) — ver [[n8n/clients/blincer/tango-integration|Tango integration]]
> - [~] Lista de credenciales/accesos a pedir al cliente → Tango/SQL/server ya en `accesos.md`; faltan TeamViewer y VPS
> - [ ] Arrancar entrevistas con Sandra lo antes posible (no esperar a Fase 0)

---

## Riesgos

| Riesgo | Impacto | Mitigación |
| --- | --- | --- |
| ~~Tango local sin API~~ → **RESUELTO** | ~~Alto~~ Bajo | Es local pero sobre SQL Server → SQL read-only (sin RPA). 2 vistas + usuario `blincer_n8n_ro` creados 2026-06-12 |
| Sandra como único knowledge holder | Alto | Empezar entrevistas YA, antes de la visita formal |
| Producción sin sistema digital | Medio | Ver en visita antes de pricear C1 |
| Resistencia al cambio del equipo | Medio | Capacitación bien estructurada en Fase 4 |
| Operaciones de exportación sensibles | Medio | Workflows idempotentes con manejo robusto de errores |
| Credenciales bancarias | Medio | Vault seguro (Doppler o Bitwarden Secrets) |

## Capturado por bot
- [ ] confirmar acceso tango (bot 11/06/2026)
