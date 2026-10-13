# 02 · Update-Pipeline

Die Automatisierungsschicht: sieben deterministische Stages, die täglich Neuigkeiten
einlesen, filtern, per LLM in minimale Patches übersetzen und als Pull Request zur
menschlichen Review stellen.

---

## 0. Design-Prinzip: LLM nur, wo Sprachverständnis nötig ist

```
fetch → classify → filter → patch → apply → update-indices → audit
 (0 LLM)  (0 LLM)   (0 LLM)  (LLM!)  (0 LLM)     (0 LLM)      (0 LLM)
```

Genau **eine** Stage ruft ein LLM auf. Alle anderen sind deterministisch:
reproduzierbar, testbar, tokenfrei, und bei Fehlern exakt diagnostizierbar.
Das ist die wichtigste Architektur-Entscheidung des Systems — sie hält Kosten,
Latenz und Halluzinationsrisiko minimal.

Alle Stages sind idempotent: Mehrfaches Ausführen ändert nichts mehr, sobald der
Steady-State erreicht ist. Jede Stage liest aus einem definierten State-File und
schreibt ein neues — die Kette ist an jeder Stelle unterbrech- und wiederaufnehmbar.

---

## 1. Stage `fetch` — Quellen einlesen

Liest RSS-/Atom-/JSON-Feeds und GitHub-Releases aus der Quellen-Registry
([templates/sources.schema.yaml](templates/sources.schema.yaml)).

- Output: `state/raw-items.json` mit `{ source, title, url, published, summary, tag }`
- **Dedup über URL-Hash** (`state/feed-hashes.json`) — Re-Runs sind idempotent
- Nur offizielle/kuratierte Quellen. Qualität der Pipeline = Qualität der Quellen-Whitelist.

## 2. Stage `classify` — manuelle Clippings scoren

Anwender werfen Web-Snippets per Web-Clipper/Drag&Drop in einen `_Inbox/`-Ordner
des Vaults. Diese Stage bewertet jedes Clipping heuristisch:

| Kriterium | Score |
|---|---|
| Domain in Quellen-Whitelist | +5 |
| ≥ 2 Registry-Keywords im Text | +3 |
| Publikation < 30 Tage | +2 |

Score ≥ Schwellwert → Clipping wird wie ein Feed-Item in die Pipeline eingespeist
und nach `_Inbox/_processed/<datum>/` verschoben; sonst nach `_Inbox/_rejected/`.
So fliessen menschliche Funde und automatische Feeds durch **denselben** Pfad.

## 3. Stage `filter` — Keyword-Match ohne LLM

Matcht raw-items gegen die Keywords der Registry — mit **Wort-Boundary-Disziplin**:

```typescript
function matches(text: string, keywords: string[]): boolean {
  return keywords.some(k => {
    const escaped = k.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    if (k.length <= 4) {
      // Kurze Keywords brauchen strikte Wort-Boundary,
      // sonst matcht z.B. "SK" in "task", "risk", "desktop"
      return new RegExp(`\\b${escaped}\\b`, 'i').test(text);
    }
    return new RegExp(escaped, 'i').test(text);
  });
}
```

Output: `state/filtered-items.json` mit `{ item, matched_slugs[] }`.
Nur Entitäten mit `enabled: true` in der Registry werden gematcht — das ist der
Ein/Aus-Schalter pro Entität.

**Gelernte Lektion:** Substring-Filter ohne Wort-Boundary bei kurzen Aliassen
(`SK`, `AI`, `ML`) fluten die Pipeline mit tausenden False Positives.

## 4. Stage `patch` — der einzige LLM-Schritt

Für jedes (Item, Slug)-Paar erzeugt das LLM einen **minimalen Patch** gegen die
bestehende Note — über **Tool-Use mit striktem Schema** (kein Freitext-Parsing):

```typescript
const tools = [{
  name: "create_note_patch",
  input_schema: {
    type: "object",
    properties: {
      slug: { type: "string" },
      changes: {
        type: "array",
        items: {
          type: "object",
          properties: {
            section: { type: "string" },   // Sektions-Header, z.B. "Changelog"
            mode: { enum: ["append", "replace_after_marker", "prepend"] },
            content: { type: "string" }
          },
          required: ["section", "mode", "content"]
        }
      },
      changelog_entry: { type: "string" },  // "- YYYY-MM-DD: <Änderung> (<Quelle>)"
      update_last_verified: { type: "string" }
    },
    required: ["slug", "changes", "changelog_entry"]
  }
}];
```

Die System-Prompt-Regeln (Kern):

