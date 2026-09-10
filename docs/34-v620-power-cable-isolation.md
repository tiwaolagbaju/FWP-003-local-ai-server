# Phase 34 — V620 Power-Cable Isolation

## Purpose

After repeated hard locks with a single Radeon Pro V620 in the same PCIe position, the investigation was narrowed to two major remaining variables: the PCIe path and the GPU auxiliary power connection.

A controlled A/B test was performed by keeping the same V620 in the previously problematic PCIe position while changing only the GPU power cable.

## Test Configuration

The validated test kept the following conditions unchanged:

- same Radeon Pro V620
- same PCIe position and host path
- same RTX 3050 display GPU
- same kernel and driver stack
- same BIOS configuration
- Resizable BAR enabled
- PCIe host path operating at full x16 width
- V620 power cap at 170 W
- AMDGPU performance policy left at `auto`
- no ROCm, inference, or GPU stress workload during the passive observation
- flight recorder, fan controller, and V620 power-cap services active

The only intentional hardware change relative to prior failing runs was use of a different GPU auxiliary power cable.

## New-Cable Result

The system remained stable well beyond the previously observed single-V620 failure windows while using the new power cable in the same PCIe position that had repeatedly hard-locked before.

This result was initially considered a successful passive stability checkpoint and strengthened suspicion of the previous cable or connection quality.

## Reverse A/B With the Prior Cable

The prior GPU power cable was then reinstalled while keeping the same V620, same PCIe position, same BIOS, same kernel/driver stack, same 170 W cap, same `auto` DPM policy, and the same passive observation method.

The reverse A/B baseline began late in the evening and remained stable for more than seven and a half hours, exceeding all previously well-recorded single-V620 passive failure windows, including the roughly 1.5-hour, 3.5-hour, and roughly 5-hour observations.

At startup for this reverse test, the V620 was cool and idle, with the same low-power state seen in prior passive tests, and the V620 power-cap, fan-control, and flight-recorder services were all active.

## Updated Interpretation

Because both the new-cable and prior-cable configurations have now remained stable for substantially longer than the earlier repeatable failure windows in the same PCIe position, the auxiliary power cable is no longer a strong standalone root-cause explanation.

The prior cable is therefore not considered proven defective. The intentionally unpopulated connector position observed on that cable should not be treated as fault evidence by itself.

One important troubleshooting confounder was discovered during the recent single-GPU testing: the original ARCTIC fan-control script required at least two AMDGPU hwmon devices. In a one-V620 configuration, this caused the service to enter fail-safe, set maximum fan PWM, exit, and restart repeatedly. The script was later corrected to support either one or two V620s. Although this restart loop does not explain earlier dual-V620 hard locks, it means some previous single-V620 passive failures occurred under a different service behavior than the current stable runs and should be interpreted with that limitation in mind.

The remaining fault domain therefore still includes:

- AMDGPU/KFD/HMM or GPU lifecycle behavior
- mixed NVIDIA/AMD interaction
- PCIe/platform behavior that is intermittent rather than strictly slot-dependent
- system firmware or processor/platform power-state interactions
- another condition that changed between the earlier failure period and the current stable observations

## Current Status

The current single-V620 configuration has now remained stable through both cable variants in the previously problematic PCIe position. This materially weakens both a simple bad-slot explanation and a simple bad-cable explanation.

Further testing should continue to change only one major variable at a time. A useful next isolation step is V620-only headless operation with the RTX 3050 physically removed, after preserving the current stable checkpoint. ROCm execution should remain paused until the underlying full-system hard-lock behavior is better understood.
