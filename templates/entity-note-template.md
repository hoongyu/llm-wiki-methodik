---
watch:           # close | standard | passive
status:          # announced | preview | ga | deprecated
last_verified:   # YYYY-MM-DD — Datum der letzten inhaltlichen Prüfung
aliases: []      # frühere oder alternative Namen
moc:
  - "[[<Root-MOC>]]"
  - "[[<Primary-Home-MOC>]]"
---

<!--
=====================================================================
TEMPLATE-ANLEITUNG (beim Instanziieren diesen Block entfernen)

ZWECK: Die Note beantwortet vier Fragen im echten Einsatz:
  1. Passt diese Entität zur Situation?        → Einsatz
  2. Kann man sie kaufen/nutzen?               → Status & Pricing
  3. Was verschweigt das Marketing?            → Limitierungen & Fallstricke
  4. Ist die Information noch aktuell?         → Changelog + Monitoring

Qualitäts-Test jeder Sektion: „Liefere ich hier mehr als die
offiziellen Docs?" — wenn nein: kürzen oder streichen.

SEKTIONS-PFLICHT nach Watch-Level:
  close    → alle Sektionen
  standard → Pflicht + Integrationen + Security & Compliance
  passive  → nur Pflicht-Sektionen

SCHREIB-PRINZIPIEN:
  - Bezieh Stellung; vage Formulierungen streichen.
  - „Unklar" ist OK — weglassen nicht.
  - Jede Aussage: Quelle (mit Datum) oder „(eigene Einschätzung)".
  - Bei Umbenennung/Änderung: Changelog-Eintrag, nie stilles Überschreiben.
=====================================================================
-->

# {Entitäts-Name}

*{1–2 Sätze Positionierung für jemanden, der die Entität nie gehört hat. In 20 Sekunden verstehen, ob sie relevant ist. Keine Feature-Liste, kein Marketing.}*

