# Aegis — Autonomes Zero-Touch-Deployment & Self-Healing Helpdesk

*English version: [README.md](README.md)*

**Ein ereignisgesteuertes Deployment- und Support-System, das Hardware erkennt, maßgeschneiderte Arbeitsplätze ausrollt, Störungen anhand echter Funktionsproben selbst heilt und aus jeder gelösten Aufgabe lernt.**

Status: **Konzept / Entwurf.** Nichts ist gebaut, nichts ist getaggt. Dieses Projekt parkt bewusst, bis **Chimera**, **woa-l**, **rag4ll** und **LLMCF** abgeschlossen sind. Die tragenden Entscheidungen stehen in [`docs/DESIGN.md`](docs/DESIGN.md).

---

## Was es tut

Klassische Deployment- und Client-Management-Systeme (wie SCCM) arbeiten mit starren Task-Sequenzen und verlassen sich oft auf bloße Exitcodes. **Aegis** dreht dieses Prinzip um: Ein abgesicherter KI-Agent baut den Installationsplan dynamisch aus der realen Hardware-Probe des Zielgeräts zusammen, überwacht jeden Schritt über echte Funktions-Gegenproben und speichert jede verifizierte Lösung dauerhaft im Gedächtnis.

Zwei Betriebsarten greifen ineinander:

1. **Zero-Touch-Rollout (PXE → fertiger Arbeitsplatz):** Ein Client bootet per PXE in ein minimales Live-System (WinPE oder Linux), führt die Hardware-Probe aus und weckt den Agenten. Dieser baut aus signierten Paketen und Treibern den passenden Installationsplan, überwacht Treiber, Software (Office, AutoCAD, VLC, WinRAR etc.) und OS-Updates und richtet das Benutzerprofil (Outlook, Netzlaufwerke, Lesezeichen) ohne Klartext-Passwörter über zeitlich begrenzte Einmal-Tokens ein.
2. **24/7/365 Self-Healing Helpdesk:** Eingehende Tickets oder Störungsmails wecken den Agenten ereignisgesteuert. Er gleicht das Fehlerbild gegen bekannte Runbooks ab, verbindet sich per WinRM, SSH oder PowerShell Direct mit dem Client, behebt die Störung, belegt den Erfolg durch eine Gegenprobe und schließt das Ticket.

---

## Kernprinzipien

* **Ereignisgesteuert statt Dauer-Polling (`woa-l`):** Der Agent existiert zwischen zwei Aufgaben nicht als teurer Dauerprozess. Eingehende PXE-Proben, Boot-Rückmeldungen oder Helpdesk-Tickets wecken ihn gezielt für genau einen Turn.
* **Erfahrungsgedächtnis mit Skill-Beförderung (`rag4ll`):** Nur grün geprüfte Lösungen landen als Runbook im Vektor- und BM25-Speicher. Tritt dasselbe Fehlerbild mehrfach auf (`N >= 10`), verdichtet der Agent die Lösung in einen deterministischen `SKILL.md`-Ablauf – danach braucht das Problem keine KI-Überlegung mehr.
* **Funktionsnachweis statt Exitcode (Regel R2):** Ein Installations- oder Reparaturschritt gilt erst als erledigt, wenn das Programm oder der Dienst nachweislich läuft.
* **Harte Privilegientrennung (Regel R1 & R4):** Der Agent hat weder Internetzugriff noch Domain-Admin-Rechte und sieht niemals Benutzerpasswörter. Befehle laufen über abgesicherte Ausführungs-Helfer mit Positivliste, destruktive Eingriffe (wie PXE-Wipe) erfordern menschliche Freigabe.

---

## Zusammenspiel mit dem Ökosystem

Aegis erfindet das Rad nicht neu, sondern verbindet vier bestehende Bausteine zu einer geschlossenen Enterprise-Plattform:

| Baustein | Herkunft | Aufgabe in Aegis |
| :--- | :--- | :--- |
| **`woa-l` (woac)** | Eigenes Projekt | Weckt den Agenten bei PXE-Boot, Neustarts und eingehenden Tickets. |
| **`rag4ll`** | Eigenes Projekt | Speichert verifizierte Runbooks und liefert bekannte Fixes vor jedem Eingriff. |
| **`hardware_probe`** | `LLMCF` / `rag4ll` | Liefert CPU-Flags, PCI-IDs, GPU, RAM und TPM im Live-System ohne Netz. |
| **Sandboxed Execution** | `Chimera` | Trennt Entscheidung (LLM) strikt von Ausführung (`sohelper`) und Netz (`tor`). |

---

## Wo was steht

| Dokument | Inhalt |
| :--- | :--- |
| [`docs/DESIGN.md`](docs/DESIGN.md) | **Einzige Wahrheitsquelle.** Ablauf, Sicherheitsgrenzen und Architekturregeln R1–R6. |
| `docs/ROADMAP.md` | *Folgt nach Abschluss der Basis-Projekte.* |
| `CHANGELOG.md` | *Folgt ab dem ersten Commit.* |

---

## Lizenz

MIT — siehe `LICENSE`.
