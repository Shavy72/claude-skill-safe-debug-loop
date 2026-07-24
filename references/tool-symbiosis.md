# Tool Symbiosis — a Cross-Cutting Duty for Every LLM Call

**Before every reasoning step: inventory → match → symbiosis plan → use actively → gap hint.**

safe-debug-loop is only as good as the tools it puts to work. This document describes how the skill assembles the optimal tool set at every step.

---

## Why this matters

An LLM reasoning step without a tool-symbiosis check wastes potential:
- Research with only `WebFetch` instead of `firecrawl + perplexity + context7 + notebooklm` in parallel = 5× slower plus far less context
- Bug validation with only self-reasoning instead of an `agent-council + codex` second opinion = a higher false-positive risk
- UI verification with only a code read instead of a `browse` browser test = not actually verified
- Endpoint testing by only reading the docs instead of curling against live = no reality check

Otherwise the skill burns tokens on naive paths while state-of-the-art tools sit unused.

---

## Standard sequence before every LLM call

### Step 1 — inventory snapshot

```bash
# Skills
ls ~/.claude/skills/

# MCPs
claude mcp list

# Plugins
ls ~/.claude/plugins/ 2>/dev/null

# Global CLI tools (what is on PATH and relevant)
which gh n8n firecrawl curl
```

The skill caches the inventory per session (a single inventory run is enough — no re-scan per step).

### Step 2 — task matching

What does the current step do? Which tools are relevant for it?

