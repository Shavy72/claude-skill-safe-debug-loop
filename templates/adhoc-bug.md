# Adhoc-Bug-Eintrag — Phase 2 Standalone (Modus C2)

**Verwendung:** Wenn der User `/safe-debug-loop Phase 2` mitten in einem aktiven Chat aufruft und kein Phase-1-Bug-Plan existiert. Der Skill legt diese Datei an unter:

`.bug-sweep/adhoc-bugs-<ISO-date>.md`

---

# Adhoc-Bug-Liste — <ISO-Date>

**Erstellt durch:** safe-debug-loop Phase 2 (Modus C2 — Mitten-im-Chat)
**Aktive Session:** `<session-id falls erkennbar>`
**Kontext-Quelle:** Letzte N Chat-Nachrichten + ggf. Code-Stand

---

## Erkannte Bugs aus Chat-Kontext

### A1 — `<bug-titel-aus-chat-erkannt>`

- **Status:** [ ] open
- **Quelle:** <z.B. "Nutzer erwähnte am <zeit>: 'X funktioniert nicht'" oder "Frühere Reasoning-Antwort identifizierte X als Bug">
- **Symptom:** <was wurde im Chat beschrieben>
- **Surface:** <wo — wenn aus Chat ableitbar>
- **Vermutete Root-Cause:** <falls schon im Chat reasoned, sonst "tbd in Step 2">
- **Fix-Plan (minimal-invasiv):** <falls schon vorhanden, sonst "tbd in Step 4">
- **Blast-Radius:** <falls erkennbar>
- **Smoke-Test-Strategie:** <Vorschlag — verfeinert in Phase 2>
- **Logging-/Doku-Upgrade-Plan:** <falls jetzt schon klar>

---

### A2 — `<weiterer-bug-falls-mehrere>`

(gleiche Struktur — nur befüllen wenn mehrere Bugs im Chat-Kontext waren)

---

## Notizen

- Adhoc-Bug-Files haben weniger Tiefe als Phase-1-Bug-Plans — die Research-Basis fehlt
- Wenn ein Adhoc-Bug komplexer wird als gedacht: Vorschlag an User, eine vollständige Phase 1 nachzuschalten
- Wenn nach Phase 2 noch Bugs offen sind: User kann optional eine Phase 1 starten um eine vollständige Liste zu bekommen

---

## Phase-2-Loop läuft jetzt

Skill startet mit Step [1] für den ersten Bug aus dieser Liste.
