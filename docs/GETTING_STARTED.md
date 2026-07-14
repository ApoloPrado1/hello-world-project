# Getting Started

A step-by-step guide to run this project for the first time. No prior Ansible
experience assumed. For the big picture first, read
[`TEAM_OVERVIEW.md`](TEAM_OVERVIEW.md).

---

## Mental model (read this first)

- This repo is just **text files** (YAML playbooks, inventory, configs). You edit
  them anywhere.
- **Ansible runs on the Ubuntu control node**, not on Windows. It is a Linux
  tool. You edit the files, then run them **from Ubuntu**.
- Ansible is **agentless**: it connects to a target (WinRM for Windows), applies
  changes, and disconnects. Nothing is installed permanently on the targets.

```
VS Code (your laptop)  --edit-->  Ubuntu control node  --ansible-playbook-->  VH host
```

---

## 1. Prepare the Ubuntu control node

On the Ubuntu VM:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ansible git curl wget vim tree python3-winrm

# sanity checks
ansible --version
python3 -c "import winrm; print('WinRM OK')"
```

Clone this repository and install the required collections:

```bash
git clone <this-repo-url> ot-lab
cd ot-lab/ansible
ansible-galaxy collection install -r requirements.yml
```

## 2. (Recommended) Edit with VS Code Remote-SSH

So you can edit comfortably on your laptop but run on Ubuntu:

1. Install the **Remote - SSH** extension in VS Code.
2. `Ctrl/Cmd+Shift+P` → **Remote-SSH: Connect to Host** → your Ubuntu VM.
3. Open the `ot-lab` folder. The integrated terminal now runs **on Ubuntu**, so
   `ansible-playbook` works from there.

## 3. Prepare a Windows host for WinRM (one-time, per host)

Ansible reaches Windows over WinRM. On each Windows Server 2025 host, run this
**once** in an elevated PowerShell to enable it for the lab:

```powershell
# Lab-only quick enable (HTTP/5985 + basic). Harden for production.
winrm quickconfig -quiet
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Service\AllowUnencrypted $true
Set-Item WSMan:\localhost\Service\Auth\Basic $true
New-NetFirewallRule -Name "WinRM-HTTP" -DisplayName "WinRM HTTP" `
  -Enabled True -Direction Inbound -Protocol TCP -LocalPort 5985 -Action Allow
```

> Lab settings only. Production should use HTTPS (5986) + NTLM/Kerberos, not
> unencrypted Basic.

## 4. Point the inventory at your host

The inventory lives in [`inventory/hosts.ini`](../ansible/inventory/hosts.ini);
Windows connection variables live in
[`group_vars/windows.yml`](../ansible/group_vars/windows.yml).

Create a host-specific file with the real address and credentials — keep secrets
out of the repo by using Ansible Vault:

```bash
# store the password encrypted
ansible-vault create host_vars/nwa-vh-01/vault.yml
```

```yaml
# host_vars/nwa-vh-01/vault.yml  (encrypted)
ansible_host: 10.0.0.11
ansible_user: Administrator
ansible_password: "your-password-here"
```

## 5. Test connectivity (your first command)

Confirm Ansible can reach the host before doing anything real:

```bash
ansible nwa-vh-01 -m ansible.windows.win_ping --ask-vault-pass
```

Expected result:

```json
nwa-vh-01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

If you get `pong`, WinRM + credentials + inventory are all correct.

## 6. Run your first playbook

Install Studio 5000 on a target (the first automated software package):

```bash
ansible-playbook playbooks/studio5000.yml --limit nwa-ews-01 --ask-vault-pass
```

Or run the whole environment in dependency order (hosts → domain → apps):

```bash
ansible-playbook site.yml --ask-vault-pass            # everything
ansible-playbook site.yml --tags hosts --limit nwa-vh-01   # just the host layer
```

Useful flags while learning:

| Flag | What it does |
| --- | --- |
| `--limit <host>` | Run against one host only |
| `--check` | Dry run — show what *would* change, change nothing |
| `--diff` | Show the actual differences being applied |
| `-v` / `-vvv` | More verbose output (good for debugging) |

---

## Troubleshooting

| Symptom | Likely cause / fix |
| --- | --- |
| `winrm or requests is not installed` | `sudo apt install -y python3-winrm` on Ubuntu |
| Timeout / connection refused | WinRM not enabled or firewall blocking 5985 on the host (step 3) |
| `the specified credentials were rejected` | Wrong user/password in `host_vars`, or Basic auth not enabled |
| Path not found during copy | Use **UNC paths**, not mapped drive letters — they are unreliable over WinRM |

## Next steps

- Read [`ARCHITECTURE.md`](ARCHITECTURE.md) for the full design.
- Pick a role to build next (the `hyperv` host role is the natural first — see
  the [roadmap](ARCHITECTURE.md#12-roadmap)).
