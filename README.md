# Aegis — Autonomous Zero-Touch Deployment & Self-Healing Helpdesk

*Deutsche Version: [README-de.md](README-de.md)*

**An event-driven deployment and support system that probes hardware, provisions tailored workstations, self-heals incidents using real functional verification, and learns from every solved problem.**

Status: **Concept / Draft.** Nothing is built, nothing is tagged. This project is intentionally parked until **Chimera**, **woa-l**, **rag4ll**, and **LLMCF** are completed. Core architectural decisions are documented in [`docs/DESIGN.md`](docs/DESIGN.md).

---

## What It Does

Traditional deployment and client management systems (such as SCCM) rely on rigid task sequences and blindly trust exit codes. **Aegis** flips this model: a deterministically sandboxed agent dynamically builds the deployment plan from the target machine's live hardware probe, verifies every step with real functional counter-checks, and permanently stores every proven fix in its memory.

Two operating modes work together:

1. **Zero-Touch Rollout (PXE → Ready Workstation):** A client boots via PXE into a minimal live environment (WinPE or Linux), runs the hardware probe, and wakes the agent. The agent assembles a tailored installation plan from signed packages and drivers, supervises driver, software (Office, AutoCAD, VLC, WinRAR, etc.), and OS updates, and pre-provisions the user profile (Outlook, network drives, bookmarks) without plaintext passwords using short-lived, single-use tokens.
2. **24/7/365 Self-Healing Helpdesk:** Incoming tickets or alert emails wake the agent on demand. It checks known runbooks first, connects to the client via WinRM, SSH, or PowerShell Direct, resolves the issue, proves recovery with a functional check, and closes the ticket.

---

## Core Principles

* **Event-Driven Instead of Polling (`woa-l`):** The agent does not exist as an expensive background process between tasks. Incoming PXE probes, boot callbacks, or helpdesk tickets wake it deterministically for a single turn.
* **Experience Memory with Skill Promotion (`rag4ll`):** Only verified green fixes are stored as runbooks in the hybrid vector + BM25 store. Once the same issue pattern is solved repeatedly (`N >= 10`), the agent distills the fix into a deterministic `SKILL.md` workflow—eliminating LLM overhead for recurring issues.
* **Functional Proof Over Exit Codes (Rule R2):** An installation or repair step is only considered complete when the target application or service is verifiably running.
* **Strict Privilege Separation (Rules R1 & R4):** The agent has no direct internet access, no Domain Admin privileges, and never sees user passwords. Execution runs through allow-listed helpers, while destructive actions (such as a PXE wipe) require explicit human approval.

---

## Ecosystem Integration

Aegis does not reinvent the wheel; it connects four existing building blocks into a unified enterprise platform:

| Component | Origin | Role in Aegis |
| :--- | :--- | :--- |
| **`woa-l` (woac)** | Own project | Wakes the agent on PXE boot, reboots, and incoming tickets. |
| **`rag4ll`** | Own project | Stores verified runbooks and retrieves known fixes prior to execution. |
| **`hardware_probe`** | `LLMCF` / `rag4ll` | Detects CPU flags, PCI IDs, GPU, RAM, and TPM offline inside the live OS. |
| **Sandboxed Execution** | `Chimera` | Strictly separates LLM reasoning from execution (`sohelper`) and network (`tor`). |

---

## Documentation

| Document | Contents |
| :--- | :--- |
| [`docs/DESIGN.md`](docs/DESIGN.md) | **Single Source of Truth.** Workflow, security boundaries, and architecture rules R1–R6. |
| `docs/ROADMAP.md` | *Follows once the foundational projects are completed.* |
| `CHANGELOG.md` | *Follows starting from the first commit.* |

---

## License

MIT — see `LICENSE`.
