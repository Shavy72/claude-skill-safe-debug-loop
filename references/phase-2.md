# Phase 2 — Iterativer Bug-Fix-Loop mit Smoke-Test-Gate

**Interaktiv, mit Copy-Paste-Prompts an User. Ein Bug nach dem anderen. Smoke-Test als Quality-Gate. Cleanup-Pflicht.**

Phase 2 fixt Bugs einzeln, sicher und nachhaltig. Jeder Loop-Durchlauf verbessert auch Logging + Doku in dem betroffenen Code-Bereich.

---

## Zwei Sub-Modi

### Modus C1 — Frische-Session (klassisch nach Phase 1)

User hat eine frische MAX-effort-Session mit `/safe-debug-loop Phase 2` gestartet, dann den Phase-1-Start-Prompt + MD-File-Path eingefügt.

- Skill liest die Bug-Plan-MD
- Skill liest die Research-Files in `.bug-sweep/research/`
- Loop läuft über alle Bugs in der Reihenfolge des Plans (priorisiert nach Kategorie + Abhängigkeit)

### Modus C2 — Mitten-im-Chat

User arbeitet schon in einem aktiven Chat. Es gibt entweder:
- **Einen konkreten Bug im Raum:** im Chat erwähnt, in einem Reasoning, in einer Task-Definition
- **Einen Plan im Raum:** eine Liste mehrerer Bugs/Issues

User ruft `/safe-debug-loop Phase 2` ohne Argument auf.

- Skill scannt den letzten Chat-Verlauf nach Bugs/Plänen
- Bei 1 konkretem Bug erkannt: Loop läuft für diesen 1 Bug
- Bei Plan erkannt: Loop läuft über die erkannte Liste
- Bei Unklarheit: kurze Rückfrage "Welcher Bug / Plan soll bearbeitet werden?"
- Wenn keine MD-File existiert: Skill legt spontan eine an unter `.bug-sweep/adhoc-bugs-<ISO>.md` (Template [../templates/adhoc-bug.md](../templates/adhoc-bug.md))
- Ab dann: identischer Loop wie Modus C1

---

## Der Loop (Sequenz pro Bug)

### [1] Tool-Symbiose-Check

Vor jedem Bug-Loop wird das Skill-/MCP-/Plugin-Inventar neu gecheckt (siehe [tool-symbiose.md](tool-symbiose.md)). Welche Tools helfen bei genau diesem Bug-Typ?

### [2] LLM-Reasoning: Bug-Auswahl + Existenz-Check (KOMBINIERT)

**Interner Reasoning-Prompt (das LLM handelt direkt nach diesem Schema, keine User-Frage nötig):**

> "Wenn ich heute nur EINEN Bug aus dem Bug-Plan fixen dürfte/müsste, welcher wäre das? Bevor ich antworte, prüfe ich kurz aus dem aktuellen Code-Stand ob der Bug überhaupt noch besteht — denn durch frühere Fixes (oder Refactors zwischen Sessions) könnte er bereits indirekt mit-gefixt worden sein. Wenn ja → markiere ihn als 'obsolet — bereits gefixt' und wähle den nächsten."

**Konkrete Verifikations-Aktionen für Existenz-Check:**
- Read der Files die der Bug-Eintrag referenziert
- Grep nach den Symptom-Patterns (z.B. fehlerhafte API-Calls, fehlende Error-Handler)
- Wenn UI-Bug: kurzer Browser-Check via `browse` oder `playwright` (read-only, kein State-Change)
- Wenn Logic-Bug: kurzer Dry-Run mit Testdaten (ohne Persistenz)

**Output Step [2]:**

```markdown
## Bug-Auswahl: #<id> — <titel>

**Existenz-Check:**
- [x] Code-Stelle untersucht: <file:line>
- [x] Symptom reproduziert / Pattern bestätigt
- [x] Bug besteht noch

**Begründung Auswahl:**
- Höchste Priorität (Kategorie 🔴 Rot)
- Andere Bugs hängen davon ab (R3, R5 sind Folge-Bugs von #R1)
- Minimal-invasiv lösbar

→ Weiter mit 12-Jährigen-Erklärung
```

**Wenn der Bug bereits gefixt ist:**

```markdown
## Bug-Auswahl: #<id> — <titel>

**Existenz-Check:**
- [x] Code-Stelle untersucht: <file:line>
- [x] Bug ist NICHT mehr reproduzierbar — wurde indirekt durch Fix #X gefixt

**Aktion:**
- Bug #<id> in MD-File markiert als "obsolet — bereits gefixt durch Folge-Effekt"
- → Loop zurück zu Step [1] mit nächstem Bug
```

### [3] Skill schlägt nächsten Copy-Paste-Prompt vor

Am Ende von Step [2] zeigt der Skill dem User einen Copy-Paste-Block:

```
╔══════════════════════════════════════════════════════════╗
║  NÄCHSTER PROMPT — kopier in den Chat:                   ║
║  ─────────────────────────────────────────────────────   ║
║  Erkläre mir diesen einen Bug, den du fixen würdest,     ║
║  für einen 12-Jährigen verständlich. Erkläre es mit dem  ║
║  wie du mir als User am besten erklären würdest laut     ║
║  deiner Erfahrung aus der Zusammenarbeit mit mir und     ║
║  nutze ein Schaubild aus der echten Welt. Stell dir vor  ║
║  ich bin 12, schwer von Begriff und kann den Bug nicht   ║
║  verstehen momentan. Antworte in einfacher Sprache für   ║
║  maximale Verständlichkeit.                              ║
║                                                          ║
║  Erkläre:                                                ║
║  1. Was der Bug macht / nicht macht                      ║
║  2. Warum das ein großes Problem ist (auch global im     ║
║     Dashboard, nicht nur lokal)                          ║
║  3. Was der Fix genau machen wird                        ║
║  4. Warum der Fix minimal-invasiv ist und keine anderen  ║
║     Teile des Dashboards kaputt machen wird              ║
║  ─────────────────────────────────────────────────────   ║
╚══════════════════════════════════════════════════════════╝
```

User kopiert, sendet ab. LLM antwortet mit der einfachen Erklärung.

### [4] Skill schlägt Fix-Ausführungs-Prompt vor

Nach der 12-Jährigen-Erklärung zeigt der Skill den nächsten Copy-Paste-Block:

```
╔══════════════════════════════════════════════════════════╗
║  NÄCHSTER PROMPT — kopier in den Chat:                   ║
║  ─────────────────────────────────────────────────────   ║
║  Bitte führe diesen einen Bugfix aus, minimal invasiv.   ║
║  Sorge dafür dass du 100% bewusst handelst und keine     ║
║  anderen Code-Teile unseres Dashboards mit deinem Fix    ║
║  kaputt machen wirst.                                    ║
║                                                          ║
║  Pflicht-Schritte:                                       ║
║  1. Pre-Check: lies alle Stellen die diese Funktion /    ║
║     diesen Endpoint / dieses State-Feld nutzen           ║
║  2. Fix anwenden (Edit/Write — kleinster Diff)           ║
║  3. Atomic Commit: fix(<scope>): <bug-titel>             ║
║  4. ISOLATED SMOKE-TEST (siehe references/smoke-test.md) ║
║  5. Cleanup aller Dummy-Daten nach Smoke-Test            ║
║  6. Logging/Doku/Code-Kommentare für diesen Bereich      ║
║     verbessern (damit zukünftige Bugs hier besser        ║
║     auffindbar sind)                                     ║
║  7. Bug in .bug-sweep/bug-plan-*.md abhaken              ║
║                                                          ║
║  Wenn ein Schritt unsicher ist oder Risiko besteht andere║
║  Teile zu brechen: STOP und frag mich.                   ║
║  ─────────────────────────────────────────────────────   ║
╚══════════════════════════════════════════════════════════╝
```

### [5] LLM führt Fix aus + Smoke-Test (Quality-Gate)

Der LLM macht:

1. **Pre-Check** — Holistik-Scan
   - Welche anderen Files/Stellen referenzieren die Funktion/den Endpoint/den State?
   - Welche Tests laufen darüber (falls vorhanden)?
   - Welche aktiven Sessions / aktiven User könnten betroffen sein?

2. **Fix-Anwendung** — minimaler Diff
   - Edit/Write nur die nötigen Stellen
   - Kein adjacent-Refactor
   - Conventional-Commit-Stil

3. **Atomic Commit** (wenn Repo aktiv)
   - `fix(<scope>): <bug-titel>`
   - Body referenziert Bug-ID + Root-Cause + Verifikation

4. **ISOLATED SMOKE-TEST** — siehe [smoke-test.md](smoke-test.md) für Strategien
   - Sandbox-Setup originalgetreu
   - Dummy-Daten / Mocks
   - Reproduziert das ursprüngliche Bug-Szenario
   - Verifiziert dass Fix greift
   - Wenn nicht 100% verifiziert: automatische Iteration (max 3 Versuche)
   - Wenn verifiziert: **Cleanup-Pflicht**

5. **Cleanup-Beweis** — explizit dokumentiert
   - Welche Dummy-Daten wurden erstellt → entfernt? ✅
   - Welche Test-Files → cleared? ✅
   - Welche Test-Endpoints/Mocks → disconnected? ✅
   - Live-Version hat KEINE Test-Reste → bestätigt ✅

6. **Logging/Doku/Code-Kommentar-Upgrade**
   - In diesem Code-Bereich Logging detaillierter machen
   - Inline-Kommentare ergänzen wo Verhalten subtil ist
   - README / Doku updaten wenn betroffen
   - Ziel: zukünftige Bugs in diesem Bereich sind in Live-Version vom LLM detailliert prüfbar

7. **Bug-Plan-MD updaten**
   - Bug-Status: ✅ done
   - Beweis: Commit-Hash + Smoke-Test-Log-Pfad
   - Welche Logging/Doku-Verbesserungen wurden gemacht

