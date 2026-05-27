# Debug-Journal — `<projekt-name>`

**Pfad:** `.bug-sweep/debug-journal.md`
**Zweck:** Session-übergreifendes Journal aller safe-debug-loop-Sessions für dieses Modul. Wächst über Zeit.

---

## Wofür dieses Journal

- Zukünftige Claude-Sessions können hier nachschauen was schon gefixt wurde
- Recurring Patterns werden sichtbar (gleicher Bug-Typ tritt immer wieder auf → architektonisches Smell)
- Logging-/Doku-Fortschritt ist nachvollziehbar
- Bei neuen Bugs: Cross-Reference auf ähnliche frühere Bugs spart Investigations-Zeit

---

## Session-Index

| # | Datum | Phase | Bugs adressiert | Status | Bug-Plan-File |
|---|---|---|---|---|---|
| 1 | `<YYYY-MM-DD>` | 1+2 | 24 (7R/12Y/5G) | ✅ done | `bug-plan-<iso>.md` |
| 2 | `<YYYY-MM-DD>` | 2 (adhoc) | 3 | ✅ done | `adhoc-bugs-<iso>.md` |
| 3 | `<YYYY-MM-DD>` | 1+2 | 8 (2R/4Y/2G) | 🔄 in progress | `bug-plan-<iso>.md` |
| ... | | | | | |

---

## Session #1 — `<YYYY-MM-DD>` — Phase 1+2 (Initial Sanierung)

**Modul/Bereich:** `<beschreibung>`
**Trigger:** "<was der User sagte / Anlass>"
**Dauer:** ~`<h>` h
**Tool-Stack-Snapshot:** `<welche Skills/MCPs/Plugins waren installiert>`

### Phase 1 Ergebnisse
- Research-Files erstellt: `<liste>`
- Bug-Plan: `<filepath>`
- Confidence-Level: 100% Council-validated / self-asserted
- Bugs gefunden: N (X 🔴 / Y 🟡 / Z 🟢)

### Phase 2 Ergebnisse
- Bugs gefixt: M von N
- Davon obsolet (bereits gefixt): K
- Davon blockiert: L (Grund: ...)
- Smoke-Tests passed: M
- Atomic Commits: M
- Logging-Upgrades in: <K Bereiche>
- Doku-Upgrades in: <J Files>

### Key Learnings dieser Session
- **<Lerneffekt 1>:** Beobachtung + warum wichtig für Zukunft
- **<Lerneffekt 2>:** ...

### Recurring Patterns (Achtung!)
- **<Pattern X>:** Tritt schon zum N-ten Mal auf → architektonisches Refactor empfohlen
- ...

### Open Items für nächste Sessions
- [ ] <was nicht geschafft wurde + warum>
- [ ] <neue Erkenntnis die in Phase 1 #2 berücksichtigt werden sollte>

---

## Session #2 — `<YYYY-MM-DD>` — Phase 2 (Adhoc)

(gleiche Struktur, kürzer wenn nur Adhoc)

---

## Cross-Session Findings

### Persistent Architectural Smells

Patterns die sich über mehrere Sessions ziehen:

- **<Smell 1>:** <beschreibung> — empfohlen: <refactor / monitor / accept>
- **<Smell 2>:** ...

### Logging-Reifegrad

Wo steht das Logging im Modul jetzt:

- ✅ <Bereiche mit gutem strukturiertem Logging>
- 🟡 <Bereiche mit teilweisem Logging>
- ❌ <Bereiche ohne sinnvolles Logging — Priorität für nächste Session>

### Doku-Reifegrad

- ✅ <gut dokumentierte Bereiche>
- 🟡 <teil-dokumentierte Bereiche>
- ❌ <undokumentierte Bereiche — Doku-Schulden>

### Tool-Coverage-Lücken

Tools die mehrfach gefehlt haben:

- `<tool x>` (mehrfach Gap-Hinweis gewesen) → empfohlen zu installieren
- ...

---

## Zukunfts-Roadmap (vom Skill vorgeschlagen)

Basierend auf den Findings über alle Sessions:

1. **<vorschlag 1>** — z.B. "Migrate API-Calls von v1 zu v2 (Doku-Mismatch in 4 Sessions hintereinander)"
2. **<vorschlag 2>** — z.B. "Strukturiertes Logging-Framework einführen (manuelle Log-Calls sind ineffizient)"
3. **<vorschlag 3>** — ...

---

## Quick-Reference für zukünftige Sessions

Wenn jemand (Claude oder Mensch) in 3 Monaten dieses Modul debuggt:

- **Modul-Übersicht:** `.bug-sweep/research/module-overview.md`
- **API-Dependencies:** `.bug-sweep/research/api-docs/`
- **Bekannte Edge-Cases:** `.bug-sweep/research/gotchas.md`
- **Logging-Konventionen:** siehe Logging-Reifegrad oben
- **Recurring Issues:** siehe Persistent Architectural Smells oben