Standard mapping (examples — the full list is in [#tool-mapping](#tool-mapping) below):

| Step type | Tool family |
|---|---|
| Scrape external docs | Web scrapers, library doc tools |
| Code inspection | Grep, Glob, code graph, subagents |
| Bug validation | Council, codex, investigate methodology |
| UI verification | Browser tools |
| API testing | curl, HTTP MCPs |
| Persistence | Notes vault, filesystem |

### Step 3 — symbiosis plan

Where possible: **in parallel**.

Example research step:
- `firecrawl` scrapes the main API docs page
- `perplexity_research` does background research on best practices
- `context7` loads the library docs
- `notebooklm` consolidates everything into a notebook

All of that in ONE tool-call batch (several tool calls in a single assistant message).

Example validation:
- `agent-council` for a multi-model verdict
- `codex consult` for a second opinion
- Self-reasoning as the third voice
→ 3 independent perspectives → real validation

### Step 4 — use them actively

Tools get called, not just mentioned. The skill makes it visible in the output which tools are running:

```
🔧 Tool symbiosis for this step:
   - firecrawl_scrape (parallel) → API docs
   - perplexity_research (parallel) → best practices
   - context7 (parallel) → library specs
   - vault write (post) → persistence
```

### Step 5 — gap hint (in passing, once)

If a state-of-the-art tool is missing that would clearly help:

```
💡 Gap hint: with the `code-review-graph` MCP I could build the architecture
   map 3× faster. Optional: install it later via npm i ...
   I will continue with Grep+Glob+Read.
```

**Important:** give the gap hint only ONCE per session per missing tool. Do not be annoying.

---

## Tool mapping

### Research (Phase 1 Part 1)

| Task | Primary | Secondary | Tertiary |
|---|---|---|---|
| Scrape external API docs | `firecrawl_scrape`, `firecrawl_crawl` | `WebFetch` | `Bash` curl |
| Library/framework docs | `context7` (resolve+query) | `WebFetch` against the official docs page | — |
| Web research (best practices, standards) | `perplexity_research` (deep) | `perplexity_search` | `WebSearch` |
| Notebook consolidation | `notebooklm` add-source + research | — | Manual markdown files |
| Social/threads/industry talk | Social scraping MCPs (Reddit, Twitter, HN) | `WebSearch` | — |
| Repo-internal understanding | `code-review-graph` (build+query) | The `Explore` subagent | Glob+Grep+Read |
| External persistence | Notes-vault MCP create file | Filesystem | — |
| Internal persistence | `Write` into `.bug-sweep/research/` | — | — |

### Bug detective (Phase 1 Part 2)

| Task | Primary | Secondary | Tertiary |
|---|---|---|---|
| Architecture mapping | `code-review-graph` (hub nodes, bridge nodes) | The `Explore` subagent | Glob+Grep+Read |
| Bug-pattern detection | `debug` methodology, `investigate` methodology | Self-reasoning with the pattern table from `audit-verify-loop` | — |
| Logging audit | Grep for log calls plus pattern analysis | `code-review-graph` find-large-functions | — |
| API reality check | `Bash` curl read-only | HTTP MCPs | Browser test |
| Confidence validation | `agent-council` | `codex consult` | Self-checklist |

### Bug fix (Phase 2 step 4)

| Task | Primary | Secondary | Tertiary |
|---|---|---|---|
| Pre-check (holistic) | `code-review-graph` get_impact_radius | Grep for call sites + Read | — |
| Code edit | `Edit`, `Write` | — | — |
| Commit | `Bash` git + conventional commit | A `commit` skill | — |
| Council pre-check (optional) | `agent-council` | `codex consult` | — |

### Smoke test (Phase 2 step 5)

| Bug type | Primary | Secondary |
|---|---|---|
| API | `Bash` curl with a dummy payload | HTTP MCPs |
| UI | `browse` skill, `playwright` MCP, `claude-in-chrome` MCP | `kapture` MCP |
| DB | `supabase` MCP execute_sql with a test schema | A direct DB client |
| n8n workflow | `n8n-mcp` validate_workflow + a manual execute with a dummy | — |
| Backend logic | Write and run a unit test | `Bash` test runner |
| Live dashboard | `browse` against staging | `playwright` |
| Mobile | `adb-android-control` | A mobile device-control setup |

### Logging/docs upgrade (Phase 2 step 6)

| Task | Primary |
|---|---|
| Logging standards | An analytics/tracking skill as a reference plus manual application |
| Code docs | A doc-authoring skill |
| README update | Manual edit |

### Persistence / knowledge base (cross-cutting)

| Task | Primary |
|---|---|
| Local | `Write` into `.bug-sweep/` |
| Notes vault | The vault MCP (if available), otherwise `Write` directly to the vault path |
| Git | `Bash` git + atomic commits |

---

## Symbiosis patterns (proven combinations)

### Pattern A — massively parallel research

When Phase 1 Part 1 runs and an external API system has to be documented:

```
PARALLEL (one tool-call batch):
- firecrawl_scrape(https://api.example.com/docs)
- firecrawl_scrape(https://api.example.com/changelog)
- perplexity_research("example.com API best practices 2026")
- context7.resolve-library-id("example-sdk") → context7.query-docs(...)
- WebSearch("example.com api rate limit gotchas")

THEN (sequential):
- Aggregate and structure into .bug-sweep/research/api-docs/example.md
- Mirror into <vault>/projects/<project>/.../example.md
```

### Pattern B — triple validation for confidence

When a 100% confidence level has to be reached:

```
1. Self-reasoning with a checklist
2. agent-council → "do I have everything?"
3. codex consult → "independent second opinion on this plan"

If all 3 are green → confidence 100%
If 1+ is red → close the gap, validate again
```

### Pattern C — holistic pre-check before a fix

Before every code change:

```
PARALLEL:
- code-review-graph.get_impact_radius(<symbol>)
- Grep "<function-name>" across the repo
- Grep "<endpoint-pattern>" across the repo

THEN:
- Analyse which files are indirectly affected
- Read the critical call sites
- Pre-test plan: what still has to work after the fix?
```

### Pattern D — smoke test with cleanup

After a fix:

```
1. Sandbox setup (per bug type)
2. Create dummy data (with the marker prefix __smoketest_*)
3. Reproduce the bug scenario
4. Verify that the fix works
5. Cleanup (grep for __smoketest_*, delete everything)
6. Cleanup verification (grep again → must return 0 hits)
7. Check the live version contains no test leftovers
```

### Pattern E — iterative logging improvement

Per bug fix:

```
1. Before the fix: what would save me later if this bug came back?
2. Add logging at the key places (correlation id, payload size, timing)
3. Add an inline comment on why this logging matters
4. Update the README section if it is domain knowledge

Goal: when a similar bug hits in 6 months, another Claude in a fresh
session can find it in 5 minutes instead of 5 hours.
```

---

## Anti-patterns (NEVER do this)

- ❌ "I don't have `firecrawl`, that's fine, I'll use `WebFetch`" WITHOUT giving the gap hint
- ❌ Running tools sequentially when they could run in parallel
- ❌ Not checking the tool inventory because "there'll be something there"
- ❌ Calling the council for every trivial step (only at real confidence gates)
- ❌ Listing tools but not calling them
- ❌ Talking to the user about tools instead of using them

---

## Performance note

- Cache the inventory scan once per session (no re-run at every step)
- Parallel calls in ONE assistant message batch (5-10× faster than sequential)
- If tools are optional (gap hint): do not wait, continue with the fallback
- For large operations (e.g. `firecrawl_crawl` over an entire docs site): run them in the background to save tokens
