---
tags:
  - n8n
  - tasks
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: m0-infra-setup
updated: 2026-06-10
status: blocked-by-oqs
---

# Tasks — M0 · Infrastructure base

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/flows/m0-infra-setup/plan|Plan]]

> Ordered, verifiable. **No empezar Build hasta tener acceso al VPS + Workspace y resueltas las OQs de M0.**

---

## Pre-build (resolución de OQs y prerequisitos del cliente)

- [ ] Acceso SSH/root al VPS + specs + OS (def transversal 🚦).
- [ ] Google Workspace: service account o OAuth para Drive (def transversal 🚦).
- [ ] Confirmar volumen ~10-15 préstamos/mes (def #8).
- [ ] OQ-M0-1: elegir vault de secrets (Doppler vs Bitwarden).
- [ ] OQ-M0-2: DB Supabase managed vs Postgres en VPS.
- [ ] OQ-M0-3 / 1.3: dominio/subdominio para n8n + acceso al DNS.

## Setup

- [ ] Hardening del VPS: usuario de servicio, SSH keys, firewall (ufw), updates.
- [ ] Instalar Node + n8n; configurar pm2 (`pm2 startup` + `pm2 save`).
- [ ] Reverse proxy (Caddy/Nginx) con HTTPS sobre el dominio del cliente.
- [ ] Provisionar DB (Supabase managed o Postgres local según OQ-M0-2).
- [ ] Configurar vault de secrets (proyecto + service token).

## Build

- [ ] Crear schema versionado con las 4 tablas:
  - [ ] `prestamos` (id, estado, monto, ltv, tipo, borrower, partner_match_json, created_at, …)
  - [ ] `capital_partners` (id, nombre, guidelines_json, activo)
  - [ ] `documentos_requeridos` (id, loan_type, slot, requerido)
  - [ ] `estado_checklist` (loan_id, slot, status, file_ref, updated_at)
- [ ] Seed mínimo de `capital_partners` (datos dummy hasta tener guidelines reales, def #1).
- [ ] Crear credencial Drive `gptlandings-drive` y `gptlandings-db` en n8n.
- [ ] Crear el **base Error Workflow** (`errorTrigger` → alerta canal interno) y setearlo como default de la instancia.
- [ ] Configurar repos dev/prod + cron de backup de DB.

## Validate

- [ ] n8n responde por HTTPS en el dominio y sobrevive reboot (`pm2 resurrect`).
- [ ] Las 4 tablas existen + seed de `capital_partners`.
- [ ] Credencial Drive de prueba referenciable desde un workflow dummy.
- [ ] Restore dry-run de un backup de DB → OK.
- [ ] Error Workflow dispara una alerta de prueba al canal interno.

## Harden

- [ ] Backups programados + retención definida.
- [ ] Monitoreo básico del VPS (disco/CPU/uptime de n8n).
- [ ] Rotación de logs de pm2/n8n.

## Document

- [ ] Documentar accesos y arquitectura en el README del workspace.
- [ ] Actualizar `spec.md` status → `live` al cerrar.
- [ ] Anotar dominio + IDs de proyecto reales acá.

## Handoff

- [ ] Confirmar a Innova que A–G pueden construirse encima.
- [ ] Escribir `retro.md`.
- [ ] Promover a `n8n/patterns/` cualquier bit reusable de setup (ej. `client-vps-n8n-bootstrap`).
