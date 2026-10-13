# 01 · Wissensarchitektur

Wie die Wissensbasis strukturiert ist: Entitäts-Notes, MOCs, das Tier/Watch-System
und die Schreibprinzipien, die den Unterschied zwischen einem Doku-Spiegel und einer
echten Beratungs-Wissensbasis ausmachen.

---

## 1. Der Qualitäts-Test jeder Note

Die Notes sind **keine Lernunterlagen und keine Feature-Kopien** aus der offiziellen
Dokumentation. Jede Sektion muss den Test bestehen:

> **„Liefere ich hier mehr als die offiziellen Docs?"** — Wenn nein: kürzen oder streichen.

Eine Entitäts-Note beantwortet vier Fragen, die im echten Einsatz (Beratung,
Architektur-Entscheid, Kaufentscheidung) gestellt werden:

1. **Passt diese Entität zur Situation?** → Sektion *Einsatz*
2. **Kann man sie kaufen/nutzen?** → Sektion *Status & Pricing*
3. **Was verschweigen die Marketing-Materialien?** → Sektion *Limitierungen & Fallstricke*
4. **Ist die Information noch aktuell?** → *Changelog* + *Monitoring*

---

## 2. Tier-System — Relevanz-Hierarchie

Jede Entität bekommt ein **Tier**, das ihre Bedeutung für die eigene Arbeit beschreibt:

| Tier | Bedeutung | Anspruch |
|---|---|---|
| **1 — Must-Know** | Kernentitäten, direkte Relevanz | in- und auswendig kennen; vollständige Note |
| **2 — Important** | fundiert einordnen können | solide Note mit Integrations- und Compliance-Teil |
| **3 — Awareness** | Überblick genügt | Minimal-Note (Pflicht-Sektionen) |
| **Deprecated** | abgekündigt | Kurz-Note mit Nachfolger + Migrationspfad; **nie empfehlen** ohne Nachfolger-Hinweis |

---

## 3. Watch-System — Beobachtungsintensität steuert Note-Tiefe

Das Feld `watch` sagt aus, wie eng eine Entität beobachtet wird — **und** welche
Sektionen ihre Note ausfüllen muss:

| Watch | Rhythmus | Wann | Sektions-Pflicht |
|---|---|---|---|
| `close` | wöchentlich | hohe Änderungsdynamik oder direkter Geschäftsimpact | **alle** Sektionen |
| `standard` | monatlich | moderate Dynamik, fundiertes Einordnen nötig | Pflicht + Integrationen + Security & Compliance |
| `passive` | quartalsweise / event-getrieben | stabil oder nur situativ relevant | nur Pflicht-Sektionen |

Zwei wichtige Regeln:

- **`watch` ist dynamisch.** Eine Entität kann wandern (z.B. ein Framework:
  `close` → `passive` → `deprecated`). Jeder Wechsel wird im Changelog festgehalten.
- **`watch` beschreibt die Beobachtungs-Frequenz, nicht zwingend die Note-Tiefe.**
  Ausnahme „consultancy-deep aber stabil": Pricing-/Lizenz-/Compliance-Themen ändern
  sich selten (`passive`), brauchen aber inhaltliche Tiefe (Hidden Costs,
  Break-Even-Rechnungen, Regions-Deltas). Faustregel: *„Braucht man in einem
  wichtigen Gespräch 5+ Minuten in dieser Note?"* — wenn ja, tiefer schreiben,
  Watch-Level trotzdem bei `passive` belassen.

---

## 4. Anatomie einer Entitäts-Note

Vollständiges Template: [templates/entity-note-template.md](templates/entity-note-template.md).
Die Pflicht-Sektionen (alle Watch-Levels) und ihr Designzweck:

### Elevator Pitch + Analogie

