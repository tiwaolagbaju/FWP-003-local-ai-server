# Phase 33 — Alternate PCIe Path Stability Validation

## Purpose

To isolate whether the repeated V620 hard-lock behavior was tied to a specific PCIe path, the same single Radeon Pro V620 used in the prior isolation tests was moved to a different CPU-connected PCIe path while keeping the rest of the system configuration unchanged.

## Controlled Test Configuration

The comparison intentionally held the major variables constant:

- same Radeon Pro V620
- same RTX 3050 display GPU
- same kernel and driver stack
- same BIOS settings
- same 170 W V620 power cap
- AMDGPU performance policy returned to `auto`
- no ROCm, inference, or GPU stress workload
- system flight recorder enabled
- dynamic ARCTIC cooling enabled

The V620 enumerated on a different PCIe path than the previously failing single-card position.

## Fan-Control Adjustment

The custom ARCTIC fan-control script had originally required two AMDGPU hwmon devices and therefore entered fail-safe mode when only one V620 was installed. The script was adjusted so that one or two V620 devices are supported while retaining the existing fail-safe behavior when no AMDGPU telemetry is available.

After the adjustment, the fan service remained active, the two connected V620 cooling fans operated normally, and the flight recorder, V620 power-cap service, and fan-control service were all verified active before the observation period.

## Baseline

The clean stability observation began on September 9, 2026 at approximately 06:48 EDT.

At baseline the single V620 was cool and idle:

- edge temperature about 27 C
- junction temperature about 28 C
- memory temperature about 28 C
- GPU power about 7 W
- core clock at the minimum reported idle state
- memory clock at the minimum reported idle state
- 170 W power cap active

## Result

The alternate-PCIe-path test remained stable for nearly nine hours and was intentionally accepted as a successful controlled observation.

This exceeds the previously recorded single-V620 hard-lock windows in the original test position, including:

- roughly 1.5 hours for one single-card passive failure
- roughly 3.5 hours for another single-card passive failure
- roughly 5 hours during a fixed-performance-profile experiment that also ultimately hard-locked

The alternate-path result therefore represents the longest controlled single-V620 stability run since the repeated hard-lock investigation began.

## Interpretation

This result materially increases suspicion around the original PCIe slot/path or another platform resource unique to that path.

It does not yet prove that the physical slot itself is defective. Other path-specific factors can include upstream PCIe switching, root-port behavior, firmware/resource allocation, signal integrity, or device/platform power-management behavior associated with that topology.

The result also weakens a simple explanation based only on:

- one defective V620
- V620 deep-idle power state
- ordinary thermal overload
- V620 power cap value
- Ubuntu desktop power profile

The system should remain on the validated alternate PCIe path while the next investigation step compares the old and new upstream PCIe topology before making unrelated BIOS or OS power-management changes.

## Current Working Conclusion

The alternate PCIe path is now the preferred known-stable single-V620 configuration.

The original PCIe path remains a leading suspect and should be investigated directly before changing multiple platform settings at once.

## Safety / Test Policy

- retain the known-good recovery kernel
- keep ROCm execution paused until the hardware/platform fault domain is narrowed further
- change one major variable per test
- keep diagnostic logs sanitized before publishing
- do not expose host-specific network identifiers, serial numbers, UUIDs, or raw private logs
