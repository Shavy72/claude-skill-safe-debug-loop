# Phase 1 — Research-Forscher + Detective-Abgleich

**Vollautonom. Kein produktiver Code wird angefasst. Ergebnis ist eine intensive Lösungs-MD-File + ein Start-Prompt für Phase 2.**

Phase 1 läuft komplett ohne User-Interaktion ab — sobald der Skill genug Kontext hat (Modul, Pfad, ggf. Live-URL), arbeitet er autonom durch bis zum Output.

---

## Voraussetzungen vor Phase-1-Start

Der Skill prüft und stellt sicher:

1. **Aktuelles Arbeitsverzeichnis ist das Projekt-Repo.** Wenn nicht: User fragen wo das Modul liegt.
2. **Modul-Scope ist klar.** Welche Datei(en), welche Sektion des Dashboards, welcher Pfad in der App. Bei Unklarheit: kurze Rückfrage an User (1× gebündelt, dann autonom).
3. **`.bug-sweep/`-Ordner existiert oder wird angelegt.**
4. **Prerequisites-Check** ([prerequisites.md](prerequisites.md)) hat einen Status-Report produziert. Bei kritischen Lücken: einmaliger Hinweis an User, dann mit dem arbeiten was da ist.
5. **Effort-Mode ist auf MAX gestellt.** Falls nicht: User-Hinweis `/effort max` auszuführen.

---

## Part 1 — Radikale Wissens-Erweiterung

### Persona

> **"Du bist ein radikaler Nachforscher auf Kokain und Ritalin. Du hörst nicht auf bevor du JEDE kleinste mögliche nützliche Info und jeden noch so entfernt relevanten Kontext gesammelt und session-übergreifend persistent gemacht hast. Hyperfokus, kein Schlaf, kein Pause-Modus."**

### Tool-Symbiose-Plan für Part 1

Vor dem Start: Inventory + Symbiose. Typische parallele Tool-Stacks (Beispiel — nutze was beim User installiert ist):

| Aufgabe | Tool-Symbiose (parallel) |
|---|---|
| Externe API-Docs scrapen | `firecrawl` + `perplexity` + `context7` + `notebooklm` + `scrape-creators` |
| Library-Docs | `context7` (mit `resolve-library-id` → `query-docs`) |
| Web-Recherche / aktuelle Standards | `perplexity_research` + `perplexity_search` |
| Repo-interne Code-Analyse | `code-review-graph` + Glob/Grep + `Explore`-Subagent |
| Endpoint-Inventar | `Bash` (curl/HTTP-Discovery) + Code-Grep + Postman/OpenAPI-Files |
| Persistenz | `obsidian-mcp` (falls da) + lokale Files in `.bug-sweep/research/` |

Wenn Tool fehlt: einmal Hinweis ("FYI — mit `firecrawl` würde das hier 3× schneller gehen"), dann mit `WebFetch`/`WebSearch` weitermachen.

### Konkrete Research-Pipeline

```
1. Modul-Code-Inventory
   - Welche Files? Welche Komponenten? Welche Endpoints? Welche Stores/State?
   - Output: .bug-sweep/research/module-overview.md
   - Tools: Glob, Grep, Read, code-review-graph

2. Abhängigkeits-Discovery (Dependencies)
   - Welche externen APIs werden gerufen? Welche internen Services?
   - Welche npm/pip-Packages spielen eine Rolle?
   - Welche Env-Vars werden genutzt?
   - Output: .bug-sweep/research/dependencies.md

3. Externe-API-Doku-Scrape (parallel pro API)
   - Komplette aktuelle Doku jeder genutzten API
   - Auch Doku zu Endpoints die NUR im Ansatz relevant sein könnten
   - Output: .bug-sweep/research/api-docs/<api-name>.md
   - Tools: firecrawl/perplexity/context7 parallel

4. Endpoint-Schemas + Beispiel-Payloads
   - Für jeden externen Call: Request-Schema, Response-Schema, Error-Cases, Rate-Limits
   - Beispiel-Payloads aus der Doku oder via API-Test (curl)
   - Output: .bug-sweep/research/endpoints.md

5. Bekannte Edge-Cases + Gotchas
   - Was steht in den API-Docs unter "common errors", "limitations", "breaking changes"?
   - Aktuelle Standards / Best-Practices für diese Domain (perplexity_research)
   - Output: .bug-sweep/research/gotchas.md

6. Internes Logging-Audit
   - Was wird aktuell geloggt? Wo? Auf welchem Level?
   - Output: .bug-sweep/research/logging-status.md

7. Obsidian-Spiegelung (wenn Vault verfügbar)
   - Alle Research-Files spiegeln in Obsidian/projects/<projekt>/safe-debug-loop/research/
   - Damit Wissen über Projekt-Grenzen hinweg verfügbar bleibt
```

