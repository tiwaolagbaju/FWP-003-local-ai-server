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