# Phase 13 — Permanent V620 Cooling Solution

## Goal

Replace the temporary single-fan cooling arrangement with a permanent directed-airflow solution for both passive AMD Radeon Pro V620 accelerators.

## Final GPU Cooling Configuration

Each V620 now has:

- a dedicated cooling shroud
- **2 × 40×20 mm Noctua NF-A4x20 PWM fans**
- four GPU cooling fans total across the system

This provides direct forced airflow through both passive accelerator heatsinks.

## Fan Header Distribution

The four 40 mm GPU fans are split across two motherboard fan circuits:

### MEM FAN1 path

- front chassis fan remains connected
- one 40×20 mm fan from V620 #1 is connected on this path
- one 40×20 mm fan from V620 #2 is connected on this path

### AUX FAN path

- one 40×20 mm fan from V620 #1 is connected here
- one 40×20 mm fan from V620 #2 is connected here

This distributes the four added GPU fans instead of placing all of them on a single header.

## Added Electrical Load

The Noctua NF-A4x20 PWM is rated at:

- 12 V
- **0.05 A maximum current per fan**
- **0.6 W maximum input power per fan**

Therefore:

- two added 40 mm fans on the MEM FAN1 path add approximately **0.10 A max / 1.2 W max**
- two added 40 mm fans on the AUX FAN path add approximately **0.10 A max / 1.2 W max**
- all four GPU fans together add approximately **0.20 A max / 2.4 W max**

These figures describe the added Noctua fan load only. The existing front chassis fan also contributes current on the MEM FAN1 circuit.

The exact motherboard header current limits have not yet been independently verified from an HP electrical specification, so header loading remains a configuration item to monitor rather than an assumed unlimited resource.

## Cooling Validation Status

Earlier controlled dual-GPU inference testing with directed airflow showed both V620s under simultaneous compute load at approximately:

- V620 #1: **71 W**, junction **51 C**
- V620 #2: **82 W**, junction **51 C**

Those measurements established good short-duration thermal behavior, but they were captured before the cooling system was finalized in its current permanent shroud/four-fan form.

The permanent arrangement should therefore be re-baselined during the next sustained model test.

## Result

| Check | Result |
|---|---|
| Dedicated V620 shroud #1 | **Installed** |
| Dedicated V620 shroud #2 | **Installed** |
| 40×20 mm fans per GPU | **2** |
| Total V620 cooling fans | **4** |
| MEM FAN1 GPU-fan load | **2 × 40 mm fans** |
| AUX FAN GPU-fan load | **2 × 40 mm fans** |
| Added fan current per circuit | **~0.10 A max** |
| Directed airflow on both passive V620s | **Implemented** |
| Short-duration dual-GPU thermal test | **Previously PASS** |
| Sustained thermal validation with permanent setup | Pending |
| Motherboard header current-limit verification | Pending |

## Engineering Significance

The passive V620s now have a purpose-built permanent cooling solution rather than an improvised fan arrangement. The design uses two fans per accelerator and splits their electrical load across separate motherboard fan circuits.

This removes one of the major remaining physical-build uncertainties before long-duration dual-GPU inference and large-model benchmarking.

## Next Validation Step

During the next larger-model run:

1. monitor both V620 edge, junction, memory temperature, clocks, and PPT
2. confirm all four cooling fans remain operating
3. run the workload for several minutes before increasing duration
4. inspect both fan circuits/connectors after the test for abnormal heating
5. review PCIe/AER logs after the run

If thermals and power-path stability remain clean, the build can move into sustained dual-GPU benchmarking.