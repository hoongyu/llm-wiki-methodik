**English** | [Deutsch](README.de.md)

# LLM-Wiki-Methodik

**A methods handbook for self-updating, LLM-maintained knowledge bases.** The handbook chapters are written in German.

This repository documents a tested method for building a curated Markdown knowledge base (Obsidian-compatible, versioned in git) about a fast-changing product ecosystem, and **keeping it up to date automatically**: LLM help only where language understanding is needed, deterministic scripting everywhere else.

> **Origin and scope:** abstracted from a real project (an LLM-maintained knowledge base about a large vendor AI
> ecosystem with about 50 entities). This repository contains **only the generalised method**: no content, no
> internal data, and no references to customers or companies from the original project.

---

## The problem

Product ecosystems that change quickly (cloud AI platforms, framework landscapes, compliance rulebooks) have a knowledge half-life of under 12 months. Classic wikis go stale faster than people can maintain them. On-demand LLM research hallucinates and does not know your own evaluation logic.

**The answer:** a dual-layer system.

```
                    ┌──────────────────────────────────────┐
                    │  AUTOMATION LAYER                    │
                    │                                      │
[RSS / blog / docs]─►  fetch          (read feeds)         │
                    │      ↓                               │
                    │  classify       (score clippings)    │
                    │      ↓                               │
                    │  filter         (keyword match, 0 LLM)│
                    │      ↓                               │
                    │  patch          (LLM tool use, delta) │
                    │      ↓                               │
                    │  apply          (Markdown mutation)  │
                    │      ↓                               │
                    │  update-indices (MOC tables)         │
                    │      ↓                               │
                    │  audit          (template gate)      │
                    │      ↓                               │
                    │  CI → pull request                   │
                    │      ↓                               │
                    │  [a human reviews and merges]        │
                    └────────────┬─────────────────────────┘
                                 │  git = single source of truth
                                 ▼
                    ┌──────────────────────────────────────┐
                    │  CONSUMPTION LAYER (read-mostly mirrors)│
                    │                                      │
                    │  document-store mirror               │
                    │      → chat agent (RAG, curated)     │
                    │                                      │
                    │  static site build                   │
                    │      → catalogue / comparison / graph│
                    └──────────────────────────────────────┘
```

**Core principle:** the LLM only does what rules cannot (understand language, choose a section, write the changelog). Everything else is deterministic scripting: reproducible, testable, and free of tokens.

---

## Contents

| Document | Topic |
|---|---|
| [01-wissensarchitektur.md](01-wissensarchitektur.md) | Knowledge architecture: note anatomy, tier/watch system, MOC theory, writing principles |
| [02-update-pipeline.md](02-update-pipeline.md) | The 7 pipeline stages, token economics, state management, idempotency |
| [03-governance.md](03-governance.md) | Git as the truth, PR gate, audits, secrets hygiene, anti-patterns |
| [04-deep-research.md](04-deep-research.md) | Tiered research prompts, three-path SOP, quota economics, prompt improvement loop |
| [05-konsumschicht.md](05-konsumschicht.md) | Consumption layer: document-store mirror, declarative chat agent, static site patterns |
| [06-adaption.md](06-adaption.md) | Fit criteria, implementation order from zero, validation questions |

**Templates** (generic, ready to instantiate):

| Template | Purpose |
|---|---|
| [templates/entity-note-template.md](templates/entity-note-template.md) | Full entity note (7 required + 3 optional sections) |
| [templates/moc-template.md](templates/moc-template.md) | Map-of-content note (topic hub) |
| [templates/registry.schema.yaml](templates/registry.schema.yaml) | Entity registry (single source of truth) |
| [templates/sources.schema.yaml](templates/sources.schema.yaml) | Feed source registry |

---

## Who is it for?

The method works for any domain with:

- **20 to 150 distinct entities** (products, frameworks, concepts, regulations)
- **a clear relevance hierarchy** (must-know / important / awareness)
- **regular external update sources** (RSS, blogs, official docs, GitHub releases)
- **a need for continuous maintenance** (half-life under 12 months)

Details and a roadmap from zero: [06-adaption.md](06-adaption.md).
