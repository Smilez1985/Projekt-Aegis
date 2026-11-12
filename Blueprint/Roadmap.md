# Projekt Aegis — Roadmap

> **Hinweis:** `design.md` enthält den Gesamtbauplan. Die Roadmap erfasst fokussiert offene und unreleased Punkte (Erledigtes wird nach `CHANGELOG.md` verschoben). Die vollständigen technischen Details zu den einzelnen Modulen und Repositories können in den jeweiligen Quellprojekten nachgelesen werden.

---

## 1. Kern-Architektur & Basis-Bausteine (Integration)

### Externe Abhängigkeiten (Dogfooding → Aegis-Integration)
- **RAG-Gedächtnis & Hardware-Probe:** Anbindung der `helpdesk_runbooks` über `rag4ll` sowie Standardisierung der Offline-SMBIOS/PCI-Erkennung (`LLM_Cross-Compiler_Framework` / `rag4ll`).
- **Wake-on-Agent & Sandbox:** Integration von Event-Triggern über `woa-l` sowie strikte Positivlisten und Gateway-Kapselung über `chimera` (`sohelper`, Tor-Prinzip).

### PXE-Boot & Zustandsverwaltung
- **Bootstrap-Vertrauen & Positivlisten:** Kryptografische Übergabe des Einmal-Kopplungscodes nach dem OS-Boot (Phase 3.1) sowie sichere Lese-Helfer mit automatischer Redaction sensibler Daten (`CBS.log`, `journalctl`, MSI-Logs nach Regel R1 & Phase 3.4).
- **Zustandsverwaltung bei Reboots:** Atomares Tracking des Installationsfortschritts in `plan.json` zwischen Zielsystem und Gateway zur lückenlosen Wiederaufnahme nach Neustarts.

---

## 2. Zweistufige Agenten-Fabrik

### Supply-Chain-Agent (Hintergrund / Staging)
- **Quarantäne & Paket-Aufbereitung:** Isolierte Trennung von Hersteller-Quellen (Regel R3), automatisierte Test-Sandbox-Prüfungen inklusive Silent-Install-Parametern, Funktions-Gegenproben (Regel R2) sowie SHA-256-Signierung via `PySignBuilder`.

### Aegis-Agent (Ereignisgesteuert)
- **Dynamischer Rollout & Self-Healing:** Zusammenstellung maßgeschneiderter Deployments aus dem verifizierten Katalog sowie autonome Störungsbehebung mit Funktionsnachweis.

---

## 3. Mobile Schnittstelle & Air-Gapped Notfallknoten (`PiPortal`)

- **Mobiler PXE-Server & Offline-Helpdesk:** Lokale Bereitstellung von iPXE-Boot-Images, Hardware-Probes, einem Paket-Katalog-Subset (Regel R3) sowie lokalem `rag4ll`-Runbook-Cache für isolierte Systeme.
- **Physisches Genehmigungs-Gate:** Einsatz von `PiPortal` als vertrauenswürdiges Terminal zur Freigabe destruktiver Eingriffe vor Ort (Regel R5).
