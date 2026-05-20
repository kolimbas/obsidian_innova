---
tags:
  - innova
  - template
  - propuesta
  - web
updated: 2026-05-20
---

# 🌐 Template — Propuesta de Sitio Web

> Spec del `.docx` que genera el [[agentes/subagente-pre-contratacion]] cuando detecta un proyecto **web**.

← Volver a [[agentes/subagente-pre-contratacion|Subagente Pre-Contratación]]

---

## Estructura del documento

### Encabezado
- Logo Innova Solutions (placeholder si no hay imagen)
- Título: **"Propuesta — Sitio Web Profesional"**
- Cliente: `[NOMBRE]`
- Fecha de reunión: `[YYYY-MM-DD]`
- Fecha del documento: `[HOY]`
- Participantes: `[LISTA]`

---

### 1. Resumen ejecutivo
3-5 líneas: qué hace el cliente, qué necesita, qué proponemos.

### 2. Contexto y necesidad
- Rubro y mercado objetivo
- Situación actual (sitio existente, redes, nada)
- Identidad de marca: paleta, tipografía, estilo, referencias visuales mencionadas
- Mensaje principal que el cliente quiere transmitir
- Audiencia

### 3. Propuesta de sitio
**Estructura propuesta** (páginas + secciones):
- Home, Servicios, Sobre Nosotros, Contacto, etc. (ajustar al rubro)

**Funcionalidades:**
- Dashboard de autogestión de contenido (edita textos, imágenes y secciones sin tocar código)
- Formulario de contacto → WhatsApp / email
- E-commerce (si aplica): botón → WhatsApp **o** checkout Stripe
- Mapa, blog, integración con redes (según corresponda)

**Stack técnico:**
| Capa | Herramienta |
| --- | --- |
| Diseño + front | Lovable |
| Backend + DB + auth | Supabase |
| Hosting + deploy | Vercel |
| Cobros (opcional) | Stripe |
| Dominio | A registrar / ya disponible |

### 4. Workflow (9 fases)
> Onboarding → Prompt Lovable → Generación → Revisión → Iteraciones → Integración (Claude Code + Supabase) → Testing → Deploy → Handoff

### 5. Plazos estimados
| Fase | Tiempo |
| --- | --- |
| Onboarding + diseño inicial | `[X días]` |
| Iteraciones (1-2 rondas típicas) | `[X días]` |
| Integración + testing | `[X días]` |
| Deploy + handoff | `[X días]` |
| **Total** | **`[X semanas]`** |

### 6. Presupuesto
- **Diseño + integración inicial**: `[$XXX]` (pago único)
- **Mantenimiento mensual**: `[$XXX/mes]` — alcance: cambios menores, backups, soporte y actualizaciones.
- **Modalidad**: por deliverable (no por hora).
- **Forma de pago**: 50% al inicio, 50% al deploy.
- **Hosting + dominio**: a cargo del cliente o pass-through con factura.

### 7. Próximos pasos
1. Confirmación de scope y presupuesto.
2. Cliente completa el formulario de onboarding (colores, fotos, referencias).
3. Inicio del diseño en Lovable.

### 8. Riesgos y supuestos
- Listado breve de cosas a confirmar antes de arrancar (fotos propias, dominio, etc.).

### Anexo A — Transcripción completa de la reunión
Texto íntegro con marcadores `=== Audio 1 ===` si hay más de uno.

---

## Notas para el subagente

> [!info]
> - Si el cliente no mencionó precios o plazos, usar `[A definir tras revisión interna]`.
> - Si el cliente menciona venta vía WhatsApp explícitamente, **no** proponer Stripe checkout.
> - El bloque de **mantenimiento mensual** siempre va, aunque el cliente no lo haya pedido — es parte del modelo de Innova.