> **Analogie:** *{Ein-Satz-Analogie als mentales Modell. Muster: „Was X für Y war, ist diese Entität für Z."}*

---

## Einsatz

### Job-to-be-done

*{Welches abstrakte Problem löst die Entität? Muster: „When I ____, I want to ____, so I can ____." 2–4 Sätze, keine Feature-Liste.}*

### Trigger-Signale

*{Konkrete Zitate/Situationen, bei denen die Entität auf den Tisch gehört:}*

- *{Zitat oder Situation 1}*
- *{Zitat oder Situation 2}*
- *{Zitat oder Situation 3}*

### Einsatz-Szenarien

1. **{Szenario-Titel}** — *{Ausgangssituation · Ziel · warum diese Entität der richtige Hebel ist}*
2. **{Szenario-Titel}** — *{…}*

*{„Gibt's bei uns aktuell nicht" ist eine legitime Antwort — ehrlicher als gekünstelte Beispiele.}*

### Voraussetzungen

| Voraussetzung | Details |
|---|---|
| **Lizenz-Baseline** | |
| **Infrastruktur** | |
| **Skills / Rollen** | |
| **Compliance-Rahmen** | |

### Aufwand & Kosten (eigene Schätzung)

| Dimension | Grössenordnung |
|---|---|
| **Setup / Einführung** | |
| **Laufende Lizenzkosten** | |
| **Laufender Betrieb** | |

### Empfehlung

**Status:** {🟢 empfehlen / 🟡 beobachten / 🔴 meiden} — *{1–2 Sätze Begründung mit konkreter Logik}*

**Nächster Schritt:** *{Was konkret tun? z.B. „PoC bis Q3" / „beobachten bis Verfügbarkeit geklärt" / „nicht verfolgen, auf Alternative Y verweisen"}*

---

## Status & Pricing

| Detail | Wert |
|---|---|
| **Aktueller Status** | *mit Datum* |
| **GA-Datum** | |
| **Preis** | *mit Sichtungs-Datum* |
| **Bundle / SKU** | |
| **Voraussetzung** | |
| **Regions-Verfügbarkeit** | *EU-/Compliance-Status explizit* |
| **Hidden Costs** | *was nicht im Standard-Preis enthalten ist* |
| **Upgrade-Pfad** | |

---

## Kernkonzept

### Was es im Kern ist

*{2–4 Absätze: Wo beginnt die Entität, wo hört sie auf? Welches zentrale Design-Prinzip? Welche Wette trifft der Hersteller mit diesem Design über die Welt? Die „Seele", die man aus keiner Feature-Liste ableiten kann.}*

### Wo es im Stack sitzt

| Layer | Rolle | Wer liefert das? |
|---|---|---|
| | | |

### Kern-Fähigkeiten

#### {Fähigkeit 1}

*{Was sie konkret macht (Mechanik, nicht Marketing) · wie sie im Kern funktioniert · wann sie relevant ist · wo sie an Grenzen stösst.}*

#### {Fähigkeit 2}

*{…}*

### Typischer Workflow

1. **Setup** — *wer macht was mit welchem Tool, wie lange*
2. **Build / Configure** — *Entwickler-Erfahrung, Debug-Loop*
3. **Deploy** — *wie kommt es in die Ziel-Umgebung*
4. **Operate** — *Monitoring, Update-Pfade*

---

## Limitierungen & Fallstricke

### Was die Entität NICHT kann

*{Harte, konzeptionelle Grenzen — keine versionsspezifischen Bugs. Jeder Punkt mit Alternative.}*

| Limitierung | Alternative / Workaround |
|---|---|
| | |

### Typische Fallstricke

- **{Fallstrick 1}** — *{was passiert · warum es übersehen wird · Gegenmittel}*
- **{Fallstrick 2}** — *{…}*

---

<!-- OPTIONAL [close + standard] -->

## Integrationen

| Mit | Zweck | Reifegrad | Friction-Points |
|---|---|---|---|
| | | | |

---

<!-- OPTIONAL [close + standard] -->

## Security & Compliance

| Thema | Status |
|---|---|
| **Data Residency** | |
| **Daten-Speicherung / Training-Opt-out** | |
| **Vertrags-Abdeckung (DPA o.ä.)** | |
| **Regulatorik-Klassifizierung** | |

### Bekannte Compliance-Lücken

*{Ehrlich: Wo hapert es?}*

---

<!-- OPTIONAL [nur close] -->

## Abgrenzung & Wettbewerb

### Intern: wann diese Entität vs. Nachbar-Entität desselben Anbieters?

| Situation | Diese Entität | Alternative |
|---|---|---|
| | ✅ wählen wenn {Kriterium} | ⚠️ stattdessen {X} wenn {Kriterium} |

### Externe Alternativen

| Dimension | {Diese Entität} | {Wettbewerber 1} | {Wettbewerber 2} |
|---|---|---|---|
| **Fokus** | | | |
| **Pricing** | | | |
| **Compliance** | | | |
| **Stärke** | | | |
| **Schwäche** | | | |

### Empfehlungs-Regel

*{Ein Satz: „Wir nehmen X über Y, wenn der Fall Z vorliegt. Sonst eher Y."}*

---

## Offizielle Referenzen & Monitoring

### Primary (Hersteller offiziell)

| Typ | Quelle | Link | Zuletzt gesichtet | Monitoring-Zweck |
|---|---|---|---|---|
| Produktseite | | | YYYY-MM-DD | allgemeine Änderungen |
| Docs | | | YYYY-MM-DD | API-/Feature-Updates |
| Release Notes | | | YYYY-MM-DD | Versionsänderungen |
| Roadmap | | | YYYY-MM-DD | geplante GAs |
| Pricing-Seite | | | YYYY-MM-DD | Preisänderungen |

### Secondary (Analysten & Industrie)

| Quelle | Link | Zuletzt gesichtet | Einschätzung |
|---|---|---|---|
| | | YYYY-MM-DD | |

### Tertiary (Community, nur nachweislich gute Signal-Rate)

| Autor | Kanal | Zuletzt gesichtet | Warum relevant |
|---|---|---|---|
| | | YYYY-MM-DD | |

### Events zum Beobachten

| Event | Datum | Erwartete Ankündigungen |
|---|---|---|
| | | |

---

## Changelog

<!-- Trigger: Preis-/GA-/Verfügbarkeits-Änderung · neue Features/Deprecations ·
     geänderte eigene Einschätzung · PoC-Erkenntnisse · neue Limitierung.
     NICHT für: Typos, Formatierung, reine Link-Updates. -->

| Datum | Autor | Änderung | Quelle |
|---|---|---|---|
| *YYYY-MM-DD* | *{Kürzel}* | *Initial-Erstellung, watch: {…}, Status {…}* | — |
