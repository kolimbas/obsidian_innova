---
tags:
  - innova
  - agente
  - n8n
  - flow-builder
  - nivel-2
estado: activo
updated: 2026-05-28
---

# 🧩 Subagente n8n Flow Builder

> Toma un requisito del usuario y construye un workflow de n8n end-to-end siguiendo el método Spec Kit. Antes de tocar n8n, deja en Obsidian un bundle completo (`spec.md`, `research.md`, `plan.md`, `tasks.md`) y al cerrar el flujo escribe `retro.md`. Aprende de los flujos previos: nunca diseña de cero si hay algo similar.

← Volver a [[agentes/agentes|Agentes]]

---

## Estado

✅ **Activo** — la base de conocimiento vive en `n8n/` (n8n KB).

---

## Cuándo se invoca

Cuando el usuario pide un nuevo flujo de n8n, ya sea con un mensaje informal ("necesito que cuando llegue X mail haga Y") o con un brief completo. El agente:

1. Confirma cliente y nombre del flujo.
2. Crea / verifica la carpeta del cliente y del flujo bajo `n8n/clients/<client>/flows/<flow>/`.
3. Arranca el ciclo Spec Kit.

No se invoca para tocar flujos ya existentes salvo que el usuario explícitamente pida una nueva iteración (entonces se versiona el bundle: `flows/<flow>/v2/`).

---

## Pipeline

| Paso | Output | Tool principal |
| --- | --- | --- |
| Intake | Carpeta de cliente + carpeta de flujo creadas | Filesystem |
| Spec | `spec.md` con objetivos, trigger, IO, criterios de éxito, open questions | — |
| Research | `research.md` citando flujos previos del cliente + cross-client + node docs + sistemas externos | `mcp__n8n__search_nodes`, `mcp__n8n__get_node`, `mcp__n8n__search_templates`, Grep sobre `n8n/clients/` y `n8n/patterns/` |
| Plan | `plan.md` con nodos, idempotencia, errores, credenciales, observabilidad, testing | — |
| Tasks | `tasks.md` con checklist verificable | — |
| Build | Workflow real en n8n + `workflow.json` exportado | `mcp__n8n__n8n_generate_workflow`, `mcp__n8n__n8n_create_workflow`, `mcp__n8n__n8n_update_partial_workflow` |
| Validate | Validaciones pasadas y ejecución de prueba OK | `mcp__n8n__validate_node`, `mcp__n8n__n8n_validate_workflow`, `mcp__n8n__n8n_test_workflow` |
| Retro | `retro.md` + bits promovidos a `patterns/` o `nodes/` + línea en `lessons-learned.md` | — |

Detalle: `n8n/METHODOLOGY.md`.

---

## Reglas duras

> [!warning] No negociables
> - **Nada se construye en n8n sin spec/research/plan/tasks aprobados.**
> - **`research.md` siempre se ejecuta**, incluso si el resultado es "no hay nada similar". Es lo que hace que el segundo flujo cueste menos que el primero.
> - **Reuso primero**: si un flujo previo cubre ≥60% del caso, se usa como base, no se diseña de cero.
> - **Idempotencia obligatoria** en todo write externo. El plan debe declarar la estrategia.
> - **Credenciales nunca inline.** Siempre en el credential store de n8n o env vars.
> - **Retro obligatorio**, aunque sean 3 bullets. Sin retro no hay aprendizaje.
> - **Toda la KB en inglés.** Las specs de agentes y notas de cliente "business" siguen en español.
> - **Open questions del spec se resuelven con el usuario antes de pasar a plan.**

---

## Output esperado por ciclo

Al cerrar un flujo, en la carpeta del flow tienen que existir:

```
n8n/clients/<client>/flows/<flow>/
├── spec.md      (status: live)
├── research.md
├── plan.md      (sincronizado con lo construido)
├── tasks.md     (todos los checks marcados o explicados)
├── retro.md
└── workflow.json
```

Y como mínimo una entrada en `n8n/lessons-learned.md`.

---

## Interacción con otros agentes

- **Pre-Contratación**: cuando una propuesta menciona flujos n8n concretos, este agente los toma como input inicial para el `spec.md`.
- **Mantenimiento**: consume los `workflow.json` versionados y los `plan.md` para entender qué workflow está corriendo y por qué.

---

## Pendientes

> [!todo] A iterar con uso real
> - [ ] Probar el ciclo end-to-end con un flujo real de Blincer cuando arranque Fase 2.
> - [ ] Definir si los `workflow.json` exportados se versionan en git o solo en n8n.
> - [ ] Política de naming de credenciales en n8n (sugerencia: `<client>-<system>-<env>`).
