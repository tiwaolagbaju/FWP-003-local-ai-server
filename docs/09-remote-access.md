# Phase 6 — Remote Administration

## Goal

Prepare the HP Z6 G4 to operate as a basement AI server that can be administered securely from the office PC without requiring a local monitor and keyboard for normal use.

The initial remote-access stack is:

- OpenSSH Server
- Tailscale
- Later: SSH key authentication and VS Code Remote SSH

## SSH Baseline

OpenSSH Server was installed and enabled with systemd.

Validation command:

```bash
systemctl status ssh --no-pager
```

Result:

- `ssh.service` loaded successfully
- Service state: **active (running)**
- Service enabled at boot
- SSH daemon listening on the standard SSH port

This establishes the local SSH service before remote overlay-network access is configured.

## Tailscale Setup — Initial Attempt

The first Tailscale repository setup command could not run because `curl` was not installed on the fresh Ubuntu installation.

Observed shell result:

```text
Command 'curl' not found
```

This is a package-dependency issue, not a networking or Tailscale failure.

## Corrective Action

Install `curl` first:

```bash
sudo apt update
sudo apt install -y curl
```

Then resume the Tailscale repository setup and installation.

Authentication URLs, Tailscale IP addresses, node names, account identifiers, and other live tailnet information will not be recorded in the public repository.

## Security / Documentation Policy

Raw screenshots from this phase are not published directly because the shell prompt and service logs contain local username / hostname information.

The final public documentation will demonstrate the architecture and validation results while omitting:

- local usernames
- live hostnames where identifying
- LAN IP addresses
- Tailscale IP addresses
- authentication URLs
- SSH keys / fingerprints
- MAC addresses
- account identifiers

## Current Status

| Check | Result |
|---|---|
| OpenSSH Server installed | PASS |
| SSH service active | PASS |
| SSH enabled at boot | PASS |
| `curl` available | Pending corrective action |
| Tailscale installed | Pending |
| Tailscale authenticated | Pending |
| Office PC can reach Z6 through Tailscale | Pending |
| SSH from office PC validated | Pending |
| SSH key authentication configured | Pending |
| VS Code Remote SSH validated | Pending |

## Engineering Takeaway

Remote administration is being built as a separate, testable layer rather than exposing services directly to the public internet. SSH is first validated locally, then secure overlay-network access is added, and credentials / live addressing information are intentionally excluded from public project artifacts.