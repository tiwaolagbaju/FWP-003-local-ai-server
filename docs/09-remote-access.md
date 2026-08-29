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

## Tailscale Setup

The first Tailscale repository setup command could not run because `curl` was not installed on the fresh Ubuntu installation.

Observed shell result:

```text
Command 'curl' not found
```

This was isolated as a package-dependency issue rather than a networking or Tailscale failure.

After `curl` was installed, the Tailscale repository and package installation completed successfully and the Z6 was authenticated to the user's private tailnet.

Current result:

- Tailscale installed
- Tailscale service connected
- Z6 successfully joined the private Tailscale network

Authentication URLs, Tailscale IP addresses, node names, account identifiers, and other live tailnet information are intentionally excluded from the public repository.

## Remote Access Architecture

The validated administration path is:

```text
Office PC
   |
   | encrypted Tailscale overlay
   v
HP Z6 G4 AI Server
   |
   +-- OpenSSH
   +-- later: VS Code Remote SSH
   +-- later: browser-based AI services
```

The design avoids exposing SSH directly to the public internet. Remote access is provided through the private overlay network instead of WAN port-forwarding.

## SSH Over Tailscale Validation

A remote SSH session from the office PC to the Z6 was successfully established over Tailscale.

This confirms:

- the office PC and Z6 can communicate across the private tailnet
- the Tailscale transport path is functioning
- the Z6 SSH service is reachable over the overlay network
- password-based SSH authentication works for the current Linux account
- normal administration can now be performed remotely from the office PC

The initial SSH login required verifying the correct Linux username and account password. No credentials were recorded in the project documentation.

## Security / Documentation Policy

Raw screenshots from this phase are not published directly because shell prompts, service logs, and Tailscale status output can expose local identifiers.

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
| `curl` available | PASS |
| Tailscale installed | PASS |
| Tailscale authenticated | **PASS** |
| Z6 connected to tailnet | **PASS** |
| Office PC can reach Z6 through Tailscale | **PASS** |
| SSH from office PC validated | **PASS** |
| SSH key authentication configured | Pending |
| Password SSH disabled | Pending — do not change until key login is proven |
| VS Code Remote SSH validated | Pending |

## Next Validation

The next checkpoint is to replace password-based remote administration with SSH public-key authentication:

1. Generate an Ed25519 key pair on the office PC if one does not already exist.
2. Install only the public key on the Z6.
3. Open a new, separate SSH session and prove key-based login works.
4. Keep the existing working SSH session open while testing to avoid accidental lockout.
5. Only after successful key authentication, consider disabling SSH password authentication.
6. Add VS Code Remote SSH as the preferred development / administration workflow.

## Engineering Takeaway

The complete remote administration path is now validated from the office PC to the basement AI server using an encrypted Tailscale overlay and OpenSSH. The next security improvement is to remove dependence on reusable account passwords by moving to SSH public-key authentication, while preserving rollback access until the key-based path is proven.