# safe-debug-loop

**Multi-Session Claude-Skill für die strategische Sanierung komplexer Dashboard-/App-Module mit zu vielen Bugs für einen MVP-Launch.**

Statt ad-hoc-Fixes baust du strategisch: erst Wissen + Bug-Inventar (Phase 1, eine Session), dann iterativ + safe fixen mit isoliertem Smoke-Test pro Bug (Phase 2, zweite Session). Jeder Loop-Durchlauf verbessert auch Logging, Doku und Code-Kommentare — damit zukünftiges Debugging session-übergreifend immer leichter wird.

---

## Wann brauchst du diesen Skill

Wenn dein Modul gebaut ist — UI/UX steht, Programm-Flow steht, Backend hängt an mehreren APIs, vielleicht ist es schon live — **aber es sind so viele Bugs drin dass du nicht launchen kannst.** Jeder ad-hoc-Fix könnte indirekt andere Stellen brechen (Multi-API-Vernetzung, geteilter State, komplexe Abhängigkeiten).

**Klassische Trigger-Phrasen:**
- "ich kann das nicht launchen so"
- "zu viele Bugs für Beta"
- "läuft im Kreis"
- "Modul ist live aber kaputt"
- "MVP-Launch unmöglich"

**Was der Skill NICHT macht:**
- Hauptfunktionalität bauen (das Modul muss schon stehen)
- Einzelne hartnäckige Bugs fixen (dafür: `deep-debugger` / `investigate`)
- Quick-Audit "läuft das?" (dafür: `audit-verify-loop`)

---

## Wie er funktioniert

### 3 Aufruf-Modi

| Aufruf | Was passiert |
|---|---|
| `/safe-debug-loop` | Onboarding: Skill erklärt Workflow + gibt 2 Copy-Paste-Console-Start-Commands für 2 Sessions raus |
| `/safe-debug-loop Phase 1` | Direkt rein in Phase 1 (in frischer MAX-effort-Session) |
| `/safe-debug-loop Phase 2` | Direkt rein in Phase 2 (frische Session ODER mitten in aktivem Chat) |

### Phase 1 — Research-Forscher + Detective (vollautonom, KEIN Code-Touch)

**Part 1: Radikale Wissens-Erweiterung**
- Persona: Forscher auf Kokain + Ritalin
- Saugt jedes relevante Wissen auf: Modul-Code, externe API-Docs (komplett scrapen), Endpoints, Schemas
- Nutzt Tool-Symbiose: alle installierten Skills + MCPs + Plugins parallel
- Persistiert: `.bug-sweep/research/` + Obsidian-Vault (falls vorhanden)
- Stop-Condition: 100% Confidence Level (validiert via `agent-council`)

**Part 2: Radikaler Abgleich**
- Persona: Weltbester Geheimdienst-/Polizei-Fahnder
- Vergleicht aktuellen Modul-Code mit gesammeltem Wissen
- Findet ALLE Bugs, kategorisiert 🔴 Rot / 🟡 Gelb / 🟢 Grün
- Erstellt vollständigen Bug-Plan mit Fix-Reihenfolge + Blast-Radius

**Output Phase 1:**
- `.bug-sweep/bug-plan-<ISO>.md` mit kompletter Bug-Liste + Plan pro Bug
- Start-Prompt für Phase 2 zum Copy-Pasten

### Phase 2 — Iterativer Bug-Fix-Loop mit Smoke-Test

Pro Bug:
1. **Tool-Symbiose-Check** — welche Skills/MCPs helfen jetzt?
2. **Bug-Auswahl + Existenz-Check** — Welcher Bug + besteht er noch? (Wenn schon gefixt → markieren + nächster)
3. **12-Jährigen-Erklärung** — Bug einfach erklären, warum global problematisch, warum Fix minimal-invasiv
4. **Fix-Ausführung** — Pre-Check (Holistik) + Edit + atomic commit
5. **Isolated Smoke-Test** — Sandbox originalgetreu, Dummy-Daten, Reproduce, Verify, Cleanup
6. **Bug abhaken + Smoke-Test-Doku**
7. **Logging/Doku/Kommentar-Upgrade** für diesen Bereich
8. **`/compact`-Vorschlag** → nächster Bug

Loop läuft bis Plan komplett abgearbeitet.

---

## Installation

### Als globaler Claude-Code-Skill

Skill wird in `~/.claude/skills/safe-debug-loop/` installiert und ist dann in JEDER Claude-Code-Session verfügbar.

