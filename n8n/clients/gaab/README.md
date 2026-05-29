---
tags:
  - n8n
  - cliente
  - nivel-3
client: gaab
status: pre-info
updated: 2026-05-29
---

# GAAB — n8n Workspace

> Hermana de Blincer (mismos dueños). Por ahora la única tarea identificada es **Productos estancados**, a la espera del detalle de Gabriel.

← [[n8n/clients/_index|Clients]]

---

## Scope (current understanding)

Tarea pendiente única: **Productos estancados.** El usuario está esperando que Gabriel mande el detalle completo para definir alcance, inputs y criterios de éxito. Hasta entonces no se abre `spec.md`.

> [!warning] No abrir spec sin el detalle de Gabriel
> El METHODOLOGY exige una requirement clara antes de Phase 1. Crear `spec.md` con suposiciones acá generaría research y plan sobre arena.

## Stack

Heredamos los supuestos de Blincer hasta que la tarea de GAAB se documente formalmente:

- Tango (versión TBC — probablemente la misma instancia o una hermana)
- HubSpot (a confirmar si tienen instancia separada)
- Periféricos a definir cuando llegue la info

## Credentials policy

A definir cuando se abra el primer flow. Por defecto se aplica la misma política que Blincer: todo en n8n credentials, nada inline.

## Flows

| Flow | Status | Trigger | Last update |
| --- | --- | --- | --- |
| _ninguno todavía_ |  |  |  |

## Open dependencies

- [ ] **Bloqueante:** Recibir de Gabriel el detalle de Productos estancados (qué se considera estancado, fuente del dato, qué acción se espera, frecuencia).
- [ ] Confirmar si GAAB usa instancias propias de Tango/HubSpot o comparte con Blincer.
- [ ] Identificar stakeholder operativo del lado GAAB (¿quién consume la salida del flow?).

## Links

- Cliente hermano: [[n8n/clients/blincer/README|Blincer]]
- Business note Blincer (referencia): `clientes/blincer.md`
