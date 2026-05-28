---
tags:
  - n8n
  - template
  - spec
  - nivel-3
client: <client-slug>
flow: <flow-slug>
status: draft   # draft | planning | building | live | paused
updated: YYYY-MM-DD
---

# Spec — <Flow Title>

> One-sentence summary of the outcome this flow produces.

← Volver a [[n8n/METHODOLOGY|Methodology]]

---

## Goal

What changes in the world when this flow runs successfully?

## Trigger

- Type: webhook | cron | manual | queue | n8n trigger node
- Schedule / endpoint / event:
- Expected frequency:

## Inputs

| Field | Type | Source | Required | Notes |
| --- | --- | --- | --- | --- |
|  |  |  |  |  |

Sample payload:

```json
{}
```

## Outputs / Side effects

- Writes to: <system>.<resource> — what fields, what semantics (upsert? append?)
- Notifies: <channel> — message shape
- Other state changes:

## Success criteria

Observable, testable conditions:

- [ ] …
- [ ] …

## Out of scope

- …

## Open questions

- [ ] …  *(must be resolved before planning)*

## Stakeholders

| Role | Person |
| --- | --- |
| Requester |  |
| Approver |  |
| End user |  |
