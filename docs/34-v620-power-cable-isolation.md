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

## Result

The system remained stable well beyond the previously observed single-V620 failure windows while using the new power cable in the same PCIe position that had repeatedly hard-locked before.

This result is considered a successful passive stability checkpoint.

## Interpretation

The result materially weakens the hypothesis that the PCIe slot/path alone is sufficient to cause the hard lock. It strengthens suspicion of the prior auxiliary power cable or its connection quality as a contributing factor.

However, the prior cable should not yet be called definitively defective. A blank connector position by itself is not sufficient evidence of a fault because some workstation GPU power cables intentionally leave a position unpopulated depending on the connector design.

The remaining possibilities include:

- intermittent contact or terminal fit on the prior cable
- conductor or crimp resistance on the prior cable
- connector seating differences
- interaction between auxiliary power quality and GPU power-state transitions
- a lower-probability platform/PCIe interaction that has not reproduced during the current run

## Next Isolation Step

A deliberate reverse A/B test can further distinguish cable from slot/path behavior by reconnecting the prior GPU power cable while keeping the same GPU, PCIe position, BIOS, kernel, driver, DPM policy, and monitoring configuration unchanged.

Because a power cable under investigation could theoretically have a poor electrical connection, any reuse should be treated cautiously and the connector should be inspected for heat damage, discoloration, recessed terminals, or poor mechanical fit before continued operation.

## Current Status

The new-cable / original-slot configuration is the strongest stable single-V620 control achieved so far and is now the preferred reference configuration for subsequent testing.
