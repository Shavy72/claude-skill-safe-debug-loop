# Prerequisites — Required + Recommended Tools

**Before every phase start: check + status report + gap hints.**

The skill works with what IS there and points out what else would be useful. Here is the structured overview.

---

## Required (critical)

These tools must be present for the skill to fulfil its core purpose. If they are missing the skill refuses to start with a clear note on what to install.

| Tool | What it does | Install |
|---|---|---|
| Claude Code (CLI) | Host environment | https://claude.com/claude-code |
| Git | Version control for atomic commits | `winget install Git.Git` (Windows) |
| Filesystem access | `.bug-sweep/` persistence | (comes with the OS) |

If these are missing → the skill cannot work. Escalate to the user with install hints.

---

## Strongly recommended (the skill runs much better with these)

These tools should be installed for solid skill performance. If they are missing: one note plus an activated fallback.

| Tool | Role | Fallback if missing |
|---|---|---|
| **`agent-council` skill** | 100% confidence validation in Phase 1 plus the smoke-test verdict | The `codex` skill as a second opinion OR self-reasoning with a checklist (warning: "self-asserted confidence") |
| **`firecrawl` MCP** | Massively parallel doc scraping in Phase 1 Part 1 | `WebFetch` (slower, less structured) |
| **`perplexity` MCP** | Web research plus best-practice lookups | `WebSearch` plus manual aggregation |
| **`context7` MCP** | Library doc lookup (npm/PyPI packages) | `WebFetch` against the official docs page |
| **`code-review-graph` MCP** | Architecture mapping, hub/bridge nodes, impact radius | Glob+Grep+Read manually (slower, less holistic) |
| **`browse` skill (gstack)** | UI smoke tests | `playwright` MCP / `claude-in-chrome` MCP / `kapture` MCP |
| **A notes-vault MCP OR a local vault** | Cross-session knowledge base | `.bug-sweep/` locally only (no cross-project lookup) |

### Install hints

#### agent-council skill
```bash
# If it is not already in ~/.claude/skills/
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
# Requires PERPLEXITY_API_KEY
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

## Recommended per bug type (nice to have)

These tools improve smoke tests and symbioses depending on the bug domain. If they are missing: a gap hint, not a blocker.

### Web/dashboard bugs
- `browse` skill (gstack) — fastest browser tests
- `playwright` MCP — full browser stack
- `claude-in-chrome` MCP — live Chrome control
- `kapture` MCP — page scraping plus DOM analysis
- `deep-page-scan` skill — structured page analysis

### Backend/API bugs
- `Bash` + `curl` (standard, always available)
- HTTP test MCPs (if available)
- `supabase` MCP (if Supabase is in the stack)

### DB bugs
- `supabase` MCP (Supabase branches for test schemas!)
- Direct DB clients via `Bash`

### n8n workflows
- `n8n-mcp` MCP — structural workflow validation plus schema lookup
- An n8n patterns skill for your own conventions

### Mobile bugs
- `adb-android-control` skill — Android device control
- iOS simulator (external)

### Build/deploy/CI
- `gh` CLI — GitHub Actions logs
- Docker / container engine
- A deploy skill if one applies to your stack

---

## Optional (quality of life)

| Tool | Role |
|---|---|
| `codex` skill | Second-opinion pattern, adversarial mode |
| `discovery-first` skill | Pre-task lookup discipline |
| `handoff` skill | Managing session transitions cleanly |
| `commit` skill | Conventional-commit helper |
| `verify` skill | Standard verification patterns |
| `qa` skill | QA loop patterns |
| `notebooklm` MCP | Knowledge consolidation |
| Social/forum scraping MCPs | Scrape industry talk / Reddit / Twitter for edge-case discovery |

---

## Prerequisites check output (example)

At every phase start the skill shows this status block:

```markdown
🔧 Prerequisites check for safe-debug-loop

✅ Required
- Claude Code: ✓
- Git: ✓
- Filesystem: ✓

✅ Strongly recommended (5/6 available)
- agent-council: ✓
- firecrawl: ✓
- perplexity: ✓
- context7: ✓
- code-review-graph: ✓
- browse: ✓
- notes vault: ✗ → fallback: .bug-sweep/ locally only (cross-project lookup disabled)
  💡 Install hint: claude mcp add <notes-vault-mcp> <...>

🟢 Bug-type coverage
- Web/UI: ✓ (browse + playwright)
- API: ✓ (Bash + curl)
- DB: ✓ (supabase)
- n8n: ✓ (n8n-mcp)
- Mobile: ✗ (adb-android-control missing) → gap hint if mobile bugs come up

✅ Effort mode: max
✅ Repo: .bug-sweep/ will be created

→ Skill starts Phase <N> ...
```

---

## What happens when tools are missing

### Critical gap (a required tool is missing)
- The skill **stops immediately**
- Clear note on what is missing
- Install command if known
- The user installs and restarts

### Important gap (a strongly recommended tool is missing)
- The skill **runs with a fallback**
- One note at the start
- Mark confidence validation explicitly as "self-asserted" instead of "council-validated"
- The skill suggests: "if you have time after this session: install X, next time it will be Y× better"

### Optional gap (bug-type coverage)
- The skill runs normally
- If a matching bug comes up: a gap hint that this bug becomes hard to verify without tool X
- Optionally: propose deferring the bug until the tool is installed

---

## Skill self-maintenance

The skill does not update its own prerequisites list automatically — that is done manually. But it gives a compact report at the end of a session:

```
📊 Skill performance report for this session

Tools used:
- firecrawl (15× parallel scrapes) ✓
- perplexity (8× research) ✓
- agent-council (3× confidence checks) ✓
- browse (12× smoke tests) ✓
- code-review-graph (5× impact checks) ✓

Tools that were missing:
- notes-vault MCP → knowledge base local only, no cross-project sync

Recommendation for next time:
- Install a notes-vault MCP for cross-session knowledge persistence
```
