# OT Lab — Infrastructure as Code

An OT (Operational Technology) cyber-security and industrial automation
laboratory, built with Infrastructure as Code so that both sites — **NWA**
(primary) and **BAC** (secondary) — are reproducible from version-controlled
source. Segmentation follows the Purdue reference model.

The lab is built **now on a development ESXi server** — six Windows Server 2025
VMs (with nested Hyper-V) stand in for the six **physical** servers that arrive
later. The IaC is structured by role so the move to real hardware changes as
little as possible: only the layer that *creates* those six hosts differs
(Terraform/ESXi in the lab, bare-metal in production). See
[`docs/ARCHITECTURE.md §3`](docs/ARCHITECTURE.md#3-deployment-model--reference-physical-vs-lab-esxi-nested).

## Documentation

| Document | Purpose |
| --- | --- |
| [`docs/TEAM_OVERVIEW.md`](docs/TEAM_OVERVIEW.md) | One-page briefing for the team — start here |
| [`docs/GETTING_STARTED.md`](docs/GETTING_STARTED.md) | Step-by-step setup: Ubuntu control node, WinRM, first playbook |
| [`docs/TEMPLATE_PREP.md`](docs/TEMPLATE_PREP.md) | Preparing the sysprepped WS2025 template and making clones WinRM-reachable |
| [`docs/GUI_AWX.md`](docs/GUI_AWX.md) | Running the project from a GUI (AWX / Semaphore) |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Full architecture: deployment model, Purdue model, site topology, compute inventory, naming, automation, Studio 5000 context |
| [`docs/PROJECT_CONTEXT.md`](docs/PROJECT_CONTEXT.md) | Continuity document — decisions, assumptions, lessons learned, roadmap |

## Repository layout

```
.
├── docs/                 # Architecture & continuity documentation
├── ansible/              # Configuration management (Windows over WinRM)
│   ├── ansible.cfg
│   ├── inventory/        # hosts.ini — Purdue-aligned groups
│   ├── group_vars/       # e.g. windows.yml (WinRM + rockwell_repo)
│   ├── host_vars/
│   ├── playbooks/        # studio5000.yml — first of several software installs
│   └── roles/            # windows-base, hyperv, studio5000, ...
└── terraform/            # optional / not used (manual VM clone instead)
```

## Getting started (control node)

The automation host is **NWA-AUTO-01** (Ubuntu). Bootstrap and quick-start steps
are in [`docs/PROJECT_CONTEXT.md`](docs/PROJECT_CONTEXT.md#2-automation-strategy).

```bash
cd ansible
ansible-playbook playbooks/studio5000.yml --limit nwa-ews-01
```
