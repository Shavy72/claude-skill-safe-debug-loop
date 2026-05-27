# Smoke-Test-Log — Bug #`<id>` — `<bug-titel>`

**Pfad:** `.bug-sweep/smoke-tests/<bug-id>-smoke-log.md`
**Erstellt durch:** safe-debug-loop Phase 2 (Step 5)
**Start-Zeit:** `<YYYY-MM-DD HH:MM:SS>`

---

## Bug-Referenz

- **Bug-ID:** `#<id>`
- **Titel:** `<titel>`
- **Bug-Plan-File:** `.bug-sweep/bug-plan-<iso>.md` (oder `adhoc-bugs-<iso>.md`)
- **Kategorie:** 🔴 Rot / 🟡 Gelb / 🟢 Grün
- **Fix-Commit:** `<hash>` (wird nach Fix-Anwendung gefüllt)

---

## Smoke-Test-Strategie

- **Typ:** API / UI / DB / Backend-Logic / n8n / Mobile / Build-Deploy
- **Sandbox-Setup:** <konkret beschreiben — z.B. "Supabase Test-Branch", "lokaler Dev-Server", "Mock-API via msw", "Test-Account auf Staging">
- **Originalgetreu-Score:** <1-10 wie nahe an Production-Realität — und Begründung>

---

## Dummy-Daten (mit Marker)

Vor dem Test erstellt — müssen am Ende cleaned werden:

- `__smoketest_<bug-id>_user_1` (User-Record in `users` Tabelle)
- `__smoketest_<bug-id>_payload_1` (API-Payload mit diesem Marker)
- `__smoketest_<bug-id>_event_*` (Event-Stream-Marker)
- ...

---

## Iteration 1

**Test-Schritte:**
1. `<schritt>`
2. `<schritt>`
3. `<schritt>`

**Outputs / Beweise:**
- `<command/click/curl>` → `<output>`
- Screenshot: `.bug-sweep/smoke-tests/screenshots/<bug-id>-iter1-step3.png`
- Console-Log:
  ```
  <relevant log lines>
  ```

**Assertions:**
- [ ] Bug-Szenario reproduziert?
- [ ] Fix greift?
- [ ] Erwartetes Verhalten beobachtet?
- [ ] Keine Side-Effects auf andere Bereiche?

**Ergebnis Iteration 1:** ✅ passed / ❌ failed

**Wenn failed:** Grund + was wird in Iteration 2 anders gemacht.

---

## Iteration 2 (nur wenn 1 failed)

(gleiche Struktur)

---

## Iteration 3 (nur wenn 2 failed)

(gleiche Struktur)

**Wenn auch Iteration 3 failed:** Eskalations-Block am Ende füllen.

---

## Cleanup-Aktionen

Nach erfolgreichem Smoke-Test (✅ passed in irgendeiner Iteration):

**Schritte:**
1. `<cleanup-aktion 1>` — z.B. `DELETE FROM users WHERE id LIKE '__smoketest_*'`
2. `<cleanup-aktion 2>` — z.B. Test-Files löschen
3. `<cleanup-aktion 3>` — z.B. Mock-Server stoppen

**Cleanup-Logs:**
- `<befehl>` → exit 0
- `<befehl>` → exit 0

---

## Cleanup-Verify (Pflicht)

Beweis dass NICHTS Test-Artefakte mehr da sind:

```
<befehl 1 zur Such-Verifikation>
→ 0 Treffer ✓

<befehl 2>
→ 0 Treffer ✓

<befehl 3 — Live-Version-Check>
→ keine __smoketest_* Spuren ✓
```

---

## Council-Verdict (optional, wenn agent-council benutzt)

```
agent-council prompt: "Ist dieser Fix solide? Hier sind die Smoke-Test-Ergebnisse: ..."

Council-Outcome: PASS / NEEDS_REWORK
Stimmen:
- Codex: PASS / NEEDS_REWORK — <begründung>
- Gemini: PASS / NEEDS_REWORK — <begründung>
- Synthesized Verdict: PASS / NEEDS_REWORK
```

---

## Final-Status

- **Status:** ✅ passed / ❌ failed (mit Begründung)
- **Iteration-Count:** 1 / 2 / 3
- **Cleanup verifiziert:** ✅ / ❌
- **Council-Verdict:** PASS / NEEDS_REWORK / not used
- **Bug in Bug-Plan abgehakt:** ✅ / ❌
- **End-Zeit:** `<YYYY-MM-DD HH:MM:SS>`
- **Dauer:** `<minuten>` min

---

## Logging-/Doku-Upgrade (parallel zum Fix gemacht)

Was wurde in diesem Code-Bereich zusätzlich verbessert um zukünftige Bugs schneller zu finden:

- **Logging:**
  - <Stelle X: neuer Log mit `<level>` für `<event>`>
  - <Stelle Y: Correlation-ID hinzugefügt>
- **Code-Kommentare:**
  - `<file:line>`: <was kommentiert wurde>
- **Doku:**
  - <README-Section / Inline-Doku aktualisiert>

---

## Eskalations-Block (nur wenn alle 3 Iterationen failed)

**Was wurde versucht:**
1. Iteration 1: <was probiert>
2. Iteration 2: <was anders gemacht>
3. Iteration 3: <was wieder anders>

**Hypothesen die nicht funktionierten:**
- <Hypothese 1 + warum sie falsch war>
- <Hypothese 2 + warum sie falsch war>
- <Hypothese 3 + warum sie falsch war>

**Empfehlung an User:**
- A) <Alternative Fix-Strategie + Rationale>
- B) <Bug als "blocked" markieren + andere Bugs zuerst>
- C) <Codex-Consult oder Council-Deep-Dive>
- D) <Manueller User-Eingriff erforderlich + was er beitragen müsste>

**Skill wartet auf User-Entscheidung.**
