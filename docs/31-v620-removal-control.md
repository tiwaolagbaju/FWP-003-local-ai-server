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

This is the first extended observation period after the repeated ROCm-era hard locks in which the V620/AMDGPU/KFD path was completely removed from the system. The successful overnight run does not by itself prove a root cause, but it materially strengthens the association between the instability and the removed AMD/V620 path rather than the base Z6 platform.

A separate CPU cooling observation remains open: during a short 24-worker CPU stress test, the CPU package reached around 80 C without an audible automatic fan ramp. The CPU workload itself completed successfully with no reported computation errors. This fan-control behavior is being investigated separately and should not be conflated with the prior V620/ROCm hard-lock issue.

## Dual-V620 Reinstallation Result

After the successful RTX-only overnight control period, both V620s were reinstalled for a passive stability observation. Before the reinstallation test, the nearby intake fan was repositioned so it no longer blew directly into the V620 cooling-fan path, and the GPU cooling baseline was raised.

The workstation later froze again with both V620s installed. No conclusion should be drawn from the airflow change alone; moving the intake fan and increasing cooling did not prevent the recurrence.

This result further weakens the intake-fan-interference hypothesis as a complete explanation and strengthens the association between system instability and the presence of the V620/AMDGPU/KFD/PCIe path. It still does not distinguish between a specific card, a specific PCIe path, the AMDGPU/KFD stack, or another platform interaction.

## Control-Test Goal

Run the workstation normally in this RTX-only configuration for an extended observation period without launching ROCm containers or making additional GPU-driver changes.

Interpretation:

- if the workstation remains stable, the repeated lockups become strongly associated with the V620/AMDGPU/KFD/PCIe side of the system
- if the workstation still hard-locks, the investigation must broaden to the platform, memory, CPU, power, kernel, or NVIDIA side

## Current Policy

During the RTX-only control period:

- do not launch ROCm containers
- do not reinstall either V620
- do not change IOMMU or PCIe settings
- keep V620-specific power-cap and cooling automation disabled
- keep the custom ARCTIC fan-controller module unloaded unless it is specifically needed for a later controlled test

After a sufficiently long stable control period, the next hardware isolation step should be to reinstall only one V620 and repeat validation one card at a time rather than returning directly to the dual-V620 configuration.