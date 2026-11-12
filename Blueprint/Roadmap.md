- Design.md ist der Volltext über den gesamten Bauplan.
- Hier hin kommt was noch offen oder unreleased ist. 
- Was erledigt ist wird aus Roadmap.md entfernt und nach Changelog.md verschoben
-
---


1. Supply-Chain-Agent (Hintergrund/Zeitplan): Hält Treiber, Pakete, Silent-Flags und Gegenproben aktuell und verifiziert sie vorab.
2. Aegis-Agent (Ereignisgesteuert): Baut daraus in Sekunden maßgeschneiderte Rollouts und heilt Störungen im Live-Betrieb.

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

Supply-Chain-Agent

Der Katalog-Agent, der  strikt in einer Quarantäne-Pipeline (Staging) arbeitet.

• Strikte Trennung zwischen Produktion und Staging: Der Katalog-Agent hat als einziger (über ein kontrolliertes Gateway) Zugriff auf Hersteller-Quellen, Winget-/Apt-Metadaten oder Vendor-Feeds. Der eigentliche Deployment-Agent (Aegis) sieht weiterhin nur den fertigen, signierten Produktiv-Katalog (Regel R3).

• Automatisierte Paket-Aufbereitung: Der Katalog-Agent erkennt neue Versionen, lädt sie in eine isolierte Test-Sandbox, ermittelt Silent-Install-Parameter, baut die passende Funktions-Gegenprobe (Regel R2) und prüft das Ergebnis in einer Wegwerf-VM.

• Kryptografische Übergabe: Erst wenn Installation und Gegenprobe in der Test-VM grün sind, berechnet die Pipeline den SHA-256-Hash, signiert das Paket und verschiebt es in den produktiven Katalog (optional mit kurzem Human-in-the-Loop-Approve für kritische Kernsoftware).

---

