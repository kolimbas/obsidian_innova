---
tags:
  - innova
  - agente
  - pre-contratacion
estado: implementado
updated: 2026-05-20
---

# 🎙️ Subagente Pre-Contratación

> Transcribe audios de reuniones presenciales cargados en Google Drive, los interpreta con Claude y genera un `.docx` con resumen, propuesta de trabajo, mantenimiento, plazo y presupuesto. Devuelve el `.docx` a Drive y deja una nota en Obsidian.

← Volver a [[agentes/agentes|Agentes]]

---

## Estado

🛠️ **Implementado** — código en `~/Innova/`. Falta hacer el setup de Google Drive OAuth (una sola vez) y arrancar el servicio.

---

## Arquitectura

```
Google Drive:
  Innova/reuniones/<cliente>/<fecha>/
      ├── audio1.m4a
      ├── audio2.m4a
      └── LISTO          ← marker que dispara el procesamiento
                            (cuando termina, queda como PROCESADO_<timestamp>)
```

### 1. Watcher 24/7 (`~/Innova/bin/procesar.py`)

- Servicio `systemd` user-level que corre un loop Python.
- Cada **60 segundos** consulta Drive vía API por archivos llamados `LISTO`.
- Cuando encuentra uno:
  1. Lee la carpeta padre (`<fecha>/`) y la abuela (`<cliente>/`).
  2. Verifica que haya al menos 1 audio.
  3. Lockea la carpeta renombrando `LISTO` → `EN_PROCESO_<ts>`.
  4. Dispara el pipeline.

### 2. Pipeline

1. **Descarga** los audios a `/tmp/innova/<reunion_id>/`.
2. **Transcribe** cada audio con `faster-whisper` (modelo `small`, idioma `es`).
3. **Concatena** los transcripts con marcadores `=== Audio 1 ===`.
4. **Llama a Claude** vía `claude -p` (modo headless, usa la suscripción) con un prompt que:
   - Recibe el transcript + el nombre del cliente.
   - Detecta el tipo de proyecto (web vs automatización).
   - Extrae los campos clave en JSON.
5. **Genera el `.docx`** con `python-docx`, eligiendo template (web o automatización).
6. **Sube** `propuesta.docx` a la misma carpeta de Drive.
7. **Escribe** una nota en `clientes/reuniones/<cliente>-<fecha>.md` del vault con resumen + link al docx.
8. **Renombra** `EN_PROCESO_*` → `PROCESADO_<ts>` en Drive.
9. **Limpia** el directorio temporal.

### 3. Idempotencia

- Si una carpeta ya tiene un archivo `PROCESADO_*`, el watcher la ignora.
- Si el pipeline falla en el medio, el lock `EN_PROCESO_*` queda. Al reiniciar el servicio se reintenta una vez; si vuelve a fallar, queda como `ERROR_<ts>` para revisión manual.

---

## Templates

- 🌐 [[agentes/templates/template-propuesta-web]] — proyectos web
- ⚙️ [[agentes/templates/template-propuesta-automatizacion]] — proyectos de automatización

---

## Stack técnico

| Componente | Tecnología | Costo |
| --- | --- | --- |
| Detección de audios | Google Drive API v3 (polling) | Gratis (cuota generosa) |
| Transcripción | `faster-whisper` local, modelo `small` | Gratis, offline |
| Análisis del transcript | `claude -p` (Claude Code headless) | Usa la suscripción, sin token extra |
| Generación `.docx` | `python-docx` | Gratis |
| Watcher | Python + systemd user service | Gratis |
| Integración Obsidian | Escritura directa de `.md` en el vault | Gratis |

---

## Setup pendiente (una sola vez, manual)

> [!warning] Pasos que hace el usuario
> 1. Crear proyecto en [Google Cloud Console](https://console.cloud.google.com) y habilitar **Google Drive API**.
> 2. Crear credenciales OAuth 2.0 tipo "Desktop app". Bajar `credentials.json` a `~/Innova/credentials/`.
> 3. Crear la carpeta `Innova/reuniones/` en su Google Drive (a mano).
> 4. Anotar el **folder ID** de `Innova/reuniones/` (último segmento del URL).
> 5. Correr `python ~/Innova/bin/drive_auth.py` — abre el navegador, autorizás, queda `token.json` guardado.
> 6. Configurar `~/Innova/.env` con el folder ID raíz.
> 7. `systemctl --user enable --now innova-watcher.service`.
>
> Detalles paso a paso en `~/Innova/README.md`.

---

## Reglas duras del agente

> [!warning] No negociables
> - **Idioma**: español argentino, voseo, sin tono robótico.
> - **Cifras**: nunca inventar precios. Si no se mencionaron, `[A definir tras revisión interna]`.
> - **Plazos**: solo lo que se dijo o un rango estimativo bien marcado.
> - **Modalidad de cobro**: **por deliverable**, no por hora (regla de Innova).
> - **Riesgos**: incluir siempre la sección, inferidos del contexto si hace falta.
> - **Transcripción completa**: como anexo del `.docx`, con marcadores entre audios.
> - **No procesar dos veces**: si la carpeta tiene `PROCESADO_*`, abortar.

---

## Riesgos operativos

- **Calidad del audio**: ruido de fondo daña la transcripción. Mitigación: grabar cerca del interlocutor; cambiar modelo a `medium` si `small` se queda corto.
- **Tiempo de proceso**: 1 hora de audio ≈ 5-15 min con `small` en CPU. El watcher loguea avance en `~/Innova/logs/`.
- **Cuota Drive API**: 1000 requests / 100s por usuario — sobra para polling cada 60s.
- **Privacidad**: audios y transcripts contienen datos sensibles. Por defecto se borran de `/tmp/` tras procesar; los originales quedan en Drive del usuario.
- **Auto-commit a Obsidian**: la nota se escribe en el vault local pero **no se commitea automáticamente** — el usuario revisa y pushea cuando quiera.
