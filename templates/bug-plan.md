# Bug plan — <module-name> — <ISO-Date>

**Module / section:** `<path or UI area>`
**Live URL (if any):** `<url>`
**Local dev URL:** `<url>`
**Research basis:** `.bug-sweep/research/`
**Created by:** safe-debug-loop Phase 1
**Confidence level:** 100% (council-validated | self-asserted — state one explicitly)
**Start date:** `<YYYY-MM-DD HH:MM>`

---

## Bug summary

| Category | Count |
|---|---|
| 🔴 Red — urgent (MVP blocker / security / data corruption) | N |
| 🟡 Yellow — important (UX disruption / edge case / missing logging) | N |
| 🟢 Green — negligible (code quality / cosmetic) | N |
| **Total** | N |

---

## Recommended fix order

1. **#R1** — `<title>` (root bug, several others depend on it)
2. **#R2** — `<title>`
3. **#R3** — `<title>` (depends on #R1)
4. ...

Reasoning for the order:
- Root bugs first (they may resolve dependent bugs automatically)
- Higher category before lower
- Within the same category: smaller blast radius first

---

## Bug list (complete)

### 🔴 R1 — `<bug-title>`

- **Status:** [ ] open
- **Category:** red — urgent
- **Symptom:** <what was observed — as concrete as possible, with an evidence path/screenshot/log>
- **Surface:** <where it occurs — URL/endpoint/function>
- **Suspected root cause:** `<file>:<line>` — <what appears to be broken>
- **Research reference:** <pointer to .bug-sweep/research/...>
- **Fix plan (minimally invasive):**
  - <step 1>
  - <step 2>
  - <step 3>
- **Dependencies:** bug #X has to be fixed first / independent
- **Blast radius:** <which other places could be affected by the fix>
- **Smoke-test strategy:** <which one from references/smoke-test.md — API/UI/DB/logic/etc>
- **Logging/docs upgrade plan:** <what gets improved along with this fix>
- **Estimate:** <S/M/L>
- **Council verdict (if validated):** <PASS / NEEDS_REWORK>

---

### 🔴 R2 — `<bug-title>`

(same structure)

---

### 🟡 Y1 — `<bug-title>`

(same structure)

---

### 🟢 G1 — `<bug-title>`

(same structure)

---

## Cross-cutting findings

Bugs that cannot be assigned cleanly but were observed architecturally:

- **Logging consistency:** <e.g. some modules log with `console.log`, others with a structured logger>
- **Error-handling pattern:** <e.g. try-catch without specific errors>
- **API versioning:** <e.g. v1 and v2 partly mixed>
- **Naming conventions:** <e.g. snake_case vs camelCase mix>

→ These are addressed alongside the respective bug fixes (logging upgrade plan).

---

## External dependencies + risks

- **API provider X:** docs version `<version>` — re-check these bugs on breaking changes
- **Library Y:** current version `<v>` — if it has to be upgraded, expect follow-on bugs
- **Auth provider:** `<name>` — auth-related bugs need a test account

---

## Persistence

- Bug-plan file: `.bug-sweep/bug-plan-<ISO-date>.md` (this file)
- Research files: `.bug-sweep/research/`
- Smoke-test logs: `.bug-sweep/smoke-tests/<bug-id>-smoke-log.md` (created in Phase 2)
- Vault mirror: `<vault>/projects/<project>/safe-debug-loop/` (if a notes vault exists)

---

## Handover to Phase 2

**Start prompt for the Phase 2 session (copy-paste):**

```
/safe-debug-loop Phase 2

Bug plan: .bug-sweep/bug-plan-<ISO>.md
Research: .bug-sweep/research/

Please work through the plan iteratively — one bug at a time,
with a smoke test per fix. Holistic check before every fix.
```

---

**Phase 1 complete.** Phase 2 takes over from here.
