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

Observed / configured:

- Slot enabled
- Option ROM Download: Enabled
- Limit PCIe Speed changed from `Auto` to **Gen3 (8 Gbps)**
- Bifurcation: Auto
- Resizable BAR: currently **Disabled**

HP documents Slot 2 as a CPU-connected **PCIe Gen3 x16** slot, so forcing Gen3 matches the platform's native capability.

### Slot 5 — Planned V620 #2

Observed / configured:

- Slot enabled
- Option ROM Download: Enabled
- Limit PCIe Speed changed from `Auto` to **Gen3 (8 Gbps)**
- Bifurcation: Auto
- Resizable BAR: currently **Disabled**

HP documents Slot 5 as a CPU-connected **PCIe Gen3 x16** slot, so forcing Gen3 also matches the native capability of this slot.

### M.2 SSD0 and SSD1

The PCIe speed limit for both native M.2 slots was changed from `Auto` to **Gen3**.

This is consistent with the Z6 G4 platform design: both native M.2 slots are PCIe Gen3 x4.

The WD Blue SN5000 is a newer-generation NVMe drive, but it is backward compatible and will operate at the Z6 platform's Gen3 limit.

### Current PCIe Speed Showing Gen1

The BIOS currently reports `Current PCIe Speed: Gen1 (2.5 Gbps)` on empty / lightly initialized slots. This is not being treated as a fault at this stage. The important setting for the planned GPUs is the configured maximum link speed of Gen3. Actual negotiated speed will be checked from Linux after the devices are installed and under a fully initialized operating system.

## Resizable BAR

Resizable BAR controls are visible per slot and are currently **Disabled** on the two planned V620 slots.

This is the next PCIe setting to evaluate. The reference dual-V620 build enabled Resizable BAR, and AMD's Linux guidance recommends enabling Resizable BAR together with large-address-space support for some large-BAR GPU configurations.

The intended next test configuration is therefore:

- Slot 2 Resizable BAR → **Enable**
- Slot 5 Resizable BAR → **Enable**

The RTX 3050 slot will be left unchanged unless testing or the software stack provides a reason to modify it.

## BIOS Change Log

| Setting | Previous | New | Reason | Validation |
|---|---|---|---|---|
| PCIe MMIO Assignment Mode | Auto | 32 Bit | Match known-working dual-V620 reference configuration | Pending |
| Slot 2 PCIe speed limit | Auto | Gen3 | Match Z6 native Gen3 x16 capability and reduce link-training uncertainty | Pending |
| Slot 5 PCIe speed limit | Auto | Gen3 | Match Z6 native Gen3 x16 capability and reduce link-training uncertainty | Pending |
| M.2 SSD0 PCIe speed limit | Auto | Gen3 | Match Z6 native Gen3 x4 M.2 interface | Pending OS validation |
| M.2 SSD1 PCIe speed limit | Auto | Gen3 | Match Z6 native Gen3 x4 M.2 interface | Pending OS validation |

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
- [ ] Enable and validate Resizable BAR on Slots 2 and 5
- [ ] Advanced → Boot Options
- [ ] Advanced → Performance Options
- [ ] Storage / NVMe detection
- [ ] Identify any additional Above-4G / large PCIe resource control if exposed elsewhere
- [ ] Record Slot 4 settings for the RTX 3050

## Planned GPU Slot Targets

| Slot | Planned Device | PCIe Target |
|---|---|---|
| Slot 2 | Radeon Pro V620 #1 | Gen3 x16, Resizable BAR enabled if validated |
| Slot 4 | NVIDIA RTX 3050 | Leave on stable/default configuration initially |
| Slot 5 | Radeon Pro V620 #2 | Gen3 x16, Resizable BAR enabled if validated |

## Engineering Takeaway

The Z6 exposes enough low-level PCIe controls to make the multi-GPU configuration reproducible. Explicitly documenting each BIOS change—including original values, reasons, and later validation results—turns what could be trial-and-error tuning into a controlled integration and troubleshooting process.
