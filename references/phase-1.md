# Phase 1 — Researcher + Detective Cross-Check

**Fully autonomous. No production code is touched. The result is an in-depth solution markdown file plus a start prompt for Phase 2.**

Phase 1 runs completely without user interaction — as soon as the skill has enough context (module, path, optionally a live URL) it works autonomously through to the output.

---

## Preconditions before Phase 1 starts

The skill checks and ensures:

1. **The current working directory is the project repo.** If not: ask the user where the module lives.
2. **The module scope is clear.** Which file(s), which section of the dashboard, which path in the app. If unclear: one short question to the user (bundled once, then autonomous).
3. **The `.bug-sweep/` folder exists or gets created.**
4. **The prerequisites check** ([prerequisites.md](prerequisites.md)) has produced a status report. For critical gaps: one note to the user, then work with what is there.
5. **Effort mode is set to MAX.** If not: tell the user to run `/effort max`.

---

## Part 1 — Radical knowledge expansion

### Persona

> **"You are a relentless investigator with total hyperfocus. You do not stop before you have collected and persisted across sessions EVERY smallest useful piece of information and every remotely relevant piece of context. No breaks, no shortcuts."**

### Tool-symbiosis plan for Part 1

Before starting: inventory plus symbiosis. Typical parallel tool stacks (examples — use whatever the user has installed):

| Task | Tool symbiosis (parallel) |
|---|---|
| Scrape external API docs | `firecrawl` + `perplexity` + `context7` + `notebooklm` + web scrapers |
| Library docs | `context7` (with `resolve-library-id` → `query-docs`) |
| Web research / current standards | `perplexity_research` + `perplexity_search` |
| Repo-internal code analysis | `code-review-graph` + Glob/Grep + the `Explore` subagent |
| Endpoint inventory | `Bash` (curl/HTTP discovery) + code grep + Postman/OpenAPI files |
| Persistence | notes-vault MCP (if present) + local files in `.bug-sweep/research/` |

If a tool is missing: note it once ("FYI — with `firecrawl` this would be 3× faster"), then continue with `WebFetch`/`WebSearch`.

### Concrete research pipeline

```
1. Module code inventory
   - Which files? Which components? Which endpoints? Which stores/state?
   - Output: .bug-sweep/research/module-overview.md
   - Tools: Glob, Grep, Read, code-review-graph

2. Dependency discovery
   - Which external APIs are called? Which internal services?
   - Which npm/pip packages matter?
   - Which env vars are used?
   - Output: .bug-sweep/research/dependencies.md

3. External API doc scrape (in parallel per API)
   - Complete current docs for every API in use
   - Also docs for endpoints that might only be marginally relevant
   - Output: .bug-sweep/research/api-docs/<api-name>.md
   - Tools: firecrawl/perplexity/context7 in parallel

4. Endpoint schemas + example payloads
   - For every external call: request schema, response schema, error cases, rate limits
   - Example payloads from the docs or via an API test (curl)
   - Output: .bug-sweep/research/endpoints.md

5. Known edge cases + gotchas
   - What do the API docs say under "common errors", "limitations", "breaking changes"?
   - Current standards / best practices for this domain (perplexity_research)
   - Output: .bug-sweep/research/gotchas.md

6. Internal logging audit
   - What is currently logged? Where? At which level?
   - Output: .bug-sweep/research/logging-status.md

7. Vault mirroring (if a notes vault is available)
   - Mirror all research files to <vault>/projects/<project>/safe-debug-loop/research/
   - So the knowledge stays available across project boundaries
```

### Stop condition Part 1

**Has 100% confidence been reached?**

Validation via `agent-council` (if installed) — prompt along the lines of:
> "I have produced the following research set: [list]. Do I have EVERY relevant piece of information? What is still missing? Are there gaps that would hinder a bug detective later on?"

If the council reports "gap X": back to Part 1 with a focus on X. Repeat until the council says "complete".

Fallback if the council is missing: self-reasoning plus a codex consult check (if `codex` is installed) plus an explicit checklist:
- [ ] All external APIs documented?
- [ ] All endpoints with a schema?
- [ ] Edge cases per API?
- [ ] Current standards/best practices?
- [ ] Logging status?
- [ ] Dependency map?

**Once 100% confidence is reached → automatic transition to Part 2.** No user input needed.

---

## Part 2 — Radical cross-check (detective)

### Persona

> **"You are the world's best intelligence and law-enforcement investigator. You meticulously compare the current code state of the module with everything learned in Part 1 and you find EVERY bug, however small. Nothing escapes you. You also look for things that are NOT there but should be (missing calls, missing logging, missing error handlers)."**

### Tool-symbiosis plan for Part 2

| Task | Tool symbiosis (parallel) |
|---|---|
| Code inspection + pattern search | Grep + Glob + Read + the `Explore` subagent |
| Bug-pattern detection | `debug` + `investigate` + `audit-verify-loop` (as methodological inspiration, not as an auto-run) |
| Architecture understanding | `code-review-graph` (hub nodes, bridge nodes, communities) |
| Doc-mismatch detection | Compare code reality ↔ `.bug-sweep/research/` |
| Endpoint validation | `Bash` curl against the actual endpoints (read-only calls, no state change!) |
| 100% confidence validation | `agent-council` + `codex` |

### Concrete detective pipeline

