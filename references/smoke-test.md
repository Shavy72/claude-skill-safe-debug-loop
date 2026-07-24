# Smoke Test — Quality Gate per Bug Fix

**Isolated. Faithful to the original. Verified with 100% confidence. With mandatory cleanup.**

The smoke test is the quality gate between "fix applied" and "bug ticked off". Without a passing smoke test a bug is NEVER marked as fixed.

---

## Iron rules

1. **As true to the original as possible.** The test should reproduce the real bug scenario — not a simplified version.
2. **Isolated.** The test environment must not touch live data or live users.
3. **Dummy data with a marker prefix.** All test data gets a recognisable prefix (e.g. `__smoketest_<bug-id>_*`) so that cleanup is verifiable.
4. **100% confidence before "passed".** If it is not unambiguously green → another iteration.
5. **Cleanup duty.** After a successful test remove all dummy data and test leftovers. NOTHING may disturb the live version.
6. **Verify the cleanup.** After cleaning up, explicitly check that all marker data is gone. No blind "it'll be fine".
7. **Max 3 iterations.** If the smoke test is not green after 3 attempts → hand off to the user.

---

## Smoke-test strategies per bug type

### API bug (external API calls, faulty payloads, response handling)

**Sandbox setup:**
- If the API has a sandbox mode: use it
- Otherwise: a test account / test user marked with `__smoketest_*`
- If neither is possible: a local mock server (e.g. msw, mockoon) OR a read-only test against live (no state change)

**Test sequence:**
```bash
# Reproduce the bug scenario
curl -X POST https://api.example.com/endpoint \
  -H "Authorization: Bearer __smoketest_token" \
  -d '{"user_id": "__smoketest_user_<bug-id>", "...": "..."}'

# Assertions
# 1. Does the HTTP status code match?
# 2. Is the response schema valid?
# 3. Are the bug-specific fields present and correct?
# 4. Error-handling path: provoke an error, verify the response
```

**Cleanup:**
- If POST/PUT/DELETE calls were made: delete all `__smoketest_*` records (delete endpoint or direct DB cleanup)
- If a test account was used: reset the account or use the delete endpoint
- If logs/events were created: remove them from the log index if possible

**Cleanup verification:**
```bash
# Search for the marker
curl https://api.example.com/list?filter=__smoketest_*
# → must return 0 hits
```

---

### UI bug (click behaviour, state updates, rendering)

**Sandbox setup:**
- If the module runs locally: the local dev URL
- If it is live only: staging / test account / test browser session
- Tools: the `browse` skill (gstack), `playwright` MCP, `claude-in-chrome` MCP

**Test sequence:**
```
1. browse navigate <url>
2. browse snapshot -i              # interactive elements
3. browse console clear            # for a clean console read after the action
4. <reproduce the bug scenario>    # clicks, form inputs, navigation
5. browse snapshot -D              # diff against the previous state
6. browse console                  # errors / warnings?
7. browse network                  # failed requests?
8. Assertions:
   - Was the target state reached?
   - No console errors?
   - Expected network calls successful?
   - Visual behaviour correct (screenshot diff)?
```

**Cleanup:**
- Log the test user out / end the session
- If test-user data was created: delete it
- Clear cookies/local storage of the test tab

**Cleanup verification:**
- Check the live session to confirm no test markers show up

---

### DB bug (query errors, schema issues, data integrity)

**Sandbox setup:**
- If a test DB / sandbox branch is available (Supabase branches!): use it
- Otherwise: a test schema or test tables with the prefix `__smoketest_*`
- Never test against live data

**Test sequence:**
```sql
-- Setup
INSERT INTO __smoketest_users (id, ...) VALUES ('__smoketest_<bug-id>_1', ...);

-- Reproduce
<the query / operation in question>

-- Assertions
SELECT * FROM __smoketest_users WHERE id LIKE '__smoketest_<bug-id>_%';
-- verify the expected behaviour
```

**Cleanup:**
```sql
DELETE FROM __smoketest_users WHERE id LIKE '__smoketest_<bug-id>_%';
-- cascading deletes for dependent tables where needed
```

**Cleanup verification:**
```sql
SELECT count(*) FROM __smoketest_users WHERE id LIKE '__smoketest_*';
-- must be 0

-- also check all other test tables
```

---

### Backend logic bug (internal function, algorithm, state machine)

**Sandbox setup:**
- Write a unit test if a test setup exists
- Otherwise: an isolated script `__smoketest_<bug-id>.py` / `.ts` that calls the function with test inputs

