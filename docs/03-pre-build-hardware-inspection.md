# Phase 2 — Pre-Build Hardware Inspection

## Purpose

Before installing the Radeon Pro V620 GPUs, the HP Z6 G4 was inspected in a baseline state to identify expansion layout, available power connections, cooling hardware, memory compatibility, storage, and configuration risks that could affect the build.

The inspection is intentionally documented before modification so later troubleshooting can be tied to a known starting point.

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

### Power Supply

The installed PSU input label is rated for:

- 100–127V AC at 12A
- 200–240V AC at 6.3A
- 50/60 Hz

These input ratings are consistent with the 1000W HP Z6 G4 power-supply configuration used as the basis for this build.

The total PSU capacity is therefore considered sufficient on paper for the planned system, but total wattage alone does not validate individual connector loading or cable compatibility. GPU-power distribution will be treated as a separate validation step.

### GPU Power Expansion Connector

A factory-style 10-pin auxiliary graphics-power connector is present inside the chassis.

A separate cable on hand has the physical form of a **10-pin to dual 6+2-pin PCIe GPU power cable**. This is the type of cable needed to provide two additional PCIe 8-pin connections for the second V620.

The cable has **not** been energized yet. Its wiring and platform compatibility must be confirmed before it is used with either V620.

**Safety decision:** no SATA-to-PCIe or Molex-to-PCIe power adapters will be used.

### Temporary Bring-Up Memory

The temporary memory was positively identified as:

**SK hynix HMA42GR7AFR4N-TF**

Per DIMM:

- 16GB
- DDR4-2133
- PC4-2133P
- 2Rx4
- Registered ECC RDIMM

Four matching DIMMs are available for a temporary total of **64GB** during baseline bring-up.

The Xeon Platinum 8168 supports faster DDR4 memory, but these temporary modules will operate at their rated 2133 MT/s. Their purpose is to establish a known-good baseline before the final 96GB DDR4-2666 configuration is installed.

DIMMs will be populated according to the HP service-panel memory-loading order, using the first four recommended positions rather than simply filling four physically adjacent sockets.

### Final Planned Memory

The target configuration remains:

- 6×16GB DDR4-2666 ECC Registered RDIMMs
- 96GB total
- all six memory channels populated

### Memory Cooling

HP specifies a dedicated **Z6 G4 Memory Cooling Solution** for configurations using more than 32GB of total system memory.

The current chassis photographs show additional cooling hardware near the CPU/DIMM area, but the complete HP memory-cooling assembly has not yet been positively identified.

Because both the temporary 64GB configuration and planned 96GB configuration exceed 32GB, this remains an open hardware-validation item.

### Baseline NVMe

The initial operating-system drive was visually verified as a:

**WD Blue SN5000 2TB NVMe SSD**

The drive is already installed in the HP M.2 carrier and is planned for the initial SSD0 / operating-system role.

Unique device serial information visible on the physical label is intentionally excluded from the public documentation.

### Baseline Display GPU

The planned display GPU is the already-owned **Yeston-branded NVIDIA RTX 3050 6GB**.

Its role during initial bring-up is to provide display output while keeping the V620 GPUs completely out of the baseline configuration. This creates a clean troubleshooting point before AMD compute hardware is introduced.

### Radeon Pro V620 Pair

Both Radeon Pro V620 GPUs have been physically inspected before installation.

Each card provides:

- 32GB VRAM
- passive cooling
- dual-slot form factor
- dual external 8-pin PCIe power inputs

Combined target AI VRAM: **64GB aggregate**.

Neither card will be installed until the baseline workstation has successfully completed POST and the GPU-power and cooling plans have been validated.

### Front / Internal Cooling

The chassis includes multiple internal fans and clear front-to-rear airflow paths. Existing system airflow will be preserved where practical because the two Radeon Pro V620 GPUs are passive server cards and depend on forced airflow.

The V620 cooling modification will be validated separately using custom shrouds and dedicated fans under sustained AI workloads.

## Documentation Security Review

Raw inspection photographs contain machine- and component-specific identifiers and are not being published in their original form.

Examples observed during inspection include:

- chassis CT / barcode identifiers
- SSD serial information
- RAM FRU / inventory labels
- workstation identification labels

Any images added to the public repository will be cropped or redacted to remove:

- serial numbers
- CT / barcode identifiers
- service tags
- MAC addresses
- asset labels
- unique storage identifiers
- other machine-specific identifiers

The objective is to demonstrate the engineering process without exposing unnecessary identifying information.

## Baseline Hardware Configuration

The first boot will intentionally use a minimal known-good configuration:

| Component | Baseline Configuration |
|---|---|
| CPU | Intel Xeon Platinum 8168 |
| Memory | 4×16GB SK hynix DDR4-2133 ECC RDIMM — 64GB |
| Storage | WD Blue SN5000 2TB NVMe |
| Display GPU | NVIDIA RTX 3050 6GB |
| Radeon Pro V620 | **Not installed** |

## Open Items Before Baseline Power-On

- [x] Confirm 1000W-class PSU configuration from electrical input ratings
- [ ] Verify the 10-pin GPU power cable wiring / platform compatibility
- [ ] Confirm whether the full HP Z6 G4 Memory Cooling Solution is installed
- [x] Record temporary RDIMM part number and specifications
- [ ] Confirm the exact first-four DIMM physical positions from the HP service-panel diagram
- [ ] Install / verify WD Blue SN5000 in SSD0
- [ ] Install RTX 3050 as the initial display GPU
- [ ] Boot with no V620s installed
- [ ] Record BIOS version and baseline hardware detection

## Engineering Takeaway

The pre-build inspection has already reduced several major integration risks before the high-power GPUs are installed:

1. **Power capacity** — the workstation is equipped with the expected 1000W-class PSU configuration.
2. **Power distribution** — an auxiliary 10-pin GPU-power path and matching-form-factor dual-PCIe cable are available, but will be electrically validated before use.
3. **Memory compatibility** — four matching 16GB ECC Registered DIMMs are available to establish a 64GB baseline configuration.
4. **Storage and display isolation** — the system can be brought up using a known NVMe drive and a separate NVIDIA display GPU before introducing the V620s.
5. **Memory thermal management** — the final high-memory configuration still requires confirmation of the OEM cooling requirement.

This incremental approach creates clean troubleshooting checkpoints and documents the same type of risk-based validation used in production infrastructure work.
