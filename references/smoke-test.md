# Smoke-Test — Quality-Gate pro Bug-Fix

**Isoliert. Originalgetreu. Mit 100% Confidence-Verifikation. Mit Cleanup-Pflicht.**

Der Smoke-Test ist das Quality-Gate zwischen "Fix angewendet" und "Bug abgehakt". Ohne bestandenen Smoke-Test wird ein Bug NIE als gefixt markiert.

---

## Iron Rules

1. **Originalgetreu wie möglich.** Test soll das echte Bug-Szenario reproduzieren — nicht eine vereinfachte Version.
2. **Isoliert.** Test-Umgebung darf die Live-Daten + Live-User nicht berühren.
3. **Dummy-Daten mit Marker-Prefix.** Alle Test-Daten bekommen einen erkennbaren Prefix (z.B. `__smoketest_<bug-id>_*`), damit Cleanup verifizierbar ist.
4. **100% Confidence vor "passed".** Wenn nicht eindeutig grün → weitere Iteration.
5. **Cleanup-Pflicht.** Nach erfolgreichem Test alle Dummy-Daten + Test-Reste raus. NICHTS darf in Live-Version stören.
6. **Cleanup-Verify.** Nach Cleanup explizit prüfen dass alle Marker-Daten weg sind. Kein blindes "wird schon".
7. **Max 3 Iterationen.** Wenn nach 3 Versuchen Smoke-Test nicht grün → User-Handoff.

---

## Smoke-Test-Strategien je Bug-Typ

### API-Bug (externe API-Calls, fehlerhafte Payloads, Response-Handling)

**Sandbox-Setup:**
- Wenn API einen Sandbox-Mode hat: nutzen
- Sonst: Test-Account / Test-User mit `__smoketest_*` Markierung
- Wenn nichts davon: Mock-Server lokal (z.B. msw, mockoon) ODER read-only Test gegen Live (kein State-Change)

**Test-Sequenz:**
```bash
# Reproduktion des Bug-Szenarios
curl -X POST https://api.example.com/endpoint \
  -H "Authorization: Bearer __smoketest_token" \
  -d '{"user_id": "__smoketest_user_<bug-id>", "...": "..."}'

# Assertions
# 1. HTTP-Status Code passt?
# 2. Response-Schema valide?
# 3. Bug-spezifische Felder vorhanden + korrekt?
# 4. Error-Handling-Path: provoziere Fehler, verifiziere Response
```

**Cleanup:**
- Wenn POST/PUT/DELETE Calls gemacht wurden: alle `__smoketest_*` Records löschen (DELETE-Endpoint oder direktes DB-Cleanup)
- Wenn Test-Account: Account-Reset oder Lösch-Endpoint
- Wenn Logs/Events erzeugt: ggf. aus Log-Index entfernen falls möglich

**Cleanup-Verify:**
```bash
# Suche nach Marker
curl https://api.example.com/list?filter=__smoketest_* 
# → muss 0 Treffer haben
```

---

### UI-Bug (Click-Verhalten, State-Update, Rendering)

**Sandbox-Setup:**
- Wenn Modul lokal lauffähig: lokale Dev-URL
- Wenn nur live: Staging / Test-Account / Test-Browser-Session
- Tool: `browse`-Skill (gstack), `playwright`-MCP, `claude-in-chrome`-MCP

**Test-Sequenz:**
```
1. browse navigate <url>
2. browse snapshot -i              # interaktive Elemente
3. browse console clear            # für sauberen Console-Read nach Action
4. <reproduziere Bug-Szenario>     # Klicks, Form-Inputs, Navigation
5. browse snapshot -D              # Diff zum Vor-Zustand
6. browse console                  # Errors / Warnings?
7. browse network                  # failed requests?
8. Assertions:
   - Zielzustand erreicht?
   - Keine Console-Errors?
   - Erwartete Network-Calls erfolgreich?
   - Visuelles Verhalten korrekt (Screenshot-Diff)?
```

