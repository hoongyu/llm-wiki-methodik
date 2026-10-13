---
type: moc
tags: [moc]
last_verified:   # YYYY-MM-DD
---

<!--
=====================================================================
MOC-TEMPLATE-ANLEITUNG (beim Instanziieren diesen Block entfernen)

Eine MOC (Map of Content) ist ein thematischer Einstiegspunkt.
Sie ist KEINE Entität und KEINE Zusammenfassung — sie:
  1. bündelt Entitäten eines Themenfelds
  2. erklärt ZUSAMMENHÄNGE (Stack-Diagramme, Eskalationsleitern,
     Decision-Trees) — das kann keine Einzel-Note leisten
  3. dient als Navigations-Hub
  4. verweist quer auf benachbarte MOCs

Entitäten verlinken per `moc:`-Frontmatter hierher; die MOC listet sie
in ihrer Tabelle (Bidirektionalität!). Die Tabelle liegt zwischen
AUTO-INDEX-Markern und wird vom Pipeline-Skript re-rendert.

PFLICHT: Zweck · Landkarte · Entitäten-Tabelle · Sammelquellen
OPTIONAL: Eskalationsleiter/Entscheidungslogik · Querverweise · Glossar
=====================================================================
-->

# {Thema} MOC

*{1–3 Sätze: Welches Themenfeld bündelt diese MOC? Was verbindet die enthaltenen Entitäten?}*

---

## Zweck

*{Für welche Situation ist diese MOC der richtige Einstieg? Welche Fragen beantwortet sie? Wann startet man hier statt direkt in einer Entitäts-Note?}*

---

## Landkarte

*{Das semantische Modell der Gruppe — wähle die Darstellung, die das Feld am klarsten macht:}*

*{— Stack-Diagramm (Schichtung) · Eskalationsleiter (einfachst → mächtigst) · Kategorien-Matrix (2 Dimensionen) · Decision-Tree („braucht X → welche Option?")}*

```
{ASCII-Diagramm oder Mermaid-Block}
```

---

## Entitäten in dieser MOC

<!-- AUTO-INDEX:START:ENTITAETEN -->
| Entität | Was es ist (1 Satz) | Tier | Watch | Status |
|---|---|---|---|---|
| [[Entität X]] | *wird vom Pipeline-Skript aus Registry + Frontmatter gerendert* | 1 | close | ga |
<!-- AUTO-INDEX:END:ENTITAETEN -->

---

## Eskalationsleiter / Entscheidungslogik

*{Wenn das Thema eine klare „Wann-welches"-Logik hat:}*

| Situation | Entität der Wahl | Begründung |
|---|---|---|
| | | |

---

## Querverweise zu anderen MOCs

- [[Andere MOC 1]] — *{Beziehung}*
- [[Andere MOC 2]] — *{Beziehung}*

---

## Offizielle Sammelquellen

*{Overview-/Hub-Seiten, die das ganze Feld beschreiben — nicht Einzel-Entitäts-Seiten.}*

| Typ | Quelle | Link | Zuletzt gesichtet |
|---|---|---|---|
| Hub-Page | | | YYYY-MM-DD |
| Blog-Serie | | | YYYY-MM-DD |

---

## Letzte Aktivität

<!-- AUTO-INDEX:START:LETZTE_AKTIVITAET -->
*{aggregierte Changelog-Zeilen aller Entitäten dieser MOC, vom Skript gerendert}*
<!-- AUTO-INDEX:END:LETZTE_AKTIVITAET -->

---

## Changelog

| Datum | Autor | Änderung |
|---|---|---|
| *YYYY-MM-DD* | *{Kürzel}* | *Initial-Erstellung der MOC* |
