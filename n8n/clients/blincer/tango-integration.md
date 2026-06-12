---
tags:
  - n8n
  - cliente
  - blincer
  - tango
  - sql-server
  - nivel-3
client: blincer
updated: 2026-06-12
---

# Blincer — Integración Tango (SQL Server read-only)

← [[n8n/clients/blincer/README|Blincer n8n]] · Negocio: `clientes/blincer.md` · 🔐 Credenciales: `accesos.md`

> [!success] Open question resuelta (2026-06-12): Tango es LOCAL, pero con acceso directo a SQL
> Se temía "Tango local sin API → CSV/RPA (frágil)". La realidad es **mejor**: corre en un servidor on-prem
> sobre **SQL Server**, así que integramos por **consultas SQL de solo-lectura** — directo, robusto, sin RPA.

---

## Arquitectura

```
n8n (VPS Hostinger)  ──túnel saliente──►  PC servidor de Tango (LAN del cliente)
   nodo Microsoft SQL        Tailscale            SQL Server 2019 :1433
        │                                              │
        └── lee 2 VISTAS read-only ◄───────────────────┘
            (usuario blincer_n8n_ro, solo SELECT a las vistas)
```

- n8n vive en el VPS; Tango está en una PC de la LAN del cliente **sin IP pública** → hace falta un **túnel saliente** (la PC llama al VPS, no se abre ningún puerto a internet).
- **Nunca se escribe en Tango** (rompe soporte). Solo SELECT, y en modo `READ UNCOMMITTED` para no bloquear producción.

## Servidor y base (detalle en `accesos.md`)

| Dato | Valor |
| --- | --- |
| Host | BLI-SVRV-TANGO (Windows Server 2019) · IP LAN 192.168.44.121 |
| SQL | SQL Server 2019 Express, **modo mixto**, TCP, puerto **1433** |
| Instancia productiva | **default** (`MSSQLSERVER`) = Tango **Gestión** |
| Base productiva | **`BLINCER_SRL`** (19 GB) |
| Descartadas | `AXSQLEXPRESS` (Astor + Sueldos, casi sin uso); bases `*_PRUEBA/*_2018/*_EJEMPLO` (backups) |

## Acceso de solo-lectura (creado 2026-06-12)

- Login SQL **`blincer_n8n_ro`** (password en `accesos.md`).
- Permisos: **GRANT SELECT solo a las 2 vistas**. Gracias al *ownership chaining* (vistas y tablas son `dbo`), el usuario **no necesita ni tiene** acceso a las tablas crudas. **Verificado:** lee las vistas y le **niega** `GVA14`.

## Las 2 vistas (creadas en `BLINCER_SRL.dbo`)

> `CREATE OR ALTER` → se pueden re-ajustar sin romper nada. SQL completo en `~/tango_comandos_recon.txt` (máquina de Fran).

### `vw_blincer_clientes_cc` → para **Credit Limit**
Por cliente: `cod_cliente, razon_social, cuit, saldo_cta_cte, saldo_documentos, deuda_total, limite_credito, pct_credito_usado, excede_limite (1/0), email, whatsapp, telefono_movil, telefono_fijo, cod_vendedor`.
- Origen: tabla **`GVA14`** (maestro de clientes). Saldo de cuenta corriente = `SALDO_CC` (positivo = debe). Límite = `CUPO_CREDI`.

### `vw_blincer_facturas_pendientes` → para **Cobranzas / WhatsApp**
Por factura pendiente: `cod_cliente, razon_social, tipo_comp, nro_comp, fecha_emision, fecha_vencimiento, dias_atraso, importe_original, saldo_pendiente, estado, mon_cte, email, whatsapp, telefono_movil, cod_vendedor`.
- Origen: vista pre-existente **`FacturasPendientesClientes`** (la dejó un integrador anterior, junto con la capa `CRM_*`) + join a `GVA14` por `COD_CLIENT`.
- Filtro de la vista: `ESTADO='PEN' AND Saldo>0`. **Vencida** = `dias_atraso > 0`.

## Hallazgos de los datos (importantes para el negocio)

- **Saldo de cuenta corriente:** confiable. 81 clientes con saldo > 0. (Ej.: GAAB LOCKS LLC, Herrajes Revelli...).
- **Límite de crédito (`CUPO_CREDI`):** cargado en 2068/2463 clientes, pero la **mayoría con valores "infinito"** (10.000M / 30.000M = sin límite real). → El bloqueo igual funciona: solo frena a quien tenga un cupo **finito** y lo supere (`excede_limite=1`). Si el cliente quiere control real sobre alguien, carga un cupo finito en Tango.
- **Facturas pendientes:** base = `FacturasPendientesClientes` (228 filas; `ESTADO='PEN'`=217 reales). Ojo: incluye **residuos históricos** (facturas de 2008–2020 con saldos de centavos) → el workflow filtra por **monto mínimo** y **antigüedad**.
- **Contacto:** **email bien cargado** (a veces varios separados por `;`). **WhatsApp (`TELEFONO_MOVIL_WA`) casi vacío** → la cobranza por WhatsApp no tiene número confiable desde Tango.

## Decisiones de negocio (a confirmar con el cliente)

- **A — Canal de cobranza MVP:** **email** (recomendado; WhatsApp queda fase 2 hasta cargar números o tirar de la capa CRM).
- **B — Saldo mínimo a reclamar:** sugerido **> $5.000** (ignora centavos).
- **C — Antigüedad máxima:** ignorar vencidas de **+12 meses** (residuos viejos).

## Conexión — túnel (pendiente)

- **Opción elegida: Tailscale** (VPN privada, sin abrir puertos, se reconecta sola). Alternativa: túnel SSH reverso.
- Falta: instalar en la PC de Tango (OK del cliente + admin) y en el VPS; abrir 1433 en el Firewall de Windows para la interfaz del túnel; crear credencial **Microsoft SQL** en n8n (host = IP del túnel, 1433, `blincer_n8n_ro`, base `BLINCER_SRL`, `trustServerCertificate`).

## Estado

- [x] Acceso al servidor (TeamViewer) + base productiva identificada
- [x] 2 vistas read-only creadas y verificadas
- [x] Usuario `blincer_n8n_ro` creado y verificado (mínimo privilegio)
- [ ] Confirmar decisiones A/B/C con el cliente
- [ ] Túnel (Tailscale) PC ↔ VPS
- [ ] Credencial Microsoft SQL en n8n + reemplazar la lectura de Tango en credit-limit y whatsapp-overdue (hoy nodos `disabled`)
