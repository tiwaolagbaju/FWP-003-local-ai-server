# Phase 2 — BIOS / PCIe Configuration

## Purpose

Before installing the Radeon Pro V620 GPUs, the HP Z6 G4 BIOS is being mapped and documented so that settings from the HP Z4 reference build can be translated to this platform rather than copied blindly.

The goal is to identify the settings that affect large PCIe devices, multi-GPU resource allocation, PCIe link speed, and boot behavior.

## BIOS Version

- HP Z6 G4 Workstation
- BIOS family: P60
- BIOS version: v03.00
- BIOS date: 04/15/2026

## Advanced → System Options

| Setting | Original / Observed State | Current State | Decision |
|---|---|---|---|
| sSATA Controller | Enabled | Enabled | Leave unchanged |
| sSATA Controller RAID Mode | Enabled | Enabled | Review later only if storage configuration requires it |
| 1TB Memory Cap | Auto | Auto | Leave unchanged |
| **PCIe MMIO Assignment Mode** | **Auto** | **32 Bit** | Intentionally changed for initial V620 testing based on the reference build |
| PCIe Training Reset | Disable | Disable | Leave unchanged for now |
| PCIe ACS | Enable | Enable | Leave unchanged for now |
| Power Button Override | 4 sec | 4 sec | Unrelated to GPU bring-up |

## PCIe MMIO Assignment Mode

The HP Z6 G4 BIOS originally had **PCIe MMIO Assignment Mode set to Auto**.

During baseline BIOS inspection, this setting was intentionally changed to **32 Bit** before the configuration photo was taken. The change was made because the dual-V620 HP Z4 reference build specifically used 32-bit MMIO assignment and reported boot issues when the required PCIe resource settings were not configured correctly.

### Change Record

- Previous value: `Auto`
- New value: `32 Bit`
- Reason: reproduce the known-working MMIO configuration from the dual-V620 reference build as a controlled starting point
- Validation status: pending V620 installation and POST testing

## Advanced → Slot Settings

The Z6 exposes per-slot PCIe controls, including speed limits, bifurcation, Option ROM loading, VROC, and **Resizable BAR**.

### Slot 2 — Planned V620 #1

Configured:

- Slot enabled
- Option ROM Download: Enabled
- Limit PCIe Speed: **Gen3 (8 Gbps)**
- Bifurcation: Auto
- Resizable BAR: **Enabled**

HP documents Slot 2 as a CPU-connected **PCIe Gen3 x16** slot, so forcing Gen3 matches the platform's native capability.

### Slot 5 — Planned V620 #2

Configured:

- Slot enabled
- Option ROM Download: Enabled
- Limit PCIe Speed: **Gen3 (8 Gbps)**
- Bifurcation: Auto
- Resizable BAR: **Enabled**

HP documents Slot 5 as a CPU-connected **PCIe Gen3 x16** slot, so forcing Gen3 also matches the native capability of this slot.

### M.2 SSD0 and SSD1

The PCIe speed limit for both native M.2 slots was changed from `Auto` to **Gen3**.

This is consistent with the Z6 G4 platform design: both native M.2 slots are PCIe Gen3 x4.

The WD Blue SN5000 is a newer-generation NVMe drive, but it is backward compatible and will operate at the Z6 platform's Gen3 limit.

### Current PCIe Speed Showing Gen1

The BIOS currently reports `Current PCIe Speed: Gen1 (2.5 Gbps)` on empty / lightly initialized slots. This is not being treated as a fault at this stage. The important setting for the planned GPUs is the configured maximum link speed of Gen3. Actual negotiated speed will be checked from Linux after the devices are installed and under a fully initialized operating system.

## Resizable BAR

Resizable BAR controls are exposed per slot on the Z6 G4.

The settings for both planned V620 slots were changed from `Disable` to **Enable**:

- Slot 2 Resizable BAR → **Enabled**
- Slot 5 Resizable BAR → **Enabled**

The change mirrors the known-working dual-V620 reference build and provides a controlled starting point for large-BAR GPU initialization. The RTX 3050 slot is intentionally being left on its stable/default configuration unless later testing provides a reason to change it.

Resizable BAR will be considered validated only after:

1. The system successfully POSTs with the V620 hardware installed.
2. Linux enumerates both GPUs.
3. `lspci` confirms the expected BAR/resource allocation.
4. Vulkan or the selected inference backend can address both devices correctly.

## Advanced → Boot Options

The Boot Options page was inspected during baseline setup.

Observed / configured values:

- Startup Delay: 0 seconds
- **Fast Boot: Disabled**
- CD-ROM Boot: Enabled
- USB Storage Boot: Enabled
- SD Card Boot: Disabled
- Network (PXE) Boot: Disabled
- After Boot Device Not Found: Stop
- After Power Loss: Power Off
- Prompt on Memory Size Change: Enabled
- UEFI Boot Order: Enabled

### Fast Boot

Fast Boot was intentionally disabled for the hardware-integration phase.

The reason is not expected AI performance; it is **troubleshooting reliability**. During a multi-GPU bring-up, a normal/full POST gives firmware more opportunity to initialize and train PCIe devices, exposes hardware warnings more consistently, and makes it easier to enter BIOS or diagnostics when a change causes a problem.

This also matches the reference dual-V620 build, which left Fast Boot disabled during setup.

