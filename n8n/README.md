---
tags:
  - n8n
  - hub
  - knowledge-base
  - nivel-1
updated: 2026-05-28
---

# n8n Knowledge Base

> Spec-driven knowledge base for building n8n workflows **almost autonomously**.
> Every flow is captured as a Spec Kit bundle (spec → research → plan → tasks → retro) and lives under its client folder so future flows can reuse prior work.

← Back to [[INNOVA_MASTER]]

---

## How it works

1. **Requirement comes in.** I take the user's request and turn it into a `spec.md` under the client folder.
2. **Spec Kit cycle runs.** `research.md` (what already exists / external docs / similar flows), then `plan.md` (architecture decisions), then `tasks.md` (build checklist).
3. **Build.** Workflow gets created in n8n (via MCP), validated, tested.
4. **Retro.** `retro.md` captures what was non-obvious, what to reuse, what hit a wall. New reusable bits are promoted to [[n8n/patterns/_index|patterns]] or [[n8n/nodes/_index|node notes]].
5. **Next flow.** Before designing, `research.md` MUST check existing client flows + the patterns library for similar work and reuse the closest match as a base.

Full process: [[n8n/METHODOLOGY|METHODOLOGY]]

---

## Layout

```
n8n/
├── README.md                  ← this file
├── METHODOLOGY.md             ← spec-kit workflow for n8n flows
├── lessons-learned.md         ← cross-flow learnings index
├── templates/
│   ├── spec.md
│   ├── research.md
│   ├── plan.md
│   ├── tasks.md
│   └── retro.md
├── clients/
│   ├── _index.md              ← clients hub
│   └── <client-slug>/
│       ├── README.md          ← client context (stack, credentials policy, scope)
│       └── flows/
│           └── <flow-slug>/
│               ├── spec.md
│               ├── research.md
│               ├── plan.md
│               ├── tasks.md
│               ├── retro.md
│               └── workflow.json   ← exported n8n workflow
├── patterns/                  ← reusable building blocks
│   └── _index.md
└── nodes/                     ← node-specific quirks & gotchas
    └── _index.md
```

---

## Entry points

| I want to… | Go to |
| --- | --- |
| Start a new flow | [[n8n/METHODOLOGY|METHODOLOGY]] → copy `templates/` into the client/flow folder |
| Find a similar past flow | [[n8n/clients/_index|Clients]] or [[n8n/patterns/_index|Patterns]] |
| Recall a node gotcha | [[n8n/nodes/_index|Nodes]] |
| Read accumulated learnings | [[n8n/lessons-learned|Lessons Learned]] |
| Understand the agent that runs this | `agentes/subagente-n8n-flow-builder.md` |

---

## Conventions

- **Language:** every file in `n8n/` is written in English.
- **Slugs:** kebab-case for client and flow folders (`blincer/`, `bank-reconciliation/`).
- **Frontmatter:** every spec/plan/etc has `client`, `flow`, `status`, `updated`.
- **n8n JSON:** the source of truth lives in n8n; the `workflow.json` in the folder is a snapshot exported after each significant change.
- **Reuse first:** never design from scratch when a prior flow or pattern covers ≥60% of the case.
