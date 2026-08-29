# Phase 3 — Ubuntu Baseline Installation

## Milestone

Ubuntu 26.04.1 LTS was successfully installed and booted on the HP Z6 G4 baseline configuration.

This establishes the first known-good Linux environment before any Radeon Pro V620 GPUs, AMD compute drivers, ROCm components, Vulkan tuning, Docker services, or AI inference software are introduced.

## Baseline Hardware During Installation

- HP Z6 G4 Workstation
- Intel Xeon Platinum 8168
- 32GB temporary ECC Registered memory configuration
- WD Blue SN5000 2TB NVMe as the operating-system drive
- NVIDIA RTX 3050 6GB as the display GPU
- No Radeon Pro V620 GPUs installed

The temporary memory was reduced from 64GB to 32GB for extended baseline testing because the workstation correctly reported that memory configurations above 32GB require the HP Z6 memory-cooling solution.

## Installation Notes

The workstation initially booted an older Ubuntu/LVM environment on the NVMe and dropped into emergency mode because an expected logical volume was missing.

The issue was isolated as a boot-source problem rather than a hardware failure. The system was then booted from the newly created Ubuntu 26.04.1 LTS USB installer and the fresh Ubuntu installation completed successfully.

This troubleshooting event reinforced the staged-build approach: because the V620 GPUs were still absent, the failed boot could be isolated to the existing storage/boot environment rather than GPU power, PCIe resource allocation, or driver configuration.

## Linux Hardware Validation

The new installation was inventoried from the terminal before any AI-specific packages were installed.

### Operating System

Validated:

- **Ubuntu 26.04.1 LTS**
- Ubuntu release: 26.04
- Kernel: **7.0.0-30-generic**

### Processor

Linux correctly detected:

- **Intel Xeon Platinum 8168 @ 2.70GHz**
- 1 socket
- 24 physical cores
- 2 threads per core
- **48 logical CPUs**
- x86-64 architecture
- NUMA nodes: 1 in the current single-socket configuration

This matches the expected processor topology and confirms that Hyper-Threading and all CPU cores are available to the operating system.

### Memory

`free -h` reported approximately:

- **30 GiB usable system memory**, consistent with the installed 32GB physical configuration
- 8 GiB swap configured

This is the intended temporary configuration for operating-system and hardware validation while the required high-memory cooling solution remains unresolved.

### Storage

Linux correctly detected the operating-system drive as a **WD Blue SN5000 2TB NVMe SSD**.

The fresh Ubuntu installation uses a layered storage configuration that includes:

- UEFI system partition
- Separate Linux boot partition
- **LUKS-encrypted main partition**
- LVM inside the encrypted container
- ext4 root filesystem

The encrypted-root configuration was retained because it improves protection of local files, model configuration data, SSH material, and other server data if the physical workstation or drive is ever removed from the homelab.

Unique disk serials, filesystem UUIDs, and encryption identifiers are intentionally excluded from the public documentation.

### Display GPU

PCIe enumeration successfully detected:

- **NVIDIA GA107 / GeForce RTX 3050 6GB**

This validates the separate NVIDIA display path before the AMD V620 compute GPUs are introduced.

### Network Interfaces

Linux detected the workstation's Intel Ethernet hardware, including:

- Intel Ethernet Connection I219-LM
- Intel Ethernet Controller X722-class interfaces

The public repository records controller models only. MAC addresses, assigned IP addresses, live interface names where unnecessary, and other network-specific identifiers are intentionally omitted.

## NVIDIA Driver Baseline

The NVIDIA display stack was validated before introducing any AMD GPUs.

`nvidia-smi` reported:

- GPU: **NVIDIA GeForce RTX 3050**
- VRAM: **6144 MiB**
- NVIDIA driver: **595.84**
- Driver-reported CUDA compatibility: **13.2**
- Idle temperature during validation: approximately **33°C**
- Idle power draw during validation: approximately **6W**
- Reported board power limit: **70W**

