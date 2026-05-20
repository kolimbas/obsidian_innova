---
tags:
  - innova
  - master
updated: 2026-05-10
---

# 🏢 INNOVA SOLUTIONS

> Agencia de sitios web profesionales con dashboard de autogestión para PyMEs argentinas.

---

## Stack

| Herramienta | Rol |
| --- | --- |
| **Lovable** | Diseño y generación de UI |
| **Claude Code** | Integración, lógica, Supabase |
| **Supabase** | Base de datos, storage, auth |
| **Vercel** | Hosting y deploy |
| **GitHub** | Control de versiones |
| **Stripe** | Cobros y suscripciones |
| **N8N + Evolution API** | Automatización comercial (WhatsApp) |

> [!info] Supabase
> **Proyecto ID:** `fcgfhoulgvusuledobwj`
> **Tablas clave:** `clients` · `site_content` · `site_styles`
> **Dashboard:** innovasolutionsdashboard.com

---

## Modelo de negocio

Cada cliente tiene un `CLIENT_ID` que conecta su sitio con su contenido en Supabase. El cliente edita su contenido desde el dashboard sin tocar código. Innova cobra diseño + integración inicial, más mantenimiento mensual.

---

## Workflow por cliente

> [!example] Las 9 fases
> **1. Onboarding** → Cliente completa formulario (nombre, colores, fotos, referencias)
> **2. Prompt Lovable** → Claude genera el prompt completo en español argentino
> **3. Generación** → Pegar en Lovable, esperar resultado
> **4. Revisión** → Screenshots a Claude, análisis + prompt de correcciones
> **5. Iteraciones** → Repetir hasta aprobación del cliente
> **6. Integración** → Claude Code conecta Supabase, lógica de dashboard
> **7. Testing** → Entorno Lovable Test (DB aislada de producción)
> **8. Deploy** → Vercel + dominio (conectar Supabase desde marketplace de Vercel)
> **9. Handoff** → Cliente recibe acceso al dashboard de autogestión

---

## Clientes

→ Ver [[clientes/clientes|Clientes]] (hub con la cartera completa)

| Cliente | Tipo | Estado | Fase |
| --- | --- | --- | --- |
| Estudio Trujillo Abogados | 🌐 Web | 🟡 En correcciones | 4 |
| ALTIUS NUTRITION | 🌐 Web | 🟡 En correcciones v2 | 4 |
| Blincer | ⚙️ Automatización | 🔵 Pre-venta | 0 |

---

## Skills activas — aplicar ahora

> [!tip] Top 5 para esta semana
> 1. **Lovable MCP Server** → Claude Code Settings → Connectors → `https://mcp.lovable.dev`
> 2. **Context7 MCP** → `context7.com/docs/clients/claude-code` — doc actualizada de Next.js y Supabase
> 3. **Lovable Themes** → Definir paleta del cliente antes del primer prompt
> 4. **Supabase Stripe Sync Engine** → Activar en dashboard Supabase al próximo proyecto con cobros
> 5. **Lovable Test/Live Environments** → Siempre trabajar en Test antes de tocar Live

→ Ver [[research/research-de-skills|Research de Skills]] (hub con todas las entradas semanales)

---

## Pendientes globales

> [!todo] Por resolver
> - [ ] Discord INNOVA WEBS — crear canales y categorías (script Node.js pendiente)
> - [ ] Trujillo — aprobación final y deploy
> - [ ] Altius — correcciones v2 + fotos reales de productos
> - [ ] Conectar Lovable MCP en Claude Code

---

## Notas clave

> [!warning] Recordar siempre
> - Todos los prompts de Lovable **en español argentino**
> - Modelo de venta de Altius es por **WhatsApp**, no ecommerce con checkout
> - Octogent (multi-agente visual) no funciona en Windows — descartado
> - Usar **Lovable Test** antes de cualquier cambio en producción
