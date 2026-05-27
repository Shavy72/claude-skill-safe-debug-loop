# Bug-Plan — <modul-name> — <ISO-Date>

**Modul / Sektion:** `<pfad oder UI-Bereich>`
**Live-URL (wenn vorhanden):** `<url>`
**Lokale Dev-URL:** `<url>`
**Research-Basis:** `.bug-sweep/research/`
**Erstellt durch:** safe-debug-loop Phase 1
**Confidence-Level:** 100% (Council-validated | self-asserted — eines davon explizit)
**Start-Datum:** `<YYYY-MM-DD HH:MM>`

---

## Bug-Summary

| Kategorie | Anzahl |
|---|---|
| 🔴 Rot — Dringend (MVP-Blocker / Sicherheit / Daten-Korruption) | N |
| 🟡 Gelb — Wichtig (UX-Störung / Edge-Case / fehlendes Logging) | N |
| 🟢 Grün — Vernachlässigbar (Code-Quality / Cosmetic) | N |
| **Gesamt** | N |

---

## Empfohlene Fix-Reihenfolge

1. **#R1** — `<titel>` (Root-Bug, mehrere andere hängen davon ab)
2. **#R2** — `<titel>`
3. **#R3** — `<titel>` (abhängig von #R1)
4. ...

Begründung der Reihenfolge:
- Root-Bugs zuerst (entlasten ggf. abhängige Bugs automatisch)
- Hohe Kategorie vor niedriger
- Bei gleicher Kategorie: kleinerer Blast-Radius zuerst

---

## Bug-Liste (vollständig)

### 🔴 R1 — `<bug-titel>`

- **Status:** [ ] open
- **Kategorie:** Rot — Dringend
- **Symptom:** <was beobachtet — möglichst konkret mit Beweis-Pfad/Screenshot/Log>
- **Surface:** <wo tritt das auf — URL/Endpoint/Funktion>
- **Vermutete Root-Cause:** `<file>:<line>` — <was scheint kaputt>
- **Recherche-Referenz:** <verweis auf .bug-sweep/research/...>
- **Fix-Plan (minimal-invasiv):**
  - <Schritt 1>
  - <Schritt 2>
  - <Schritt 3>
- **Abhängigkeiten:** Bug #X muss vorher gefixt sein / unabhängig
- **Blast-Radius:** <welche anderen Stellen könnten vom Fix betroffen sein>
- **Smoke-Test-Strategie:** <welche aus references/smoke-test.md — API/UI/DB/Logic/etc>
- **Logging-/Doku-Upgrade-Plan:** <was wird mit diesem Fix verbessert>
- **Estimate:** <S/M/L>
- **Council-Verdict (wenn validiert):** <PASS / NEEDS_REWORK>

---

### 🔴 R2 — `<bug-titel>`

(gleiche Struktur)

---

### 🟡 Y1 — `<bug-titel>`

(gleiche Struktur)

---

### 🟢 G1 — `<bug-titel>`

(gleiche Struktur)

---

## Cross-Cutting-Findings

Bugs die nicht eindeutig zuordenbar sind aber architektonisch beobachtet wurden:

- **Logging-Konsistenz:** <z.B. einige Module loggen mit `console.log`, andere mit strukturiertem Logger>
- **Error-Handling-Pattern:** <z.B. Try-Catch ohne spezifische Errors>
- **API-Versionierung:** <z.B. teilweise v1 v2 gemischt>
- **Naming-Konventionen:** <z.B. snake_case vs camelCase Mix>

→ Diese werden bei den jeweiligen Bug-Fixes mit-adressiert (Logging-Upgrade-Plan).

---

## Externe Abhängigkeiten + Risiken

- **API-Provider X:** Doku-Version `<version>` — bei Breaking-Changes diese Bugs neu prüfen
- **Library Y:** Aktuelle Version `<v>` — wenn upgegradet werden muss, ggf. Folge-Bugs
- **Auth-Provider:** `<name>` — Auth-bezogene Bugs erfordern Test-Account

---

## Persistenz

- Bug-Plan-File: `.bug-sweep/bug-plan-<ISO-date>.md` (diese Datei)
- Research-Files: `.bug-sweep/research/`
- Smoke-Test-Logs: `.bug-sweep/smoke-tests/<bug-id>-smoke-log.md` (kommen in Phase 2)
- Obsidian-Spiegelung: `Obsidian/projects/<projekt>/safe-debug-loop/` (falls Vault vorhanden)

---

## Phase-2-Übergang

**Start-Prompt für Phase 2 Session (Copy-Paste):**

```
/safe-debug-loop Phase 2

Bug-Plan: .bug-sweep/bug-plan-<ISO>.md
Research: .bug-sweep/research/

Bitte arbeite den Plan iterativ durch — einen Bug nach dem anderen,
mit Smoke-Test pro Fix. Holistik-Check vor jedem Fix.
```

---

**Phase 1 abgeschlossen.** Phase 2 übernimmt von hier.
