# Phase 35 — Fan Controller Restart-Loop Correlation

## Why This Was Reviewed

After the single-V620 configuration began remaining stable through multiple passive tests, attention shifted from PCIe slot and auxiliary-power-cable theories toward a software/service condition that had changed during troubleshooting.

A review of prior service logs identified a significant difference between at least one known failing single-V620 run and the current stable runs.

## Prior Single-GPU Behavior

The original fan-control script was written to require two AMDGPU hwmon devices. When only one V620 was installed, the script treated that state as a fault, set the GPU cooling fans to full PWM, exited with a failure status, and was immediately restarted by systemd.

During a known failing single-V620 run, this produced a continuous restart loop every few seconds. The service restart counter exceeded five thousand before the run ended. Each cycle repeated device discovery, fan-controller access, PWM writes, process creation, and journal logging.

The script was later corrected so that a one-V620 configuration is considered valid. The fail-safe now triggers only when no AMDGPU hwmon device is found.

## Current Stable Behavior

In the post-fix single-V620 runs, the fan-control service starts once and remains active continuously. The service reports its normal and full PWM values at startup and does not enter the previous failure/restart cycle.

This stable service behavior has now coincided with successful passive operation using both tested GPU power cables in the same PCIe position that had previously reproduced hard locks.

## Interpretation

The timing makes the former fan-service restart loop a meaningful contributing-factor candidate for the single-V620 hard locks. It is not yet proven to be the root cause.

A userspace service restart should not ordinarily hard-lock an entire Linux host. However, this service communicates with an out-of-tree fan-controller kernel module through hwmon/sysfs. Repeated hardware-control operations every few seconds could expose a driver, controller, locking, or bus-level issue that would not be expected from an ordinary shell-script restart loop alone.

The restart-loop explanation also does not account for all historical failures. Earlier dual-V620 freezes occurred in configurations where the two-GPU requirement should have been satisfied. Therefore the specific single-GPU restart condition cannot be treated as a universal explanation for every previous hard lock.

## Current Working Hypothesis

The fan-control stack is now considered a higher-priority investigation area, particularly for the single-V620 failures. The strongest evidence is the contrast between:

- a known failing single-V620 run with thousands of fan-service restarts; and
- current stable single-V620 runs with one clean fan-service start and continuous operation.

The PCIe-slot and GPU-power-cable theories have both weakened because the system has remained stable in the previously problematic PCIe position with both cable variants.

## Next Step

Do not deliberately restore the broken restart-loop configuration. Instead, continue using the corrected fan service and compare historical kernel logs for fan-controller, hwmon, USB/bus, workqueue, lockup, and watchdog messages around known failure windows.

ROCm execution remains paused until the full-system hard-lock behavior is better understood.
