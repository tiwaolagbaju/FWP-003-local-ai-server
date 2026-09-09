# Phase 33 — V620 Fixed-Profile Stability Test

## Purpose

After repeated hard locks with either Radeon Pro V620 installed individually, the next controlled test changed only the V620 power/performance behavior while keeping the same one-V620 hardware configuration, RTX 3050, kernel, BIOS, cooling, and 170 W power cap.

The goal was to determine whether the previously observed low-power idle state or aggressive AMDGPU clock/power-gating behavior was required to reproduce the failure.

## Result

The workstation hard-locked again during the fixed-profile observation period.

The final preserved telemetry sample was captured at roughly 4 hours and 58 minutes of uptime and showed that the V620 was **not** in its earlier deep-idle state immediately before the freeze:

- V620 core clock around 2 GHz
- V620 memory clock 1000 MHz
- V620 power about 74 W
- V620 edge/junction temperature about 56 C
- V620 memory temperature about 52 C
- CPU package about 34 C
- system load below 1
- system memory largely free with swap unused
- V620 PCIe endpoint remained in D0 with full x16 width
- PCIe AER corrected, non-fatal, and fatal counters remained at zero
- EDAC corrected and uncorrected memory counters remained at zero
- no failed systemd units were recorded

The telemetry stream again stopped abruptly without an orderly shutdown sequence.

## Interpretation

This result is important because the hard lock is no longer associated only with the V620's lowest-power idle condition. Earlier failures were captured at roughly 6 W with minimum reported clocks, while this failure occurred with the V620 held at substantially higher clocks and power.

Therefore:

- the 6–7 W deep-idle state is **not required** to reproduce the hard lock
- a simple "deep idle is broken" explanation is weakened
- forcing higher clocks / reducing ordinary idle clock transitions did **not** eliminate the issue
- gradual thermal overload remains a poor fit because all recorded temperatures were moderate
- ordinary logged PCIe AER and ECC memory faults were again absent

The remaining fault domain is more consistent with a broader AMDGPU/KFD/HMM, PCIe path, firmware/platform, mixed-GPU, or system-level power-management interaction rather than one specific V620 or one specific low-power GPU state.

## Next Isolation Direction

The next controlled test should move upward from GPU-local DPM to platform-level power management while changing one major variable at a time. Candidate next steps include reviewing HP Z6 BIOS power-management settings, especially CPU/package C-states and any workstation performance/idle-power policy, while restoring the V620 to its normal automatic performance mode.

Ubuntu's desktop power profile is already set to performance, Intel P-state is active, and the CPU exposes C1, C1E, and C6 idle states. The deeper CPU/platform idle path therefore remains a reasonable area for a BIOS-level A/B test.

ROCm and GPU stress testing remain paused until platform stability improves.
