# Linux OS Hardening

> Took a fresh Ubuntu server and applied a repeatable, 8-point hardening baseline — then
> verified each control with the same commands an auditor would use.

*CNIT 271 — Cybersecurity Fundamentals II, Lab 6.*

## Summary

I built a hardening checklist, applied it to a live Ubuntu VM, and documented the before/after state
for each control. The goal was a defensible security baseline for a general-purpose Linux server:
patched, firewalled, minimal services, strong auth, and audited.

## Lab setup

- Ubuntu VM from the `CNIT271VVM` template on Purdue's Nutanix cluster
- Static IP via `netplan`, verified network connectivity
- Services in scope: SSH, Apache2, MySQL
- Tools: `apt`, `ss`, `ufw`, `systemctl`, `auditd`, `aide`, `nmap`

## The hardening baseline

| Area | Control | Command / change |
| --- | --- | --- |
| Patching | Apply all updates | `apt update && apt upgrade -y` |
| Port audit | Inventory listening services | `ss -tulpn` |
| Firewall | Default deny, allow only needed ports | `ufw enable`; allow 22/80; `deny 3306/tcp` |
| Services | Disable anything not required | `systemctl stop/disable <svc>` |
| Accounts | Review interactive accounts | `grep -E "/bin/bash\|/bin/sh" /etc/passwd` |
| SSH | Restrict root login & retries | `PermitRootLogin no`, `MaxAuthTries 3`, restart sshd |
| Database | Bind MySQL to localhost | `bind-address = 127.0.0.1`, restart mysql |
| Logging | Enable the audit daemon | `apt install auditd`, `enable`, `start` |

## What I found

- An open-port audit is the fastest way to catch services you didn't mean to expose — MySQL on
  `3306` was reachable before the firewall rule and is now denied at the host.
- SSH is the most-attacked service on a typical host, so `PermitRootLogin no` + `MaxAuthTries 3`
  (and ideally key-only auth) is high-value for low effort.
- File-integrity monitoring with `aide` gives you a tripwire for changes after the baseline is set —
  it only helps if you snapshot a known-good state first.

## Screenshots

See [`screenshots/`](screenshots/) for the port audit, UFW rules, SSH config changes, MySQL bind
change, and `auditd` running.

## Skills demonstrated

Host hardening · firewall configuration · service minimization · SSH hardening · audit logging ·
file integrity monitoring · security baselining & documentation
