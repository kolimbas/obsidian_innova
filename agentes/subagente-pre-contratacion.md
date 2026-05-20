---
tags:
  - innova
  - agente
  - pre-contratacion
estado: spec-completa
updated: 2026-05-20
---

# 🎙️ Subagente Pre-Contratación

> Transcribe audios de reuniones presenciales grabados en iPhone, interpreta el contenido y genera un `.docx` con resumen, propuesta de trabajo, mantenimiento, plazo y presupuesto.

← Volver a [[agentes/agentes|Agentes]]

---

## Estado

✅ **Spec cerrada** — lista para implementar. Templates definidos en [[agentes/templates/template-propuesta-web]] y [[agentes/templates/template-propuesta-automatizacion]].

---

## Arquitectura

```
~/Innova/reuniones/<cliente>/<fecha>/
    ├── audio1.m4a
    ├── audio2.m4a
    ├── ...
    └── LISTO          ← marker file que dispara el watcher
```

### 1. Watcher (proceso 24/7)

- Servicio `systemd` corriendo `inotifywait` sobre `~/Innova/reuniones/` recursivo.
- Cuando detecta **creación del archivo `LISTO`** en una subcarpeta `<cliente>/<fecha>/`:
  1. Confirma que la carpeta tiene al menos un audio.
  2. Loguea la activación (`~/Innova/reuniones/.watcher.log`).
  3. Invoca Claude Code en modo headless pasándole la ruta:
     ```
     claude -p "Procesá la reunión en <ruta>" --agent pre-contratacion
     ```
  4. Espera a que termine para volver a escuchar (evita reentradas).

### 2. Pipeline del subagente

1. Recibe la ruta `~/Innova/reuniones/<cliente>/<fecha>/`.
2. Lista los audios y los ordena por nombre o fecha de modificación.
3. Transcribe cada uno con **`faster-whisper`** (modelo `small` o `medium`, `language="es"`).
4. Concatena las transcripciones con marcadores `=== Audio 1 ===`, `=== Audio 2 ===`.
5. Detecta el **tipo de proyecto** según contenido del transcript:
   - Si menciona sitio web, dominio, hosting, e-commerce, dashboard → **web**.
   - Si menciona procesos, ERP, automatización, integración, n8n → **automatización**.
   - Si es ambiguo, usa **web** por defecto y deja una nota al final.
6. Elige el template correspondiente.
7. Extrae los campos clave del transcript:
   - Cliente, participantes, contexto, dolor, scope deseado, plazos discutidos, presupuesto mencionado, próximos pasos.
   - **Si un campo no se mencionó en el audio**, lo deja como `[A definir]` (no inventa).
8. Genera el `.docx` con **`python-docx`** siguiendo el template.
9. Guarda en `~/Innova/reuniones/<cliente>/<fecha>/propuesta.docx`.
10. Renombra `LISTO` → `PROCESADO_<timestamp>` para no disparar dos veces.
11. (Opcional fase 2) Actualiza la nota del cliente en `clientes/` con un resumen.

---

## Templates

- 🌐 [[agentes/templates/template-propuesta-web|Propuesta Web]] — clientes tipo Trujillo / Altius
- ⚙️ [[agentes/templates/template-propuesta-automatizacion|Propuesta Automatización]] — clientes tipo Blincer

---

## Dependencias del sistema

| Paquete | Origen | Para qué |
| --- | --- | --- |
| `inotify-tools` | `apt` | Watcher de filesystem |
| Python 3.10+ | sistema | Runtime |
| `faster-whisper` | `pip` | Transcripción local gratis |
| `python-docx` | `pip` | Generación `.docx` |
| Modelo Whisper `small` o `medium` | descarga automática | ~500 MB - 1.5 GB |
| Claude Code CLI | ya instalado | Ejecuta el subagente |

---

## Implementación pendiente

> [!todo] Para hacer
> - [ ] Crear `~/Innova/reuniones/` y agregar al gitignore si versionamos
> - [ ] Script de watcher: `~/Innova/bin/watcher.sh`
> - [ ] Servicio systemd: `~/.config/systemd/user/innova-watcher.service`
> - [ ] Pipeline Python: `~/Innova/bin/procesar-reunion.py` (transcribe + extrae + genera docx)
> - [ ] Archivo del subagente: `.claude/agents/pre-contratacion.md` con la system prompt
> - [ ] Templates `.docx` propiamente dichos (los `.md` de este vault son la spec; el subagente los rinde)
> - [ ] Probar end-to-end con un audio real

---

## Reglas duras para el agente

> [!warning] No negociables
> - **Idioma**: español argentino, voseo, sin tono robótico.
> - **Cifras**: nunca inventar precios. Si no se mencionó, `[A definir tras revisión interna]`.
> - **Plazos**: lo mismo — solo lo que se dijo o un rango razonable marcado como estimativo.
> - **Modalidad de cobro**: **por deliverable**, no por hora (regla de Innova).
> - **Riesgos**: incluir siempre la sección, incluso si el agente tiene que inferirlos del contexto.
> - **Transcripción completa**: siempre como anexo, con marcadores entre audios.
> - **No procesar dos veces**: si encuentra `PROCESADO_*` en la carpeta, abortar.

---

## Riesgos operativos

- **Calidad del audio**: ruido de fondo en reuniones presenciales puede dañar la transcripción. Mitigación: grabar cerca del interlocutor; modelo `medium` si `small` se queda corto.
- **Tiempo de transcripción**: 1 hora de audio ≈ 5-15 min con `faster-whisper small` en CPU. Avisar progreso en log.
- **Reentrada accidental**: marker file `LISTO` se elimina/renombra apenas arranca el procesamiento.
- **Privacidad**: audios y transcripts contienen datos sensibles. Definir política de retención (¿borrar audio post-procesamiento? ¿conservar transcript?).
