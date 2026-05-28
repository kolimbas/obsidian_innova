---
tags:
  - innova
  - agente
  - pre-contratacion
  - nivel-2
estado: activo
updated: 2026-05-20
cssclasses:
---

# 🎙️ Subagente Pre-Contratación

> Transcribe audios de reuniones presenciales cargados en Google Drive, los interpreta con Claude y genera un `.docx` con resumen, propuesta de trabajo, mantenimiento, plazo y presupuesto. Devuelve el `.docx` a Drive y deja una nota en Obsidian.

← Volver a [[agentes/agentes|Agentes]]

---

## Estado

✅ **Activo** — servicio `innova-watcher.service` corriendo, polling Drive cada 60s. Listo para procesar reuniones reales.

---

## 🔗 Recursos

| Recurso                               | Ubicación                                                                                  |
| ------------------------------------- | ------------------------------------------------------------------------------------------ |
| 📂 Drive — INNOVA_MASTER              | [Abrir en Drive](https://drive.google.com/drive/folders/1UlR_MklVV0bMz0CjXTm6O8MDecTL4t4M) |
| 📂 Drive — reuniones (drop de audios) | [Abrir en Drive](https://drive.google.com/drive/folders/1AGKCx0QzToF3o-nB04Qwkm7OKCOIfWiG) |
| 💻 Código del pipeline                | `~/Innova/` (fuera del vault)                                                              |
| 📋 README del pipeline                | `~/Innova/README.md`                                                                       |
| 📜 Logs                               | `~/Innova/logs/watcher.log`                                                                |
| ⚙️ Servicio                           | `systemctl --user status innova-watcher`                                                   |

---

## Arquitectura

```
Google Drive:
  INNOVA_MASTER/reuniones/<cliente>/<fecha>/
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

## Cómo usarlo (uso diario)

1. Entrá a [INNOVA_MASTER/reuniones/](https://drive.google.com/drive/folders/1AGKCx0QzToF3o-nB04Qwkm7OKCOIfWiG) en Drive.
2. Creá la carpeta del cliente si no existe: `<cliente>/`.
3. Adentro creá la carpeta de la fecha: `<YYYY-MM-DD>/`.
4. Subí todos los audios de la reunión.
5. Cuando terminaste, creá un archivo vacío llamado **`LISTO`** en la carpeta de la fecha.
6. Esperá 1-2 min. Va a aparecer `propuesta.docx` en la misma carpeta, y una nota en `clientes/reuniones/<cliente>-<fecha>.md` del vault.

> Para ver el progreso en vivo: `tail -f ~/Innova/logs/watcher.log`.

---

## Setup ✅ Completado el 2026-05-20

- [x] Proyecto Google Cloud creado (`scenic-scholar-496919-u1`)
- [x] Drive API habilitada
- [x] OAuth Desktop credentials → `~/Innova/credentials/credentials.json`
- [x] `INNOVA_MASTER/reuniones/` creada en Drive (ID `1AGKCx0QzToF3o-nB04Qwkm7OKCOIfWiG`)
- [x] `drive_auth.py` corrido → `token.json` guardado
- [x] `.env` configurado
- [x] `innova-watcher.service` instalado, **enabled** y **active (running)**

### Pendientes operativos

- [ ] `sudo loginctl enable-linger ventas4` — para que el servicio sobreviva al logout
- [ ] Probar end-to-end con un audio real
- [x] Regenerar `client_secret` OAuth (el original quedó expuesto en chat el 2026-05-20)
- [x] Revocar PAT de GitHub que se usó para los primeros pushes

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
