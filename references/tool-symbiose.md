# Tool-Symbiose — Cross-Cutting-Pflicht für jeden LLM-Call

**Vor jedem Reasoning-Schritt: Inventory → Match → Symbiose-Plan → aktiv nutzen → Gap-Hinweis.**

Der safe-debug-loop ist nur so gut wie die Tools die er einsetzt. Diese Datei beschreibt wie der Skill bei jedem Schritt das optimale Tool-Set zusammenstellt.

---

## Warum das wichtig ist

Ein LLM-Reasoning ohne Tool-Symbiose-Check ist verschenktes Potenzial:
- Research mit nur `WebFetch` statt `firecrawl + perplexity + context7 + notebooklm` parallel = 5× langsamer + viel weniger Kontext
- Bug-Validation mit nur Self-Reasoning statt `agent-council + codex` Second-Opinion = höheres False-Positive-Risiko
- UI-Verify mit nur Code-Read statt `browse`-Browser-Test = nicht echt verifiziert
- Endpoint-Test mit nur Doku-Lesen statt curl-gegen-Live = keine Realitäts-Validierung

Der Skill verbrennt sonst Tokens auf naive Pfade während State-of-the-Art-Tools ungenutzt rumliegen.

---

## Standard-Sequenz vor jedem LLM-Call

### Schritt 1 — Inventory-Snapshot

```bash
# Skills
ls ~/.claude/skills/

# MCPs
claude mcp list

# Plugins
ls ~/.claude/plugins/ 2>/dev/null

# Globale CLI-Tools (was im PATH ist + relevant)
which gh n8n firecrawl curl
```

Skill cached das Inventory pro Session (Single Inventory-Run reicht — kein Re-Scan pro Step).

### Schritt 2 — Task-Matching

Was macht der aktuelle Schritt? Welche Tools sind dafür relevant?