**Cleanup:**
- Test-User abmelden / Session beenden
- Wenn Test-User-Daten erstellt: löschen
- Browser-Cookies/LocalStorage des Test-Tabs leeren

**Cleanup-Verify:**
- Live-Session prüfen dass Test-Marker nicht auftauchen

---

### DB-Bug (Query-Fehler, Schema-Issues, Datenintegrität)

**Sandbox-Setup:**
- Wenn Test-DB / Sandbox-Branch verfügbar (Supabase Branches!): dort
- Sonst: Test-Schema oder Test-Tabellen mit Prefix `__smoketest_*`
- Nie auf Live-Daten testen

**Test-Sequenz:**
```sql
-- Setup
INSERT INTO __smoketest_users (id, ...) VALUES ('__smoketest_<bug-id>_1', ...);

-- Reproduce
<die fragliche Query / Operation>

-- Assertions
SELECT * FROM __smoketest_users WHERE id LIKE '__smoketest_<bug-id>_%';
-- erwartetes Verhalten verifizieren
```

**Cleanup:**
```sql
DELETE FROM __smoketest_users WHERE id LIKE '__smoketest_<bug-id>_%';
-- ggf. cascading deletes für abhängige Tabellen
```

**Cleanup-Verify:**
```sql
SELECT count(*) FROM __smoketest_users WHERE id LIKE '__smoketest_*';
-- muss 0 sein

-- alle anderen Test-Tabellen auch checken
```

---

### Backend-Logic-Bug (interne Funktion, Algorithmus, State-Maschine)

**Sandbox-Setup:**
- Unit-Test schreiben falls Test-Setup vorhanden
- Sonst: isoliertes Script `__smoketest_<bug-id>.py` / `.ts` das die Funktion mit Test-Inputs aufruft

**Test-Sequenz:**
```typescript
// __smoketest_<bug-id>.ts
import { functionUnderTest } from './...';

const testCases = [
  { input: <bug-szenario>, expected: <korrekt> },
  { input: <edge-case-1>, expected: <korrekt> },
  { input: <edge-case-2>, expected: <korrekt> },
];

for (const tc of testCases) {
  const result = functionUnderTest(tc.input);
  console.assert(deepEqual(result, tc.expected), `FAIL: ${JSON.stringify(tc)}`);
}
```

**Cleanup:**
- `__smoketest_<bug-id>.ts` Datei löschen
- Falls Tests dauerhaft sein sollen: in normales Test-Verzeichnis verschieben + Marker entfernen + commiten

**Cleanup-Verify:**
```bash
ls -la | grep __smoketest_  # muss leer sein
```

---

### n8n-Workflow-Bug

**Sandbox-Setup:**
- n8n hat Test-Workflow-Mode: Workflow kopieren als `__smoketest_<original-name>`, dort fixen + testen
- Triggers manuell ausführen (nicht automatisch)
- Wenn Webhook-basiert: Test-Webhook-URL nutzen, nicht Production-URL

**Test-Sequenz:**
- `n8n-mcp` validate_workflow → strukturelle Validierung
- Manual Execute mit Dummy-Payload
- Execution-Log prüfen: alle Nodes grün? Welcher Output?

**Cleanup:**
- Test-Executions in n8n löschen
- Falls Test-Workflow-Kopie: löschen
- Falls Test-Credentials erstellt: entfernen

**Cleanup-Verify:**
- `__smoketest_*` Workflows in n8n-Liste → muss leer sein
- Execution-Liste prüfen dass keine `__smoketest_*` Executions hängen

---

### Mobile-App-Bug

**Sandbox-Setup:**
- ADB für Android (`adb-android-control`) — Test auf eigenem Device
- iOS: Simulator wenn vorhanden
- Test-Account verwenden, nicht Live-Account

