# 05 · Konsumschicht

Wie die git-versionierte Wissensbasis für Menschen nutzbar wird: ein
Dokument-Store-Mirror als RAG-Grundlage für einen Chat-Agenten, und eine statische
Website für Browsing, Vergleich und Stakeholder-Demos. Beide Layer sind
**read-mostly** — sie konsumieren das Repo, nie umgekehrt.

---

## 1. Dokument-Store-Mirror (Chat-Agent-Grundlage)

Chat-Agenten in Kollaborations-Plattformen (M365 Copilot, Google Agentspace,
Slack-Bots mit RAG etc.) indizieren typischerweise einen Dokument-Store
(SharePoint-Library, Google Drive, S3-Bucket). Das Repo wird dorthin gespiegelt.

### Variante A · CI-gesteuert (empfohlen für Teams)

Ein Workflow triggert bei Push auf `main`, wenn sich Note-Dateien geändert haben:

1. **OAuth Client-Credentials-Flow** mit einer dedizierten App-Registration
   → Access-Token (nur im CI, token maskiert)
2. Ziel-Site/Drive-ID über die API auflösen
3. **Diff seit letztem Commit** (`git diff --diff-filter=AM HEAD^ HEAD`)
   → Liste geänderter `.md`-Dateien
4. Pro Datei: Upload via API (`PUT .../root:/<Ordner>/<Datei>:/content`)
5. **Keine Deletes** — Schutz vor versehentlichem Massen-Löschen;
   Aufräumen im Ziel bleibt menschlich
6. Concurrency-Group verhindert parallele Schreib-Konflikte

Berechtigungs-Prinzip: **Least Privilege** — der App nur Schreibrecht auf die
eine Ziel-Site geben (z.B. Graph `Sites.Selected` statt `Sites.ReadWrite.All`),
Secret-Rotation terminieren, Zugangsdaten ausschliesslich als CI-Secrets.

### Variante B · lokal gesteuert (Ein-Personen-Betrieb)

Wenn das Vault lokal die Arbeitskopie ist: Spiegelung per `robocopy`/`rsync` in
einen synchronisierten Ordner (OneDrive-/Drive-Client), getriggert durch den
OS-Scheduler (z.B. bei Anmeldung + alle 3 Stunden). Einfacher, aber an eine
Maschine gebunden — Pfade als Konstanten am Skript-Kopf konfigurierbar halten.

**In beiden Varianten gilt:** Der Store ist Mirror, nie Editier-Ort.

---

## 2. Deklarativer Chat-Agent

Für reine Q&A-Konsultation der Wissensbasis reicht ein **deklarativer Agent**
(Konfigurations-JSON: Instructions + Knowledge-Quelle + Conversation-Starters) —
um Grössenordnungen billiger, schneller und wartbarer als ein Custom-Engine-Agent.

### Anti-Halluzinations-Regeln (Kern der Instructions)

```
- Antworte AUSSCHLIESSLICH auf Basis der angeschlossenen Wissensbasis.
- Wenn die Wissensbasis keine Antwort hergibt, sage explizit:
  "Dazu gibt das Wiki keinen Stand her — Empfehlung: Deep-Research anstossen."
  (kein Web-Fallback; Web-Suche im Agent ABGESCHALTET)
- Niemals Produktnamen, Preise, Regionen, Funktionen oder Termine erfinden.
- Deprecated-Entitäten NIE empfehlen ohne Nachfolger-Hinweis.
```

### Erzwungene Antwort-Struktur

Jede Empfehlung MUSS enthalten: Entitäts-Name + Tier + Watch-Level ·
Ein-Satz-Beschreibung · Compliance-Status explizit (✅/⚠️/❌) ·
Lizenz-Voraussetzung · 2–3 Stärken · 1–2 Schwächen · mind. 1 Alternative mit
Kurzvergleich · **Quellen-Pfad zur Markdown-Note**.

Der Quellen-Pfad ist entscheidend: Er macht jede Agent-Antwort im Vault
nachprüfbar und trainiert Nutzer, der kuratierten Basis zu vertrauen statt dem
Agenten blind.

Weitere bewährte Instruction-Bausteine: das Tier-System mit Beispielen ins Prompt
kodieren („Tier 1 kennst du auswendig, Tier 3 auf Awareness-Level") ·
Deprecated-Migrationspfade explizit auflisten · Entscheidungs-Heuristiken statt
„es kommt darauf an" verlangen (immer 2–3 konkrete Entscheidungsfaktoren nennen).

### Validierungs-Fragen

Drei Fragen, die jede Agent-Iteration korrekt beantworten muss:

1. **„Welche Option für Anforderung X mit Compliance-Bedingung Y?"** —
   erwartet: 2–3 Optionen mit Trade-offs, alle mit Compliance-Status und Quellen-Pfad.
2. **„Migrationspfad von <deprecated Entität>?"** — erwartet: konkreter Nachfolger,
   nie die deprecated Entität selbst empfehlen.
3. **„Status von <Entität mit unklarer Lage>?"** — erwartet: Tier/Watch/Status +
   Datum der letzten Änderung.

**Wenn eine Frage versagt: Instructions schärfen, nicht das Wiki neu indexieren** —
das Wiki ist meistens nicht das Problem.

---

## 3. Statische Website (Browsing- und Demo-Layer)

Ein Static-Site-Generator (z.B. Astro mit React-Islands) parst **zur Build-Zeit**
die Registry + Markdown-Notes. Kein Backend, keine API, kein CMS — die Site ist
eine weitere Projektion desselben Repos.

### Bewährte Seiten-Typen

| Seite | Zweck |
|---|---|
| Katalog (Grid + Tabelle) | filterbar nach Tier / MOC / Watch / Status, Volltext-Suche |
| Detail-Seiten | auto-generiert pro Note (`/products/<slug>`) |
| Ökosystem-Graph | Entitäten + typisierte Beziehungs-Kanten, Hover = Upstream/Downstream |
| Konfigurator | Decision-Tree-Wizard mit transparentem Filter-Funnel („45 → 23 → 8, weil …") |
| Vergleich | Side-by-Side-Matrix, deep-linkbar via `?products=slug1,slug2` |
| Insights | kuratierte, nicht-offensichtliche Findings quer über die Basis |

### Techniken

- **Filter-State teilen**: ein gemeinsamer Filter-Hook über alle Views
  (Grid/Tabelle/Graph), damit die Auswahl beim View-Wechsel erhalten bleibt.
- **Heuristische Fakten-Extraktion**: Vergleichs-Flags (Compliance-Status,
  Preismodell, Voraussetzungen) zur Build-Zeit aus dem Markdown-Body parsen statt
  eine separate Matrix zu pflegen — eine Wahrheitsquelle weniger.
- **Wikilink-Auflösung**: `[[Note]]`-Links per Remark-Plugin in Site-Routen
  übersetzen.
- **Zugriffsschutz für interne Sites**: Edge-Auth vor die statische Site
  (z.B. Cloudflare Access / IAP) mit E-Mail-Allowlist bzw. Service-Token —
  die Site selbst bleibt dumm und statisch.

### Verhältnis zum Chat-Agenten

Die Site beantwortet **Browsing-Fragen** („was gibt es, wie hängt es zusammen?"),
der Agent **Situations-Fragen** („was nehme ich für Fall X?"). Beide lesen dieselbe
Quelle; keiner von beiden ist ein Schreib-Ort.
