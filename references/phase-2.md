# Phase 2 — Iterative Bug-Fix Loop with a Smoke-Test Gate

**Interactive, with copy-paste prompts for the user. One bug at a time. Smoke test as the quality gate. Cleanup is mandatory.**

Phase 2 fixes bugs one by one, safely and sustainably. Every loop iteration also improves logging and documentation in the affected code area.

---

## Two sub-modes

### Mode C1 — fresh session (the classic path after Phase 1)

The user started a fresh MAX-effort session with `/safe-debug-loop Phase 2` and then pasted the Phase 1 start prompt plus the markdown file path.

- The skill reads the bug-plan markdown
- The skill reads the research files in `.bug-sweep/research/`
- The loop runs over all bugs in the order of the plan (prioritised by category plus dependencies)

### Mode C2 — mid-chat

The user is already working in an active chat. There is either:
- **A concrete bug on the table:** mentioned in the chat, in some reasoning, or in a task definition
- **A plan on the table:** a list of several bugs/issues

The user invokes `/safe-debug-loop Phase 2` without an argument.

- The skill scans the recent chat history for bugs/plans
- If 1 concrete bug is detected: the loop runs for that single bug
- If a plan is detected: the loop runs over the detected list
- If it is unclear: one short question — "which bug / plan should I work on?"
- If no markdown file exists: the skill creates one on the spot at `.bug-sweep/adhoc-bugs-<ISO>.md` (template [../templates/adhoc-bug.md](../templates/adhoc-bug.md))
- From then on: the loop is identical to mode C1

---

## The loop (sequence per bug)

### [1] Tool-symbiosis check

Before every bug loop the skill/MCP/plugin inventory is re-checked (see [tool-symbiosis.md](tool-symbiosis.md)). Which tools help with exactly this type of bug?

### [2] LLM reasoning: bug selection + existence check (COMBINED)

**Internal reasoning prompt (the LLM acts directly on this schema, no user question needed):**

> "If I were only allowed to fix ONE bug from the bug plan today, which one would it be? Before I answer I briefly check the current code state to see whether the bug still exists at all — because earlier fixes (or refactors between sessions) may already have fixed it indirectly. If so → mark it 'obsolete — already fixed' and pick the next one."

**Concrete verification actions for the existence check:**
- Read the files referenced by the bug entry
- Grep for the symptom patterns (e.g. faulty API calls, missing error handlers)
- If it is a UI bug: a quick browser check via `browse` or `playwright` (read-only, no state change)
- If it is a logic bug: a quick dry run with test data (without persistence)

**Output of step [2]:**

```markdown
## Bug selection: #<id> — <title>

**Existence check:**
- [x] Code location inspected: <file:line>
- [x] Symptom reproduced / pattern confirmed
- [x] The bug still exists

**Reason for the selection:**
- Highest priority (category 🔴 red)
- Other bugs depend on it (R3, R5 are follow-on bugs of #R1)
- Solvable in a minimally invasive way

→ Continue with the explain-it-to-a-12-year-old step
```

**If the bug has already been fixed:**

```markdown
## Bug selection: #<id> — <title>

**Existence check:**
- [x] Code location inspected: <file:line>
- [x] The bug is NOT reproducible any more — it was fixed indirectly by fix #X

**Action:**
- Bug #<id> marked in the markdown file as "obsolete — already fixed as a side effect"
- → Loop back to step [1] with the next bug
```

### [3] The skill proposes the next copy-paste prompt

At the end of step [2] the skill shows the user a copy-paste block:

```
╔══════════════════════════════════════════════════════════╗
║  NEXT PROMPT — paste this into the chat:                 ║
║  ─────────────────────────────────────────────────────   ║
║  Explain this one bug that you would fix so that a       ║
║  12-year-old understands it. Explain it the way you      ║
║  know from working with me explains things best, and     ║
║  use an analogy from the real world. Imagine I am 12,    ║
║  slow on the uptake, and cannot understand the bug       ║
║  right now. Answer in simple language for maximum        ║
║  clarity.                                                ║
║                                                          ║
║  Explain:                                                ║
║  1. What the bug does / does not do                      ║
║  2. Why that is a big problem (globally across the       ║
║     dashboard, not just locally)                         ║
║  3. What exactly the fix will do                         ║
║  4. Why the fix is minimally invasive and will not       ║
║     break other parts of the dashboard                   ║
║  ─────────────────────────────────────────────────────   ║
╚══════════════════════════════════════════════════════════╝
```

The user copies it and sends it. The LLM answers with the simple explanation.

### [4] The skill proposes the fix-execution prompt

After the 12-year-old explanation the skill shows the next copy-paste block:

```
╔══════════════════════════════════════════════════════════╗
║  NEXT PROMPT — paste this into the chat:                 ║
║  ─────────────────────────────────────────────────────   ║
║  Please apply this one bug fix, minimally invasive.      ║
║  Make sure you act with 100% awareness and will not      ║
║  break any other code parts of our dashboard with your   ║
║  fix.                                                    ║
║                                                          ║
║  Mandatory steps:                                        ║
║  1. Pre-check: read every place that uses this           ║
║     function / this endpoint / this state field          ║
║  2. Apply the fix (edit/write — smallest diff)           ║
║  3. Atomic commit: fix(<scope>): <bug-title>             ║
║  4. ISOLATED SMOKE TEST (see references/smoke-test.md)   ║
║  5. Clean up all dummy data after the smoke test         ║
║  6. Improve logging/docs/code comments for this area     ║
║     (so future bugs here are easier to find)             ║
║  7. Tick the bug off in .bug-sweep/bug-plan-*.md         ║
║                                                          ║
║  If any step is uncertain or risks breaking other        ║
║  parts: STOP and ask me.                                 ║
║  ─────────────────────────────────────────────────────   ║
╚══════════════════════════════════════════════════════════╝
```