1–2 Sätze Positionierung für jemanden, der die Entität nie gehört hat — in 20 Sekunden
verstehen, ob sie relevant ist. Dazu **eine Ein-Satz-Analogie** als mentales Modell,
die an ein bereits bekanntes Konzept anknüpft (Muster: „Was X für Y war, ist diese
Entität für Z.").

### Einsatz — die wichtigste Sektion

Hier verbindet sich abstraktes Wissen mit konkreter Handlungsempfehlung. Nicht
„welcher Nutzertyp?" (zu starr), sondern **„welche Situation macht diese Entität
relevant?"**. Unterstruktur:

- **Job-to-be-done** — welches abstrakte Problem löst sie? Muster:
  *„When I ____, I want to ____, so I can ____."* Kein Feature-Listing.
- **Trigger-Signale** — konkrete Zitate/Situationen, bei denen die Entität auf den
  Tisch gehört. Das ist der Aha-Moment für den Leser (Muster: „Jemand sagt:
  ‚Wir haben jetzt drei Agents und keiner weiss mehr, wer welche Berechtigung hat.'").
- **Einsatz-Szenarien** — 2–3 konkrete Situationen. „Gibt's bei uns aktuell nicht"
  ist eine legitime, ehrliche Antwort — besser als gekünstelte Beispiele.
- **Voraussetzungen** — Lizenz-Baseline, Infrastruktur, Skills/Rollen, Compliance-Rahmen.
- **Aufwand & Kosten (eigene Schätzung)** — Setup, laufende Kosten, Betrieb.
- **Empfehlung** — klarer Standpunkt mit genau einem Signal:
  🟢 empfehlen · 🟡 beobachten · 🔴 meiden. Plus **nächster konkreter Schritt**
  („PoC bis Q3", „beobachten bis EU-Verfügbarkeit geklärt", „nicht verfolgen,
  auf Alternative Y verweisen").

### Status & Pricing

Kommerzielle Realität, präzise und **mit Sichtungs-Datum** (Preise und GA-Termine
verschieben sich häufig): Status, GA-Datum, Preise, Bundles, Voraussetzungen,
Regions-Verfügbarkeit (EU-Status explizit!), **Hidden Costs** (was nicht im
Standard-Preis steckt), Upgrade-Pfade.

### Kernkonzept

Die wichtigste Wissens-Sektion. Ziel: Wer die Note einmal liest, versteht die Entität
wirklich — Architektur, Design-Prinzip, Nutzungsgefühl. Ausdrücklich **ohne
Feature-Limit**. Unterstruktur:

- **Was es im Kern ist** — die „Seele" der Entität, die man aus keiner Feature-Liste
  ableiten kann: Wo beginnt sie, wo hört sie auf? Welche Wette über die Welt trifft
  der Hersteller mit diesem Design?
- **Wo es im Stack sitzt** — Layer-Tabelle (was passiert auf welchem Layer, wer liefert es).
- **Kern-Fähigkeiten** — pro Fähigkeit: was sie konkret macht (Mechanik, nicht
  Marketing) · wie sie im Kern funktioniert (Pattern/Architektur) · wann sie relevant
  ist · wo sie an Grenzen stösst (Brücke zur Limitierungen-Sektion).
- **Typischer Workflow** — chronologisch Setup → Build → Deploy → Operate. Welche
  Rollen machen was mit welchen Tools? Das ist der Verstehens-Hebel, den
  Marketing-Material nie liefert.

### Limitierungen & Fallstricke

Was der Hersteller nicht auf die Produktseite schreibt — der Haupt-Mehrwert einer
guten Wissensbasis und der einzige Ort, der Vertrauen schafft. Zwei Teile:

- **Was die Entität NICHT kann** — harte, *konzeptionelle* Grenzen (keine
  versionsspezifischen Bugs, die veralten). Jeder Punkt mit Alternative/Workaround.
- **Typische Fallstricke** — was übersehen Anwender erfahrungsgemäss? Was führt zu
  gescheiterten Projekten? Konkret, konzeptionell, mit Gegenmittel.

### Offizielle Referenzen & Monitoring

Doppelfunktion: (1) **Quellen-Nachweis** für die Note, (2) **Monitoring-Anker** —
wo schaut man regelmässig hin? Drei Ebenen, jede Zeile mit `Zuletzt gesichtet`-Datum:

- **Primary** — offizielle Quellen des Herstellers (Produktseite, Docs, Release Notes,
  Roadmap, Pricing-Seite, Konferenz-Sessions).
- **Secondary** — Analysten und vertrauenswürdige Industrie-Quellen.
- **Tertiary** — einzelne Community-Stimmen/Blogs mit nachweislich guter Signal-Rate.
  Keine Marketing-Posts.

Dazu: **Events zum Beobachten** (wo werden Ankündigungen erwartet, nächste Termine).

### Changelog

Chronologisches Protokoll aller **substanziellen** Änderungen, neueste oben.
Trigger: Preis-/GA-/Verfügbarkeits-Änderung, neue Features/Deprecations, geänderte
eigene Einschätzung, PoC-Erkenntnisse, neu entdeckte Limitierung.
**Nicht** für Typos, Formatierung, reine Link-Updates.

### Optionale Sektionen (nach Watch-Level)

| Sektion | Levels | Inhalt |
|---|---|---|
| **Integrationen** | close + standard | beratungsrelevante Verbindungen und ihre Friction-Points — nicht „200 Connector-Namen" |
| **Security & Compliance** | close + standard | Data Residency, Prompt-/Output-Speicherung, DPA-Abdeckung, Regulatorik-Klassifizierung, ehrliche Compliance-Lücken |
| **Abgrenzung & Wettbewerb** | nur close | Wann diese Entität vs. Nachbar-Entität desselben Herstellers (häufigste Verwechslungsfrage!) + externe Alternativen als Dimensions-Matrix + **eine** klare Empfehlungs-Regel |

---

## 5. MOC-Theorie — Maps of Content

Eine **MOC** ist ein thematischer Einstiegspunkt im Vault. Sie ist **kein Produkt und
keine Zusammenfassung**, sondern:

1. bündelt Entitäten, die zusammen ein Themenfeld beschreiben,
2. erklärt **Zusammenhänge** (Stack-Diagramme, Eskalationsleitern, Decision-Trees) —
   das kann keine Einzel-Note leisten,
3. dient als Navigations-Hub,
4. verweist quer auf benachbarte MOCs.

### Hybrid-Struktur: Home-MOCs, Lens-MOCs, Querschnitts-MOCs

| Typ | Regel | Beispielrolle |
|---|---|---|
| **Root-MOC** | genau eine; jede Note referenziert sie | Gesamt-Landkarte des Ökosystems |
| **Primary-Home-MOCs** | jede Entität hat **genau eine** Home-MOC | Themenfelder („Agents", „Security", „Daten & Wissen", …) |
| **Lens-MOCs** | verlinken Entitäten **einseitig quer**, sind nie Home | Frage-Perspektiven („Welches SDK für welchen Build?", „Welches RAG-Backend wofür?") |
| **Querschnitts-MOCs** | optional zusätzlich zur Home-MOC | kommerzielle Sicht (Lizenzen, SKUs, Bundles) |

**Zuordnungs-Heuristik:** Jede Note = Root-MOC + genau eine Primary-Home-MOC
(+ ggf. Querschnitts-MOC). Lens-MOCs referenzieren die Entität von ihrer Seite aus —
die Entität verweist *nicht* zurück auf die Lens.

### Bidirektionalität — die wichtigste Pflege-Regel

Wer eine neue Note anlegt, muss die Ziel-MOC(s) ergänzen (Tabelle „Entitäten in
dieser MOC"), sonst bleibt die Verknüpfung einseitig und die MOC unvollständig.
Frontmatter-Feld `moc:` in der Note ↔ Tabellen-Zeile in der MOC.

In der Automatisierung wird diese Pflege maschinell abgesichert: MOC-Tabellen liegen
zwischen `AUTO-INDEX`-Markern und werden von einem Skript aus der Registry
re-rendert (siehe [02-update-pipeline.md](02-update-pipeline.md), Stage 6).

### Landkarten-Formen

Die MOC-„Landkarte" wählt die Darstellung, die das Feld am klarsten macht:

- **Stack-Diagramm** — Schichtung (wer sitzt über wem)
- **Eskalationsleiter** — vom einfachsten zum mächtigsten Werkzeug
- **Kategorien-Matrix** — zwei Dimensionen (z.B. Prevention/Detection/Response × Plattform)
- **Decision-Tree** — „Kunde braucht X → welche Option?"

---

## 6. Datenmodell: Registry + Frontmatter (bewusst redundant)

Eine zentrale YAML-Registry (siehe
[templates/registry.schema.yaml](templates/registry.schema.yaml)) listet alle
Entitäten: `slug`, `note`, `tagline`, `tier`, `watch`, `status`, `layer`,
`primary_home_moc`, `keywords`, `aliases`, `sources`, `enabled`.
**Alles andere — Notes, MOC-Tabellen, Filter, UI — wird von dieser Datei abgeleitet.**

Jede Note trägt zusätzlich eigenes Frontmatter (`watch`, `status`, `last_verified`,
`aliases`, `moc`). Die Redundanz ist **gewollt**: Die Note muss auch ohne Registry
für sich allein lesbar sein (Obsidian-Workflow), und der Audit-Step prüft die
Konsistenz beider Quellen maschinell.

---

## 7. Schreibprinzipien

1. **Bezieh Stellung.** Vage Formulierungen („kann hilfreich sein") streichen.
2. **„Unklar" ist OK — weglassen nicht.** Eine dokumentierte Wissenslücke ist
   wertvoller als eine stillschweigende.
3. **Jede Aussage braucht eine Quelle** (unter Referenzen, mit Datum) **oder die
   explizite Markierung „(eigene Einschätzung)"**.
4. **Nie überschreiben bei Umbenennungen/Änderungen** — Changelog-Eintrag schreiben;
   alte Information als „Historisch" belassen, wenn sie für Bestandsnutzer relevant ist.
5. **Ehrlich statt beschönigend** — besonders in Limitierungen & Compliance-Lücken.
   Genau dort entsteht der Vertrauens-Vorsprung gegenüber Marketing-Material.
