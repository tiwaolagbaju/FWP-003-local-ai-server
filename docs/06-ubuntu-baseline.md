# Phase 3 — Ubuntu Baseline Installation

## Milestone

Ubuntu 26.04.1 LTS was successfully installed and booted on the HP Z6 G4 baseline configuration.

This establishes the first known-good Linux environment before any Radeon Pro V620 GPUs, AMD compute drivers, ROCm components, Vulkan tuning, Docker services, or AI inference software are introduced.

## Baseline Hardware During Installation

- HP Z6 G4 Workstation
- Intel Xeon Platinum 8168
- Temporary ECC Registered memory configuration
- WD Blue SN5000 2TB NVMe as the operating-system drive
- NVIDIA RTX 3050 6GB as the display GPU
- No Radeon Pro V620 GPUs installed

## Installation Notes

The workstation initially booted an older Ubuntu/LVM environment on the NVMe and dropped into emergency mode because an expected logical volume was missing.

The issue was isolated as a boot-source problem rather than a hardware failure. The system was then booted from the newly created Ubuntu 26.04.1 LTS USB installer and the fresh Ubuntu installation completed successfully.

This troubleshooting event reinforced the staged-build approach: because the V620 GPUs were still absent, the failed boot could be isolated to the existing storage/boot environment rather than GPU power, PCIe resource allocation, or driver configuration.

## Security / Documentation Policy

Linux validation data will be sanitized before it is added to the public repository.

The public documentation will not include:

- Public or private IP addresses unless generalized for an example
- MAC addresses
- Personally identifying usernames or hostnames
- Tailscale identifiers
- SSH keys or fingerprints tied to the live system
- Hardware serial numbers
- Filesystem UUIDs where they add no technical value
- Credentials, tokens, or secrets

Command output used as evidence will be trimmed to the fields needed to demonstrate hardware detection and configuration.

## Next Validation Checkpoint

Before installing any AI software, the base operating system will be inventoried and validated.

Planned checks:

```bash
cat /etc/os-release
uname -r
lscpu
free -h
lsblk -o NAME,SIZE,TYPE,FSTYPE,MODEL
lspci | grep -Ei 'vga|3d|display|non-volatile|ethernet'
```

Additional checks may be added after reviewing the initial output.

## Validation Goals

- [x] Ubuntu installer boots successfully
- [x] Ubuntu installs successfully
- [x] Fresh Ubuntu environment boots successfully
- [ ] Confirm Ubuntu release from inside the OS
- [ ] Confirm kernel version
- [ ] Confirm Xeon Platinum 8168 / 24-core, 48-thread topology
- [ ] Confirm installed memory
- [ ] Confirm WD Blue NVMe detection
- [ ] Confirm RTX 3050 PCIe detection
- [ ] Confirm Ethernet adapter detection
- [ ] Apply OS updates
- [ ] Establish a clean Linux baseline before AMD/Vulkan/ROCm work

## Engineering Takeaway

A working Linux baseline is being established before accelerator integration so later failures can be attributed to a specific configuration change. This creates a reproducible troubleshooting path and separates operating-system bring-up from the much more complex multi-GPU AI stack.
