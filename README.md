# LLM-Wiki-Methodik

**Methodik-Handbuch für selbst-aktualisierende, LLM-gepflegte Wissensbasen.**

Dieses Repo dokumentiert eine erprobte Methodik, um eine kuratierte Markdown-Wissensbasis
(Obsidian-kompatibel, git-versioniert) über ein sich schnell änderndes Produkt-Ökosystem
aufzubauen und **automatisch aktuell zu halten** — mit LLM-Unterstützung nur dort, wo
Sprachverständnis nötig ist, und deterministischem Scripting überall sonst.

> **Herkunft & Abgrenzung:** Abstrahiert aus einem realen Projekt (eine LLM-gepflegte
> Wissensbasis über ein grosses Vendor-AI-Ökosystem mit ~50 Entitäten). Dieses Repo
> enthält **ausschliesslich die verallgemeinerte Methodik** — keine Inhalte, keine
> internen Daten, keine Kunden- oder Firmenbezüge des Ursprungsprojekts.

---

## Das Problem

Produkt-Ökosysteme mit hoher Änderungsdynamik (Cloud-AI-Plattformen, Framework-Landschaften,
Compliance-Regelwerke) haben eine Wissens-Halbwertszeit von unter 12 Monaten. Klassische
Wikis veralten schneller, als Menschen sie pflegen. Reine LLM-Recherche on demand
halluziniert und kennt die eigene Bewertungslogik nicht.

**Die Antwort:** ein Dual-Layer-System.

```
                    ┌──────────────────────────────────────┐
                    │  AUTOMATISIERUNGSSCHICHT             │
                    │                                      │
[RSS / Blog / Docs]─►  fetch          (Feeds einlesen)     │
                    │      ↓                               │
                    │  classify       (Clippings scoren)   │
                    │      ↓                               │
                    │  filter         (Keyword-Match, 0 LLM)│
                    │      ↓                               │
                    │  patch          (LLM Tool-Use, Delta) │
                    │      ↓                               │
                    │  apply          (Markdown-Mutation)  │
                    │      ↓                               │
                    │  update-indices (MOC-Tabellen)       │
                    │      ↓                               │
                    │  audit          (Template-Gate)      │
                    │      ↓                               │
                    │  CI → Pull Request                   │
                    │      ↓                               │
                    │  [Mensch reviewt & merged]           │
                    └────────────┬─────────────────────────┘
                                 │  git = Single Source of Truth
                                 ▼
                    ┌──────────────────────────────────────┐
                    │  KONSUMSCHICHT (read-mostly Mirrors) │
                    │                                      │
                    │  Dokument-Store-Mirror                │
                    │      → Chat-Agent (RAG, kuratiert)   │
                    │                                      │
                    │  Static-Site-Build                    │
                    │      → Katalog / Vergleich / Graph   │
                    └──────────────────────────────────────┘
```

**Kernprinzip:** Das LLM macht nur, was Regeln nicht können (Sprache verstehen,
Sektion wählen, Changelog formulieren). Alles andere ist deterministisches Scripting —
reproduzierbar, testbar, tokenfrei.

---

## Inhalt

| Dokument | Thema |
|---|---|
| [01-wissensarchitektur.md](01-wissensarchitektur.md) | Note-Anatomie, Tier/Watch-System, MOC-Theorie, Schreibprinzipien |
| [02-update-pipeline.md](02-update-pipeline.md) | Die 7 Pipeline-Stages, Token-Ökonomie, State-Management, Idempotenz |
| [03-governance.md](03-governance.md) | Git als Wahrheit, PR-Gate, Audits, Secrets-Hygiene, Anti-Patterns |
| [04-deep-research.md](04-deep-research.md) | Tiered-Research-Prompts, Drei-Pfade-SOP, Quota-Ökonomie, Prompt-Verbesserungs-Loop |
| [05-konsumschicht.md](05-konsumschicht.md) | Dokument-Store-Mirror, deklarativer Chat-Agent, Static-Site-Patterns |
| [06-adaption.md](06-adaption.md) | Eignungs-Kriterien, Implementierungs-Reihenfolge von null, Validierungs-Fragen |

**Templates** (generisch, direkt instanziierbar):

| Template | Zweck |
|---|---|
| [templates/entity-note-template.md](templates/entity-note-template.md) | Vollständige Entitäts-Note (7 Pflicht- + 3 optionale Sektionen) |
| [templates/moc-template.md](templates/moc-template.md) | Map-of-Content-Note (Themen-Hub) |
| [templates/registry.schema.yaml](templates/registry.schema.yaml) | Entitäten-Registry (Single Source of Truth) |
| [templates/sources.schema.yaml](templates/sources.schema.yaml) | Feed-Quellen-Registry |

---

## Für wen ist das?

Die Methodik funktioniert für jede Domäne mit:

- **20–150 abgrenzbaren Entitäten** (Produkte, Frameworks, Konzepte, Regularien)
- **klarer Relevanz-Hierarchie** (Must-Know / Important / Awareness)
- **regelmässigen externen Update-Quellen** (RSS, Blogs, offizielle Docs, GitHub-Releases)
- **Bedarf nach kontinuierlicher Pflege** (Halbwertszeit < 12 Monate)

Details und ein Von-null-Fahrplan: [06-adaption.md](06-adaption.md).