### Stop-Condition Part 1

**100% Confidence Level erreicht?**

Validation via `agent-council` (wenn installiert) — Prompt sinngemäß:
> "Ich habe folgendes Research-Set produziert: [Liste]. Habe ich JEDE relevante Info? Was fehlt noch? Gibt es Lücken die einen Bug-Detective später behindern würden?"

Bei Council-Feedback "Lücke X": zurück zu Part 1 mit Fokus auf X. Wiederholen bis Council "complete" sagt.

Fallback wenn Council fehlt: Self-Reasoning + Codex-Consult-Check (falls `codex` installiert) + explizite Checkliste:
- [ ] Alle externen APIs dokumentiert?
- [ ] Alle Endpoints mit Schema?
- [ ] Edge-Cases je API?
- [ ] Aktuelle Standards/Best-Practices?
- [ ] Logging-Status?
- [ ] Dependencies-Map?

**Sobald 100% Confidence → automatischer Übergang zu Part 2.** Kein User-Input nötig.

---

## Part 2 — Radikaler Abgleich (Detective)

### Persona

> **"Du bist der weltbeste Geheimdienst- und Polizei-Behörden-Fahnder. Du gleichst akribisch den aktuellen Code-Stand des Moduls ab mit allem Wissen aus Part 1 und findest JEDEN noch so kleinen Bug. Nichts entgeht dir. Du suchst auch nach Dingen die NICHT da sind aber da sein sollten (fehlende Calls, fehlendes Logging, fehlende Error-Handler)."**

### Tool-Symbiose-Plan für Part 2

| Aufgabe | Tool-Symbiose (parallel) |
|---|---|
| Code-Inspektion + Pattern-Search | Grep + Glob + Read + `Explore`-Subagent |
| Bug-Pattern-Erkennung | `debug` + `investigate` + `audit-verify-loop` (als Methoden-Inspiration, nicht als Auto-Run) |
| Architektur-Verständnis | `code-review-graph` (Hub-Nodes, Bridge-Nodes, Communities) |
| Doku-Mismatch-Detection | Vergleich Code-Realität ↔ `.bug-sweep/research/` |
| Endpoint-Validation | `Bash` curl gegen tatsächliche Endpoints (read-only Calls, kein State-Change!) |
| 100%-Confidence-Validation | `agent-council` + `codex` |

### Konkrete Detective-Pipeline

```
1. Code-Realität vs. API-Doku-Abgleich
   - Pro externem Call: Schema-Match? Response-Handling vollständig?
   - Fehlerhafte/veraltete Endpoints? Deprecated-Felder?

2. Internal-Flow-Abgleich
   - State-Transitions konsistent? Fehlende Loading-/Error-States?
   - Race-Conditions möglich? Stale-Data-Risiken?

3. Logging-Gap-Analyse
   - Wo fehlt Logging das man im Bug-Fall bräuchte?
   - Inkonsistente Log-Levels? Fehlende Korrelations-IDs?

4. Doku-Mismatch-Detection
   - Code-Kommentare die nicht mehr stimmen?
   - README / Inline-Doku ↔ tatsächliches Verhalten?

5. Dead-Code / Code-Smells
   - Ungenutzte Imports/Functions?
   - Copy-Paste-Duplikate?
   - Magic Numbers ohne Erklärung?

6. Fehl-Calls / Wrong Arguments
   - Funktions-Calls mit falschen Parametern?
   - APIs mit deprecated Endpoints/Headern?

7. Error-Handling-Gaps
   - Try-Catch-Blöcke die nichts tun?
   - Unhandled Promise-Rejections?
   - Generic-Error-Messages wo spezifisch sein sollten?
```

### Kategorisierung jedes Bugs

| Kategorie | Wann | Beispiel |
|---|---|---|
| 🔴 **Rot — Dringend** | Bug bricht Hauptfunktion / verhindert MVP / Sicherheits-relevant / Daten-Korruption-Risiko | Submit-Button macht nichts, Auth-Token wird im LocalStorage gespeichert, DB-Query ohne Sanitization |
| 🟡 **Gelb — Wichtig** | Bug stört User-Experience / führt zu Konfusion / verhindert Edge-Cases | Loading-State fehlt, Error-Toast zeigt generischen Text statt konkreter Info, Rate-Limit-Handler fehlt |
| 🟢 **Grün — Vernachlässigbar** | Cosmetic / Code-Quality / nice-to-have | Inkonsistentes Naming, fehlende JSDoc, Magic Number ohne Konstante |

### Bug-Plan-Erstellung

Pro Bug wird ein Eintrag im Bug-Plan-MD geschrieben. Template: [../templates/bug-plan.md](../templates/bug-plan.md).

