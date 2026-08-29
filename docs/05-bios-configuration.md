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

The following values were observed before any changes were made:

| Setting | Observed Value | Current Decision |
|---|---|---|
| sSATA Controller | Enabled | Leave unchanged |
| sSATA Controller RAID Mode | Enabled | Review later only if storage configuration requires it |
| 1TB Memory Cap | Auto | Leave unchanged |
| **PCIe MMIO Assignment Mode** | **32 Bit** | **Leave at 32 Bit for initial V620 testing** |
| PCIe Training Reset | Disable | Leave unchanged for now |
| PCIe ACS | Enable | Leave unchanged for now |
| Power Button Override | 4 sec | Unrelated to GPU bring-up |

## PCIe MMIO Assignment Mode

The most important finding on this page is that **PCIe MMIO Assignment Mode is already set to 32 Bit**.

This matches the configuration used in the dual-V620 HP Z4 reference build, where the creator reported that the system would not boot correctly with the large-GPU configuration unless this setting was configured appropriately.

Because the Z6 is already set to 32 Bit, no change is required at this checkpoint.

This setting will be validated empirically once the V620s are installed rather than changed preemptively.

## Current BIOS Change Policy

No BIOS settings will be changed simply to imitate the reference video.

The process is:

1. Record the Z6 default/current state.
2. Identify the equivalent Z4 setting from the reference build.
3. Determine whether the Z6 already satisfies that requirement.
4. Change only settings that are necessary for successful hardware initialization.
5. Record every change and the resulting system behavior.

This preserves a known-good baseline and makes troubleshooting reversible.

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

The Z6 BIOS already contains a platform-specific PCIe MMIO control that directly relates to the large-address-space requirements of multi-GPU configurations. Recording the current state before modifying anything reduces risk and creates a defensible troubleshooting record for the final portfolio documentation.
