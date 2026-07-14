# Terraform — ESXi Provisioning (lab only)

Terraform runs on the `NWA-AUTO-01` control node and is responsible for **one
isolated layer**: creating the **six Windows Server 2025 VMs** on the development
**ESXi** server that stand in for the six physical `VH` hosts
(`NWA-VH-01..04`, `BAC-VH-01/02`). See sizing in
[`../docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md#6-compute-inventory).

Why this is deliberately a thin, separate layer: in production the six `VH` are
physical servers, so this Terraform step is simply **replaced by bare-metal
provisioning**. Everything above it — enabling Hyper-V, host-level software,
creating the guest VMs, installing applications, and network config — is
identical Ansible and does not change. See the deployment model in
[`../docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md#3-deployment-model--reference-physical-vs-lab-esxi-nested).

Intended responsibilities:

- Create the six WS2025 host VMs on ESXi (CPU/RAM/disk per the capacity plan).
- Enable nested virtualisation on each ("Expose hardware assisted virtualization").
- Hand off to Ansible (`../ansible`) for all in-guest and host configuration.

Status: **not yet implemented** — placeholder for Roadmap Phase 6.
