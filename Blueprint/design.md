# Projekt Aegis (Arbeitstitel) — Autonomes Zero-Touch-Deployment & Self-Healing Helpdesk

Was und warum. Tragende Entscheidungen, Architekturregeln, Grenzen. Wer etwas ändert, das hier beschrieben ist, ändert **zuerst dieses Dokument**.

Stand: 2026-11-12. Konzept-Entwurf (Geparkt bis Abschluss von Chimera, woa-l, rag4ll und LLMCF).

---

## 1. Was dieses System ist

Ein ereignisgesteuertes, selbstlernendes Zero-Touch-Deployment- und Helpdesk-System für Windows- und Linux-Clients. Es ersetzt starre Task-Sequenzen (wie klassisches SCCM) durch einen **deterministisch abgesicherten Agenten**, der Hardware-Profile live einliest, maßgeschneiderte Installationspläne baut, Installationen anhand echter Funktionsproben überwacht, Fehler aus Logs selbstständig behebt und jede verifizierte Lösung dauerhaft im Gedächtnis speichert.

Zwei Dinge unterscheiden es von herkömmlichen Deployment-Werkzeugen:

1. **Kein starres Paket-Korsett.** Treiber, Basis-OS, Rollen-Software und bekannte Workarounds werden erst nach der realen Hardware-Probe des Zielgeräts zusammengestellt.
2. **Kein blindes Vertrauen auf Exitcodes.** Jede Installation wird durch eine echte Gegenprobe am Programm oder Dienst belegt; ein stummer Exitcode 0 gilt nicht als Nachweis.

---

## 2. Herkunft und Systemgrenzen

| Baustein | Quelle / Vorbild | Rolle im System |
| :--- | :--- | :--- |
| **Weckruf & Event-Bus** | `woa-l` (eigen) | Ereignissteuerung (`woac`): PXE-Probe, Boot-Rückmeldung und Tickets wecken den Agenten als Turn. |
| **Runbook-Gedächtnis** | `rag4ll` (eigen) | Hybrid-Suche (Vektor + BM25) über verifizierte Problemlösungen; Beförderung wiederkehrender Fixes zu Skills. |
| **Hardware-Probe** | `LLMCF` / `rag4ll` (eigen) | Plattformunabhängige Erkennung von CPU-Flags, PCI-IDs, GPU, RAM, TPM und Speicher (`hardware_probe`). |
| **Privilegientrennung** | `Chimera` (eigen) | Agent ohne direkten Netzwerk- oder Domain-Admin-Zugriff; Ausführung ausschließlich über isolierte Gateways/Helfer. |

---

## 3. Ablauf: Vom PXE-Boot zum fertigen Arbeitsplatz

### Phase 1 — PXE-Boot und Live-Probe
1. Der Client bootet manuell oder per Wake-on-LAN über **iPXE** in ein minimales Live-System (WinPE für Windows, schlankes Alpine/Debian-Initramfs für Linux).
2. Im Live-System läuft sofort die `hardware_probe` (ohne Netzabhängigkeit, reines Auslesen von SMBIOS, PCI-IDs, TPM, Storage und RAM).
3. Das Live-System sendet das Hardware-Profil zusammen mit MAC-Adresse, angeforderter Zielrolle (z. B. *CAD-Arbeitsplatz*, *Büro-Standard*) und einem Einmal-Kopplungscode per `woa-l`-Weckruf an das Deployment-Gateway.

### Phase 2 — Dynamischer Bauplan statt starrer Task-Sequenz
1. Der Agent gleicht die PCI-IDs gegen das Treiber-Repository und das `rag4ll`-Gedächtnis ab (bekannte Treiber-Konflikte oder Pflicht-Reihenfolgen werden sofort berücksichtigt).
2. Aus einem signierten Paket-Katalog (nur geprüfte Quellen und Hashes, **Regel R3**) erstellt der Agent eine maßgeschneiderte `unattend.xml` (bzw. Preseed/Kickstart) sowie ein deklaratives Installations-Manifest (`plan.json`).
3. Das Live-System zieht das Basis-Image und den Plan über das isolierte Deployment-Gateway und startet die Grundinstallation.

### Phase 3 — Selbstheilende Installation & Nachweis
1. Nach dem ersten Boot ins Ziel-OS meldet sich ein minimaler Bootstrap-Dienst per `woa-l` zurück.
2. Der Agent arbeitet das Installations-Manifest ab (Treiber, Runtimes, Applikationen wie Office, AutoCAD, VLC, WinRAR, danach OS-Updates).
3. Nach jedem Schritt läuft eine **Funktions-Gegenprobe** (**Regel R2**): Nicht nur `msiexec` Exitcode 0, sondern echter Starttest, Dienststatus oder Datei-/Registry-Nachweis.
4. Schlägt ein Schritt fehl, liest der Agent gezielt die relevanten Log-Abschnitte (`AppEnforce`, `CBS.log`, MSI-Logs, `journalctl`), sucht in `rag4ll` nach bekannten Fixes, wendet einen zulässigen Workaround an und wiederholt den Nachweis.

