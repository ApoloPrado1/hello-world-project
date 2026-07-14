# Terraform — ESXi Provisioning (planned)

Terraform runs on the `NWA-AUTO-01` control node and provisions the ESXi layer
that underpins the OT lab (Roadmap Phase 6).

Intended responsibilities:

- Create Hyper-V host VMs on ESXi (see compute sizing in
  [`../docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md#5-compute-inventory)).
- Hand off to Ansible for in-guest configuration (`../ansible`).

Status: **not yet implemented** — placeholder for the IaC provisioning flow
`Git → Terraform → ESXi → Hyper-V → Windows → Rockwell`.
