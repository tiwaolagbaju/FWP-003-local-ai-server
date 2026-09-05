# Phase 20 — ARCTIC GPU Fan Controller Integration

## Goal

Improve control and observability of the dedicated cooling fans used on the two passive Radeon Pro V620 GPUs.

An ARCTIC 10-port USB/PWM fan controller was added so each GPU fan could be monitored and controlled independently from Linux rather than relying on shared workstation fan headers.

## Hardware Layout

- 2× Radeon Pro V620 32 GB
- 4× dedicated 40 mm PWM fans total
- Fan-controller channels 1–2: V620 #1
- Fan-controller channels 3–4: V620 #2
- Remaining controller channels unused during validation

## Linux Integration

The controller was detected over USB and exposed as an ARCTIC Fan Controller.

The running Linux kernel did not yet include the controller's native hwmon driver, so the upstream ARCTIC hwmon driver was built as an out-of-tree kernel module against the installed kernel headers. Secure Boot was disabled, allowing the test module to load normally.

The driver exposed:

- `fan1_input` through `fan10_input` for RPM monitoring
- `pwm1` through `pwm10` for independent PWM control

A small local initialization change was made so the driver's PWM cache matched the intended startup control state before the first outbound PWM report. Implementation-specific local paths are intentionally omitted from this public documentation.

## Initial Validation

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

After returning all four channels to the 90% baseline, a later validation showed:

| Channel | PWM | RPM |
|---|---:|---:|
| Fan 1 | 230 / ~90% | 4,029 |
| Fan 2 | 230 / ~90% | 3,794 |
| Fan 3 | 230 / ~90% | 4,323 |
| Fan 4 | 230 / ~90% | 4,382 |

## GPU Idle Check After Reinstallation

After reinstalling both V620s and booting normally, both cards were detected and showed low idle temperatures before any AI workload:

- V620 #1: approximately 34 C junction, 6 W
- V620 #2: approximately 36 C junction, 6 W

## Persistent Linux Fan Control

The out-of-tree fan-controller module was installed into the active kernel module tree and registered for automatic loading during Ubuntu startup.

The original fan service applied a fixed PWM 230 (~90%) baseline after Linux boot. Reboot validation confirmed that the kernel module and fan-control service both loaded automatically and the four GPU fans returned to the expected high-airflow baseline without manual intervention.

## Dynamic Load-Based Fan Control

After validating the fixed 90% baseline, the service was upgraded from a one-shot startup action to a continuously running controller.

The current policy is intentionally aggressive because acoustics are not a design constraint for this server:

- normal baseline: PWM 230 (~90%)
- full-speed trigger: either V620 reaches 50 W or more, or either GPU junction reaches 60 C
- full-speed state: PWM 255 (100%) on all four dedicated GPU fans
- cooldown return: once the workload has ended and the hottest V620 junction is 55 C or lower, all four fans return to PWM 230
- fail-safe behavior: if required GPU temperature telemetry becomes unavailable, the controller defaults to full fan speed

The GPU power policy remains separate from the fan controller. Both V620s remain capped at **170 W**.

## Dynamic Control Validation

A short inference load confirmed that the power-based trigger worked correctly:

- GPU power crossed 50 W
- the controller immediately switched all four channels to PWM 255
- when the load ended and the GPUs cooled, the service returned the fans to PWM 230

The same behavior was then validated during a sustained dual-V620 inference workload. The controller detected a peak load near the 170 W GPU cap and switched to full fan speed early in the run.

Observed full-speed fan RPMs during the sustained workload were approximately:

| Channel | PWM | RPM |
|---|---:|---:|
| Fan 1 | 255 / 100% | 4,294 |
| Fan 2 | 255 / 100% | 4,205 |
| Fan 3 | 255 / 100% | 4,382 |
| Fan 4 | 255 / 100% | 4,558 |

A representative end-of-run thermal snapshot showed both V620s operating close to their fixed 170 W limits, with junction temperatures in the high-80s to approximately 90 C range. GPU memory temperatures remained much lower, around the low-50s C.

The controller intentionally remained at full speed after inference stopped while the warmer card was still above the 55 C cooldown threshold. Once the hottest GPU reached 55 C, the service automatically returned to the 90% baseline.

Observed post-cooldown fan speeds were approximately:

| Channel | PWM | RPM |
|---|---:|---:|
| Fan 1 | 230 / ~90% | 4,000 |
| Fan 2 | 230 / ~90% | 3,882 |
| Fan 3 | 230 / ~90% | 4,147 |
| Fan 4 | 230 / ~90% | 4,264 |

## Current Operating Plan

- The HP workstation's PCIe idle fan setting is configured for maximum chassis airflow.
- Once Linux loads, the four dedicated V620 fans run at a high ~90% baseline.
- Any meaningful GPU workload automatically pushes all four dedicated fans to 100%.
- The fans remain at 100% during cooldown until the hottest V620 reaches 55 C or lower.
- Both V620s remain capped at 170 W for efficiency and thermal control.
- The fan-control service and out-of-tree kernel module are treated as kernel-dependent components and are revalidated after kernel upgrades.

This cooling strategy was developed after long-context testing showed that sustained prompt prefill can create substantially higher thermal load than short-prompt inference. The resulting configuration prioritizes thermal margin and sustained reliability over noise.