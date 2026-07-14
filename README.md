# OT Lab — Infrastructure as Code

An OT (Operational Technology) cyber-security and industrial automation
laboratory, built with Infrastructure as Code so that both sites — **NWA**
(primary) and **BAC** (secondary) — are reproducible from version-controlled
source. Segmentation follows the Purdue reference model; automation flows
`Git → Terraform → ESXi → Hyper-V → Windows → Rockwell`.

## Documentation

| Document | Purpose |
| --- | --- |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Full architecture: Purdue model, site topology, compute inventory, naming, automation, Studio 5000 context |
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
└── terraform/            # ESXi provisioning (planned)
```

## Getting started (control node)

The automation host is **NWA-AUTO-01** (Ubuntu). Bootstrap and quick-start steps
are in [`docs/PROJECT_CONTEXT.md`](docs/PROJECT_CONTEXT.md#2-automation-strategy).

```bash
cd ansible
ansible-playbook playbooks/studio5000.yml --limit nwa-ews-01
```
