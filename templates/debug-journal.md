# Debug journal — `<project-name>`

**Path:** `.bug-sweep/debug-journal.md`
**Purpose:** a cross-session journal of all safe-debug-loop sessions for this module. It grows over time.

---

## What this journal is for

- Future Claude sessions can look up what has already been fixed
- Recurring patterns become visible (the same bug type keeps coming back → an architectural smell)
- Logging/docs progress stays traceable
- For new bugs: cross-referencing similar earlier bugs saves investigation time

---

## Session index

| # | Date | Phase | Bugs addressed | Status | Bug-plan file |
|---|---|---|---|---|---|
| 1 | `<YYYY-MM-DD>` | 1+2 | 24 (7R/12Y/5G) | ✅ done | `bug-plan-<iso>.md` |
| 2 | `<YYYY-MM-DD>` | 2 (ad hoc) | 3 | ✅ done | `adhoc-bugs-<iso>.md` |
| 3 | `<YYYY-MM-DD>` | 1+2 | 8 (2R/4Y/2G) | 🔄 in progress | `bug-plan-<iso>.md` |
| ... | | | | | |

---

## Session #1 — `<YYYY-MM-DD>` — Phase 1+2 (initial remediation)

**Module/area:** `<description>`
**Trigger:** "<what the user said / the occasion>"
**Duration:** ~`<h>` h
**Tool stack snapshot:** `<which skills/MCPs/plugins were installed>`

### Phase 1 results
- Research files created: `<list>`
- Bug plan: `<filepath>`
- Confidence level: 100% council-validated / self-asserted
- Bugs found: N (X 🔴 / Y 🟡 / Z 🟢)

### Phase 2 results
- Bugs fixed: M of N
- Of those obsolete (already fixed): K
- Of those blocked: L (reason: ...)
- Smoke tests passed: M
- Atomic commits: M
- Logging upgrades in: <K areas>
- Docs upgrades in: <J files>

### Key learnings from this session
- **<learning 1>:** observation + why it matters going forward
- **<learning 2>:** ...

### Recurring patterns (watch out!)
- **<pattern X>:** this is the Nth time it shows up → an architectural refactor is recommended
- ...

### Open items for future sessions
- [ ] <what was not finished + why>
- [ ] <a new insight that should be taken into account in the next Phase 1>

---

## Session #2 — `<YYYY-MM-DD>` — Phase 2 (ad hoc)

(same structure, shorter if it was ad hoc only)

---

## Cross-session findings

### Persistent architectural smells

Patterns that span several sessions:

- **<smell 1>:** <description> — recommended: <refactor / monitor / accept>
- **<smell 2>:** ...

### Logging maturity

Where logging stands in the module now:

- ✅ <areas with good structured logging>
- 🟡 <areas with partial logging>
- ❌ <areas without meaningful logging — priority for the next session>

### Docs maturity

- ✅ <well documented areas>
- 🟡 <partly documented areas>
- ❌ <undocumented areas — documentation debt>

### Tool coverage gaps

Tools that were missing repeatedly:

- `<tool x>` (has been a gap hint several times) → recommended to install
- ...

---

## Future roadmap (proposed by the skill)

Based on the findings across all sessions:

1. **<proposal 1>** — e.g. "migrate API calls from v1 to v2 (doc mismatch in 4 sessions in a row)"
2. **<proposal 2>** — e.g. "introduce a structured logging framework (manual log calls are inefficient)"
3. **<proposal 3>** — ...

---

## Quick reference for future sessions

If someone (Claude or a human) debugs this module in 3 months:

- **Module overview:** `.bug-sweep/research/module-overview.md`
- **API dependencies:** `.bug-sweep/research/api-docs/`
- **Known edge cases:** `.bug-sweep/research/gotchas.md`
- **Logging conventions:** see logging maturity above
- **Recurring issues:** see persistent architectural smells above
