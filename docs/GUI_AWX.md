# Running this project from a GUI (AWX / Semaphore)

This is a standard Ansible project, so it works with any Ansible GUI. You never
*have* to use one — the CLI in [`GETTING_STARTED.md`](GETTING_STARTED.md) does
everything — but a GUI adds a web dashboard, scheduling, role-based access, and a
button to run playbooks with logs.

## Which GUI?

| Option | Best for | Notes |
| --- | --- | --- |
| **Semaphore** | Getting started | Lightweight, single container, easy install. Recommended first GUI. |
| **AWX** | Full features, free | Powerful (the upstream of Ansible Automation Platform) but heavier — needs Kubernetes/minikube. |
| **Ansible Automation Platform** | Enterprise / support | Red Hat's paid product; same concepts as AWX. |

All three run **this same repo unchanged**.

## How a GUI maps to this repo

Whatever tool you pick, you configure the same four things:

1. **Project** → point it at this Git repo + branch. It syncs the playbooks and
   auto-installs collections from [`../collections/requirements.yml`](../collections/requirements.yml).
2. **Inventory** → use `ansible/inventory/hosts.ini` (import it, or source it from
   the project).
3. **Credentials**:
   - a *Machine/Windows* credential (the WinRM user/password), and
   - a *Vault* credential (the Ansible Vault password that decrypts our secrets —
     DSRM, domain admin, template admin).
4. **Job Templates** → one per playbook you want a button for, e.g.
   `ansible/site.yml` (build order), or `ansible/playbooks/studio5000.yml`.

## Quick start — AWX (Job Template)

1. **Project**: SCM type Git, URL of this repo, branch
   `claude/audible-architecture-docs-lwmh9w` (or `master` once merged).
2. **Inventory**: create one, then add a *Source* “Sourced from a Project” →
   file `ansible/inventory/hosts.ini`.
3. **Credentials**: add a *Vault* credential (paste the vault password) and a
   *Machine* credential (WinRM user + password).
4. **Job Template**: Playbook = `ansible/site.yml`, attach both credentials,
   optionally set *Limit* (e.g. `nwa-vh-01`) and *Tags* (e.g. `hosts`).
5. Click **Launch**.

## Quick start — Semaphore

1. Create a **Key Store** entry for the WinRM login and one for the Vault
   password.
2. Create a **Repository** pointing at this Git repo/branch.
3. Create an **Inventory** using `ansible/inventory/hosts.ini`.
4. Create a **Task Template** → playbook `ansible/site.yml` → run it.

## Note on paths

The GUIs expect `collections/requirements.yml` and (optionally)
`roles/requirements.yml` at the **repo root** — `collections/requirements.yml`
is already here. The CLI keeps using `ansible/requirements.yml`; both list the
same collections.