8. **Live-Schaltung (wenn anwendbar)**
   - Wenn Modul schon live war: Fix wird live deployt (merge / push / re-deploy je nach Setup)
   - Wenn nur lokal: bleibt lokal
   - Post-Live-Check: kurz dass Live-Version den Fix zeigt

9. **Council-Optional**
   - Wenn `agent-council` installiert: kurzer Council-Check ob der Fix solide ist
   - Council-Verdict wird im Smoke-Test-Log dokumentiert

### [6] Zusammenfassung an User

Der LLM antwortet mit:

```markdown
## ✅ Bug #<id> gefixt

**Was wurde gemacht:** <kurze technische Zusammenfassung>
**Commit:** <hash>
**Smoke-Test:** ✅ passed (siehe .bug-sweep/smoke-tests/<id>-smoke-log.md)
**Cleanup:** ✅ verifiziert
**Logging-Upgrade:** <was wurde verbessert>
**Doku-Upgrade:** <was wurde verbessert>

### Warum dieser Fix gut ist
<Erklärung warum technisch solide, warum minimal-invasiv>

### Was zukünftig dadurch besser ist
<Erklärung welche zukünftigen Bug-Klassen jetzt schneller erkennbar sind>

### Was du als User langfristig beobachten sollst
<konkrete Anhaltspunkte: Logs, Metriken, Verhalten — damit User selbst sehen kann ob Fix langfristig hält>
```

### [7] /compact-Vorschlag

Am Ende der Antwort:

```
╔══════════════════════════════════════════════════════════╗
║  NÄCHSTER SCHRITT — optional:                            ║
║  ─────────────────────────────────────────────────────   ║
║  /compact Focus: bug-plan, nächste Bugs aus Phase 1      ║
║                                                          ║
║  Empfehlung: jetzt /compact ausführen damit Context      ║
║  für nächste Bug-Iteration frei wird. Der Skill liest    ║
║  den Bug-Plan beim nächsten Durchgang neu ein.           ║
║  ─────────────────────────────────────────────────────   ║
╚══════════════════════════════════════════════════════════╝

Wenn du bereit bist für den nächsten Bug, schick einfach
"weiter" oder einen leeren /safe-debug-loop Phase 2 Aufruf —
ich starte dann automatisch die nächste Loop-Iteration.
```

### [8] Nächste Iteration

User schickt "weiter" oder Loop wird automatisch fortgesetzt. Skill springt zurück zu Step [1] mit dem nächsten Bug aus dem Plan.

---

## Loop-Ende (alle Bugs durch)

Wenn die Bug-Liste komplett abgearbeitet ist (alle ✅ oder als "obsolet" markiert):

```markdown
## 🎉 Phase 2 Abgeschlossen

**Bugs gefixt:** N (davon X 🔴, Y 🟡, Z 🟢)
**Bugs als obsolet markiert (bereits gefixt):** M
**Commits:** N atomic Commits
**Logging-Upgrades:** in K Code-Bereichen verbessert
**Doku-Upgrades:** in J Files ergänzt

### MVP-Readiness-Check
<Skill bewertet: ist das Modul jetzt MVP-tauglich? Welche Risiken bleiben?>

### Empfohlene nächste Schritte
- [ ] Live-Deploy (wenn noch nicht passiert)
- [ ] Beta-Test mit kleinem User-Kreis
- [ ] Monitoring der neuen Logging-Pfade in den ersten 24h
- [ ] Optional: weitere Phase-1-Iteration in 2-4 Wochen für neue Erkenntnisse aus Live-Daten

### Persistierte Wissens-Basis
- `.bug-sweep/bug-plan-<ISO>.md` (komplett durchgearbeitet)
- `.bug-sweep/smoke-tests/` (alle Smoke-Test-Logs)
- `.bug-sweep/research/` (Wissens-Basis für künftige Sessions)
- Obsidian-Vault (gespiegelt, wenn vorhanden)
```

---

## Eskalations-Regeln im Loop

- **Smoke-Test 3× nicht bestanden für selben Bug:** STOP, User-Handoff mit Frage "A) andere Fix-Strategie B) Bug als 'blocked' markieren C) externe Hilfe (Codex / Council)"
- **Blast-Radius >5 Files:** STOP, User-Bestätigung holen
- **Live-Deploy schlägt fehl:** STOP, Rollback prüfen, User informieren
- **Cleanup-Verifikation schlägt fehl:** STOP, manuelles Aufräumen erforderlich, kein Bug abhaken bevor Cleanup 100%

---

## Was Phase 2 NIE tut

- ❌ Mehrere Bugs gleichzeitig fixen
- ❌ Bug abhaken ohne Smoke-Test-Bestätigung
- ❌ Test-Reste in Live-Version zurücklassen
- ❌ Fix ohne Pre-Check (Holistik) anwenden
- ❌ Refactor angrenzender Stellen ohne expliziten User-OK
- ❌ Mehrere Bugs in einem Commit
- ❌ Bug als gefixt deklarieren ohne Beweis (Smoke-Test-Log)
