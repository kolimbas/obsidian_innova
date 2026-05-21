---
tags:
  - innova
  - agente
  - mantenimiento
estado: diseño
updated: 2026-05-21
---

# 🔧 Subagente Mantenimiento

> Monitorea las instancias de n8n de cada cliente, detecta ejecuciones fallidas o workflows con problemas, y los arregla automáticamente cuando puede. Si no puede, manda un mail al equipo con diagnóstico.

← Volver a [[agentes/agentes|Agentes]]

---

## Estado

🚧 **Diseño cerrado** — pendiente: alias del equipo, transporte SMTP, registro inicial de clientes.

---

## Cuándo actúa

El subagente corre como **servicio 24/7** (systemd, igual que [[agentes/subagente-pre-contratacion|Pre-Contratación]]). Cada ciclo (configurable, por defecto cada **15 minutos**) hace una pasada por todos los clientes registrados y procesa los incidentes que detecta.

No se invoca a mano. Es totalmente automático.

---

## Arquitectura

```
~/Innova/
├── clients-n8n.yaml         ← registro de clientes con su n8n
├── bin/
│   └── mantenimiento.py     ← watcher + pipeline
└── logs/
    └── mantenimiento.log
```

### Registro de clientes — `clients-n8n.yaml`

```yaml
clients:
  - id: blincer                              # slug, matchea clientes/blincer.md
    name: Blincer
    n8n_url: https://n8n.blincer.example
    api_key: ${BLINCER_N8N_API_KEY}          # interpolado desde .env
    contact_email: guillermo@blincer.example # solo informativo
    poll_interval_minutes: 15
    paused: false                            # toggle manual si hay maintenance
```

Cada cliente con n8n productivo se suma como entrada. Si `paused: true`, el subagente lo ignora.

### Pipeline por cliente, por ciclo

1. **Health check** — `GET /healthz` o equivalente. Si la instancia no responde, ya es un incidente.
2. **Lista de ejecuciones recientes** — `n8n_executions` filtrando `status=error` y `lastN=X` (X = config por cliente).
3. **Para cada ejecución fallida**:
   - Si ya fue procesada antes (ID en el log de procesados): saltar.
   - Si no: pasar a diagnóstico.
4. **Diagnóstico** — llamada a `claude -p` con:
   - El error completo del nodo que falló (mensaje + stack + input).
   - El workflow en JSON.
   - Histórico breve de fallas previas en ese workflow.
   - Claude devuelve JSON con: categoría del error, fix sugerido, nivel de confianza, plan de verificación.
5. **Aplicación del fix** según categoría (ver tabla más abajo).
6. **Verificación** — test execution con un payload conocido o re-ejecución de la falla original.
7. **Cierre**:
   - ✅ Si verifica OK: log + marcar como resuelto.
   - ❌ Si no verifica o no se puede fixear: agregar a la cola de email.
8. **Al final del ciclo**: si hay cola de incidentes no resueltos, mandar **un solo mail batch** al equipo con todos los pendientes (evita spam).

---

## Capacidades de auto-fix (modo **amplio** elegido)

| Categoría | Acción automática | Guardrail |
| --- | --- | --- |
| **Retry simple** (timeout, 503, rate-limit) | Re-ejecutar con backoff exponencial (1m, 5m, 15m) | Máximo 3 intentos antes de escalar |
| **Auth expirado** (401 con token OAuth/API) | Disparar refresh del credential. Si no hay refresh token disponible, escalar | Si el refresh falla 2 veces seguidas, escalar |
| **Config simple cambió** (URL/header/param) | Actualizar el nodo afectado en el workflow | Snapshot del workflow ANTES de modificar (vía `n8n_workflow_versions`) |
| **API breaking change** (cambió schema, v1→v2) | Reescribir el nodo según patrón conocido | Snapshot + test ejecución antes de marcar como resuelto. Si test falla, **rollback automático** |
| **Workflow desactivado por error** | Re-activar si los últimos N runs son OK | No re-activar si fue desactivado manualmente (heurística por timestamp del cambio) |
| **Error nuevo / desconocido** | NO auto-fix. Escalar inmediatamente | — |

### Guardrails universales (no negociables)

> [!warning] Reglas duras del auto-fix
> - **Snapshot antes de modificar**: cualquier cambio en un workflow se precede de `n8n_workflow_versions` (crear backup) o snapshot manual del JSON.
> - **Test antes de cerrar**: nada se marca como resuelto sin una verificación de éxito.
> - **Rollback si falla la verificación**: si el fix rompe algo más, restaurar el snapshot inmediatamente y escalar.
> - **Cap de cambios por workflow por día**: máximo 3 modificaciones automáticas en 24h por workflow. Si llega a 3, escalar.
> - **Log exhaustivo**: cada acción queda en `mantenimiento.log` con timestamp + diff.