### Phase 4 — Sicheres User-Pre-Provisioning (Zero-Trust AD)
1. Ist das System vollständig durchgeprüft, fordert der Agent beim Identity-Gateway ein zeitlich eng begrenztes Einmal-Token an (z. B. *Temporary Access Pass* / Kerberos-Delegation, **Regel R4**).
2. Der Agent meldet das Benutzerprofil einmalig lokal an, initialisiert Outlook/Exchange-Konten, Netzlaufwerke und Browser-Lesezeichen aus dem User-Profil-Speicher und widerruft das Einmal-Token sofort nach Abschluss.
3. Das System meldet „Bereit“ und legt den gesamten Ablauf als verifiziertes Runbook in `rag4ll` ab.

---

## 4. Architekturregeln

### Regel R1 — Die Isolation des Agenten ist hart (Vorbild Chimera)
Der Sprachmodell-Agent besitzt **keinen** direkten Zugriff auf Domain-Admin-Konten, Paket-Shares oder das Internet.
* Alle Befehle auf dem Zielclient laufen über einen signierten, abgesicherten Ausführungs-Dienst (`sohelper`-Prinzip) mit strikter Positivliste.
* Alle Netzwerk- und Repository-Zugriffe laufen durch ein zentrales Gateway (`tor`-Prinzip).

### Regel R2 — Stiller Erfolg ist verboten (Funktionsnachweis statt Exitcode)
Ein Installationsschritt gilt erst als abgeschlossen, wenn sein Ergebnis am System nachgewiesen ist.
* Beispiel Office: Programmdatei vorhanden, Lizenzstatus prüfbar, Prozess startet ohne Fehlerdialog.
* Jede Prüfung braucht eine Gegenprobe: Schlägt der Nachweis bei absichtlich kaputtem Zustand nicht fehl, ist die Prüfung wertlos.

### Regel R3 — Nur signierte Pakete, kein freies Nachladen aus dem Netz
Der Agent darf Softwarepakete und Treiber **ausschließlich** aus dem lokalen, SHA-256-verifizierten Paket- und Treiber-Katalog zusammenstellen.
* Fehlt ein Paket oder Treiber im Katalog, meldet der Agent einen benannten Befund an den Administrator, statt selbstständig fremde Binaries aus dem Internet herunterzuladen.

### Regel R4 — Niemals Klartext-Passwörter beim User-Onboarding
Für die Erstanmeldung und Einrichtung des Benutzerprofils (Outlook, Laufwerke, Lesezeichen) werden **keine** Benutzerpasswörter gespeichert, abgefragt oder weitergereicht.
* Zulässig sind ausschließlich kurzlebige, auf einen Vorgang und einen Host beschränkte Einmal-Tokens (z. B. TAP / Zertifikats-Delegation), die unmittelbar nach dem Provisioning serverseitig ungültig gemacht werden.

### Regel R5 — Destruktive Eingriffe brauchen menschliche Freigabe
Auf bestehenden Produktiv-Clients im Helpdesk-Betrieb (Ticket-Bearbeitung über WinRM / SSH / PowerShell Direct) darf der Agent Dienste neu starten, Caches leeren, Pakete reparieren und Logs auslesen.
* Neuinstallationen (PXE-Wipe), Änderungen an AD-Gruppen, Firewall-Regeln oder das Löschen von Benutzerdaten erfordern zwingend eine explizite Freigabe über das Genehmigungs-Gate.

### Regel R6 — Lernen nur aus grün geprüften Lösungen
In das `rag4ll`-Runbook-Gedächtnis (`collection: helpdesk_runbooks`) wird **ausschließlich** geschrieben, was durch einen erfolgreichen Funktionsnachweis (**Regel R2**) bestätigt wurde.
* Ungelöste Versuche landen im Audit-Log, niemals im Runbook-Speicher.
* Ab einer festen Schwelle erfolgreicher Wiederholungen (z. B. `N >= 10` verifizierte Treffer für dasselbe Fehlerbild) darf der Agent einen deterministischen `SKILL.md`-Entwurf erzeugen, der nach Review in die Skill-Ablage wandert.

---

## 5. Helpdesk-Betrieb (24/7/365 Ticket-Loop)

Neben dem Neu-Rollout bedient dieselbe Architektur den laufenden Support:

1. **Ticket-Eingang:** Ein neues Ticket oder eine Störungsmail wird über `woa-l` auf das 6.000-Zeichen-Format gekürzt und weckt den zuständigen Support-Agenten.
2. **RAG-Vorprüfung:** Vor jedem Zugriff auf den Client sucht der Agent in `rag4ll` nach bekannten Runbooks oder bereits beförderten Skills.
3. **Diagnose & Fix:** Über WinRM, SSH oder PowerShell Direct prüft der Agent den Client, wendet den Fix an und führt die Gegenprobe aus.
4. **Abschluss:** Ticket wird mit einer verständlichen Zusammenfassung geschlossen, das Runbook in `rag4ll` aktualisiert (Zähler +1) und der Agent kehrt in den Schlafzustand zurück.

---

## 6. Verzeichnis der Architekturregeln

* **R1** — Die Isolation des Agenten ist hart
* **R2** — Stiller Erfolg ist verboten (Funktionsnachweis statt Exitcode)
* **R3** — Nur signierte Pakete, kein freies Nachladen aus dem Netz
* **R4** — Niemals Klartext-Passwörter beim User-Onboarding
* **R5** — Destruktive Eingriffe brauchen menschliche Freigabe
* **R6** — Lernen nur aus grün geprüften Lösungen
