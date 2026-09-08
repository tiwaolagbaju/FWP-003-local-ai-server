# Phase 26 — V620 Removal / RTX-Only Control Baseline

## Purpose

After repeated hard locks during and after ROCm/KFD testing, both Radeon Pro V620 cards were physically removed to create a clean hardware-isolation baseline.

This control configuration intentionally removes the V620/AMDGPU/KFD path from the system without changing the RTX 3050 display stack.

## Verified State

Post-removal validation confirmed:

- the RTX 3050 is the only display/3D GPU enumerated
- the AMDGPU kernel module is not loaded
- `/dev/kfd` is not present
- the RTX 3050 is operational with the expected NVIDIA driver
- the V620 power-cap and ARCTIC fan-control services are disabled
- the prior expected V620 service failure was cleared, leaving zero failed systemd units
- the custom ARCTIC fan-controller kernel module was unloaded successfully for the control period

The ARCTIC controller's earlier `sensors` PWM percentage labels were misleading; direct sysfs inspection showed raw PWM values on a 0–255 scale. This did not provide evidence that fan duty itself caused the ROCm-related hard locks.

## Baseline Health Check

With both V620s physically absent and the ARCTIC module unloaded, the initial baseline check showed:

- CPU package temperature in the high-30 C range at idle
- PCH temperature in the mid-40 C range
- NVMe temperatures within normal operating range
- roughly 90 GiB of usable system memory with the host largely idle
- no observed MCE, EDAC memory error, NVIDIA Xid, watchdog lockup, or explicit PCIe/AER fault in the reviewed boot log

The boot log continues to report the platform's existing ACPI/PCIe capability limitations, but no new hardware-error event was identified in this RTX-only state.

## Overnight Stability Checkpoint

The workstation remained powered on and responsive overnight in the RTX-only control configuration, with both V620s physically removed, AMDGPU/KFD absent, V620-specific services disabled, and the custom ARCTIC fan-controller module unloaded.

This was the first extended observation period after the repeated ROCm-era hard locks in which the V620/AMDGPU/KFD path was completely removed from the system. The successful overnight run does not by itself prove a root cause, but it materially strengthens the association between the instability and the removed AMD/V620 path rather than the base Z6 platform.

A separate CPU cooling observation remains open: during a short 24-worker CPU stress test, the CPU package reached around 80 C without an audible automatic fan ramp. The CPU workload itself completed successfully with no reported computation errors. This fan-control behavior is being investigated separately and should not be conflated with the V620-related hard-lock issue.

## Dual-V620 Reinstallation Result

After the successful RTX-only overnight control period, both V620s were reinstalled for passive stability observation. Before the reinstallation test, the nearby intake fan was repositioned so it no longer blew directly into the V620 cooling-fan path, and the GPU cooling baseline was raised.

The workstation later froze again with both V620s installed. No ROCm workload, inference workload, or stress test was required to reproduce the failure.

This result weakens the intake-fan-interference hypothesis as a complete explanation and strengthens the association between system instability and the presence of the V620/AMDGPU/KFD/PCIe path. It still does not distinguish between a specific card, a specific PCIe path, the AMDGPU/KFD stack, or another platform interaction.

## Passive Flight-Recorder Capture

A full-system telemetry recorder was enabled during a later passive dual-V620 observation. The recorder sampled temperatures, GPU power, fan RPM, system load, memory state, GPU PCIe link state, endpoint AER counters, and EDAC counters while the system was otherwise left idle.

The workstation hard-locked again. The final complete pre-freeze sample was captured after roughly 3 hours and 38 minutes of uptime and showed a very light system load with no thermal or hardware-error trend immediately before logging stopped:

- CPU package about 30 C, with cores in the mid-20s to low-30s C
- both V620 edge temperatures about 26 C
- V620 junction temperatures about 28-29 C
- V620 memory temperatures about 26-28 C
- both V620s at only about 6-7 W and effectively idle clocks
- RTX 3050 about 32 C and about 6 W
- PCH about 39 C
- both V620 PCIe endpoints remained D0 with full-width links
- endpoint PCIe AER corrected, non-fatal, and fatal counters remained at zero
- EDAC corrected and uncorrected memory counters remained at zero
- no persistent RAS record was recovered after reboot
- pstore contained no crash record
- the previous-boot kernel journal contained no GPU reset, ring timeout, machine-check, watchdog, thermal, OOM, or explicit PCIe fault near the hard lock

The telemetry stream simply stopped after this apparently healthy sample, followed later by a manual reboot. This materially weakens gradual overheating, ordinary ECC memory failure, GPU load, and a conventional logged PCIe AER event as explanations for this occurrence. It does not exclude an abrupt device, driver, power-state, bus-level, or platform stall that prevents the kernel from recording its final state.

## Current Interpretation

The evidence now supports a narrower hardware/software isolation path:

- the base system remained stable for an extended period with both V620s physically absent
- passive dual-V620 operation can still hard-lock the workstation without a ROCm userspace workload
- the final pre-freeze telemetry remained cool and lightly loaded
- no obvious EDAC, RAS, or endpoint-AER precursor was captured

The next controlled step should isolate one V620 at a time. If both individual cards are stable alone but instability returns only with two cards installed, the investigation should focus on dual-device AMDGPU/KFD/HMM behavior, PCIe/platform interaction, resource mapping, or power-delivery interaction rather than a single defective card.

## Safety / Test Policy

Until the isolation matrix is complete:

- do not launch ROCm containers
- do not run GPU stress workloads during passive-control tests
- avoid changing IOMMU or PCIe settings at the same time as a hardware-isolation test
- change only one major variable per test
- retain the known-good recovery kernel
- keep diagnostic logs sanitized before publishing