**Test-Sequenz:**
- App auf Test-State setzen
- Reproduce Bug
- Screenshot + Logcat-Analyse
- Verifiziere Fix-Behavior

**Cleanup:**
- App-Daten via ADB löschen (`pm clear <package>`)
- Test-Account-Daten zurücksetzen

---

### Build/Deploy/CI-Bug

**Sandbox-Setup:**
- Lokaler Build / Container-Build statt CI-Run wenn möglich
- Wenn CI: Branch mit `__smoketest_<bug-id>` Prefix, CI dort laufen lassen

**Test-Sequenz:**
- Pre-Build-Check: Dependencies installierbar?
- Build-Run mit Verbose-Logs
- Post-Build: Artefakte vorhanden + valide?
- Wenn Deploy: Smoke-Endpoint auf Staging treffen

**Cleanup:**
- `__smoketest_*` Branches löschen
- Test-Container stoppen + removen
- Staging-Artefakte ggf. zurückrollen

---

## Smoke-Test-Log (Pflicht-Doku pro Bug)

Pro Bug wird ein Smoke-Test-Log geschrieben unter:
`.bug-sweep/smoke-tests/<bug-id>-smoke-log.md`

Template: [../templates/smoke-test-log.md](../templates/smoke-test-log.md)

Pflicht-Felder:
- Bug-ID + Titel
- Test-Strategie (welche aus diesem Doc)
- Sandbox-Setup-Details
- Dummy-Daten erstellt (Liste mit Markern)
- Test-Schritte + Outputs
- Assertions-Ergebnisse
- Iteration-Count (1, 2, 3)
- Cleanup-Aktionen
- Cleanup-Verify-Beweis
- Council-Verdict (wenn benutzt)
- Status: ✅ passed / ❌ failed (mit Grund)

---

## Wann der Smoke-Test "passed" ist (Definition)

ALLE diese Kriterien müssen erfüllt sein:

- [x] Bug-Szenario wurde im Sandbox-Setup reproduziert
- [x] Fix wurde dort verifiziert (Verhalten korrekt)
- [x] Alle definierten Assertions grün
- [x] Keine Side-Effects auf andere Bereiche erkannt
- [x] Cleanup wurde durchgeführt
- [x] Cleanup-Verify zeigt 0 Test-Reste
- [x] Live-Version (wenn schon deployed) zeigt keine Test-Daten
- [x] Council-Verdict (wenn benutzt) ist "PASS"

Wenn auch nur EIN Kriterium nicht erfüllt → ❌ failed → Iteration

---

## Failure-Eskalation

**Nach 3 fehlgeschlagenen Smoke-Test-Iterationen:** STOP. Skill macht User-Handoff:

```
🚨 Smoke-Test 3× nicht bestanden für Bug #<id>

Was wurde versucht:
1. Iteration 1: <was probiert, was fehlte>
2. Iteration 2: <was probiert, was fehlte>
3. Iteration 3: <was probiert, was fehlte>

Mögliche Optionen:
A) Andere Fix-Strategie — neuer Root-Cause-Hypothese
B) Bug als "blocked" markieren — andere Bugs zuerst, später zurückkommen
C) Externe Hilfe — Codex-Consult oder Council-Deep-Dive
D) Manueller User-Eingriff — User testet selbst und gibt Feedback

Welche Option?
```

---

## Anti-Patterns

- ❌ "Test ist grün, wahrscheinlich passt's auch in Live" — NEIN, Smoke-Test MUSS originalgetreu sein
- ❌ Test-Daten im Code-Repo committen mit echtem Marker → könnte später aktiviert werden
- ❌ Cleanup "vergessen" oder "kann später" → NEIN, sofort
- ❌ Cleanup-Verify überspringen weil "vertraue ich" → NEIN, immer explizit prüfen
- ❌ Bei Smoke-Test-Fail: Bug trotzdem abhaken weil "müsste eigentlich gehen" → NEIN, Fail = Fail = Iteration oder Handoff
