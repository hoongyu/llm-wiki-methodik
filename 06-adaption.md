# 06 · Adaption & Implementierung von null

Wann sich die Methodik lohnt, wie man sie auf eine neue Domäne überträgt, und in
welcher Reihenfolge man das System aufbaut.

---

## 1. Eignungs-Kriterien

Die Methodik funktioniert für jede Domäne mit:

| Kriterium | Richtwert |
|---|---|
| Abgrenzbare Entitäten | 20–150 (Produkte, Frameworks, Konzepte, Regularien) |
| Relevanz-Hierarchie | klar teilbar in Must-Know / Important / Awareness |
| Externe Update-Quellen | RSS / Blogs / offizielle Docs / GitHub-Releases vorhanden |
| Änderungsdynamik | Wissens-Halbwertszeit < 12 Monate |

Bei < 20 Entitäten lohnt die Automatisierung nicht (Handpflege ist billiger);
bei > 150 braucht es zusätzlich Sub-Domänen-Aufteilung.

## 2. Anpassungspunkte pro Domäne

Pipeline-Code, CI/CD und Konsum-Layer bleiben identisch — nur Schemas und
Whitelists ändern sich:

| Aspekt | Beispiel: Vendor-AI-Ökosystem | Beispiel: Cloud-Native-Stack |
|---|---|---|
| Registry-Schema | tier / watch / layer / status | tier / maturity / cncf-stage |
| MOCs | „Agents", „Security", „Lizenzen" | „Service-Mesh", „Observability" |
| Quellen-Whitelist | Vendor-Blogs, offizielle Docs | cncf.io, kubernetes.io, Projekt-Repos |
| Deprecated-Pfade | Alt-SDK → Nachfolge-Framework | Helm v2 → v3 |
| Validierungs-Fragen | Compliance, Region, Lizenz | CNCF-Stage, Production-Readiness, OSS-Lizenz |

## 3. Implementierungs-Reihenfolge

Als Walking Skeleton (siehe [03-governance.md](03-governance.md), §4): erst die
ganze Kette mit minimalem Inhalt, dann Inhalt in Breite.

### Phase 1 · Foundation (Tag 1)

1. Git-Repo + Verzeichnis-Struktur:
   ```
   .automation/{scripts,prompts,state}/
   .github/workflows/
   <vault>/{Products,MOCs,_Templates}/
   _Inbox/{_processed,_rejected}/
   hooks/
   ```
2. Registry mit ~5 Beispiel-Entitäten anlegen
   ([templates/registry.schema.yaml](templates/registry.schema.yaml))
3. Note- und MOC-Templates instanzieren
   ([templates/](templates/))
4. `.gitignore` vollständig ab Commit 1 (Secrets, transiente States, Editor-Zustände)

### Phase 2 · Pipeline-Skelett (Tag 2)

5. `fetch` + `filter` schreiben (deterministisch, kein LLM)
6. 3–5 Notes von Hand schreiben, gegen Template validiert
7. `update-indices` mit AUTO-INDEX-Marker-Logik
8. `audit` mit der Tier×Watch-Pflichtsektions-Matrix
9. Lokaler Dry-Run: gesamte Kette grün ohne API-Key

### Phase 3 · LLM-Integration (Tag 3)

10. `patch` mit Tool-Use-Schema (siehe [02-update-pipeline.md](02-update-pipeline.md), §4)
11. System-Prompt als eigene Datei unter `prompts/`
12. Mit echten News-Items dry-runnen, Patch-Qualität prüfen
13. `apply` einmal scharf laufen lassen, Diff reviewen

### Phase 4 · CI/CD (Tag 4)

14. Daily-Sync-Workflow mit allen Stages + PR-Erzeugung
15. API-Key als CI-Secret
16. `workflow_dispatch` mit `dry_run=true` als Probelauf
17. Pre-Commit-Hook für lokale Index-Konsistenz

### Phase 5 · Inhalt (Tage 5–10)

18. Deep-Research für Tier-1-Notes ([04-deep-research.md](04-deep-research.md))
19. Tier-2-Notes, dann Tier-3-Stubs
20. MOCs schreiben (manuelle Sektionen: Landkarte, Decision-Trees)
21. Audit auf 0 Errors bringen

### Phase 6 · Konsumschicht (Tage 11–12)

22. Statische Site aufsetzen (Katalog → Detail → Vergleich → Graph)
23. Dokument-Store-Mirror + Sync-Workflow ([05-konsumschicht.md](05-konsumschicht.md))
24. Deklarativen Chat-Agenten konfigurieren, 3 Validierungs-Fragen testen

### Phase 7 · Steady State (ab Tag 13)

25. Daily-Sync läuft selbständig; PRs einmal täglich reviewen
26. Mirror-Sync triggert nach jedem Merge
27. Neue Entität: Registry-Stub → Note via Deep-Research → automatischer Roundtrip

---

## 4. Bootstrap-per-LLM-Pattern

Die gesamte Methodik lässt sich als **ein selbsttragendes Bootstrap-Prompt** an
einen LLM-Coding-Agenten übergeben („baue dieses System für Domäne X von null").
Bewährte Regeln dafür:

- **Erwartete Outputs explizit ordnen**: Repo-Skelett → Registry mit 3 echten
  Beispiel-Entitäten → Templates → ein vollständig lauffähiges Skript
  (`update-indices` zuerst — es hat die klarste Spezifikation) → CI-Workflows →
  Agent-Konfiguration → README mit Schritt-für-Schritt-Anleitung.
- **Frage-Gates vor Code**: Der Agent muss nachfragen (und darf nicht anfangen),
  solange (1) die Domänen-Abgrenzung, (2) die Quellen-Whitelist, (3) die
  Tier-Logik oder (4) Ziel-Umgebungen (Tenant/Hosting) unklar sind.
- **Platzhalter statt Umgebungs-Werte**: Tenant-Namen, Site-URLs, Pfade als
  `<placeholder>` — die Instanziierung passiert pro Umgebung, nie im Code.

---

## 5. Erfolgskriterien im Betrieb

| Signal | Zielzustand |
|---|---|
| Daily-Sync-PRs | klein, reviewbar in < 5 Min, nie direkt auf `main` |
| Audit | dauerhaft 0 Errors im Bestand |
| Changelog-Dichte | jede substanzielle Änderung hinterlässt genau einen Eintrag |
| Agent-Validierungsfragen | alle 3 dauerhaft korrekt (siehe [05](05-konsumschicht.md), §2) |
| Handpflege-Aufwand | < 30 Min/Tag (Review + gelegentliche Research-Runs) |
| Registry = Realität | jede Entität im Vault hat einen Registry-Eintrag und umgekehrt |
