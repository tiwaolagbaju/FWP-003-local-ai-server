# Phase 2 — Pre-Build Hardware Inspection

## Purpose

Before installing the Radeon Pro V620 GPUs, the HP Z6 G4 was inspected in a baseline state to identify expansion layout, available power connections, cooling hardware, and any configuration risks that could affect the build.

The inspection is intentionally documented before modification so that later troubleshooting can be tied to a known starting point.

## Initial Findings

### Chassis and Expansion Area

The workstation is a single-CPU HP Z6 G4 configuration with the primary PCIe expansion area available for the planned multi-GPU layout.

Planned configuration remains:

| Slot | Planned Device |
|---|---|
| Slot 2 | Radeon Pro V620 #1 |
| Slot 4 | NVIDIA RTX 3050 |
| Slot 5 | Radeon Pro V620 #2 |

This layout will still be validated after the NVMe and GPU configuration is finalized.

### GPU Power Expansion Connector

A factory-style 10-pin auxiliary graphics-power connector is present inside the chassis.

A separate cable on hand has the physical form of a **10-pin to dual 6+2-pin PCIe GPU power cable**. This matches the connector style used by the HP Z4 G4 / Z6 G4 GPU power cable commonly identified as **HP part number L15907-001**.

This is a promising path for supplying the additional PCIe power required by the second V620; however, the cable will not be energized until its compatibility and wiring are confirmed.

**Safety decision:** no SATA-to-PCIe or Molex-to-PCIe power adapters will be used.

Reference:
- HP Z4/Z6 G4 10-pin to dual 6+2-pin GPU power cable, P/N L15907-001: https://pcserverandparts.com/hp-z4-z6-g4-pcie-gpu-power-cable-10-pin-dual-6-2-pin-l15907-001/

### Memory Cooling

HP specifies a dedicated **Z6 G4 Memory Cooling Solution** for configurations using more than 32GB of total system memory.

The current chassis photographs do **not** clearly show the full HP 2HW44AA / 916799-001 memory-cooling baffle assembly. A separate fan is present near the DIMM/CPU area, but that should not be assumed to be the complete HP memory-cooling solution without further verification.

Because the planned system memory is 96GB, this is an open hardware item that must be resolved before the final memory configuration is considered complete.

HP reference:
- HP Z6 G4 Memory Cooling Solution, option kit 2HW44AA
- HP Z6 G4 QuickSpecs: configurations with greater than 32GB total system memory require the memory cooling solution.

### Front / Internal Cooling

The chassis includes multiple internal fans and clear front-to-rear airflow paths. The existing system airflow will be preserved where practical because the two Radeon Pro V620 GPUs are passive server cards and depend on forced airflow.

The V620 cooling modification will be validated separately using custom shrouds and dedicated fans under sustained AI workloads.

## Documentation Security Review

Raw inspection photographs contain machine-specific CT/barcode identifiers and are not being published in their original form.

Any images added to the public repository will be cropped or redacted to remove:

- CT / barcode identifiers
- serial numbers
- service tags
- MAC addresses
- asset labels
- other machine-specific identifiers

The objective is to demonstrate the engineering process without exposing unnecessary identifying information.

## Open Items Before Baseline Power-On

- [ ] Confirm the workstation PSU rating / part number
- [ ] Verify the 10-pin GPU power cable part number or wiring
- [ ] Confirm whether the full HP Z6 G4 Memory Cooling Solution is installed
- [ ] Record temporary RDIMM part number and specifications
- [ ] Confirm DIMM population order for the baseline boot
- [ ] Install the WD Blue SN5000 in SSD0
- [ ] Install the RTX 3050 as the initial display GPU
- [ ] Boot with no V620s installed
- [ ] Record BIOS version and baseline hardware detection

## Engineering Takeaway

This inspection identified two important integration issues before high-power GPUs were installed:

1. **Power distribution** — the Z6 platform provides an auxiliary 10-pin GPU-power path that may allow the system to support the additional PCIe connectors required for dual V620s.
2. **Memory thermal management** — the planned 96GB memory configuration introduces an OEM cooling requirement that must be validated rather than ignored.

Identifying these constraints before assembly reduces troubleshooting complexity and helps establish a controlled, repeatable build process.
