---
tags:
  - n8n
  - template
  - tasks
  - nivel-3
client: <client-slug>
flow: <flow-slug>
updated: YYYY-MM-DD
---

# Tasks — <Flow Title>

← Volver a [[n8n/METHODOLOGY|Methodology]]

> Ordered, verifiable. Update as work progresses; do not delete completed items.

---

## Setup

- [ ] Create / verify n8n credentials listed in `plan.md`
- [ ] Set env vars: …
- [ ] Prepare test payload(s) and place in `./test-payloads/` if reused
- [ ] Confirm sandbox vs prod target

## Build

- [ ] Node 1: <name> — params per plan
- [ ] Node 2: <name> — params per plan
- [ ] …
- [ ] Wire error branches per plan
- [ ] Run `mcp__n8n__validate_node` on each node
- [ ] Run `mcp__n8n__n8n_validate_workflow` on the full workflow

## Validate

- [ ] Execute with test payload — success path
- [ ] Execute with malformed payload — fails into error branch
- [ ] Assert each success criterion from `spec.md`
- [ ] Confirm idempotency: re-run with same input does not duplicate output

## Harden

- [ ] Retries configured on flaky nodes
- [ ] Alert wired to <channel>
- [ ] Rate-limit / concurrency settings reviewed

## Document

- [ ] Export workflow as `workflow.json`
- [ ] Update `spec.md` status → `live`
- [ ] Add row to client `README.md` flows table

## Handoff

- [ ] Notify requester with execution URL + how to monitor
- [ ] Write `retro.md`
- [ ] Promote reusable bits to `n8n/patterns/` or `n8n/nodes/`
- [ ] Append one-liner to `n8n/lessons-learned.md`
