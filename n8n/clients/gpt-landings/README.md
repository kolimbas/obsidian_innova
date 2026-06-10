---
tags:
  - n8n
  - cliente
  - nivel-3
client: gpt-landings
status: pre-sales
updated: 2026-06-10
---

# GPT Landings — n8n Workspace

> Private lender — first-position mortgages for investors, South Florida. Closing the contract (USD 5.000 per phase × 4 phases). Business context lives in `clientes/gpt-landings.md` (Spanish).

← [[n8n/clients/_index|Clients]]

---

## Scope — Phase 1 "Loan processing & origination"

Automate a loan from intake to the moment processing starts, removing the repetitive work (matching, term sheets, doc tracking, folder setup). **Underwriting judgment stays human.** Seven modules (M0 + A–G); see `clientes/gpt-landings.md` for the 4-phase roadmap and pricing model.

> [!warning] Module F — conditional carve-out
> **F (`track-record-validation`) is OUT of Phase 1's fixed price.** It enters only as a **conditional add-on**, gated on whether **Elementix exposes an API** (OQ-F-1 / discovery def #6). The PDF itself flags F as the riskiest block — validate technical feasibility before committing scope or price.

**Canonical stack (from the Fase-1 PDF, to confirm in scoping):** dedicated **client-owned VPS** · n8n self-hosted (pm2 + HTTPS/domain) · **Supabase/Postgres** for state & parameters · **Claude API** for extraction/reasoning · **Google Drive API** · e-sign platform (DocuSign vs PandaDoc — TBD) · WhatsApp Business API (provider TBD).

> [!note] New, separate n8n instance
> This project runs on the **client's own VPS** — a brand-new n8n instance, independent from Blincer's (`n8n.srv1512692.hstgr.cloud`). **No Blincer credentials are reused — only patterns and structure.**

## Stack

| System | Use | Integration risk |
| --- | --- | --- |
| **VPS (client-owned)** | Host for n8n + supporting services | Medium — depends on access + specs (M0) |
| **n8n self-hosted (pm2 + HTTPS)** | Orchestration | Low — known setup |
| **Supabase / Postgres** | State: `prestamos`, `capital_partners`, `documentos_requeridos`, `estado_checklist` | Low/Medium — managed vs VPS-hosted TBD |
| **Claude API** | Borderline matching (A), OCR/extraction (F) | Medium — budget + model choice |
| **Google Drive API** | Loan folders + document storage (D, G) | Low — service account / OAuth |
| **e-sign (DocuSign / PandaDoc)** | Term-sheet signing (C) | Medium — vendor + API access TBD |
| **WhatsApp Business API** | Document-checklist reminders to the borrower (E) | **Medium — HSM template approval has Meta lead time** |
| **Elementix / public records** | Track-record cross-check (F) | **High — API vs UI-only unknown (carve-out)** |

## Credentials policy

- All credentials live in n8n's credential store, never in node parameters or env vars in plaintext.
- Secrets vault (non-n8n secrets): TBD — Doppler vs Bitwarden Secrets — decide in M0.
- **No reuse of Blincer credentials** (separate instance). Naming convention proposal: `<client>-<system>-<env>` → e.g. `gptlandings-hubspot`, `gptlandings-whatsapp`, `gptlandings-drive`.

## Flows (Phase 1 bundles)

| Flow | Status | Trigger | Last update |
| --- | --- | --- | --- |
| [[n8n/clients/gpt-landings/flows/m0-infra-setup/spec\|M0 · Infrastructure base]] | draft · blocked-by-oqs | n/a (setup) | 2026-06-10 |
| [[n8n/clients/gpt-landings/flows/partner-matching-engine/spec\|A · Capital-partner matching engine]] | draft · blocked-by-oqs | Webhook/form (loan intake) | 2026-06-10 |
| [[n8n/clients/gpt-landings/flows/term-sheet-generation/spec\|B · Term-sheet generation]] | draft · blocked-by-oqs | Called from A (qualifying JSON) | 2026-06-10 |
| [[n8n/clients/gpt-landings/flows/approval-and-esign/spec\|C · Approval & e-sign]] | draft · blocked-by-oqs | Manager approval → e-sign API + "signed" webhook | 2026-06-10 |
| [[n8n/clients/gpt-landings/flows/document-intake-form/spec\|D · Document intake (backend)]] | draft · blocked-by-oqs | Webhook ← web form | 2026-06-10 |
| [[n8n/clients/gpt-landings/flows/whatsapp-doc-reminder/spec\|E · WhatsApp document reminders]] | draft · blocked-by-oqs | Daily cron | 2026-06-10 |
| [[n8n/clients/gpt-landings/flows/track-record-validation/spec\|F · Track-record validation]] | **CARVE-OUT (outside Phase-1 price)** · blocked-by-oqs | Per incoming document | 2026-06-10 |
| [[n8n/clients/gpt-landings/flows/drive-folder-structure/spec\|G · Drive folder structure]] | draft · blocked-by-oqs | On loan → `processing` | 2026-06-10 |

> [!note] Status 2026-06-10 — Spec Kit bundles, pre-build
> All 8 bundles are documented (spec/research/plan/tasks/retro), `status: draft`, plans `blocked-by-oqs`. **Nothing is built yet** — pre-contract, no VPS access, and the 8 scoping definitions are unanswered (see discovery). No `workflow.json` exists. Build order once unblocked: M0 → A+B → C → D → G → E (in parallel once the form exists) → F last (validate feasibility first).

## Module D — web deliverable note

Module D is a **double deliverable**: the **front-end intake form** (Next.js/React — Innova's web line: Lovable / Claude Code / Supabase / Vercel) lives documented in `clientes/gpt-landings.md` § "Entregable web". The bundle [[n8n/clients/gpt-landings/flows/document-intake-form/spec|document-intake-form]] here covers **only the n8n backend** (ingest webhook, Drive/Supabase persistence, per-item checklist state, triggering E and F).

## Open dependencies (the 8 scoping definitions + transversales)

- [ ] **VPS access** (SSH/root, specs, OS) — blocks M0 🚦
- [ ] **Google Workspace** of the client (service account / OAuth for Drive) 🚦
- [ ] Confirm expected **volume** (~10-15 loans/month) to size infra ⚠️ (def #8)
- [ ] Internal alert channel / `processing@` (email vs Slack vs WhatsApp)
- [ ] Format + count of **capital-partner guidelines** (A) 🚦 (def #1)
- [ ] **Term-sheet template** + exact fields + real example (B) 🚦 (def #2)
- [ ] **e-sign**: DocuSign vs PandaDoc + account/API (C) 🚦 (def #3)
- [ ] Exact **document list per loan type** (D) 🚦 (def #4)
- [ ] **WhatsApp Business API** provider + Meta template approval — start NOW (lead time) (E) 🚦 (def #5)
- [ ] **Elementix: API or UI-only?** + public-records sources (F) 🚦🚦 (def #6 — most important)
- [ ] Who approves term sheets and on what channel (C) ⚠️ (def #7)

## Links

- [[n8n/clients/gpt-landings/discovery-2026-06-10|Discovery questionnaire — Phase 1 (2026-06-10)]] (to send to GPT Landings manager / IT)
- Business note: `clientes/gpt-landings.md`
- Methodology: [[n8n/METHODOLOGY|Spec Kit]]
- Source proposal: `~/Descargas/desglose-tareas-fase1.pdf`
