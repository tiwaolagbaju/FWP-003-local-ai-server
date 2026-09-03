# Phase 20 — ARCTIC GPU Fan Controller Integration

## Goal

Improve control and observability of the dedicated cooling fans used on the two passive Radeon Pro V620 GPUs.

An ARCTIC 10-port USB/PWM fan controller was added so each GPU fan could be monitored and controlled independently from Linux rather than relying on shared workstation fan headers.

## Hardware Layout

- 2× Radeon Pro V620 32 GB
- 4× dedicated 40 mm PWM fans total
- Fan-controller channels 1–2: V620 #1
- Fan-controller channels 3–4: V620 #2
- Remaining controller channels unused during initial validation

## Linux Integration

The controller was detected over USB and exposed as an ARCTIC Fan Controller.

The running Linux kernel did not yet include the controller's native hwmon driver, so the upstream ARCTIC hwmon driver was built as an out-of-tree kernel module against the installed kernel headers. Secure Boot was disabled, allowing the test module to load normally.

The driver exposed:

- `fan1_input` through `fan10_input` for RPM monitoring
- `pwm1` through `pwm10` for independent PWM control

A small local initialization change was made so the driver's PWM cache matched the intended startup control state before the first outbound PWM report. Implementation-specific local paths are intentionally omitted from this public documentation.

## Validation

At the controller's startup/default airflow, the four connected fans reported approximately:

| Channel | RPM |
|---|---:|
| Fan 1 | 2,000 |
| Fan 2 | 1,882 |
| Fan 3 | 2,117 |
| Fan 4 | 2,323 |

All unused channels reported 0 RPM.

Linux PWM control was then validated. With channels 2–4 commanded to approximately 90% and channel 1 temporarily commanded to 100%, observed speeds were:

| Channel | PWM | RPM |
|---|---:|---:|
| Fan 1 | 255 / 100% | 4,029 |
| Fan 2 | 230 / ~90% | 3,500 |
| Fan 3 | 230 / ~90% | 4,088 |
| Fan 4 | 230 / ~90% | 4,147 |

The substantial RPM increase confirmed that both USB output control and RPM feedback were functioning correctly.

## GPU Idle Check After Reinstallation

After reinstalling both V620s and booting normally, both cards were detected and showed low idle temperatures before any AI workload:

- V620 #1: approximately 34 C junction, 6 W
- V620 #2: approximately 36 C junction, 6 W

## Current Operating Plan

- Controller hardware startup behavior remains unchanged during POST/BIOS.
- Once Linux loads, the four dedicated GPU fans will be commanded to a high fixed baseline speed.
- Sustained AI workloads will be validated again only after boot-time fan control is made persistent.
- GPU power limiting will be evaluated separately after cooling control is fully verified.

This fan-controller work was prompted by the earlier long-context stress test, which showed that near-maximum sustained prompt prefill can create substantially higher thermal load than short-prompt inference.