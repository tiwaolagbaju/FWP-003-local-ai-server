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

The V620 power-cap service reports failed because the hardware it targets is no longer installed. This is expected in the control configuration and is not treated as a new system fault.

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
- keep the V620-specific power-cap and cooling automation disabled or stopped to avoid false service failures and unnecessary hardware-control logic

After a sufficiently long stable control period, the next hardware isolation step should be to reinstall only one V620 and repeat validation one card at a time rather than returning directly to the dual-V620 configuration.