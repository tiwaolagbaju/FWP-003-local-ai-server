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

## Findings

All four fans respond correctly to PWM control.

The useful control range is concentrated below roughly PWM 160–192. Above that point, fan speed is effectively saturated, with little or no additional RPM gained by increasing the command to 230 or 255.

The existing controller's 230-to-255 transition therefore provides almost no additional airflow with the replacement fans. The current high baseline remains intentionally conservative until thermal behavior is measured under the established dual-GPU workload.

## Current Status

The upgraded fan hardware and all four control channels are validated at idle.

Thermal validation under load is still pending. The next step is to repeat the established dual-V620 Vulkan inference test at the fixed 170 W/card baseline and compare junction temperatures against the previous cooling configuration.

ROCm multi-GPU testing remains paused until the cooling upgrade has been validated under the known-good Vulkan control workload.