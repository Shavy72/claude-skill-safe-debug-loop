# Ad-hoc bug entry — Phase 2 standalone (mode C2)

**Usage:** when the user invokes `/safe-debug-loop Phase 2` in the middle of an active chat and no Phase 1 bug plan exists. The skill creates this file at:

`.bug-sweep/adhoc-bugs-<ISO-date>.md`

---

# Ad-hoc bug list — <ISO-Date>

**Created by:** safe-debug-loop Phase 2 (mode C2 — mid-chat)
**Active session:** `<session id if identifiable>`
**Context source:** the last N chat messages plus, where relevant, the code state

---

## Bugs detected from the chat context

### A1 — `<bug-title-detected-from-chat>`

- **Status:** [ ] open
- **Source:** <e.g. "the user mentioned at <time>: 'X does not work'" or "an earlier reasoning answer identified X as a bug">
- **Symptom:** <what was described in the chat>
- **Surface:** <where — if derivable from the chat>
- **Suspected root cause:** <if already reasoned about in the chat, otherwise "tbd in step 2">
- **Fix plan (minimally invasive):** <if already available, otherwise "tbd in step 4">
- **Blast radius:** <if identifiable>
- **Smoke-test strategy:** <proposal — refined in Phase 2>
- **Logging/docs upgrade plan:** <if already clear>

---

### A2 — `<further-bug-if-several>`

(same structure — only fill this in if there were several bugs in the chat context)

---

## Notes

- Ad-hoc bug files have less depth than Phase 1 bug plans — the research basis is missing
- If an ad-hoc bug turns out to be more complex than expected: suggest to the user that a full Phase 1 be run afterwards
- If bugs are still open after Phase 2: the user can optionally start a Phase 1 to get a complete list

---

## The Phase 2 loop starts now

The skill starts with step [1] for the first bug from this list.