Fast Boot may be revisited after the final system is stable, but there is little benefit to enabling it on a remotely administered server where predictable hardware initialization is more valuable than saving a few seconds during boot.

### USB Storage Boot

USB Storage Boot remains enabled so Ubuntu installation media can be used without another BIOS change.

### After Power Loss

`After Power Loss` is currently set to **Power Off**. This is acceptable during bench configuration, but it will be reconsidered before the system is moved to its final basement-server role. An automatic recovery option may be preferable for a headless homelab server after an unexpected utility outage, provided the final configuration has been validated for safe unattended startup.

## Advanced → Performance Options

The performance page was reviewed and no changes were made.

| Setting | Current Value | Decision |
|---|---|---|
| Turbo Mode | Enabled | Keep enabled |
| Intel Hyper-Threading Technology | Enabled | Keep enabled |
| Active CPU Cores Per Processor | All | Keep all cores enabled |
| Sub-NUMA Clustering | Disabled | Leave disabled for baseline |
| Isoc Mode | Disabled | Leave disabled |
| Workload Configuration | Balanced | Leave unchanged for first benchmarks |
| Performance Control | Performance Mode | Keep enabled |

### Rationale

**Turbo Mode** and **Hyper-Threading** are useful for the general-purpose CPU work this server will perform, including model loading, prompt processing, container workloads, decompression, and CPU/RAM offload. Disabling cores or threads at this stage would reduce available compute without solving a known problem.

**Sub-NUMA Clustering** remains disabled for the first baseline. It can expose a more granular NUMA topology to software, but that adds complexity and is not needed to prove the initial multi-GPU configuration. NUMA tuning can be tested later if CPU-offloaded inference or memory-bandwidth benchmarks show a reason to investigate it.

**Isoc Mode** remains disabled because the target workload is AI inference rather than a specialized isochronous / latency-reservation workload.

**Workload Configuration = Balanced** will be kept for the first benchmark set so the machine begins from a conservative, reproducible configuration. If CPU-bound or offload-heavy tests later justify another profile, it will be changed and benchmarked as a separate experiment.

**Performance Control = Performance Mode** is already aligned with the project's goal of maximizing sustained compute performance, so no change is required.

HP's BIOS documentation exposes these same performance controls on the Z4/Z6-class workstation firmware; the current approach is to leave them stable until measured workloads give a reason to tune them further.

## BIOS Change Log

| Setting | Previous | New | Reason | Validation |
|---|---|---|---|---|
| PCIe MMIO Assignment Mode | Auto | 32 Bit | Match known-working dual-V620 reference configuration | Pending |
| Slot 2 PCIe speed limit | Auto | Gen3 | Match Z6 native Gen3 x16 capability and reduce link-training uncertainty | Pending |
| Slot 5 PCIe speed limit | Auto | Gen3 | Match Z6 native Gen3 x16 capability and reduce link-training uncertainty | Pending |
| Slot 2 Resizable BAR | Disable | Enable | Prepare V620 #1 for large-BAR / multi-GPU testing | Pending |
| Slot 5 Resizable BAR | Disable | Enable | Prepare V620 #2 for large-BAR / multi-GPU testing | Pending |
| M.2 SSD0 PCIe speed limit | Auto | Gen3 | Match Z6 native Gen3 x4 M.2 interface | Pending OS validation |
| M.2 SSD1 PCIe speed limit | Auto | Gen3 | Match Z6 native Gen3 x4 M.2 interface | Pending OS validation |
| Fast Boot | Enabled | Disabled | Improve visibility and consistency during PCIe / multi-GPU bring-up | Baseline BIOS change |

## Current BIOS Change Policy

BIOS changes are being treated as controlled experiments rather than undocumented tweaks.

1. Record the original state whenever possible.
2. Identify the equivalent setting used in the Z4 reference build.
3. Confirm the setting makes architectural sense on the Z6.
4. Change one relevant class of settings at a time.
5. Record the reason for each change.
6. Validate POST, device detection, and stability.
7. Revert or revise anything not supported by testing.

## Remaining BIOS Pages / Settings to Inspect

- [x] Advanced → Slot Settings
- [x] Enable Resizable BAR on Slots 2 and 5
- [x] Advanced → Boot Options
- [x] Advanced → Performance Options
- [ ] Validate Resizable BAR with installed V620 hardware
- [ ] Storage / NVMe detection
- [ ] Identify any additional Above-4G / large PCIe resource control if exposed elsewhere
- [ ] Record Slot 4 settings for the RTX 3050
- [ ] Revisit `After Power Loss` before final headless deployment

## Planned GPU Slot Targets

| Slot | Planned Device | PCIe Target |
|---|---|---|
| Slot 2 | Radeon Pro V620 #1 | Gen3 x16, Resizable BAR enabled |
| Slot 4 | NVIDIA RTX 3050 | Leave on stable/default configuration initially |
| Slot 5 | Radeon Pro V620 #2 | Gen3 x16, Resizable BAR enabled |

## Engineering Takeaway

The Z6 exposes enough low-level PCIe and performance controls to make the multi-GPU configuration reproducible. Explicitly documenting each BIOS change—and equally documenting when settings are reviewed and deliberately left unchanged—turns what could be trial-and-error tuning into a controlled integration and troubleshooting process.
