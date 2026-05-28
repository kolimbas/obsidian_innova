---
tags:
  - innova
  - workflow
  - nivel-2
updated: 2026-05-28
---

# 🚀 Onboarding de cliente nuevo

> Pasos a ejecutar cuando entra un cliente nuevo, desde la firma hasta el handoff de Fase 1.

← Volver a [[workflows/workflows|Workflows]]

---

## 1. Apertura

- Crear nota del cliente en `clientes/<slug>.md` con frontmatter (`estado: pre-venta`, `fase: 0`).
- Agregar fila en `clientes/clientes.md`.
- Crear carpeta del cliente en Google Drive (`INNOVA_MASTER/reuniones/<cliente>/`).

## 2. Discovery

- Reunión presencial o videollamada. Grabar audio.
- Subir audio a `reuniones/<cliente>/<YYYY-MM-DD>/` en Drive y crear el archivo `LISTO`.
- El subagente Pre-Contratación transcribe y genera la propuesta `.docx` automáticamente.

## 3. Propuesta y firma

- Revisar el `.docx` generado, ajustar pricing si hace falta.
- Mandar al cliente por mail.
- Si aprueba: firma de propuesta + 50% del primer deliverable.

## 4. Setup técnico

- Crear repo (si aplica).
- Provisionar Supabase / n8n / dominio según corresponda.
- Pedir credenciales y accesos del lado del cliente.

## 5. Handoff a Fase 1

- Cambiar `estado` en la nota del cliente a `en-curso`.
- Actualizar tabla en `clientes/clientes.md`.
- Arrancar Fase 1 según tipo de proyecto (web → workflow web, automatización → spec de n8n).
