---
tags:
  - innova
  - agentes
  - hub
updated: 2026-05-20
---

# 🤖 Agentes

> Hub de subagentes de Claude Code construidos para el flujo de trabajo de Innova.

← Volver a [[INNOVA_MASTER]]

---

## Catálogo

| Agente | Función | Estado |
| --- | --- | --- |
| [[agentes/subagente-pre-contratacion\|Pre-Contratación]] | Transcribe audios de reuniones presenciales (iPhone) y genera propuesta en `.docx` con scope, mantenimiento, plazo y presupuesto | ✅ Spec cerrada |
| _Segundo agente_ | _Pendiente de definir_ | — |

---

## Templates

- 🌐 [[agentes/templates/template-propuesta-web]] — usado por Pre-Contratación para proyectos web
- ⚙️ [[agentes/templates/template-propuesta-automatizacion]] — usado por Pre-Contratación para proyectos de automatización

---

## Convenciones

- Cada agente vive en una nota en `agentes/`.
- La spec en Obsidian es la fuente de verdad de **qué hace**, **cuándo se invoca**, y **qué entrega**.
- La implementación real va en `.claude/agents/<nombre>.md` del proyecto donde se use.
- Cambios en el comportamiento del agente → primero se actualiza la spec acá, después se aplica al archivo de Claude Code.
