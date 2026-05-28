---
tags:
  - innova
  - template
  - propuesta
  - automatizacion
  - nivel-3
updated: 2026-05-20
---

# ⚙️ Template — Propuesta de Automatización

> Spec del `.docx` que genera el [[agentes/subagente-pre-contratacion]] cuando detecta un proyecto **de automatización**.

← Volver a [[agentes/subagente-pre-contratacion|Subagente Pre-Contratación]]

---

## Estructura del documento

### Encabezado
- Logo Innova Solutions (placeholder si no hay imagen)
- Título: **"Propuesta — Automatización de Procesos"**
- Cliente: `[NOMBRE]`
- Fecha de reunión: `[YYYY-MM-DD]`
- Fecha del documento: `[HOY]`
- Participantes: `[LISTA + roles]`

---

### 1. Resumen ejecutivo
3-5 líneas: qué hace la empresa, qué duele, qué proponemos.

### 2. Contexto y diagnóstico
- **Empresa**: rubro, tamaño aproximado, áreas involucradas.
- **Stack actual del cliente**: ERP, bancos, herramientas (Mailchimp, Google, etc.). Marcar si tiene API o no.
- **Personas clave**: nombre, rol, área. Identificar *knowledge holders* críticos.
- **Procesos identificados**: lista de tareas manuales, repetitivas, propensas a error.
- **Urgencia / drivers**: jubilaciones, deadlines, presión de socios, regulatorio, etc.

### 3. Proyectos propuestos
| Proyecto | Descripción | Impacto estimado | Prioridad |
| --- | --- | --- | --- |
| P1 | `[ej. Conciliación bancaria]` | `~X hs/mes ahorradas` | Alta |
| P2 | `[ej. Trazabilidad pedido → despacho]` | Estratégico | Alta |
| ... | | | |

**Tareas NO automatizables — reasignar:**
Lista de cosas que requieren criterio humano (relación con socios, arqueo físico, etc.).

### 4. Fases del proyecto
| Fase | Descripción | Deliverables |
| --- | --- | --- |
| **Fase 0** | Discovery — visita, entrevistas, accesos, confirmar stack | Doc de procesos · lista de credenciales · roadmap detallado |
| **Fase 1** | Infraestructura | VPS · n8n · Postgres · conexiones base · vault de secretos |
| **Fase 2** | Quick wins | Implementación de los proyectos verde (alto impacto, baja complejidad) |
| **Fase 3** | Proyectos estratégicos | Trazabilidad, omnicanal, dashboards, etc. |
| **Fase 4** | Transferencia + capacitación | Docs operativos · sesiones con el equipo · runbooks |
| **Retainer** | Soporte mensual | Tickets · mejoras · monitoring · backups |

### 5. Plazos estimados
| Fase | Tiempo |
| --- | --- |
| Fase 0 | `[X semanas]` |
| Fase 1 | `[X semanas]` |
| Fase 2 | `[X semanas]` |
| Fase 3 | `[X semanas]` |
| Fase 4 | `[X semanas]` |
| **Total hasta retainer** | **`[X meses]`** |

### 6. Presupuesto
- **Modalidad: por deliverable** (cada fase tiene precio cerrado, no se cobra por hora).
- **Desglose**:
  | Fase | Precio | Forma de pago |
  | --- | --- | --- |
  | Fase 0 | `[$XXX]` | 100% al cierre |
  | Fase 1 | `[$XXX]` | 50% inicio / 50% cierre |
  | Fase 2 | `[$XXX]` | 50/50 |
  | Fase 3 | `[$XXX]` | 50/50 |
  | Fase 4 | `[$XXX]` | 100% al cierre |
- **Retainer mensual**: `[$XXX/mes]` — alcance: soporte, monitoreo, mejoras menores, X horas/mes incluidas.
- **Buffer de riesgo**: si hay incertidumbre técnica (ej. ERP local sin API → RPA), aplicar +20-30% al precio de la fase afectada.

### 7. Riesgos y supuestos
| Riesgo | Impacto | Mitigación |
| --- | --- | --- |
| ERP sin API | Alto | RPA / migración a versión cloud |
| Knowledge holder único | Alto | Entrevistas tempranas + documentación |
| Producción sin sistema digital | Medio | Validar en visita antes de pricear |
| Resistencia al cambio | Medio | Capacitación bien estructurada en Fase 4 |
| Credenciales sensibles | Medio | Vault seguro (Doppler / Bitwarden Secrets) |

### 8. Próximos pasos
1. Confirmación de scope y presupuesto.
2. Agenda para visita a planta / oficina del cliente.
3. Lista de credenciales y accesos a pedir.
4. Inicio de entrevistas con los *knowledge holders* lo antes posible.

### Anexo A — Transcripción completa de la reunión
Texto íntegro con marcadores `=== Audio N ===`.

---

## Notas para el subagente

> [!info]
> - **Por deliverable, nunca por hora** — regla dura de Innova.
> - Si el cliente tiene un *knowledge holder* que se va (jubilación, renuncia), marcar como **riesgo crítico** y proponer arrancar entrevistas **antes** de la Fase 0 formal.
> - Si menciona un ERP, confirmar versión (local vs cloud) — es la prioridad #1 de la Fase 0.
> - Si no se mencionó algo, usar `[A definir]` o `[Confirmar en visita]`. Nunca inventar.
> - El retainer siempre va, aunque el cliente no lo haya pedido — es parte del modelo de Innova.
