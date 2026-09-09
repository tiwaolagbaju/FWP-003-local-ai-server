# Phase 32 — Independent V620 Isolation Result

## Purpose

After repeated dual-V620 hard locks, each Radeon Pro V620 was tested independently in the same PCIe test position while the RTX 3050 remained installed. No ROCm, inference, or GPU stress workload was intentionally running during the passive observations.

## Result

Both V620 cards independently reproduced the system hard lock.

The second-card test remained responsive for roughly 3.5 hours before the telemetry stream and journal stopped abruptly. The final complete recorder sample showed an apparently healthy, lightly loaded system immediately before the lock:

- CPU package around 31 C
- system load below 1
- system memory largely free, with swap unused
- V620 around 26 C edge, 28 C junction, and 28 C memory temperature
- V620 power around 6 W
- V620 core clock reported at 0 MHz and memory clock at 96 MHz
- RTX 3050 around 28 C and about 5 W in an idle performance state
- V620 PCIe endpoint in D0 with a full-width link
- PCIe AER corrected, non-fatal, and fatal counters at zero
- EDAC corrected and uncorrected memory counters at zero
- no failed systemd units
- no kernel messages preserved in the several-minute window around the lock
- pstore empty and no persistent RAS record recovered

The first independently tested V620 had previously failed under a similarly cool, low-power idle condition. Taken together, these results make a single defective V620 substantially less likely as the sole cause.

## Interpretation

The hard-lock issue is not limited to dual-V620 operation and is reproducible with either V620 installed individually alongside the RTX 3050. Previous failures have also occurred during or near GPU/ROCm activity, so the issue should not be described as idle-only.

The remaining common fault domain includes:

- AMDGPU/KFD/HMM behavior
- dynamic GPU power/clock-state transitions
- mixed NVIDIA/AMD GPU interaction
- the shared PCIe path or platform resource behavior
- workstation firmware/platform interaction

The repeated low-power final snapshots make a DPM or power-state transition hypothesis worth testing, but they do not prove that power management is the root cause.

## Next Controlled A/B Test

Keep the same single-V620 hardware configuration and current software stack. Change only the AMDGPU performance-policy behavior temporarily, then repeat passive stability observation.

Baseline:

- `power_dpm_force_performance_level=auto`
- V620 can reach approximately 0 MHz core / 96 MHz memory at idle
- hard lock reproduced

Next test:

- temporarily set `power_dpm_force_performance_level=profile_standard`
- keep the same card, slot, kernel, driver, BIOS, cooling, RTX placement, and 170 W power cap
- do not install OS updates or launch ROCm/inference workloads during the passive observation
- retain the flight recorder

If the system remains stable for an extended passive period, follow with a controlled workload while keeping the same power profile. If the system still hard-locks, restore `auto` and continue isolating the PCIe/platform and driver paths.

## Safety / Test Policy

- one major variable per test
- keep ROCm execution paused during passive isolation
- retain the known-good recovery kernel
- do not publish raw private logs or host-specific identifiers