---

## Notificación por email

### Cuándo se envía
- Al final de cada ciclo, **si y solo si** hay al menos 1 incidente que no se pudo resolver.
- Máximo **1 mail por cliente por hora** (evita spam si un cliente está roto en cascada).

### Destino
- **A**: alias del equipo (a definir — ej. `soporte@innovasolutions...`)
- **De**: cuenta SMTP de Innova (Gmail App Password recomendado para arrancar)

### Estructura del mail

```
Asunto: [Innova maintenance] {cliente} — {N} incidente(s) no resuelto(s)

Resumen:
  Cliente: {cliente}
  Workflows afectados: {N}
  Ciclo: {timestamp}

Por cada incidente:
  - Workflow: {nombre del workflow}
  - Nodo: {nodo que falló}
  - Error: {mensaje crudo}
  - Categoría detectada: {auth | api-change | quota | desconocido}
  - Intentos automáticos: {qué se probó}
  - Sugerencia: {plan que dio Claude pero que requiere intervención humana}
  - Link directo: https://n8n.cliente.com/workflow/{id}

— Innova maintenance agent (automatizado)
```

---

## Stack técnico

| Componente | Tecnología | Costo |
| --- | --- | --- |
| Watcher | Python + systemd user service | Gratis |
| Conexión a n8n | API REST de n8n (per-cliente con API key) | Gratis |
| Diagnóstico | `claude -p` (modo headless) | Suscripción |
| Auto-fix de workflows | API REST de n8n (`workflows`, `executions`, `credentials`) | Gratis |
| Envío de mail | SMTP (Gmail App Password o Mailgun) | Gratis hasta tier |
| Logs | Archivos planos en `~/Innova/logs/` | Gratis |
| Snapshots | Tabla de versiones de workflow en cada n8n | Nativo de n8n |

---

## Reglas duras del agente

> [!warning] No negociables
> - **Snapshot antes de cualquier modificación**. Sin excepción.
> - **Verificación antes de cerrar**. Sin excepción.
> - **Rollback automático si la verificación falla**. Sin excepción.
> - **Cap de 3 modificaciones por workflow por día**. Llegado el cap, escala.
> - **No tocar credenciales manualmente cargadas por el cliente**. Solo refresh de OAuth/API automatizables.
> - **Idioma del mail**: español argentino, voseo, tono técnico pero claro.
> - **Si hay duda, escalar**. Es preferible mandar un mail de más que romper un workflow productivo.

---

## Implementación pendiente

> [!todo] A definir antes de implementar
> - [ ] **Alias del equipo** para recibir los mails
> - [ ] **Cuenta SMTP** de Innova + App Password (Gmail) o servicio equivalente
> - [ ] **Registro inicial** de clientes con n8n productivo (URL + API key por cliente)
> - [ ] Definir si el log de incidentes resueltos se versiona en Obsidian (ej. `clientes/<cliente>/incidentes-<mes>.md`) o solo en logfile local
> - [ ] Política de retención de logs y snapshots

## Próximos pasos

1. Resolver los pendientes de arriba.
2. Implementar `~/Innova/bin/mantenimiento.py` siguiendo el patrón del watcher de pre-contratación.
3. Crear unit systemd `innova-mantenimiento.service`.
4. Probar contra un cliente piloto (ej. cuando Blincer arranque Fase 3).
5. Iterar.

---

## Riesgos operativos

- **Falsos positivos**: una ejecución fallida puede ser intencional (cliente cortó manualmente). Mitigación: el "Workflow desactivado por error" sólo se reactiva si los últimos N runs eran OK y no hubo intervención humana cercana.
- **Cascada de fallos**: si una API externa cae, todos los clientes que la usan generan errores simultáneos. Mitigación: deduplicación por categoría + cap de 1 mail/hora/cliente.
- **Auto-fix que rompe más cosas**: por eso el guardrail de snapshot + verificación + rollback.
- **API key del cliente compromete acceso total**: el subagente tiene poder de modificar workflows. Almacenar API keys con cuidado (en `.env` gitignored, idealmente en un vault tipo Bitwarden Secrets para producción).
- **Costo de Claude por incidente**: cada diagnóstico cuesta ~$0.05-0.15. Si hay 100 errores por ciclo, son ~$10 por ciclo. Monitorear y poner cap si hace falta.
