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

The intended administration path is:

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
| Office PC can reach Z6 through Tailscale | Pending |
| SSH from office PC validated | Pending |
| SSH key authentication configured | Pending |
| VS Code Remote SSH validated | Pending |

## Next Validation

The next checkpoint is to test the full remote path from the office PC:

1. Confirm the office PC is connected to the same Tailscale network.
2. Verify basic reachability to the Z6 through Tailscale.
3. Establish an SSH session from the office PC.
4. Once password-based SSH is proven, configure Ed25519 key authentication.
5. Disable SSH password authentication only after key-based login has been validated in a separate session.
6. Add VS Code Remote SSH as the preferred development / administration workflow.

## Engineering Takeaway

Remote administration is being built as a separate, testable layer rather than exposing services directly to the public internet. SSH was validated first, Tailscale was then added as the secure transport layer, and live addressing / authentication information is deliberately excluded from public project artifacts.