# 03 · Governance

Wie das System vertrauenswürdig bleibt: Git als einzige Wahrheit, menschliche
Review-Gates, maschinelle Audits, Secrets-Hygiene — und die Anti-Patterns, die man
kennen muss, bevor man sie baut.

---

## 1. Git ist die Single Source of Truth

Das Vault **ist** ein Git-Repo. Jede Änderung ist ein Commit. Alle Konsum-Layer
(Dokument-Store-Mirror, Chat-Agent, Static Site) sind **read-mostly Mirrors** —
niemand ausser dem Pipeline-Bot schreibt dorthin, und nichts fliesst von dort zurück.

Warum so strikt: Sobald ein zweiter Schreibort existiert (z.B. direkte Edits im
Dokument-Store), entsteht Drift, Änderungen gehen beim nächsten Sync verloren, und
niemand weiss mehr, welche Version stimmt.

---

## 2. PR-Gate: Automatik schreibt nie direkt auf `main`

**Automatische Änderungen kommen ausnahmslos als Pull Request** und werden manuell
reviewt und gemerged. Das ist die zentrale Absicherung gegen das Grundrisiko
LLM-gepflegter Inhalte: *plausibel, aber falsch*.

- Der tägliche Sync öffnet **einen** PR mit allen Patches des Tages.
- Der Audit-Output steht in der PR-Beschreibung — der Reviewer sieht
  Template-Verstösse ohne selbst zu prüfen.
- Die Patch-Stage eskaliert Widersprüche explizit (`needs_human_review`
  statt stillschweigendem Überschreiben).
- **Manuelle Änderungen** (selbst geschriebene Notes) werden normal committet —
  das PR-Gate gilt für die Automatik, nicht für Menschen.

### Merge-Disziplin absichern

Unreviewte Sync-PRs stapeln sich sonst unbemerkt. Gegenmittel: ein zweiter, winziger
Scheduled-Workflow (wöchentlich), der ein Reminder-Issue anlegt/aktualisiert, solange
ungemergte Sync-PRs existieren — und es automatisch schliesst, wenn keine mehr offen
sind. Kostet nichts, hält die Review-Schleife am Leben.

---

## 3. Lokale Konsistenz: Pre-Commit-Hook

Auch manuelle Edits müssen die Auto-Index-Tabellen konsistent halten. Ein
Pre-Commit-Hook rebuildet die MOC-Indizes, wenn Vault-Dateien oder die Registry
gestaged sind, und staged die resultierenden MOC-Änderungen automatisch mit:

```bash
#!/bin/bash
cd "$(git rev-parse --show-toplevel)"
npx tsx .automation/scripts/update-indices.ts
git add <vault>/MOCs/*.md
```

Aktivierung versionierbar über `git config core.hooksPath hooks` (per
Install-Skript), statt `.git/hooks` von Hand zu kopieren.

---

## 4. Walking-Skeleton-Ansatz

Das System wird als **Walking Skeleton** aufgebaut: Die gesamte Pipeline
(fetch → filter → patch → apply → audit → PR) läuft von Anfang an durch, aber jede
Inhalts-Kategorie ist bewusst mit **genau einer Beispiel-Datei** instanziiert
(eine Tier-1-Note, eine MOC, eine Deprecated-Note, ein Inbox-Item).

Vorteile:

- Architektur-Fehler zeigen sich, bevor 50 Notes existieren.
- Jede weitere Note folgt einem belegten, auditierten Muster.
- Der Dry-Run-Modus (`DRY_RUN=1 DRY_WRITE=1`) macht die Kette ohne API-Key und
  ohne Vault-Writes durchlaufbar — der Smoke-Test für jede Umgebung.

---

## 5. Secrets- und Daten-Hygiene

Regeln, die sich bewährt haben:

1. **Kein Credential im Repo, nie.** API-Keys ausschliesslich als CI-Secrets bzw.
   lokale `.env` (gitignored von Commit 1 an). Auch OAuth-Flows im CI referenzieren
   nur `secrets.*`, Tokens werden im Log maskiert.
2. **Umgebungs-Spezifika parametrisieren.** Tenant-Namen, Site-URLs, lokale Pfade
   gehören in Variablen/Secrets oder eine lokale Konfig-Datei — nicht hartkodiert
   in Skripte und Workflows. (Auch wenn sie „nur" interne Infrastruktur beschreiben:
   Sie machen ein Repo unveröffentlichbar und die Skripte unportabel.)
3. **`.gitignore` von Anfang an vollständig**: `.env`, `*.pem`, transiente
   Pipeline-States, lokale Editor-/Tool-Zustände (z.B. Obsidian
   `workspace*.json`), Embedding-Caches von Plugins, `node_modules`.
4. **Personennamen minimieren.** Changelog-Autoren als Kürzel, keine Kollegen-Mails
   in Docs — macht das Repo teilbar, ohne PII zu verteilen.
5. **Third-Party-Code (z.B. Editor-Plugins) nicht mit-committen**, sondern
   ignorieren — Redistribution ist ein Lizenzthema und bläht das Repo auf.

---

## 6. Anti-Patterns

Erprobte „bau das nicht"-Liste:

- ❌ **Dokument-Store als Source of Truth.** Edits dort gehen verloren oder erzeugen
  Drift. Git ist die Wahrheit.
- ❌ **Direktes Pushen der Pipeline auf `main`.** Immer PR-basiert — Halluzinationen
  müssen abfangbar sein.
- ❌ **Ein Mega-LLM-Skript, das alles macht.** In deterministische Stages mit klaren
  Inputs/Outputs zerlegen; LLM nur, wo Sprachverständnis nötig ist.
- ❌ **Web-Suche im Konsum-Agent aktiv lassen.** Verleitet zu Halluzinationen aus
  Drittquellen und untergräbt das kuratierte Wiki.
- ❌ **Substring-Filter ohne Wort-Boundary** bei kurzen Keywords/Aliassen —
  flutet die Pipeline mit False Positives.
- ❌ **MOC-Sektionen ohne AUTO-INDEX-Marker automatisch überschreiben** — zerstört
  manuelle Kuratierung.
- ❌ **Deletes im Mirror automatisieren.** Versehentlicher Massen-Drop ist
  katastrophal; Aufräumen im Ziel bleibt eine menschliche Aktion.
- ❌ **Schwergewichts-Agent, wo ein deklarativer reicht.** Für reine
  Q&A-Konsultation ist ein deklarativer RAG-Agent um Grössenordnungen billiger,
  schneller und wartbarer als ein Custom-Engine-Agent.
- ❌ **Proprietäres Wissens-Repository (CMS, Datenbank) als primäre Ablage.**
  Man verliert Markdown-Diffs, den Editor-Workflow und die Volltext-Suche.

---

## 7. Versionierungs-Modell (Zusammenfassung)

| Was | Wie |
|---|---|
| Automatische Inhalts-Änderungen | PR, menschlicher Merge, nie direkt `main` |
| Manuelle Notes | normale Commits |
| Pipeline-State (Hashes, Seen-Items) | committed — reproduzierbarer Status |
| Transiente Artefakte | gitignored |
| Registry-Änderung (neue Entität) | gleiche PR wie zugehörige Note-Änderung |
| Template-Änderung | eigene PR + Abgleich der Research-Prompts (siehe [04-deep-research.md](04-deep-research.md)) |