```
Du bist ein präziser Knowledge-Base-Editor. Du erhältst (1) eine bestehende
Markdown-Note und (2) ein neues News-Item. Erzeuge einen MINIMALEN Patch:

- Niemals ganze Sektionen umschreiben, wenn nur 1 Fakt sich geändert hat.
- Immer eine Zeile in ## Changelog ergänzen (mit Datum + Quelle).
- Wenn die News der Note widerspricht: KEIN Patch, sondern
  `needs_human_review` mit kurzer Begründung.
- Roadmap-Spekulation (kein offizielles Statement): nur als Signal mit
  "(spekulativ)"-Marker eintragen, NIE als Fakt.
- Niemals Funktionen, Preise oder Termine erfinden, die nicht im Item stehen.
- NICHTS zwischen AUTO-INDEX-Markern ändern — das wird deterministisch
  neu gerendert.
```

**Token-Ökonomie:**

- **Prompt-Caching** für den System-Prompt und die Template-Beschreibung —
  bei N Items pro Tag zahlt man den statischen Kontext nur einmal.
- **Nur Delta**: Das LLM sieht die betroffene Note + das eine News-Item,
  nie das ganze Vault.
- **`DRY_RUN`-Modus**: erzeugt deterministische Stub-Patches ohne API-Call —
  die gesamte Kette ist ohne API-Key durchlaufbar (CI-Smoke-Test, lokale Entwicklung).

Output: `state/patches.json`.

## 5. Stage `apply` — Patches anwenden

Wendet Patches als Text-Mutationen auf die Markdown-Dateien an: findet Sektionen
anhand der Header-Zeile, respektiert AUTO-INDEX-Marker, schreibt zurück.
`DRY_WRITE=1` loggt nur, schreibt nichts — der zweite Schalter für gefahrlose Tests.

## 6. Stage `update-indices` — MOC-Tabellen re-rendern

MOCs enthalten manuell kuratierte Sektionen **und** automatisch gepflegte Tabellen.
Nur letztere liegen zwischen HTML-Kommentar-Markern:

```markdown
<!-- AUTO-INDEX:START:PRODUKTE -->
| Entität | Tagline | Tier | Watch | Status |
| ...     | (vom Skript aus Registry + Frontmatter re-rendert) |
<!-- AUTO-INDEX:END:PRODUKTE -->
```

Das Skript liest Registry + Note-Frontmatter + die letzten Changelog-Einträge und
überschreibt **ausschliesslich** den Bereich zwischen den Markern. Typische
Auto-Index-Blöcke: `STAND` (Datum + Anzahl pro Tier), `PRODUKTE` (Tabelle),
`RESEARCH-STATUS` (deep/stub), `LETZTE_AKTIVITAET` (aggregierte Changelogs).

**Gelernte Lektion:** MOC-Sektionen ohne Marker automatisch zu überschreiben
zerstört die manuelle Kuratierung — der wertvollste Teil jeder MOC.

## 7. Stage `audit` — Template-Gate

Validiert das gesamte Vault maschinell:

- Frontmatter vollständig und **konsistent mit der Registry**
- Tier × Watch → Pflicht-Sektionen vorhanden (hartkodierte Matrix)
- Deprecated-Notes haben `successor` + Banner
- Quellen-Sektion enthält mindestens 1 URL
- Changelog absteigend sortiert
- keine Alias-Kollisionen
- AUTO-INDEX-Marker korrekt gepaart

Exit-Code ≠ 0 bei Verstössen. Im CI läuft der Audit mit `continue-on-error`,
**aber sein Output wird in die PR-Beschreibung eingebettet** — der Reviewer sieht
jeden Verstoss, ohne dass Alt-Bestands-Verstösse den täglichen Sync blockieren.

---

## State-Management: committen vs. wegwerfen

| State-Datei | Behandlung | Warum |
|---|---|---|
| `feed-hashes.json`, `seen-items.json`, `last-run.json` | **committed** | Teil des reproduzierbaren Pipeline-Status — ohne sie würde jeder CI-Lauf alles neu verarbeiten |
| `raw-items.json`, `filtered-items.json`, `patches.json` | **gitignored** | flüchtige Zwischenprodukte eines einzelnen Laufs |
| Sync-/Lauf-Logs | **gitignored** | lokal-only |

---

## CI-Verdrahtung (Tages-Rhythmus)

Ein Workflow (cron, z.B. „täglich 06:17 UTC" — bewusst nicht xx:00, um
Feed-Server-Spitzen zu meiden) führt alle Stages aus und öffnet **einen PR** über
eine Action wie `peter-evans/create-pull-request`:

- Branch `auto/kb-sync-<run-id>`, Labels `kb-sync, needs-review`
- PR-Body enthält den Audit-Output
- `workflow_dispatch` mit `dry_run`-Input für manuelle Tests
- API-Key ausschliesslich als CI-Secret, nie im Code

Der PR wird **immer von einem Menschen gemerged** — siehe
[03-governance.md](03-governance.md).
