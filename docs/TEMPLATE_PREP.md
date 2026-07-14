# Golden Template Preparation

Everything in this project is cloned from a **sysprepped Windows Server 2025
template**. The current template is sysprepped but **does not have WinRM
enabled**, which matters because Ansible reaches Windows over WinRM. This page
covers how to make clones reachable.

There are two kinds of machine cloned from a template, and each solves the WinRM
problem differently.

---

## 1. VH hosts (WS2025 VMs on ESXi)

These are plain ESXi VMs — there is no host underneath them we can use to reach
in, so **WinRM must exist the moment the clone boots**. Bake it into the golden
template (recommended), or enable it on first boot via the sysprep answer file.

### Option A — bake WinRM into the image (recommended)

Before running sysprep on the golden VM, enable WinRM so every clone inherits it:

```powershell
# Enable PS remoting + WinRM service (auto start)
Enable-PSRemoting -Force
Set-Service -Name WinRM -StartupType Automatic

# Lab transport (HTTP/5985). Harden to HTTPS/NTLM for production.
Set-Item WSMan:\localhost\Service\Auth\Basic $true
Set-Item WSMan:\localhost\Service\AllowUnencrypted $true
New-NetFirewallRule -Name 'WinRM-HTTP-In' -DisplayName 'WinRM HTTP' `
  -Protocol TCP -LocalPort 5985 -Direction Inbound -Action Allow
```

Then sysprep with an **unattend.xml** that completes OOBE unattended and sets the
local Administrator password (so the clone boots straight to a usable state):

```
sysprep.exe /generalize /oobe /shutdown /unattend:C:\unattend.xml
```

### Option B — enable WinRM on first boot (SetupComplete.cmd)

Place a script at `C:\Windows\Setup\Scripts\SetupComplete.cmd` in the image; it
runs automatically after sysprep/OOBE completes:

```bat
powershell -ExecutionPolicy Bypass -Command "Enable-PSRemoting -Force; Set-Item WSMan:\localhost\Service\Auth\Basic $true; Set-Item WSMan:\localhost\Service\AllowUnencrypted $true; New-NetFirewallRule -Name WinRM-HTTP-In -DisplayName 'WinRM HTTP' -Protocol TCP -LocalPort 5985 -Direction Inbound -Action Allow"
```

---

## 2. Guest VMs (inside Hyper-V on a VH host)

Here we have an advantage: the VH host is already Ansible-managed over WinRM, and
it can reach its own guests over the **Hyper-V VM bus** using **PowerShell
Direct** — no guest network or WinRM required. So even with a template that lacks
WinRM, the `hyperv` role bootstraps it automatically:

1. `hyperv` role creates + starts the guest from the template VHDX.
2. It connects with `Invoke-Command -VMName <guest> -Credential <local admin>`
   (PowerShell Direct) and enables WinRM **inside** the guest.
3. Ansible then targets the guest directly over WinRM for all further config.

This needs the guest's **local admin credentials** (baked into the template).
Provide them via vault and reference from the host/group vars:

```yaml
hyperv_guest_bootstrap_winrm: true
hyperv_guest_admin_user: "Administrator"
hyperv_guest_admin_password: "{{ vault_template_admin_password }}"
```

Requirements for PowerShell Direct: host and guest both Windows Server
2016+/Windows 10+ (WS2025 qualifies), guest running and past OOBE, authenticated
with a **guest-local** account.

> If you prefer, you can also bake WinRM into the template (§1 Option A) and set
> `hyperv_guest_bootstrap_winrm: false` — then guests are reachable directly and
> the bootstrap step is skipped.

---

## Summary

| Machine | Reached via | WinRM comes from |
| --- | --- | --- |
| VH host (ESXi clone) | Network WinRM | Baked into template (§1 A) or first-boot script (§1 B) |
| Guest VM (Hyper-V) | PowerShell Direct → then WinRM | Bootstrapped by the `hyperv` role, or baked into template |

Keeping WinRM in the golden template is the simplest path for **both** cases; the
PowerShell Direct bootstrap is what lets us proceed even with the current
template that does not have it.
