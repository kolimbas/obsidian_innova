---
tags:
  - accesos
  - credenciales
  - privado
updated: 2026-06-12
---

# 🔐 Accesos y credenciales

> [!danger] Archivo LOCAL — no se sube a GitHub
> Este archivo está en `.gitignore` a propósito: vive **solo en esta máquina**, NO se pushea al repo.
> No lo copies a otro archivo que sí se sincronice. A futuro, lo ideal es pasar todo a **Bitwarden / Doppler**.

---

## BLINCER — Tango Gestión (servidor local)

### App Tango (login de usuario)

```
TANGO GESTION
Usuario: SUPERVISOR
Contraseña: 1259
BLINCER SRL
```

> Las 4 líneas: **programa** (Tango Gestión) · **usuario** · **contraseña** · **empresa** a elegir al entrar.

### Servidor / red

| Dato | Valor |
| --- | --- |
| Host | **BLI-SVRV-TANGO** (Windows Server 2019) |
| IP LAN | **192.168.44.121** (/24, gateway 192.168.44.254) |
| Acceso remoto | TeamViewer — ID: `<completar>` · Clave: `<completar>` |
| Internet | Sí (ok para el túnel) |

### SQL Server (donde vive la base de Tango)

| Dato | Valor |
| --- | --- |
| Motor | SQL Server 2019 Express — **modo mixto** (acepta usuario/clave SQL) |
| Instancia productiva | **default** (`MSSQLSERVER`) → se conecta como `.` o `BLI-SVRV-TANGO` |
| Puerto | **1433** (fijo, TCP habilitado) |
| Base productiva | **BLINCER_SRL** (19 GB — Tango Gestión) |
| Otra instancia | `AXSQLEXPRESS` (Tango Astor + Sueldos, casi sin uso) |
| Admin local | autenticación **Windows** (cuenta del server): `sqlcmd -S . -E` |

#### Usuario SQL solo-lectura para n8n (creado 2026-06-12)

| Dato | Valor |
| --- | --- |
| Login | **`blincer_n8n_ro`** |
| Password | **`Krn7.Vxq2-Lpm9_Wtz5`** |
| Permisos | SELECT **solo** sobre `dbo.vw_blincer_clientes_cc` y `dbo.vw_blincer_facturas_pendientes` (no puede leer tablas crudas — probado) |
| Base por defecto | BLINCER_SRL |

---

## BLINCER — n8n

| Dato | Valor |
| --- | --- |
| URL | https://n8n.srv1512692.hstgr.cloud |
| API key (REST) | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIwYjg0MDk5Mi0yZmI1LTQyODAtYmNkMy02YzBiZjFiODMyMDEiLCJpc3MiOiJuOG4iLCJhdWQiOiJwdWJsaWMtYXBpIiwianRpIjoiODA2ZGZlN2YtZDU1Yy00NjgxLTg4ZjEtMWZlY2RlZDY0ZGZkIiwiaWF0IjoxNzgwOTM4OTkyfQ.nLdmTQtqCa1S2ZkDu0pSgJz6M_RljMnXPyGaUv-p2f4` (también en `~/blincer/connection.env`, chmod 600) |
| Credenciales internas | HubSpot app-token (`A3JekIL652cjutl4`), Google Sheets (`NNpCFCk3F2rhlxUk`), etc. → viven en el **credential store de n8n**, no en archivos |

---

## VPS (Hostinger) donde corre n8n

| Dato | Valor |
| --- | --- |
| Host | srv1512692.hstgr.cloud |
| Acceso | `<completar — SSH / panel Easypanel>` |

---

> Pendiente de completar: ID/clave de TeamViewer del server de Tango, y acceso al VPS.