**Klon-Variante (empfohlen für Updates):**
```bash
git clone https://github.com/Shavy72/claude-skill-safe-debug-loop ~/.claude/skills/safe-debug-loop
```

**Manuelle Kopie:**
- Verzeichnis `safe-debug-loop/` aus diesem Repo nach `~/.claude/skills/` kopieren.

**Verifizieren:**
```bash
ls ~/.claude/skills/safe-debug-loop/
# Sollte zeigen: SKILL.md, references/, templates/, README.md, LICENSE
```

In Claude Code testen: einfach `/safe-debug-loop` aufrufen — der Skill triggert.

### Prerequisites

**Required:**
- Claude Code CLI
- Git
- Filesystem-Zugang

**Strongly Recommended (für volle Skill-Power):**
- `agent-council` Skill (für 100%-Confidence-Validation)
- `firecrawl` MCP (für massiv-parallel Doc-Scraping)
- `perplexity` MCP (für Web-Research)
- `context7` MCP (für Library-Docs)
- `code-review-graph` MCP (für Architektur-Mapping)
- `browse` Skill (gstack) ODER `playwright`-MCP (für UI-Smoke-Tests)
- `obsidian-mcp` ODER lokaler Vault (für Cross-Projekt-Wissensbasis)

Skill funktioniert auch ohne — gibt aber Gap-Hinweise und fällt auf langsamere Fallbacks zurück.

---

## Persistenz-Struktur

Im Projekt-Repo legt der Skill an:

```
.bug-sweep/
├── research/                    # Phase-1-Wissensbasis
│   ├── module-overview.md
│   ├── api-docs/
│   ├── endpoints.md
│   └── dependencies.md
├── bug-plan-<ISO>.md            # Phase-1-Output
├── adhoc-bugs-<ISO>.md          # Phase-2-Standalone-Bugs
├── smoke-tests/
│   └── <bug-id>-smoke-log.md    # Per-Bug-Smoke-Test-Doku
└── debug-journal.md             # Session-übergreifend
```

Wenn Obsidian-Vault verfügbar, wird das `research/` zusätzlich gespiegelt unter `Obsidian/projects/<projekt>/safe-debug-loop/`.

---

## Beispiel-Workflow

```
[Terminal 1]
$ cd ~/projekte/dashboard-modul
$ claude
> /effort max
> /safe-debug-loop Phase 1

→ Skill arbeitet autonom ~20-90 Min je nach Modul-Größe
→ Ergebnis: .bug-sweep/bug-plan-2026-05-27.md mit 24 Bugs
→ Start-Prompt für Phase 2 wird ausgegeben

[Terminal 2 — parallel]
$ cd ~/projekte/dashboard-modul
$ claude
> /effort max
> /safe-debug-loop Phase 2

→ User kopiert Start-Prompt aus Terminal 1 rein
→ Skill startet Loop für ersten Bug
→ User durchläuft pro Bug 3 Copy-Paste-Prompts + Smoke-Test
→ Pro Bug ein atomic Commit
→ Nach jedem Bug: /compact-Vorschlag

→ Nach N Bugs: Modul ist MVP-tauglich
```

---

## Iron Laws

1. **Phase 1 ändert KEINEN produktiven Code.** Nur `.bug-sweep/` und Obsidian-Vault wachsen.
2. **Vor jedem LLM-Call: Tool-Symbiose-Check.** Alle installierten Skills + MCPs + Plugins in Symbiose nutzen.
3. **Pro Bug: 1 Loop. 1 Smoke-Test. 1 atomic Commit.** Niemals mehrere Bugs parallel.
4. **Holistik bei jedem Schritt.** Kein Fix darf andere funktionierende Stellen brechen.
5. **Logging + Doku iterativ ausbauen.** Bei jedem Bug wird auch Logging/Doku besser.
6. **Cleanup-Pflicht.** Nach Smoke-Test alle Dummy-Daten + Test-Reste raus.

---

## Wann eskalieren

- Smoke-Test 3× nicht bestanden → User-Handoff mit konkreten Optionen
- Blast-Radius >5 Files für 1 Bug → User-Confirm holen
- Security-sensitive Change unsicher → STOP, User entscheidet

---

## Pflege + Beiträge

Skill wird unter [Shavy72/claude-skill-safe-debug-loop](https://github.com/Shavy72/claude-skill-safe-debug-loop) gepflegt.

Issues + PRs willkommen.

---

## Lizenz

MIT. Siehe [LICENSE](LICENSE).
