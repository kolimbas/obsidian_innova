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

| Agente                                                   | Función                                                                                                                                                   | Estado   |
| -------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| [[agentes/subagente-pre-contratacion\|Pre-Contratación]] | Transcribe audios de reuniones presenciales (iPhone) cargados en Google Drive y genera propuesta en `.docx` con scope, mantenimiento, plazo y presupuesto | ✅ Activo |
| [[agentes/subagente-mantenimiento\|Mantenimiento]] | Monitorea los n8n de cada cliente, arregla solo lo que puede (retries, reauth, rewrites con snapshot+rollback) y avisa por mail al equipo cuando no puede | 🚧 Diseño cerrado |
| [[agentes/subagente-n8n-flow-builder\|n8n Flow Builder]] | Toma un requisito y construye el workflow en n8n siguiendo Spec Kit (spec → research → plan → tasks → build → retro). Reusa flujos previos y deja todo documentado en [[n8n/README\|n8n KB]] | ✅ Activo |

---

## Recursos compartidos

- 📂 [INNOVA_MASTER en Drive](https://drive.google.com/drive/folders/1UlR_MklVV0bMz0CjXTm6O8MDecTL4t4M) — carpeta raíz para todos los agentes

---

## Convenciones

- Cada agente vive en una nota en `agentes/`.
- La spec en Obsidian es la fuente de verdad de **qué hace**, **cuándo se invoca**, y **qué entrega**.
- La implementación real va en `.claude/agents/<nombre>.md` del proyecto donde se use.
- Cambios en el comportamiento del agente → primero se actualiza la spec acá, después se aplica al archivo de Claude Code.
