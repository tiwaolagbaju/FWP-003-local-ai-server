# Phase 6 — Remote Administration

## Goal

Prepare the HP Z6 G4 to operate as a basement AI server that can be administered securely from the office PC without requiring a local monitor and keyboard for normal use.

The initial remote-access stack is:

- OpenSSH Server
- Tailscale
- SSH public-key authentication
- Later: VS Code Remote SSH

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
   +-- Ed25519 key authentication
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

## Ed25519 SSH Key Authentication

A dedicated Ed25519 SSH key pair was generated on the office PC for this server workflow.

The public key was added to the Linux user's `~/.ssh/authorized_keys` file and permissions were set appropriately.

A separate SSH session was then opened using the dedicated private key. The connection succeeded, validating public-key authentication.

This confirms:

- the office PC possesses the required private key
- the Z6 recognizes the matching public key
- SSH public-key authentication is functional over Tailscale
- reusable Ubuntu account passwords are no longer required for normal remote administration from this client

During setup, care was taken to distinguish the public key from the private key. The private key remains only on the office PC and is not stored in the repository or shared in screenshots.

## Security / Documentation Policy

Raw screenshots from this phase are not published directly because shell prompts, service logs, SSH configuration, and Tailscale status output can expose local identifiers.

The final public documentation will demonstrate the architecture and validation results while omitting:

- local usernames
- live hostnames where identifying
- LAN IP addresses
- Tailscale IP addresses
- authentication URLs
- SSH private keys
- full public keys or fingerprints
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
| Dedicated Ed25519 key generated | **PASS** |
| Public key installed on Z6 | **PASS** |
| SSH key authentication validated | **PASS** |
| Password SSH disabled | Pending — optional hardening after alias / VS Code validation |
| VS Code Remote SSH validated | Pending |

## Next Validation

The next checkpoint is to improve usability and then complete remote development access:

1. Create a local SSH config entry on the office PC with a short alias for the Z6.
2. Verify the alias uses the dedicated Ed25519 identity automatically.
3. Install / configure VS Code Remote SSH to use the same SSH config entry.
4. Validate interactive shell and file access through VS Code.
5. Only after all key-based workflows are proven, consider disabling SSH password authentication.

## Engineering Takeaway

The complete remote administration path is now validated from the office PC to the basement AI server using an encrypted Tailscale overlay, OpenSSH, and Ed25519 public-key authentication. The system can be administered securely without exposing SSH directly to the public internet or relying on reusable account passwords for normal access.