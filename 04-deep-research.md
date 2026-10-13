# 04 · Deep-Research-Workflow

Die Tages-Pipeline ([02](02-update-pipeline.md)) hält bestehende Notes mit
Mikro-Patches aktuell. Für **neue Notes** und **grosse Überarbeitungen** braucht es
tiefe Recherche — durch einen Research-Agenten (Deep-Research-Tool, LLM-Agent mit
Websuche o.ä.). Dieser Workflow regelt, wann man ihn einsetzt, mit welchen Prompts,
und wie der Output kontrolliert ins Vault kommt.

---

## 1. Tiered-Prompt-System

Vier wiederverwendbare Prompts, deckungsgleich mit dem Tier/Watch-System der
Wissensarchitektur:

| Prompt | Erzeugt | Watch-Level | Umfang |
|---|---|---|---|
| `tier1-deep-research` | vollständige Note, alle Sektionen inkl. Wettbewerbs-Profil | close | gross |
| `tier2-standard-research` | Pflicht + Integrationen + Security & Compliance | standard | mittel |
| `tier3-awareness-research` | nur Pflicht-Sektionen, bewusst knapp | passive | klein |
| `deprecated-rewrite` | Status-Wechsel `ga → deprecated`: Kurz-Note + Migrationspfad | passive | klein |

Auswahl-Baum:

```
Neues Produkt fürs Wiki?
├── hohe Dynamik / direkter Impact        → tier1 (watch: close)
├── moderate Dynamik, Einordnen reicht    → tier2 (watch: standard)
└── stabil / situativ, Überblick reicht   → tier3 (watch: passive)

Bestehendes Produkt wird deprecated (offizielle EOS-Ankündigung + Nachfolger)?
└── → deprecated-rewrite

Bestehende Note braucht Refresh?
├── Major (mehrere Sektionen, Tier-Wechsel) → Tier-Prompt + Diff im PR begründen
└── Mikro (1 Fakt, 1 Datum, 1 Preis)        → KEIN Research-Agent,
                                              das macht die Tages-Pipeline
```

**Prompt-Anforderungen** (in jedem Tier-Prompt kodiert):

- Quellen-Priorität explizit: offizielle Docs > Hersteller-Blogs > Community-Praxis
  (kritische Quelle, nicht für Fakten). **Verboten:** Marketing-Whitepapers,
  Press-Releases, Listicles.
- Pflicht-Checks pro Sektion: Compliance konkret mit Region + Datum; Pricing mit
  SKU-Namen; mind. 2 Wettbewerber; Roadmap-Signale nur mit Datum + Quelle,
  Spekulation nie als Fakt; Stärken/Schwächen ehrlich kritisch.
- Output: vollständige Markdown-Note **inkl. Frontmatter**, ready für Commit —
  muss den maschinellen Audit grün durchlaufen.

---

## 2. Quota-Ökonomie

Research-Agenten sind teuer (Query-Budgets, Rechenzeit). Sparsamkeit ist Pflicht.

**Wann KEIN Research-Agent:**

| Situation | Stattdessen |
|---|---|
| 1 Fakt, 1 Datum, 1 Preis | Tages-Pipeline (Mikro-Patch via Feed) |
| Typo / Formatierung | Direkt-Commit |
| Eigene Erfahrung dokumentieren | Direkt-Edit — der Agent kennt die eigene Praxis nicht |
| MOC-Index-Update | macht `update-indices` automatisch |
| Neue Entität taucht in News auf | erst Registry-Stub, dann Tages-Pipeline beobachten; Deep-Research erst, wenn nach 1–2 Wochen substanziell zu schreiben ist |

**Budget-Verteilung** (Richtwert pro Monat bei ~25 Queries):

| Aktivität | Anteil |
|---|---|
| Tier-1-Recherchen (neue close-watch Entitäten) | 5–8 |
| Tier-2-Recherchen | 4–6 |
| Tier-3-Refresh-Tests (Mikro-Deltas) | 3–5 |
| Major-Refreshes | 2–4 |
| Deprecated-Rewrites | 1–2 |
| Reserve (ad-hoc, Nachschärfungs-Loops) | 3–5 |

Bei aufgebrauchter Reserve: Tier-3 auf herkömmliche Recherche zurückstufen,
Agent nur noch für Tier 1+2.

---

## 3. Drei-Pfade-SOP: Wie Output ins Vault kommt

Nach jedem Research-Lauf entscheidet der Output-Charakter über den Pfad:

