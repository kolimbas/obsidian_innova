---
tags:
  - n8n
  - methodology
  - spec-kit
  - nivel-2
updated: 2026-05-28
---

# Methodology — Spec Kit for n8n Flows

> Hard rule: **no node is created in n8n until `spec.md`, `research.md`, `plan.md` and `tasks.md` exist and the plan has been confirmed.** This keeps reruns cheap and outputs reusable.

← Back to [[n8n/README|n8n KB]]

---

## The five artifacts

| File | Question it answers | When it's written |
| --- | --- | --- |
| `spec.md` | **What does the user need?** Inputs, outputs, triggers, success criteria, out-of-scope. | First. From the raw request. |
| `research.md` | **What do we already know?** Similar past flows, reusable patterns, node docs, external API constraints. | Right after the spec. |
| `plan.md` | **How will we build it?** Node-level architecture, error handling, idempotency, credentials, env vars. | After research, before any build. |
| `tasks.md` | **What are the concrete steps?** Ordered checklist with validation points. | Together with the plan. |
| `retro.md` | **What did we learn?** Surprises, reusable bits to promote, gotchas, time spent. | After the flow is live. |

The templates live in `n8n/templates/`: [[n8n/templates/spec|spec]] · [[n8n/templates/research|research]] · [[n8n/templates/plan|plan]] · [[n8n/templates/tasks|tasks]] · [[n8n/templates/retro|retro]].

---

## Step-by-step

### 0. Intake

User gives a requirement (could be a paragraph, a meeting note, a Loom). Identify:

- **Client** → does the client folder exist under `n8n/clients/<slug>/`? If not, create it with `README.md` (stack, credentials policy, scope).
- **Flow slug** → kebab-case, descriptive: `bank-reconciliation`, `whatsapp-inbound-router`, `stock-alert`.
- Create `n8n/clients/<client>/flows/<flow>/` and copy the five templates in.

### 1. Spec (`spec.md`)

Translate the requirement into:

- **Goal** — one sentence outcome.
- **Trigger** — webhook, cron, manual, queue, etc.
- **Inputs** — payload shape, source systems.
- **Outputs / side effects** — where data lands, what gets notified.
- **Success criteria** — observable conditions ("a row exists in X with…", "the user receives…").
- **Out of scope** — explicit exclusions.
- **Open questions** — to confirm with user before planning.

Stop here and confirm with the user if any open question is load-bearing.

### 2. Research (`research.md`)

This is the **reuse step**. Always run, in this order:

1. **Past flows of the same client** — read every `spec.md` and `retro.md` under the client folder.
2. **Cross-client patterns** — scan `n8n/patterns/` and other clients' flow folders for similar shapes (same trigger type, same target system, same data flow).
3. **n8n node docs / templates** — use the n8n MCP:
   - `mcp__n8n__search_nodes` to find candidate nodes
   - `mcp__n8n__get_node` for properties
   - `mcp__n8n__search_templates` for community workflows
4. **External system docs** — auth model, rate limits, idempotency keys, webhook semantics.

Record findings as: **What we will reuse** + **Why each external decision was made**. Cite file paths and links.

If a prior flow covers ≥60% of the case, declare it as the **base flow** and the plan starts from a copy.

### 3. Plan (`plan.md`)

Node-level architecture. For each node list: purpose, key parameters, error-handling branch. Decide upfront:

- **Idempotency strategy** — dedup keys, upserts, lookups before write.
- **Error handling** — retry policy, dead-letter, alert channel.
- **Credentials & env** — which credential, which secret store, never inline.
- **Observability** — what gets logged where; how a failed run is detected.
- **Testing approach** — test payload(s), sandbox vs prod environment, rollback plan.

End with a tiny mermaid diagram of the flow.

### 4. Tasks (`tasks.md`)

Ordered checklist. Each task is verifiable. Group by phase:

- Setup (credentials, env vars, test data)
- Build (node by node, in execution order)
- Validate (run with test payload, assert outputs)
- Harden (error branches, retries, alerts)
- Document (export `workflow.json`, link from client README)
- Handoff (notify user, retro)

### 5. Build

Use the n8n MCP tools:

- `mcp__n8n__n8n_generate_workflow` for first-cut scaffold from the plan
- `mcp__n8n__n8n_create_workflow` / `n8n_update_partial_workflow` for surgical edits
- `mcp__n8n__validate_node` and `mcp__n8n__n8n_validate_workflow` before activating
- `mcp__n8n__n8n_test_workflow` with the spec's test payload
- Export JSON → save as `workflow.json` in the flow folder

Tick tasks as they're done. Any deviation from the plan → update `plan.md` (don't let docs and reality drift).

### 6. Retro (`retro.md`)

Mandatory before closing. Sections:

- **What worked / didn't**
- **Reusable bits** — promote to `n8n/patterns/<name>.md` if generic, or note in client README if client-specific
- **Node gotchas** — promote to `n8n/nodes/<node>.md`
- **Time spent** vs estimate
- **Open follow-ups**

Add a one-liner to `n8n/lessons-learned.md`.

---

## Decision: when to skip steps

**Never skip spec, plan, tasks.** They are the contract.

`research.md` can be a 5-line "checked X, Y, Z, nothing relevant" — but the search must still happen. The whole point of this KB is the second flow is cheaper than the first.

`retro.md` can be 3 bullets if the flow was trivial — but it must exist, otherwise patterns never accumulate.

---

## Anti-patterns

- ❌ Designing a flow before reading prior client flows.
- ❌ Hardcoding credentials or URLs in the spec/plan; those belong in n8n credentials + env vars.
- ❌ Skipping idempotency "because it'll be fine"; every external write needs a dedup story.
- ❌ Closing the flow without a retro — that's how reuse rots.
- ❌ Letting `plan.md` lie about what's built. If the build diverged, the doc gets fixed.