### [5] The LLM applies the fix + smoke test (quality gate)

The LLM does the following:

1. **Pre-check** — holistic scan
   - Which other files/places reference the function/the endpoint/the state?
   - Which tests cover it (if any)?
   - Which active sessions / active users could be affected?

2. **Apply the fix** — minimal diff
   - Edit/write only the necessary places
   - No adjacent refactoring
   - Conventional-commit style

3. **Atomic commit** (if the repo is active)
   - `fix(<scope>): <bug-title>`
   - The body references the bug ID, the root cause and the verification

4. **ISOLATED SMOKE TEST** — see [smoke-test.md](smoke-test.md) for strategies
   - Faithful sandbox setup
   - Dummy data / mocks
   - Reproduces the original bug scenario
   - Verifies that the fix works
   - If not 100% verified: automatic iteration (max 3 attempts)
   - If verified: **cleanup duty**

5. **Cleanup evidence** — documented explicitly
   - Which dummy data was created → removed? ✅
   - Which test files → cleared? ✅
   - Which test endpoints/mocks → disconnected? ✅
   - The live version has NO test leftovers → confirmed ✅

6. **Logging/docs/code-comment upgrade**
   - Make logging more detailed in this code area
   - Add inline comments where behaviour is subtle
   - Update the README / docs if affected
   - Goal: future bugs in this area are inspectable in detail by an LLM in the live version

7. **Update the bug-plan markdown**
   - Bug status: ✅ done
   - Evidence: commit hash + smoke-test log path
   - Which logging/docs improvements were made

8. **Going live (if applicable)**
   - If the module was already live: the fix is deployed live (merge / push / re-deploy depending on the setup)
   - If it is local only: it stays local
   - Post-live check: briefly confirm the live version shows the fix

9. **Council (optional)**
   - If `agent-council` is installed: a short council check on whether the fix is solid
   - The council verdict is documented in the smoke-test log

### [6] Summary for the user

The LLM answers with:

```markdown
## ✅ Bug #<id> fixed

**What was done:** <short technical summary>
**Commit:** <hash>
**Smoke test:** ✅ passed (see .bug-sweep/smoke-tests/<id>-smoke-log.md)
**Cleanup:** ✅ verified
**Logging upgrade:** <what was improved>
**Docs upgrade:** <what was improved>

### Why this fix is good
<explain why it is technically solid and why it is minimally invasive>

### What is better going forward
<explain which future bug classes are now easier to spot>

### What you as the user should watch long term
<concrete signals: logs, metrics, behaviour — so the user can tell whether the fix holds>
```

### [7] /compact suggestion

At the end of the answer:

```
╔══════════════════════════════════════════════════════════╗
║  NEXT STEP — optional:                                   ║
║  ─────────────────────────────────────────────────────   ║
║  /compact Focus: bug plan, next bugs from Phase 1        ║
║                                                          ║
║  Recommendation: run /compact now so context is freed    ║
║  for the next bug iteration. The skill re-reads the      ║
║  bug plan on the next pass.                              ║
║  ─────────────────────────────────────────────────────   ║
╚══════════════════════════════════════════════════════════╝

When you are ready for the next bug, just send "next" or an
empty /safe-debug-loop Phase 2 call — I will then start the
next loop iteration automatically.
```

### [8] Next iteration

The user sends "next" or the loop continues automatically. The skill jumps back to step [1] with the next bug from the plan.

---

## End of the loop (all bugs done)

Once the bug list is fully worked through (everything ✅ or marked "obsolete"):

```markdown
## 🎉 Phase 2 complete

**Bugs fixed:** N (of which X 🔴, Y 🟡, Z 🟢)
**Bugs marked obsolete (already fixed):** M
**Commits:** N atomic commits
**Logging upgrades:** improved in K code areas
**Docs upgrades:** extended in J files

### MVP readiness check
<the skill assesses: is the module MVP-ready now? Which risks remain?>

### Recommended next steps
- [ ] Live deploy (if it has not happened yet)
- [ ] Beta test with a small group of users
- [ ] Monitor the new logging paths for the first 24h
- [ ] Optional: another Phase 1 iteration in 2-4 weeks for new insights from live data

### Persisted knowledge base
- `.bug-sweep/bug-plan-<ISO>.md` (fully worked through)
- `.bug-sweep/smoke-tests/` (all smoke-test logs)
- `.bug-sweep/research/` (knowledge base for future sessions)
- Notes vault (mirrored, if available)
```

---

## Escalation rules inside the loop

- **Smoke test failed 3× for the same bug:** STOP, hand off to the user with the question "A) different fix strategy B) mark the bug as 'blocked' C) external help (codex / council)"
- **Blast radius >5 files:** STOP, get user confirmation
- **Live deploy fails:** STOP, check the rollback, inform the user
- **Cleanup verification fails:** STOP, manual cleanup required, do not tick the bug off before cleanup is 100%

---

## What Phase 2 NEVER does

- ❌ Fix several bugs at once
- ❌ Tick a bug off without smoke-test confirmation
- ❌ Leave test leftovers in the live version
- ❌ Apply a fix without a pre-check (holistic)
- ❌ Refactor adjacent places without explicit user approval
- ❌ Put several bugs in one commit
- ❌ Declare a bug fixed without evidence (smoke-test log)