```
Research-Output liegt vor
│
├── Vollständige Note, Entität ist NEU
│   → PFAD A · Direktablage
│     1. Registry-Eintrag anlegen (slug, tagline, tier, watch, keywords, enabled)
│     2. Note direkt unter <vault>/Products/<Name>.md ablegen
│     3. Audit lokal laufen lassen (0 Errors erwartet)
│     4. Branch + PR mit Research-Label
│
├── Vollständige Note, Entität EXISTIERT (Major-Refresh / Tier-Wechsel)
│   → PFAD A' · Direktablage mit Diff-Begründung
│     wie A, aber die PR-Beschreibung MUSS erklären:
│     „Was ist anders, warum, was wurde bewusst NICHT übernommen"
│
├── Sektions-Update / Fund betrifft mehrere Notes / Mikro-Deltas
│   → PFAD B · Inbox-Routing
│     1. Delta-Datei in _Inbox/ ablegen (Frontmatter: source_url,
│        source: researcher, date_clipped, scope: <slugs>)
│     2. Body: pro Delta → Ziel-Sektion, aktuelle Formulierung,
│        vorgeschlagener Patch, Quelle mit Datum
│     3. Nächster Tages-Sync erzeugt die Patches automatisch als PR
│     4. Review, Merge → Datei wandert automatisch nach _Inbox/_processed/
│
└── Spekulativ / unbelegt / Marketing-kontaminiert
    → REJECT — verwerfen oder mit schärferem Prompt wiederholen.
      Niemals unbelegt ins Vault, auch nicht „vorläufig".
```

**Pre-Audit-Checkliste** vor jedem PR (menschlich, 2 Minuten):
Frontmatter ab Zeile 1 vollständig · H1 ist der Entitäts-Name (nicht
„Researcher Output") · Root-MOC + genau eine Home-MOC im `moc:`-Feld ·
alle Pflicht-Sektionen des Tiers vorhanden · Referenzen enthalten URLs
(nicht nur Quellen-Namen) · Changelog hat eine Zeile mit heutigem Datum ·
Empfehlungs-Block enthält genau ein 🟢/🟡/🔴.

---

## 4. Gelernte Lektion: Refresh-Test → Mikro-Deltas

Die wichtigste empirische Erkenntnis aus dem Praxisbetrieb:
**Research-Output ist kein Ersatz für eine bestehende Note, sondern eine Quelle
für Mikro-Deltas.**

| Was der Research-Agent gut kann | Was er nicht kann |
|---|---|
| aktuelle Pricing-Daten präzise erheben | die eigene Bewertungslogik (🟢/🟡/🔴 mit konkretem nächsten Schritt) |
| Quellen-Diskrepanzen und Doku-Lücken aufspüren | eigene Runbooks und Migrations-Snippets |
| frische Quell-URLs für die Monitoring-Tabelle | domänenspezifische Patterns aus eigener Projekterfahrung |
| Regions-Verfügbarkeit Stand heute | „Gegenmittel"-Empfehlungen für Anti-Patterns |

Folge-Workflow nach jedem Refresh-Test:

1. Bestehende Note **nicht überschreiben**
2. Research-Output Zelle für Zelle gegen die Note diffen
3. Deltas identifizieren, wo der Agent aktueller/genauer ist
4. Inhalte identifizieren, die dem Agent fehlen → **bewusst behalten**
5. Inbox-Delta-Datei anlegen (Pfad B)
6. PR-Beschreibung enthält die „Was nicht übernommen wurde"-Tabelle

Der handkuratierte Bewertungs-Layer (Empfehlungen, Erfahrung, Runbooks) ist genau
der Teil, den keine automatische Recherche ersetzen kann — er ist der eigentliche
Wert der Wissensbasis und wird bei jedem Refresh aktiv geschützt.

---

## 5. PR-Beschreibungs-Template für Research-PRs

```markdown
## <Pfad-Label> · <slug>

**Research-Tier**: <1 / 2 / 3 / Deprecated>
**Quota-Verbrauch**: <N> von <Budget> (<Monat>)

### Was diese PR tut
<1–3 Sätze>

### Was NICHT in dieser PR ist
<bewusste Auslassungen aus dem Research-Output, mit Begründung>

### Audit-Status
- [ ] Audit lokal grün
- [ ] MOC-Wikilinks verifiziert
- [ ] Frontmatter-Pflichtfelder vorhanden

### Quellen-Stichprobe
<2–3 zentrale Source-URLs zur Reviewer-Verifikation>
```

---

## 6. Prompt-Verbesserungs-Loop

Die Prompt-Dateien sind **lebende Dokumente**. Nach jedem Research-Lauf:

1. Welche Audit-Errors blieben nach dem Output übrig?
2. Welche manuellen Korrekturen waren nötig?
3. Prompt nachschärfen (konkretere Regel oder Beispiel) → eigene PR mit
   Prompt-Tuning-Label.
4. Changelog-Block am Ende jeder Prompt-Datei führen
   (`- YYYY-MM-DD: Regel X verschärft, Grund: …`).

**Halbjährliche Wartung:** alle Prompt-Dateien gegen das Note-Template abgleichen —
wurde das Template um eine Sektion erweitert, ziehen die Prompts nach.

Typische Fehlerbilder und Nachschärfungen:

| Symptom | Nachschärfung |
|---|---|
| Output ohne YAML-Frontmatter | „Beginne mit `---` und dem Frontmatter exakt wie spezifiziert" |
| Pflicht-Sektion fehlt | „Sektion X ist nicht ausgefüllt — ergänze, ohne den Rest zu ändern" |
| Marketing-Sprech | „Sachlich-beratender Stil. ‚X is the leader in…' → <Faktum mit Quelle>" |
| halluzinierte Preise/Termine | Sektion mit `nicht öffentlich verfügbar (Stand <Datum>)` ersetzen lassen |
| fehlende URLs in Quellen | „Jede Faktenaussage muss in den Referenzen verlinkt sein" |