The CUDA value shown by `nvidia-smi` represents the CUDA API level supported by the installed driver; it does **not** by itself confirm that a CUDA toolkit is installed.

`lspci -k` confirmed:

- NVIDIA GA107 / GeForce RTX 3050 6GB detected
- **Kernel driver in use: `nvidia`**

This provides a known-good proprietary NVIDIA driver baseline before AMD compute devices are added.

## Package Update Check

`sudo apt update` completed successfully and reported that the system packages were already up to date at this checkpoint.

No AI runtime, ROCm stack, Vulkan tuning, Docker deployment, or local-model software has been installed yet.

## Baseline Validation Summary

| Check | Result |
|---|---|
| Ubuntu 26.04.1 LTS boots | PASS |
| Kernel detected | PASS — 7.0.0-30-generic |
| Xeon Platinum 8168 detected | PASS |
| 24 cores / 48 threads available | PASS |
| Temporary 32GB memory available | PASS |
| WD Blue SN5000 2TB detected | PASS |
| LUKS + LVM encrypted root active | PASS |
| RTX 3050 detected over PCIe | PASS |
| NVIDIA proprietary driver active | PASS — 595.84 |
| `nvidia-smi` functional | PASS |
| Intel Ethernet controllers detected | PASS |
| Ubuntu package index current | PASS |
| Radeon Pro V620 installed | No — intentionally deferred |

## Security / Documentation Policy

Linux validation data is sanitized before being added to the public repository.

The public documentation will not include:

- Public or private IP addresses unless generalized for an example
- MAC addresses
- Personally identifying usernames or hostnames
- Tailscale identifiers
- SSH keys or fingerprints tied to the live system
- Hardware serial numbers
- Filesystem UUIDs where they add no technical value
- Credentials, tokens, or secrets

The original terminal screenshots used during validation are **not published as-is** because the shell prompt contains local username/hostname information. Portfolio screenshots will be recreated, cropped, or redacted later using sanitized identifiers.

## Commands Used for Baseline Inventory

```bash
cat /etc/os-release
uname -r
lscpu
free -h
lsblk -o NAME,SIZE,TYPE,FSTYPE,MODEL
lspci | grep -Ei 'vga|3d|display|non-volatile|ethernet'
nvidia-smi
lspci -k | grep -A3 -i nvidia
```

These commands provide enough hardware inventory information to establish a reproducible baseline without exposing unnecessary network or device identifiers.

## Validation Goals

- [x] Ubuntu installer boots successfully
- [x] Ubuntu installs successfully
- [x] Fresh Ubuntu environment boots successfully
- [x] Confirm Ubuntu release from inside the OS
- [x] Confirm kernel version
- [x] Confirm Xeon Platinum 8168 / 24-core, 48-thread topology
- [x] Confirm installed memory
- [x] Confirm WD Blue NVMe detection
- [x] Confirm RTX 3050 PCIe detection
- [x] Confirm Ethernet adapter detection
- [x] Confirm encrypted-root storage layout
- [x] Check package-update state
- [x] Validate NVIDIA driver state
- [x] Establish a clean Linux baseline before AMD/Vulkan/ROCm work

## Next Hardware Checkpoint

The next phase is **single-V620 integration**.

Before power is applied to the first V620, the following must be validated:

- Dedicated V620 fan shroud installation
- Fan orientation and airflow direction
- Fan power / control connection
- Two native PCIe 8-pin power connections for the first V620
- Physical clearance in Slot 2
- BIOS Slot 2 Gen3 and Resizable BAR configuration already prepared

The second V620 and auxiliary 10-pin GPU-power cable remain deferred until one V620 has been proven stable.

## Engineering Takeaway

The Linux baseline validates the core workstation platform independently of the accelerator configuration. CPU topology, memory, NVMe storage, encryption, display GPU, NVIDIA driver, and Ethernet hardware are all working before the V620s are introduced.

This creates a strong fault-isolation point: any later problem that appears after adding GPU power, passive-GPU cooling, AMD drivers, Vulkan, ROCm, or multi-GPU inference can be compared against a documented known-good operating-system state.
