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

## Cooling Validation

Before the permanent cooling system was finalized, a controlled dual-GPU inference test showed both V620s under simultaneous compute load at approximately:

- V620 #1: **71 W**, junction **51 C**
- V620 #2: **82 W**, junction **51 C**

After the permanent shrouds and four-fan configuration were installed, a fresh idle health check showed:

- V620 #1: **33 C edge / 36 C junction / 30 C memory / ~7 W**
- V620 #2: **33 C edge / 35 C junction / 30 C memory / ~6 W**

Both cards were enumerated correctly by Vulkan/llama.cpp with approximately **30.7 GB usable VRAM each**, and the kernel log showed no AMD GPU reset, timeout, PCIe bus error, AER error, uncorrected error, or Surprise Link Down event.

A subsequent dual-V620 llama.cpp inference run using Qwen2.5-7B-Instruct Q4_K_M produced the following active-load sensor snapshot with the permanent cooling setup:

### V620 #1

- edge: **57 C**
- junction: **59 C**
- memory: **52 C**
- PPT: **74 W**
- sclk: **923 MHz**
- mclk: **1000 MHz**

### V620 #2

- edge: **50 C**
- junction: **54 C**
- memory: **50 C**
- PPT: **83 W**
- sclk: **1 GHz**
- mclk: **1000 MHz**

The workload generated at approximately **64.8 tokens/s**, confirming active simultaneous inference while both passive accelerators remained below **60 C junction temperature** in the captured load snapshot.

The first card ran approximately 5 C warmer at the junction than the second card in this snapshot, but both remained far below their reported critical temperature thresholds.

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
| Idle dual-GPU health check | **PASS** |
| Idle V620 junction temperatures | **35–36 C** |
| Captured V620 #1 load junction | **59 C** |
| Captured V620 #2 load junction | **54 C** |
| Captured V620 #1 memory temp | **52 C** |
| Captured V620 #2 memory temp | **50 C** |
| Captured dual-GPU PPT | **74 W / 83 W** |
| Dual-GPU inference under permanent cooling | **PASS — short duration** |
| GPU/PCIe error check | **Clean at idle health check** |
| Long-duration thermal validation | Pending |
| Motherboard header current-limit verification | Pending |

## Engineering Significance

The passive V620s now have a purpose-built permanent cooling solution rather than an improvised fan arrangement. The design uses two fans per accelerator and splits their electrical load across separate motherboard fan circuits.

The permanent setup demonstrated very low idle temperatures and kept both GPU junction temperatures below 60 C during active dual-GPU inference at roughly 74–83 W per card. This is an encouraging thermal result for passive server accelerators adapted to a workstation chassis.

## Next Validation Step

The permanent cooling system has passed its first active-compute check. The next useful validation is a longer and larger-model workload while monitoring both V620 edge, junction, memory temperature, clocks, and PPT separately, followed by a PCIe/AER log review.

If those results remain clean, the cooling portion of the physical build can be considered validated for sustained benchmarking.