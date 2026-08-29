# Phase 6 — Remote Administration

## Goal

Prepare the HP Z6 G4 to operate as a basement AI server that can be administered securely from the office PC without requiring a local monitor and keyboard for normal use.

The validated remote-access stack is:

- OpenSSH Server
- Tailscale
- SSH public-key authentication
- SSH client alias
- VS Code Remote SSH

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

## Tailscale Setup

The first Tailscale repository setup command could not run because `curl` was not installed on the fresh Ubuntu installation. After installing `curl`, the Tailscale repository/package setup completed and the Z6 successfully joined the private tailnet.

Authentication URLs, Tailscale IP addresses, node names, account identifiers, and other live tailnet information are intentionally excluded from this public repository.

## Validated Remote Access Architecture

```text
Office PC
   |
   | encrypted Tailscale overlay
   v
HP Z6 G4 AI Server
   |
   +-- OpenSSH
   +-- Ed25519 key authentication
   +-- short SSH client alias
   +-- VS Code Remote SSH
   +-- later: browser-based AI services
```

The design avoids exposing SSH directly to the public internet. Remote access is provided through the private overlay network instead of WAN port-forwarding.

## SSH Over Tailscale

A remote SSH session from the office PC to the Z6 was successfully established over Tailscale. This proved the overlay-network path, SSH reachability, and remote administration workflow.

## Ed25519 SSH Key Authentication

A dedicated Ed25519 SSH key pair was generated on the office PC. The public key was installed in the Linux user's `~/.ssh/authorized_keys` file and a separate session successfully authenticated using the matching private key.

The private key remains only on the office PC and is not stored in the repository or published in screenshots.

## SSH Client Alias

A local OpenSSH client configuration entry was created and validated using:

```text
ssh z6
```

The actual Tailscale hostname and local key path details are intentionally omitted from public documentation.

## VS Code Remote SSH

VS Code Remote SSH was configured on the office PC using the same `z6` SSH client profile.

The remote connection completed successfully. This establishes the preferred development workflow for the server, allowing remote shell access and Linux file/folder editing from the office PC without local interaction at the Z6.

## Security / Documentation Policy

Public documentation omits:

- local usernames
- identifying hostnames
- LAN IP addresses
- Tailscale IP addresses
- authentication URLs
- SSH private keys
- full public keys / fingerprints
- MAC addresses
- account identifiers

## Current Status

| Check | Result |
|---|---|
| OpenSSH Server installed | PASS |
| SSH service active / enabled at boot | PASS |
| Tailscale installed / authenticated | PASS |
| Z6 connected to private tailnet | PASS |
| SSH from office PC over Tailscale | PASS |
| Dedicated Ed25519 key generated | PASS |
| SSH key authentication | PASS |
| Short SSH alias (`ssh z6`) | PASS |
| VS Code Remote SSH | **PASS** |
| Password SSH disabled | Pending — optional hardening |

## Engineering Takeaway

The Z6 can now be administered and developed against remotely from the office PC through an encrypted Tailscale overlay, OpenSSH, Ed25519 public-key authentication, a reusable SSH client profile, and VS Code Remote SSH. No inbound WAN port-forward is required.