Pro Eintrag MIN:
- ID + Titel
- Kategorie (Rot/Gelb/Grün)
- Symptom (was beobachtet)
- Vermutete Root-Cause (Datei + Zeile wenn möglich)
- Fix-Plan (was tun, minimal-invasiv)
- Abhängigkeiten (welche anderen Bugs müssen vorher gefixt sein?)
- Blast-Radius (welche anderen Stellen könnten betroffen sein vom Fix?)
- Smoke-Test-Strategie (Vorschlag, Detail-Strategie in Phase 2)
- Logging-/Doku-/Kommentar-Verbesserungen die mit-fixed werden sollten

### Stop-Condition Part 2

**100% Confidence Level: alle Bugs gefunden + jeder Plan macht 100% Sinn?**

Validation via `agent-council` — Prompt sinngemäß:
> "Hier ist mein Bug-Plan: [Liste]. Habe ich JEDEN relevanten Bug gefunden? Macht der Plan pro Bug 100% Sinn? Sind die Abhängigkeiten korrekt? Ist der Blast-Radius realistisch eingeschätzt?"

Bei Council-Feedback "Lücke X" oder "Plan Y unklar": zurück zu Detective-Pipeline. Wiederholen bis 100%.

---

## Output von Phase 1

### 1. Bug-Plan-MD

`<repo>/.bug-sweep/bug-plan-<ISO-date>.md`

Struktur siehe [../templates/bug-plan.md](../templates/bug-plan.md). Beispiel-Anfang:

```markdown
# Bug-Plan — <modul-name> — <ISO-Date>

**Modul:** <pfad>
**Live-URL (wenn vorhanden):** <url>
**Research-Quellen:** .bug-sweep/research/
**Created by:** safe-debug-loop Phase 1
**Confidence-Level:** 100% (Council-validated)

## Bug-Summary
- 🔴 Rot: 7 Bugs
- 🟡 Gelb: 12 Bugs
- 🟢 Grün: 5 Bugs

## Empfohlene Fix-Reihenfolge
1. Bug #R1 (Root-Bug, alle anderen indirekt betroffen)
2. Bug #R2
...

---

## 🔴 R1 — <bug-titel>
- **Symptom:** ...
- **Root-Cause:** ...
- **Fix-Plan:** ...
- **Blast-Radius:** ...
- **Smoke-Test:** ...
- **Status:** [ ] open
```

### 2. Start-Prompt für Phase 2 (im Chat ausgegeben)

Der Skill druckt am Ende einen Copy-Paste-Block an den User:

```
╔══════════════════════════════════════════════════════════╗
║  PHASE 1 ABGESCHLOSSEN                                   ║
║  Bug-Plan: .bug-sweep/bug-plan-<ISO>.md                  ║
║  Confidence: 100% (Council-validated)                    ║
║  Found: 7 rot / 12 gelb / 5 grün                         ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  COPY-PASTE in deine Phase-2-Session:                    ║
║                                                          ║
║  ─────────────────────────────────────────────────────   ║
║  /safe-debug-loop Phase 2                                ║
║                                                          ║
║  Bug-Plan: .bug-sweep/bug-plan-<ISO>.md                  ║
║  Research: .bug-sweep/research/                          ║
║                                                          ║
║  Bitte arbeite den Plan iterativ durch — einen Bug nach  ║
║  dem anderen, mit Smoke-Test pro Fix. Holistik-Check     ║
║  vor jedem Fix.                                          ║
║  ─────────────────────────────────────────────────────   ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

User kopiert den eingerahmten Block in seine Phase-2-Session.

---

## Was Phase 1 NICHT tut

- ❌ Code ändern
- ❌ Files schreiben außer in `.bug-sweep/` und (optional) Obsidian-Vault
- ❌ Commits machen (außer ggf. ein Commit der `.bug-sweep/`-Artefakte)
- ❌ Live-Endpoints mit State-Change-Calls treffen (nur read-only Calls erlaubt)
- ❌ User mit Zwischenfragen löchern (nur 1× bündeln am Anfang wenn nötig)

---

## Edge-Cases + Eskalation

- **Repo zu groß für 100%-Research:** Skill priorisiert nach Code-Review-Graph-Hub-Nodes → fokussiert auf High-Impact-Bereich. Hinweis an User dass Scope reduziert wurde.
- **Externe API nicht erreichbar / Cloudflare:** Hinweis an User, Fallback auf gecachte Docs oder gleich `WebFetch` mit Browser-Headers. Bei harten Blocks: User-Handoff.
- **Council nicht installiert + Codex nicht installiert:** Skill macht strukturierte Self-Validation per Checkliste + warnt User dass Confidence-Level "self-asserted" ist statt "Council-validated".
- **Endlosschleife in 100%-Confidence-Suche:** Hard-Limit auf 3 Iterations pro Confidence-Round. Danach: an User mit konkreter Frage was noch fehlt.
