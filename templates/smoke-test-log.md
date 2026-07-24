# Smoke-test log — bug #`<id>` — `<bug-title>`

**Path:** `.bug-sweep/smoke-tests/<bug-id>-smoke-log.md`
**Created by:** safe-debug-loop Phase 2 (step 5)
**Start time:** `<YYYY-MM-DD HH:MM:SS>`

---

## Bug reference

- **Bug ID:** `#<id>`
- **Title:** `<title>`
- **Bug-plan file:** `.bug-sweep/bug-plan-<iso>.md` (or `adhoc-bugs-<iso>.md`)
- **Category:** 🔴 red / 🟡 yellow / 🟢 green
- **Fix commit:** `<hash>` (filled in after the fix is applied)

---

## Smoke-test strategy

- **Type:** API / UI / DB / backend logic / n8n / mobile / build-deploy
- **Sandbox setup:** <describe concretely — e.g. "Supabase test branch", "local dev server", "mock API via msw", "test account on staging">
- **Faithfulness score:** <1-10, how close to production reality — plus a justification>

---

## Dummy data (with markers)

Created before the test — must be cleaned up at the end:

- `__smoketest_<bug-id>_user_1` (user record in the `users` table)
- `__smoketest_<bug-id>_payload_1` (API payload with this marker)
- `__smoketest_<bug-id>_event_*` (event-stream marker)
- ...

---

## Iteration 1

**Test steps:**
1. `<step>`
2. `<step>`
3. `<step>`

**Outputs / evidence:**
- `<command/click/curl>` → `<output>`
- Screenshot: `.bug-sweep/smoke-tests/screenshots/<bug-id>-iter1-step3.png`
- Console log:
  ```
  <relevant log lines>
  ```

**Assertions:**
- [ ] Bug scenario reproduced?
- [ ] Fix works?
- [ ] Expected behaviour observed?
- [ ] No side effects on other areas?

**Result of iteration 1:** ✅ passed / ❌ failed

**If failed:** the reason plus what will be done differently in iteration 2.

---

## Iteration 2 (only if 1 failed)

(same structure)

---

## Iteration 3 (only if 2 failed)

(same structure)

**If iteration 3 also fails:** fill in the escalation block at the end.

---

## Cleanup actions

After a successful smoke test (✅ passed in any iteration):

**Steps:**
1. `<cleanup action 1>` — e.g. `DELETE FROM users WHERE id LIKE '__smoketest_*'`
2. `<cleanup action 2>` — e.g. delete test files
3. `<cleanup action 3>` — e.g. stop the mock server

**Cleanup logs:**
- `<command>` → exit 0
- `<command>` → exit 0

---

## Cleanup verification (mandatory)

Evidence that NO test artefacts remain:

```
<command 1 for search verification>
→ 0 hits ✓

<command 2>
→ 0 hits ✓

<command 3 — live version check>
→ no __smoketest_* traces ✓
```

---

## Council verdict (optional, if agent-council was used)

```
agent-council prompt: "Is this fix solid? Here are the smoke-test results: ..."

Council outcome: PASS / NEEDS_REWORK
Votes:
- Codex: PASS / NEEDS_REWORK — <justification>
- Gemini: PASS / NEEDS_REWORK — <justification>
- Synthesised verdict: PASS / NEEDS_REWORK
```

---

## Final status

- **Status:** ✅ passed / ❌ failed (with a justification)
- **Iteration count:** 1 / 2 / 3
- **Cleanup verified:** ✅ / ❌
- **Council verdict:** PASS / NEEDS_REWORK / not used
- **Bug ticked off in the bug plan:** ✅ / ❌
- **End time:** `<YYYY-MM-DD HH:MM:SS>`
- **Duration:** `<minutes>` min

---

## Logging/docs upgrade (done alongside the fix)

What else was improved in this code area to find future bugs faster:

- **Logging:**
  - <place X: new log at `<level>` for `<event>`>
  - <place Y: correlation ID added>
- **Code comments:**
  - `<file:line>`: <what was commented>
- **Docs:**
  - <README section / inline docs updated>

---

## Escalation block (only if all 3 iterations failed)

**What was tried:**
1. Iteration 1: <what was tried>
2. Iteration 2: <what was done differently>
3. Iteration 3: <what was done differently again>

**Hypotheses that did not work:**
- <hypothesis 1 + why it was wrong>
- <hypothesis 2 + why it was wrong>
- <hypothesis 3 + why it was wrong>

**Recommendation to the user:**
- A) <alternative fix strategy + rationale>
- B) <mark the bug as "blocked" + other bugs first>
- C) <codex consult or council deep dive>
- D) <manual user intervention required + what they would need to contribute>

**The skill waits for the user's decision.**
