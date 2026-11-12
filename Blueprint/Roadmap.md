- Design.md ist der Volltext über den gesamten Bauplan.
- Hier hin kommt was noch offen oder unreleased ist. 
- Was erledigt ist wird aus Roadmap.md entfernt und nach Changelog.md verschoben
-
---

PXE-Boot:

• Bootstrap-Vertrauen nach dem ersten OS-Boot (Phase 3.1): Der Übergang vom Live-System (WinPE / Alpine) zum minimalen Bootstrap-Dienst im installierten Ziel-OS braucht eine saubere kryptografische Übergabe des Einmal-Kopplungscodes aus Phase 1, damit sich kein fremder Host am Gateway als der frisch installierte Client ausgeben kann.

• Positivliste vs. Log-Analyse (Regel R1 & Phase 3.4): Wenn der Agent bei Fehlern gezielt Log-Abschnitte (CBS.log, journalctl, MSI-Logs) liest, muss der sohelper-Ausführungsdienst sauber begrenzte Lese-Helfer mit Redaction (Schwärzung sensibler Daten) bereitstellen, damit keine zufällig geloggten Geheimnisse ins LLM wandern.

• Zustandsverwaltung bei Reboots: Da Treiber- und OS-Installationen oft mehrere Neustarts erfordern, muss plan.json den Fortschritt („Step 3 von 12 erledigt“) atomar auf dem Zielsystem oder im Gateway tracken, damit der Agent nach dem woa-l-Weckruf exakt an der richtigen Stelle weitermacht.

---

RAG: siehe repo RAG4LL

-
-
---

Hardware_Probe: siehe LLMCF & RAG4LL

-
-
---

Wake on Agent - Loop: siehe Repo wao-l

-
-
---
