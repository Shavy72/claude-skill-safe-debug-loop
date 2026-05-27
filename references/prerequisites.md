# Prerequisites — Required + Recommended Tools

**Vor jedem Phase-Start: Check + Status-Report + Gap-Hinweise.**

Der Skill arbeitet mit dem was DA ist und gibt Hinweise was zusätzlich nützlich wäre. Hier ist die strukturierte Übersicht.

---

## Required (kritisch)

Diese Tools müssen vorhanden sein damit der Skill seinen Kern erfüllen kann. Bei Fehlen: Skill verweigert den Start mit klarem Hinweis was zu installieren ist.

| Tool | Was es macht | Install |
|---|---|---|
| Claude Code (CLI) | Host-Umgebung | https://claude.com/claude-code |
| Git | Versionskontrolle für atomic commits | `winget install Git.Git` (Windows) |
| Filesystem-Zugang | `.bug-sweep/` Persistenz | (kommt mit OS) |

Wenn diese fehlen → Skill kann nicht arbeiten. Eskalation an User mit Install-Hinweisen.

---

## Strongly Recommended (Skill läuft viel besser damit)

Diese Tools sollten installiert sein für solide Skill-Performance. Bei Fehlen: einmaliger Hinweis + Fallback aktivieren.

| Tool | Rolle | Fallback wenn fehlt |
|---|---|---|
| **`agent-council` Skill** | 100%-Confidence-Validation in Phase 1 + Smoke-Test-Verdict | `codex` Skill als 2nd opinion ODER Self-Reasoning mit Checkliste (Warnung: "self-asserted Confidence") |
| **`firecrawl` MCP** | Massiv-Parallel-Doc-Scraping in Phase 1 Part 1 | `WebFetch` (langsamer, weniger structured) |
| **`perplexity` MCP** | Web-Research + Best-Practices-Lookup | `WebSearch` + manuelles Aggregieren |
| **`context7` MCP** | Library-Doc-Lookup (npm/pypi-Packages) | `WebFetch` zur offiziellen Doku-Seite |
| **`code-review-graph` MCP** | Architektur-Mapping, Hub/Bridge-Nodes, Impact-Radius | Glob+Grep+Read manuell (langsamer, weniger holistisch) |
| **`browse` Skill (gstack)** | UI-Smoke-Tests | `playwright`-MCP / `claude-in-chrome`-MCP / `kapture`-MCP |
| **`obsidian-mcp` ODER lokaler Vault** | Session-übergreifende Wissens-Basis | nur `.bug-sweep/` lokal (kein Cross-Projekt-Lookup) |

### Install-Hinweise

#### agent-council Skill
```bash
# Falls noch nicht im ~/.claude/skills/ vorhanden
git clone https://github.com/<source>/agent-council ~/.claude/skills/agent-council
```

#### firecrawl MCP
```bash
claude mcp add firecrawl <command>
# Details: https://docs.firecrawl.dev/mcp
```

#### perplexity MCP
```bash
claude mcp add perplexity <command>
# Erfordert PERPLEXITY_API_KEY
```

#### context7 MCP
```bash
claude mcp add context7 <command>
# https://github.com/upstash/context7
```

#### code-review-graph MCP
```bash
claude mcp add code-review-graph <command>
```

---

## Recommended per Bug-Typ (gut zu haben)

Diese Tools verbessern Smoke-Tests + Symbiosen je nach Bug-Domain. Bei Fehlen: Gap-Hinweis, kein Block.

### Web-/Dashboard-Bugs
- `browse` Skill (gstack) — schnellste Browser-Tests
- `playwright`-MCP — voller Browser-Stack
- `claude-in-chrome`-MCP — Live-Chrome-Steuerung
- `kapture`-MCP — Page-Scraping + DOM-Analyse
- `deep-page-scan` Skill — strukturierte Page-Analyse

### Backend/API-Bugs
- `Bash` + `curl` (Standard, immer da)
- HTTP-Test-MCPs (falls vorhanden)
- `supabase`-MCP (wenn Supabase im Stack)

