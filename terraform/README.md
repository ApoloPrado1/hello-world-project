# Terraform — Optional / Future (NOT used today)

**This project does not use Terraform.** API access to create VMs on the ESXi
server is restricted, so the six Windows Server 2025 hosts (and the network
appliance VMs) are **cloned manually**; Ansible takes over from first boot.

This directory is a placeholder kept only for a possible future where API-based
provisioning becomes available. If that happens, Terraform would own **one thin,
isolated layer**: creating the six WS2025 host VMs on ESXi (sizing per the
capacity plan in
[`../docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md#6-compute-inventory)) and
enabling nested virtualisation on them. Everything above that layer stays
identical Ansible — see the deployment model in
[`../docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md#35-what-changes-between-lab-and-production).

Status: **out of scope** — do not implement unless provisioning access changes.
