# Hardware Inventory & Cost Tracker

## Purpose

A central goal of FWP-003 is to quantify how much practical local-AI capability can be achieved using carefully selected used enterprise/workstation hardware.

This file tracks the hardware investment alongside the capability each component contributes to the system.

## Current Hardware

| Component | Specification / Role | Status | Cost |
|---|---|---|---:|
| HP Z6 G4 | Enterprise workstation chassis/platform | Purchased | Included below |
| Intel Xeon Platinum 8168 | 24 cores / 48 threads, 6-channel DDR4, 48 PCIe 3.0 lanes | Purchased with Z6 | $366.88 total with Z6 |
| AMD Radeon Pro V620 #1 | 32GB VRAM, passive AI GPU | Purchased | Included below |
| AMD Radeon Pro V620 #2 | 32GB VRAM, passive AI GPU | Purchased | $850.00 total for pair |
| NVIDIA RTX 3050 6GB | Display / secondary compute | Already owned | $0 incremental |
| A-Tech DDR4-2666 ECC RDIMM | Planned 96GB configuration | Purchased / expanding | $253.81 documented to date |
| SK Hynix DDR4-2133 ECC RDIMM | 4×16GB temporary bring-up memory | Available | Already owned / no project cost recorded |
| WD Blue SN5000 2TB | Initial NVMe / OS and applications | Already owned | $0 incremental |
| SanDisk Optimus 5100 2TB | Secondary NVMe / models and project storage | Ordered | $262.87 |
| V620 cooling shrouds | Custom airflow adaptation | Purchased | $50.84 |
| 4× Noctua NF-A4x20 PWM | 2 fans planned per V620 | Purchased | $38.03 |
| HP 1000W PSU | System power supply | Included with workstation | Included |

## Current Documented Spend

| Category | Cost |
|---|---:|
| Dual Radeon Pro V620 GPUs | $850.00 |
| HP Z6 G4 + Xeon Platinum 8168 | $366.88 |
| ECC RAM purchased to date | $253.81 |
| GPU cooling fans | $38.03 |
| GPU cooling shrouds | $50.84 |
| SanDisk 2TB NVMe | $262.87 |
| **Current documented new spend** | **$1,822.43** |

Already-owned components are tracked separately so the final project can distinguish:

1. **New money spent specifically for FWP-003**
2. **Total effective hardware value**, if useful for comparison

This prevents the cost analysis from understating the role of existing hardware while still making the incremental project cost clear.

## Value Metrics to Calculate Later

Once the system is validated, this document will be expanded with measurable value metrics such as:

- Cost per GB of aggregate GPU VRAM
- Total system cost per successfully supported model class
- Cost vs tokens/second for representative workloads
- Cost vs maximum model size successfully loaded
- Cost vs newer alternative platforms where comparison is technically fair
- Power and thermal tradeoffs required to achieve the lower acquisition cost

## Initial VRAM Cost Observation

The two V620 GPUs provide:

- 32GB VRAM each
- 64GB aggregate GPU VRAM
- $850 combined acquisition cost

This represents an acquisition cost of approximately:

**$13.28 per GB of GPU VRAM**

This figure describes acquisition cost only. It does not account for performance, power consumption, cooling modifications, software compatibility, or the fact that aggregate VRAM does not behave identically across every workload. Those limitations will be addressed in the final analysis.

## Planned Final Configuration

### CPU

Intel Xeon Platinum 8168

- 24 cores / 48 threads
- 2.7 GHz base frequency
- 6-channel DDR4 memory
- 48 PCIe 3.0 lanes

### Memory

Target:

- 6×16GB DDR4 ECC Registered DIMMs
- 96GB total
- Full six-channel population

Temporary bring-up:

- 4×16GB SK Hynix DDR4-2133 ECC RDIMMs

### GPU Layout

| Slot | Planned Device |
|---|---|
| 2 | Radeon Pro V620 #1 |
| 4 | RTX 3050 |
| 5 | Radeon Pro V620 #2 |

### Storage

| Device | Planned Role |
|---|---|
| WD Blue SN5000 2TB | Initial OS / application NVMe |
| SanDisk Optimus 5100 2TB | Model / data / experiment storage |

## Open Hardware Risks / Unknowns

These items must be resolved before the final dual-GPU configuration is considered validated:

### GPU Power

The workstation has sufficient total PSU wattage on paper, but the current system does not have enough confirmed native GPU power connectors for both V620s.

There is an unused 10-pin PSU-side connector that must be positively identified and its intended use/pinout verified before it is used for GPU power.

**Safety rule:** SATA-to-PCIe and Molex-to-PCIe GPU power adapters will not be used.

### GPU Cooling

The V620 is a passive server GPU and depends on chassis airflow. The planned custom fan/shroud solution must be validated under sustained load before the system is considered stable.

### PCIe / BIOS Resources

The Z6 must successfully allocate resources to two large GPUs plus the display GPU and NVMe devices. BIOS/PCIe settings will be validated on the actual platform rather than copied directly from an HP Z4 reference build.

## Cost Tracking Rule

Every new hardware purchase made specifically for FWP-003 should be added here with:

- Item
- Purpose
- Purchase cost
- Whether it was new, used, or already owned
- Any value or compatibility tradeoff discovered during the build

This will allow the final project to make a defensible **AI capability-per-dollar** argument based on actual measured results.