### DB-Bugs
- `supabase`-MCP (Supabase-Branches für Test-Schemas!)
- Direkte DB-Clients über `Bash`

### n8n-Workflows
- `n8n-mcp` MCP — strukturelle Workflow-Validierung + Schema-Lookup
- `n8n-mastery` Skill — User-spezifische n8n-Patterns

### Mobile-Bugs
- `adb-android-control` Skill — Android-Device-Control
- iOS Simulator (extern)

### Build/Deploy/CI
- `gh` CLI — GitHub-Actions-Logs
- Docker / Container-Engine
- `n8n-deploy` Skill wenn n8n-relevant

---

## Optional (Quality-of-Life)

| Tool | Rolle |
|---|---|
| `codex` Skill | 2nd-Opinion-Pattern, Adversarial-Mode |
| `discovery-first` Skill | Pre-Task-Lookup-Discipline |
| `handoff` Skill | Session-Übergänge sauber managen |
| `commit` Skill | Conventional-Commit-Helper |
| `verify` Skill | Standard-Verify-Patterns |
| `qa` Skill | QA-Loop-Patterns |
| `notebooklm` MCP | Wissens-Konsolidierung |
| `scrape-creators` MCP | Industrie-Talk / Reddit / Twitter scrapen für Edge-Case-Discovery |

---

## Prerequisites-Check-Output (Beispiel)

Bei jedem Phase-Start zeigt der Skill diesen Status-Block:

```markdown
🔧 Prerequisites-Check für safe-debug-loop

✅ Required
- Claude Code: ✓
- Git: ✓
- Filesystem: ✓

✅ Strongly Recommended (5/6 verfügbar)
- agent-council: ✓
- firecrawl: ✓
- perplexity: ✓
- context7: ✓
- code-review-graph: ✓
- browse: ✓
- obsidian-mcp: ✗ → Fallback: nur .bug-sweep/ lokal (Cross-Projekt-Lookup deaktiviert)
  💡 Install-Hinweis: claude mcp add obsidian-mcp <...>

🟢 Bug-Type-Coverage
- Web/UI: ✓ (browse + playwright)
- API: ✓ (Bash + curl)
- DB: ✓ (supabase)
- n8n: ✓ (n8n-mcp)
- Mobile: ✗ (adb-android-control fehlt) → falls Mobile-Bugs auftauchen: Gap-Hinweis

✅ Effort-Mode: max
✅ Repo: .bug-sweep/ wird angelegt

→ Skill startet Phase <N> ...
```

---

## Was passiert wenn Tools fehlen

### Kritische Lücke (Required fehlt)
- Skill **stoppt sofort**
- Klarer Hinweis was fehlt
- Install-Command wenn bekannt
- User soll installieren + neu starten

### Wichtige Lücke (Strongly Recommended fehlt)
- Skill **läuft mit Fallback**
- Einmaliger Hinweis am Anfang
- Bei Confidence-Validation explizit "self-asserted" markieren statt "Council-validated"
- Skill schlägt vor: "Wenn du nach dieser Session Zeit hast: X installieren, beim nächsten Mal Y× besser"

### Optionale Lücke (Bug-Type-Coverage)
- Skill läuft normal
- Wenn passender Bug auftaucht: Gap-Hinweis dass dieser Bug schwer verifizierbar wird ohne Tool X
- Optional: Vorschlag den Bug zu deferren bis Tool installiert ist

---

## Skill-Self-Maintenance

Der Skill aktualisiert seine eigene Prerequisites-Liste nicht automatisch — das passiert manuell. Aber er gibt am Ende einer Session einen kompakten Report:

```
📊 Skill-Performance-Report dieser Session

Tools genutzt:
- firecrawl (15× parallel-scrapes) ✓
- perplexity (8× research) ✓
- agent-council (3× confidence-checks) ✓
- browse (12× smoke-tests) ✓
- code-review-graph (5× impact-checks) ✓

Tools die gefehlt haben:
- obsidian-mcp → Wissensbasis nur lokal, kein Cross-Projekt-Sync

Empfehlung für nächstes Mal:
- obsidian-mcp installieren für Cross-Session-Wissens-Persistenz
```