```
1. Code reality vs. API docs
   - Per external call: does the schema match? Is response handling complete?
   - Faulty/outdated endpoints? Deprecated fields?

2. Internal flow cross-check
   - Are state transitions consistent? Missing loading/error states?
   - Race conditions possible? Stale-data risks?

3. Logging gap analysis
   - Where is logging missing that you would need when a bug hits?
   - Inconsistent log levels? Missing correlation IDs?

4. Doc-mismatch detection
   - Code comments that no longer hold?
   - README / inline docs ↔ actual behaviour?

5. Dead code / code smells
   - Unused imports/functions?
   - Copy-paste duplicates?
   - Magic numbers without explanation?

6. Wrong calls / wrong arguments
   - Function calls with wrong parameters?
   - APIs with deprecated endpoints/headers?

7. Error-handling gaps
   - Try-catch blocks that do nothing?
   - Unhandled promise rejections?
   - Generic error messages where they should be specific?
```

### Categorising every bug

| Category | When | Example |
|---|---|---|
| 🔴 **Red — urgent** | The bug breaks the main function / blocks the MVP / is security relevant / risks data corruption | Submit button does nothing, auth token stored in local storage, DB query without sanitisation |
| 🟡 **Yellow — important** | The bug hurts the user experience / causes confusion / breaks edge cases | Loading state missing, error toast shows generic text instead of concrete info, rate-limit handler missing |
| 🟢 **Green — negligible** | Cosmetic / code quality / nice to have | Inconsistent naming, missing JSDoc, magic number without a constant |

### Building the bug plan

Every bug gets an entry in the bug-plan markdown. Template: [../templates/bug-plan.md](../templates/bug-plan.md).

Every entry needs at least:
- ID + title
- Category (red/yellow/green)
- Symptom (what was observed)
- Suspected root cause (file + line if possible)
- Fix plan (what to do, minimally invasive)
- Dependencies (which other bugs have to be fixed first?)
- Blast radius (which other places could the fix affect?)
- Smoke-test strategy (a proposal; the detailed strategy comes in Phase 2)
- Logging/docs/comment improvements that should be fixed along the way

### Stop condition Part 2

**100% confidence: all bugs found and every plan makes complete sense?**

Validation via `agent-council` — prompt along the lines of:
> "Here is my bug plan: [list]. Have I found EVERY relevant bug? Does the plan make complete sense per bug? Are the dependencies correct? Is the blast radius realistically assessed?"

If the council reports "gap X" or "plan Y unclear": back to the detective pipeline. Repeat until 100%.

---

## Phase 1 output

### 1. Bug-plan markdown

`<repo>/.bug-sweep/bug-plan-<ISO-date>.md`

For the structure see [../templates/bug-plan.md](../templates/bug-plan.md). Example opening:

```markdown
# Bug plan — <module-name> — <ISO-Date>

**Module:** <path>
**Live URL (if any):** <url>
**Research sources:** .bug-sweep/research/
**Created by:** safe-debug-loop Phase 1
**Confidence level:** 100% (council-validated)

## Bug summary
- 🔴 Red: 7 bugs
- 🟡 Yellow: 12 bugs
- 🟢 Green: 5 bugs

## Recommended fix order
1. Bug #R1 (root bug, all others indirectly affected)
2. Bug #R2
...

---

## 🔴 R1 — <bug-title>
- **Symptom:** ...
- **Root cause:** ...
- **Fix plan:** ...
- **Blast radius:** ...
- **Smoke test:** ...
- **Status:** [ ] open
```

### 2. Start prompt for Phase 2 (printed in the chat)

At the end the skill prints a copy-paste block for the user:

```
╔══════════════════════════════════════════════════════════╗
║  PHASE 1 COMPLETE                                        ║
║  Bug plan: .bug-sweep/bug-plan-<ISO>.md                  ║
║  Confidence: 100% (council-validated)                    ║
║  Found: 7 red / 12 yellow / 5 green                      ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  COPY-PASTE into your Phase 2 session:                   ║
║                                                          ║
║  ─────────────────────────────────────────────────────   ║
║  /safe-debug-loop Phase 2                                ║
║                                                          ║
║  Bug plan: .bug-sweep/bug-plan-<ISO>.md                  ║
║  Research: .bug-sweep/research/                          ║
║                                                          ║
║  Please work through the plan iteratively — one bug at   ║
║  a time, with a smoke test per fix. Holistic check       ║
║  before every fix.                                       ║
║  ─────────────────────────────────────────────────────   ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

The user copies the framed block into their Phase 2 session.

---

## What Phase 1 does NOT do

- ❌ Change code
- ❌ Write files outside `.bug-sweep/` and (optionally) the notes vault
- ❌ Make commits (except possibly one commit of the `.bug-sweep/` artefacts)
- ❌ Hit live endpoints with state-changing calls (read-only calls only)
- ❌ Pester the user with interim questions (bundle them once at the start if needed)

---

## Edge cases + escalation

- **Repo too large for 100% research:** the skill prioritises by code-review-graph hub nodes → focuses on the high-impact area. Tell the user that the scope was reduced.
- **External API unreachable / behind Cloudflare:** tell the user, fall back to cached docs or straight to `WebFetch` with browser headers. On hard blocks: hand off to the user.
- **Neither council nor codex installed:** the skill runs a structured self-validation via checklist and warns the user that the confidence level is "self-asserted" instead of "council-validated".
- **Infinite loop in the 100%-confidence search:** hard limit of 3 iterations per confidence round. After that: go to the user with a concrete question about what is still missing.