**Test sequence:**
```typescript
// __smoketest_<bug-id>.ts
import { functionUnderTest } from './...';

const testCases = [
  { input: <bug-scenario>, expected: <correct> },
  { input: <edge-case-1>, expected: <correct> },
  { input: <edge-case-2>, expected: <correct> },
];

for (const tc of testCases) {
  const result = functionUnderTest(tc.input);
  console.assert(deepEqual(result, tc.expected), `FAIL: ${JSON.stringify(tc)}`);
}
```

**Cleanup:**
- Delete the `__smoketest_<bug-id>.ts` file
- If the tests should be permanent: move them into the normal test directory, remove the marker and commit them

**Cleanup verification:**
```bash
ls -la | grep __smoketest_  # must be empty
```

---

### n8n workflow bug

**Sandbox setup:**
- n8n has a test workflow mode: copy the workflow as `__smoketest_<original-name>`, fix and test it there
- Run triggers manually (not automatically)
- If it is webhook based: use the test webhook URL, not the production URL

**Test sequence:**
- `n8n-mcp` validate_workflow → structural validation
- Manual execute with a dummy payload
- Check the execution log: are all nodes green? What is the output?

**Cleanup:**
- Delete the test executions in n8n
- Delete the test workflow copy if one was made
- Remove any test credentials that were created

**Cleanup verification:**
- `__smoketest_*` workflows in the n8n list → must be empty
- Check the execution list for lingering `__smoketest_*` executions

---

### Mobile app bug

**Sandbox setup:**
- ADB for Android (`adb-android-control`) — test on your own device
- iOS: the simulator if available
- Use a test account, not a live account

**Test sequence:**
- Put the app into the test state
- Reproduce the bug
- Screenshot plus logcat analysis
- Verify the fixed behaviour

**Cleanup:**
- Wipe app data via ADB (`pm clear <package>`)
- Reset the test-account data

---

### Build/deploy/CI bug

**Sandbox setup:**
- A local build / container build instead of a CI run where possible
- If CI: a branch with the `__smoketest_<bug-id>` prefix, run CI there

**Test sequence:**
- Pre-build check: are the dependencies installable?
- Build run with verbose logs
- Post-build: are the artefacts present and valid?
- If deploying: hit the smoke endpoint on staging

**Cleanup:**
- Delete `__smoketest_*` branches
- Stop and remove test containers
- Roll back staging artefacts if needed

---

## Smoke-test log (mandatory documentation per bug)

A smoke-test log is written per bug at:
`.bug-sweep/smoke-tests/<bug-id>-smoke-log.md`

Template: [../templates/smoke-test-log.md](../templates/smoke-test-log.md)

Mandatory fields:
- Bug ID + title
- Test strategy (which one from this document)
- Sandbox setup details
- Dummy data created (list with markers)
- Test steps + outputs
- Assertion results
- Iteration count (1, 2, 3)
- Cleanup actions
- Cleanup verification evidence
- Council verdict (if used)
- Status: ✅ passed / ❌ failed (with a reason)

---

## When the smoke test counts as "passed" (definition)

ALL of these criteria must be met:

- [x] The bug scenario was reproduced in the sandbox setup
- [x] The fix was verified there (behaviour is correct)
- [x] All defined assertions are green
- [x] No side effects on other areas were detected
- [x] The cleanup was carried out
- [x] The cleanup verification shows 0 test leftovers
- [x] The live version (if already deployed) shows no test data
- [x] The council verdict (if used) is "PASS"

If even ONE criterion is not met → ❌ failed → iterate

---

## Failure escalation

**After 3 failed smoke-test iterations:** STOP. The skill hands off to the user:

```
🚨 Smoke test failed 3× for bug #<id>

What was tried:
1. Iteration 1: <what was tried, what was missing>
2. Iteration 2: <what was tried, what was missing>
3. Iteration 3: <what was tried, what was missing>

Possible options:
A) A different fix strategy — a new root-cause hypothesis
B) Mark the bug as "blocked" — other bugs first, come back later
C) External help — a codex consult or a council deep dive
D) Manual user intervention — the user tests it and gives feedback

Which option?
```

---

## Anti-patterns

- ❌ "The test is green, it'll probably work in live too" — NO, the smoke test MUST be faithful to the original
- ❌ Committing test data into the repo with a real marker → it could be activated later
- ❌ "Forgetting" the cleanup or deferring it to later → NO, do it immediately
- ❌ Skipping the cleanup verification because "I trust it" → NO, always check explicitly
- ❌ Ticking a bug off after a failed smoke test because "it should work" → NO, fail = fail = iterate or hand off
