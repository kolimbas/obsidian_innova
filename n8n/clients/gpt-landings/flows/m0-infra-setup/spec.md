---
tags:
  - n8n
  - spec
  - gpt-landings
  - nivel-3
client: gpt-landings
flow: m0-infra-setup
status: draft
updated: 2026-06-10
---

# Spec — M0 · Infrastructure base

> Stand up the foundational infrastructure on the client's VPS so modules A–G can be built on top.

← Volver a [[n8n/METHODOLOGY|Methodology]] · [[n8n/clients/gpt-landings/README|GPT Landings]]

> [!note] Setup module, not a flow
> M0 has no trigger/inputs/outputs in the n8n sense. The usual "Trigger / Inputs / Outputs" sections are adapted to **Prerequisites / Deliverables / Definition of Done**.

---

## Goal

Leave the GPT Landings project infrastructure operational on the client-owned VPS: a new self-hosted n8n instance (pm2 + HTTPS/domain), a Postgres/Supabase DB with the four core tables, a secrets vault, Google Drive access, and dev/prod repos with backups — so that modules A–G can be built and credentialed without re-doing plumbing.

## Trigger

`n/a — foundational setup` (no event trigger).

## Prerequisites (client-side)

| Item | Source | Required | Notes |
| --- | --- | --- | --- |
| SSH/root access to the VPS | IT/DevOps | yes | specs (CPU/RAM/disk) + OS — OQ-M0 / def transversal |
| Google Workspace access | IT/DevOps | yes | service account or OAuth for Drive (D, G) |
| Domain / subdomain for n8n | IT/DevOps | yes | for HTTPS + webhook URLs |
| Expected volume (~10-15 loans/mo) | Manager | no | sizing only (def #8) |
| Supabase project (if managed) | Innova/IT | conditional | vs Postgres on the VPS |

## Deliverables (side effects)

- n8n running at `https://<client-domain>` under pm2 with restart-on-boot (`pm2 resurrect`).
- DB with tables `prestamos`, `capital_partners`, `documentos_requeridos`, `estado_checklist` (versioned schema) + a minimal `capital_partners` seed.
- Secrets vault (Doppler or Bitwarden — TBD) holding non-n8n secrets.
- Google Drive service account / OAuth with create-folder + upload permissions.
- Dev/prod repos + scheduled DB backups.
- A base **Error Workflow** (style of Blincer's `T-000`) wired as the instance default.

## Success criteria (Definition of Done)

- [ ] n8n responds over HTTPS on the client domain and survives a reboot (`pm2 resurrect`).
- [ ] The 4 tables exist with their schema and a minimal `capital_partners` seed row.
- [ ] A test credential (Drive) is stored in the vault and referenceable from n8n.
- [ ] A DB backup is verified via a restore dry-run.
- [ ] Base Error Workflow exists and is set as the default for the instance.

## Out of scope

- Any business logic of A–G.
- The Module D front-end (web deliverable — see `clientes/gpt-landings.md`).

## Open questions

- [ ] **OQ-M0-1** — Secrets vault: Doppler vs Bitwarden Secrets. 🚦
- [ ] **OQ-M0-2** — DB: Supabase managed vs Postgres on the VPS. ⚠️
- [ ] **OQ-M0-3** — Volume to size infra (~10-15/mo?). ⚠️ *(ref def #8)*
- [ ] **OQ-M0-4** — Drive: service account vs OAuth.
- [ ] **Transversales** — VPS access 🚦 + Google Workspace 🚦 (see [[n8n/clients/gpt-landings/discovery-2026-06-10|discovery]] §0).

## Stakeholders

| Role | Person |
| --- | --- |
| Requester | Manager/owner GPT Landings |
| Approver | Manager/owner |
| End user | Innova (builds A–G on top) + IT del cliente (operación) |
