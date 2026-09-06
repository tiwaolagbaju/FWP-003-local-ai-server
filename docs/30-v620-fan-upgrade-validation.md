# Phase 26 — V620 Cooling Fan Upgrade Validation

## Goal

Validate the replacement cooling fans installed for the dual Radeon Pro V620 GPUs before resuming sustained GPU workloads.

## Baseline State

The workstation remained on the validated `7.0.0-31-generic` kernel with both V620 power caps fixed at 170 W. The dynamic ARCTIC fan-control service remained enabled and active.

At idle, both V620s reported approximately:

- 28 C edge temperature
- 30 C junction temperature
- 6–7 W board power

## Fan Controller Validation

The ARCTIC controller exposes independent PWM channels for each connected fan. The control script was inspected and confirmed to write the requested PWM value to all four active channels.

A manual PWM sweep was performed across all four GPU cooling fans.

| PWM | Fan 1 RPM | Fan 2 RPM | Fan 3 RPM | Fan 4 RPM |
|---:|---:|---:|---:|---:|
| 32 | 1941 | 2000 | 2058 | 2117 |
| 64 | 3647 | 3794 | 3794 | 3941 |
| 96 | 5000 | 5264 | 5147 | 5323 |
| 128 | 5735 | 6088 | 5852 | 6088 |
| 160 | 6058 | 6411 | 6205 | 6500 |
| 192 | 6088 | 6647 | 6264 | 6588 |
| 220 | 6117 | 6647 | 6323 | 6647 |
| 230 | 6147 | 6676 | 6294 | 6676 |
| 255 | 6029 | 6647 | 6264 | 6617 |

## Fan-Control Findings

All four fans respond correctly to PWM control.

The useful control range is concentrated below roughly PWM 160–192. Above that point, fan speed is effectively saturated, with little or no additional RPM gained by increasing the command to 230 or 255.

The existing controller's 230-to-255 transition therefore provides almost no additional airflow with the replacement fans. The high baseline remains intentionally conservative for GPU cooling.

## Load Thermal Validation

The established dual-V620 Vulkan inference workload was repeated at the fixed 170 W/card baseline after the fan upgrade.

During the completed run, the monitoring capture showed:

| GPU | Junction Temperature | Board Power |
|---|---:|---:|
| V620 #1 | 80 C | 170 W |
| V620 #2 | 71 C | 163 W |

The prior comparable cooling configuration had reached approximately 92 C and 86 C junction temperature under the same 170 W-class workload. The new cooling configuration therefore showed a substantial reduction in observed junction temperature while maintaining the established power target.

Because the monitoring capture is an observed point from the completed run rather than an independently logged maximum-temperature trace, these values are documented as observed load temperatures rather than guaranteed absolute peaks.

## Current Status

The replacement fan hardware, independent PWM control across all four active channels, and thermal behavior under the known-good dual-V620 Vulkan workload are validated.

The cooling upgrade materially improves thermal margin at the established 170 W/card operating point.

ROCm multi-GPU testing can now resume cautiously, beginning with low-risk single-GPU validation and local telemetry before retrying controlled dual-GPU concurrency.