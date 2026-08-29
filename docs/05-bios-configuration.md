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

The following values were observed during BIOS inspection:

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

This is therefore a **documented configuration change**, not the factory/default state of this workstation.

### Change Record

- Previous value: `Auto`
- New value: `32 Bit`
- Reason: reproduce the known-working MMIO configuration from the dual-V620 reference build as a controlled starting point
- Validation status: pending V620 installation and POST testing

If later testing shows that `Auto` or another setting is more appropriate on the Z6 G4, the configuration will be reverted or adjusted and the result documented.

## Current BIOS Change Policy

BIOS changes will be treated as controlled experiments rather than undocumented tweaks.

The process is:

1. Record the original Z6 state whenever possible.
2. Identify the equivalent setting used in the Z4 reference build.
3. Make one relevant change at a time.
4. Record the reason for the change.
5. Validate POST, device detection, and system stability.
6. Revert or revise a setting if testing does not support it.

This preserves a troubleshooting trail and makes the final build reproducible.

## Remaining BIOS Pages to Inspect

- [ ] Advanced → Slot Settings
- [ ] Advanced → Boot Options
- [ ] Advanced → Performance Options
- [ ] Storage / NVMe detection
- [ ] Any Resizable BAR setting
- [ ] Any Above-4G / large BAR / large PCIe resource setting
- [ ] Per-slot PCIe generation controls for Slots 2, 4, and 5

## Planned GPU Slot Targets

| Slot | Planned Device | Intended Link Setting |
|---|---|---|
| Slot 2 | Radeon Pro V620 #1 | PCIe Gen3 if manually selectable |
| Slot 4 | NVIDIA RTX 3050 | PCIe Gen3 if required / appropriate |
| Slot 5 | Radeon Pro V620 #2 | PCIe Gen3 if manually selectable |

No slot-speed changes will be made until the actual Slot Settings page is reviewed.

## Engineering Takeaway

The Z6 BIOS exposes a platform-specific PCIe MMIO control that may be important for allocating address space to multiple large GPUs. More importantly, the configuration process is being documented as a sequence of intentional, reversible changes rather than presenting modified values as defaults.
