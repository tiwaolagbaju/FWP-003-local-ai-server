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

## Fixed-Profile DPM Test

A later A/B test temporarily changed only the V620 performance policy from the normal automatic DPM behavior to a fixed profiling mode intended to reduce clock/power-gating transitions.

That test also reproduced the hard lock after roughly five hours. The final pre-freeze sample was materially different from the earlier idle-state captures:

- V620 power was about 74 W rather than the usual single-digit idle draw
- core clock was around 2 GHz and memory clock around 1000 MHz
- GPU temperatures remained moderate, in the low-to-mid 50 C range
- PCIe endpoint remained in D0 at full link width
- PCIe AER counters remained at zero
- EDAC corrected and uncorrected memory counters remained at zero
- system memory and CPU temperatures remained normal

This result weakens the theory that the lowest V620 idle clock/power state by itself is required to trigger the freeze. It does not eliminate AMDGPU power-management behavior more broadly, but it shifts attention toward shared driver, PCIe-path, platform, and firmware interactions.

## Alternate PCIe Path Test

The same V620 was then moved to a different CPU-connected PCIe path while keeping the rest of the software and hardware configuration unchanged. The V620 was returned to its normal `auto` DPM state, the same 170 W cap was retained, and no ROCm, inference, or stress workload was launched.

The cooling script was also corrected to support a one-V620 configuration. Its previous two-GPU assumption caused the service to enter fail-safe and restart continuously when only one V620 was installed; the fail-safe itself held the connected fans at maximum speed and was not evidence of a GPU thermal problem.

At the latest checkpoint, the alternate-slot configuration had remained stable for more than six hours, exceeding the prior approximately 3.5-hour single-card automatic-DPM failure and the approximately five-hour fixed-profile failure window. This is a meaningful divergence, but one successful partial-day run is not yet sufficient to identify the original PCIe path as the root cause.

If the alternate-slot configuration remains stable overnight, the original PCIe slot/path and its upstream platform resources become substantially more suspicious. If the alternate slot also hard-locks, the investigation should shift back toward shared AMDGPU/KFD/HMM, mixed-GPU, and platform/firmware behavior.

## Interpretation

The hard-lock issue is not limited to dual-V620 operation and is reproducible with either V620 installed individually alongside the RTX 3050. Previous failures have also occurred during or near GPU/ROCm activity, so the issue should not be described as idle-only.

The remaining common fault domain includes:

- AMDGPU/KFD/HMM behavior
- dynamic GPU power/clock-state transitions
- mixed NVIDIA/AMD GPU interaction
- PCIe path or platform resource behavior
- workstation firmware/platform interaction

The fixed-profile failure weakens a simple low-idle-state explanation. The alternate-slot test is currently the strongest hardware-path A/B comparison and should remain otherwise untouched until it either fails or completes an extended overnight observation.

## Safety / Test Policy

- one major variable per test
- keep ROCm execution paused during passive isolation
- retain the known-good recovery kernel
- do not publish raw private logs or host-specific identifiers
