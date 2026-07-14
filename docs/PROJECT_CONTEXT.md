# Project Context & Continuity Document

> **Purpose:** A future engineer or AI agent should be able to continue this
> project from this document alone, without access to previous chats.

---

## 1. What this project is

An **OT Cyber Security & Industrial Automation laboratory**, built entirely with
Infrastructure as Code. Two sites — **NWA** (primary) and **BAC** (secondary) —
are modelled on the Purdue reference architecture and must ultimately be
reproducible from version-controlled code.

Full technical detail lives in [`ARCHITECTURE.md`](ARCHITECTURE.md). This file
captures the *why*: decisions, assumptions, working code, dead ends, and next
steps.

---

## 2. Automation strategy

- **Control node:** `NWA-AUTO-01` (Ubuntu Linux) — runs Git, Terraform, Ansible,
  documentation, and software-repository access.
- **Windows management:** Ansible over **WinRM** (validated for copy, PowerShell,
  and long installs).
- **Provisioning direction:** `Git → Terraform (ESXi) → Hyper-V → Network → Windows → Rockwell`.
- **Golden rule:** reference software media through the `rockwell_repo` variable,
  never a hard-coded path.

### Ubuntu control-node bootstrap (reference)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ansible git curl wget vim tree python3-winrm

# sanity checks
ansible --version
python3 -c "import winrm; print('WinRM OK')"

# workspace
mkdir -p ~/ot-lab/ansible/{inventory,playbooks,roles,group_vars,host_vars}
mkdir -p ~/ot-lab/ansible/roles/{hyperv,ad,historian,assetcentre,rockwell,windows-base}
mkdir -p ~/ot-lab/software
```

---

## 3. Key decisions

| Decision | Rationale |
| --- | --- |
| Purdue-level segmentation | Standard OT reference model; drives firewall/VLAN design |
| Single Ubuntu control node | One place for Git/Terraform/Ansible/docs |
| `rockwell_repo` variable | Software source will move from VMware share to `\\NWA-FILE-01\Software` |
| UNC paths over mapped drives | Mapped letters are unreliable across WinRM sessions |
| No separate roles for FactoryTalk Linx / Activation Manager / ControlFLASH | They install as part of Studio 5000 |
| Studio 5000 installed with `/QS /IAcceptAllLicenseTerms /AutoRestart` only | `/Product=` example from installer help does not work for this media |
| **Lab = 6x WS2025 VMs on one ESXi dev server (nested Hyper-V); production = 6 physical servers** | Build and validate now, before the physical hardware arrives |
| **Isolate the "create the VH" layer** (Terraform/ESXi in lab, bare-metal in prod) | Everything above it is identical Ansible → minimal-change migration |
| **Two install layers: host (WS2025) vs guest (Hyper-V VMs)** | e.g. EPP installs on the host; SCADA/AD/Historian install inside the VMs |

---

## 4. Confirmed deployment model

Confirmed with the project owner (not an assumption):

- **Target (final):** six **physical** servers — one per dashed host box in the
  design diagram (`NWA-VH-01..04`, `BAC-VH-01/02`). Each runs Windows Server 2025
  with the **Hyper-V role bare-metal**; OT workloads run as Hyper-V VMs inside.
- **Now (lab):** one **development ESXi** server. On it, **six WS2025 VMs** stand
  in for the six physical servers, each with **Hyper-V enabled (nested)**; the
  workload VMs run inside that nested Hyper-V.
- **Two software layers:** some software installs on the **WS2025 host** itself
  (e.g. Endpoint Protection and other host-level agents); the rest installs
  **inside the Hyper-V VMs**.
- **Goal:** IaC structured by role so moving from the ESXi lab to the physical
  hardware is as close to a no-op as possible — only the layer that *creates* the
  six `VH` machines changes.

### Standing assumptions

- Windows Server 2025 for servers; Windows 11 Pro for workstations/clients.
- Current software repo is the VMware shared folder; production will be a file
  server (`NWA-FILE-01` / `BAC-FILE-01`).
- `BAC-VH-02` exists in the design (hosts `BAC-IDS-01` / `BAC-SEM-01`) but is not
  yet sized in the capacity plan (0/0/0 — TBD).
- Field-device names corrected per diagram: `NWA-LVL-01/02` (not `LVI`); added
  `NWA-GW-02` and `NWA-VRU-01`. `BAC-SW-02` shown in L1 per the diagram.

---

## 5. Open questions / to confirm

- How to perform Windows Updates for VMs running inside ESXi (raised with Costas).
- IP addressing / subnetting is not yet populated in the capacity plan.
- FortiGate and OPSWAT automation approach (API vs. Ansible modules) not chosen.

---

## 6. Failed / rejected approaches

- **`Setup.exe /Product="Studio Enterprise"`** — documented in installer help but
  fails on this media ("does not support Install now"). Rejected.
- **Mapped drive letters (`X:`) over WinRM** — unreliable across sessions. Use UNC.

---

## 7. Current status & next actions

- ✅ Architecture, inventory, and naming documented.
- ✅ Studio 5000 unattended install validated — this is the **first** of several
  software packages to be automated; it was chosen to prove out the install
  pattern that later packages will reuse.
- ⏳ Next: complete the `studio5000` role, then extend the same pattern to the
  remaining software packages, then the `hyperv` role (Roadmap Phases 1–2).

See the Roadmap table in [`ARCHITECTURE.md`](ARCHITECTURE.md#12-roadmap).