Standard-Mapping (Beispiele — Liste in [#tool-mapping](#tool-mapping) unten):

| Schritt-Typ | Tool-Familie |
|---|---|
| Externe Doku scrapen | Web-Scrapers, Library-Doc-Tools |
| Code-Inspektion | Grep, Glob, Code-Graph, Subagents |
| Bug-Validation | Council, Codex, Investigate-Methodologie |
| UI-Verify | Browser-Tools |
| API-Test | curl, HTTP-MCPs |
| Persistenz | Obsidian, File-System |

### Schritt 3 — Symbiose-Plan

Wo möglich: **parallel**.

Beispiel Research-Schritt:
- `firecrawl` scrapt API-Doku-Hauptseite
- `perplexity_research` macht Hintergrund-Recherche zu Best-Practices
- `context7` lädt Library-Doku
- `notebooklm` konsolidiert in Notebook

All das in EINEM Tool-Call-Batch (mehrere Tool-Calls in einer Assistant-Message).

Beispiel Validation:
- `agent-council` für Multi-Model-Verdict
- `codex consult` für 2nd Opinion
- Self-Reasoning als 3. Stimme
→ 3 unabhängige Perspektiven → echte Validation

### Schritt 4 — Aktiv nutzen

Tools werden aufgerufen, nicht nur erwähnt. Skill macht im Output sichtbar welche Tools laufen:

```
🔧 Tool-Symbiose für diesen Schritt:
   - firecrawl_scrape (parallel) → API-Doku
   - perplexity_research (parallel) → Best-Practices
   - context7 (parallel) → Library-Specs
   - obsidian_create_file (post) → Persistenz
```

### Schritt 5 — Gap-Hinweis (nebenbei, einmalig)

Wenn ein State-of-the-Art-Tool fehlt das deutlich helfen würde:

```
💡 Gap-Hinweis: Mit `code-review-graph` MCP würde ich die Architektur-Map
   3× schneller bauen können. Optional: später installieren via npm i ...
   Ich arbeite weiter mit Grep+Glob+Read.
```

**Wichtig:** Gap-Hinweis nur EINMAL pro Session pro fehlendes Tool. Nicht nervig.

---

## Tool-Mapping

### Research (Phase 1 Part 1)

| Aufgabe | Primär | Sekundär | Tertiär |
|---|---|---|---|
| External API-Doku scrapen | `firecrawl_scrape`, `firecrawl_crawl` | `WebFetch` | `Bash` curl |
| Library/Framework-Docs | `context7` (resolve+query) | `WebFetch` zur offiziellen Doku-Seite | — |
| Web-Recherche (Best-Practices, Standards) | `perplexity_research` (deep) | `perplexity_search` | `WebSearch` |
| Notebook-Konsolidierung | `notebooklm` Add-Source + Research | — | Manuelle MD-Files |
| Social/Threads/Industrie-Talk | `scrape-creators` (Reddit, Twitter, HN) | `WebSearch` | — |
| Repo-internes Verständnis | `code-review-graph` (build+query) | `Explore`-Subagent | Glob+Grep+Read |
| Persistenz extern | `obsidian-mcp` create_file | File-System | — |
| Persistenz intern | `Write` in `.bug-sweep/research/` | — | — |

### Bug-Detective (Phase 1 Part 2)

| Aufgabe | Primär | Sekundär | Tertiär |
|---|---|---|---|
| Architektur-Mapping | `code-review-graph` (hub-nodes, bridge-nodes) | `Explore`-Subagent | Glob+Grep+Read |
| Bug-Pattern-Erkennung | `debug`-Methodologie, `investigate`-Methodologie | Self-Reasoning mit Pattern-Tabelle aus `audit-verify-loop` | — |
| Logging-Audit | Grep nach Log-Calls + Pattern-Analyse | `code-review-graph` find-large-functions | — |
| API-Realitäts-Check | `Bash` curl read-only | HTTP-MCPs | Browser-Test |
| Confidence-Validation | `agent-council` | `codex consult` | Self-Checkliste |

### Bug-Fix (Phase 2 Step 4)

| Aufgabe | Primär | Sekundär | Tertiär |
|---|---|---|---|
| Pre-Check (Holistik) | `code-review-graph` get_impact_radius | Grep nach Callsite + Read | — |
| Code-Edit | `Edit`, `Write` | — | — |
| Commit | `Bash` git + Conventional-Commit | `commit` Skill | — |
| Council-Pre-Check (optional) | `agent-council` | `codex consult` | — |

### Smoke-Test (Phase 2 Step 5)

| Bug-Typ | Primär | Sekundär |
|---|---|---|
| API | `Bash` curl mit Dummy-Payload | HTTP-MCPs |
| UI | `browse`-Skill, `playwright`-MCP, `claude-in-chrome`-MCP | `kapture`-MCP |
| DB | `supabase`-MCP execute_sql mit Test-Schema | direkter DB-Client |
| n8n-Workflow | `n8n-mcp` validate_workflow + manual execute mit Dummy | — |
| Backend-Logic | Unit-Test schreiben + ausführen | `Bash` Test-Runner |
| Live-Dashboard | `browse` gegen Staging | `playwright` |
| Mobile | `adb-android-control` | `mobile-use-setup` |

### Logging/Doku-Upgrade (Phase 2 Step 6)

| Aufgabe | Primär |
|---|---|
| Logging-Standards | `analytics-tracking` Skill als Reference + manuelle Anwendung |
| Code-Doku | `doc-coauthoring` Skill |
| README-Update | manuell Edit |

### Persistenz / Wissens-Basis (Cross-Cutting)

| Aufgabe | Primär |
|---|---|
| Lokal | `Write` in `.bug-sweep/` |
| Obsidian | `obsidian-mcp` (falls vorhanden), sonst `Write` direkt im Vault-Pfad |
| Git | `Bash` git + atomic commits |

---

## Symbiose-Patterns (Bewährte Kombinationen)

### Pattern A — Massiv-Research-Parallel

Wenn Phase 1 Part 1 läuft und ein externes API-System dokumentiert werden muss:

```
PARALLEL (ein Tool-Call-Batch):
- firecrawl_scrape(https://api.example.com/docs)
- firecrawl_scrape(https://api.example.com/changelog)
- perplexity_research("example.com API best practices 2026")
- context7.resolve-library-id("example-sdk") → context7.query-docs(...)
- WebSearch("example.com api rate limit gotchas")

DANN (sequential):
- Aggregation + Strukturierung in .bug-sweep/research/api-docs/example.md
- obsidian_create_file(Obsidian/projects/<proj>/.../example.md)
```

### Pattern B — Triple-Validation für Confidence

Wenn 100%-Confidence-Level erreicht werden soll:

```
1. Self-Reasoning mit Checkliste
2. agent-council → "Habe ich alles?"
3. codex consult → "Independent 2nd opinion auf diesen Plan"

Wenn alle 3 grün → Confidence 100%
Wenn 1+ rot → Lücke schließen, neu validieren
```

### Pattern C — Holistik-Pre-Check vor Fix

Vor jeder Code-Änderung:

```
PARALLEL:
- code-review-graph.get_impact_radius(<symbol>)
- Grep "<function-name>" über Repo
- Grep "<endpoint-pattern>" über Repo

DANN:
- Analyse welche Files indirekt betroffen
- Read der kritischen Callsites
- Pre-Test-Plan: was muss nach Fix noch funktionieren?
```

### Pattern D — Smoke-Test mit Cleanup

Nach Fix:

```
1. Sandbox-Setup (je Bug-Typ)
2. Dummy-Daten erstellen (mit Marker-Prefix __smoketest_*)
3. Reproduktion des Bug-Szenarios
4. Verify dass Fix greift
5. Cleanup (Grep nach __smoketest_*, alles löschen)
6. Cleanup-Verify (nochmal Grep → muss 0 Treffer haben)
7. Live-Version-Check dass nichts Test-Reste enthält
```

### Pattern E — Iterative Logging-Verbesserung

Pro Bug-Fix:

```
1. Vor Fix: was würde mich später retten wenn dieser Bug wiederkommt?
2. Logging an Schlüssel-Stellen einbauen (correlation-id, payload-size, timing)
3. Inline-Kommentar warum dieses Logging wichtig ist
4. README-Section updaten falls Domain-Knowledge

Ziel: bei nächstem ähnlichem Bug in 6 Monaten kann ein anderer Claude
in einer frischen Session den Bug in 5 Minuten finden statt 5 Stunden.
```

---

## Anti-Patterns (NIE so)

- ❌ "Ich habe `firecrawl` nicht, ist OK, ich nehme `WebFetch`" OHNE den Gap-Hinweis zu setzen
- ❌ Tools sequenziell wenn sie parallel könnten
- ❌ Tool-Inventory nicht checken weil "wird schon was da sein"
- ❌ Council aufrufen für jeden trivialen Schritt (nur bei echten Confidence-Gates)
- ❌ Tools auflisten aber nicht aufrufen
- ❌ Über Tools reden mit User statt sie zu nutzen

---

## Performance-Hinweis

- Inventory-Scan einmal pro Session cachen (kein Re-Run bei jedem Schritt)
- Parallel-Calls in EINEM Assistant-Message-Batch (5-10× schneller als sequenziell)
- Wenn Tools optional sind (Gap-Hinweis): nicht warten, weiter mit Fallback
- Bei großen Operations (z.B. `firecrawl_crawl` einer ganzen Doku-Site): im Background laufen lassen, Token-Spar
