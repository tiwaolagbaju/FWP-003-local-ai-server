# Phase 22 — CPU Cooling Validation

## Goal

Validate CPU thermals after returning the original CPU-cooler fans to service and compare the result against the previously tested quieter replacement fans.

## Test Method

A repeatable CPU-only stress test was used so the cooling change could be evaluated without GPU load affecting the result.

- CPU: Intel Xeon Platinum 8168
- workload: 24 `stress-ng` CPU workers
- test duration: 5 minutes
- telemetry: `turbostat` and Linux hardware sensors

Because the processor exposes 48 hardware threads, 24 stress workers produced roughly 50% aggregate CPU busy time while still driving sustained package power near 200 W.

## Results

During the validation run, the CPU maintained approximately:

- busy frequency: ~3.4 GHz
- package power: typically ~200–208 W
- package temperature: generally mid-60s to low-70s C
- observed peak package temperature: approximately 72 C
- thermal-throttle indicator: 0 throughout the observed load

The CPU frequency remained effectively flat during the test rather than falling as temperature increased, indicating that the processor was not thermally throttling.

## Comparison

The earlier test with the quieter replacement CPU-cooler fans reached approximately 83 C at a similar ~200 W package load while maintaining the same ~3.4 GHz operating frequency.

Returning to the original CPU-cooler fans reduced peak CPU package temperature by roughly 10–11 C under the same style of workload.

## Decision

The original CPU-cooler fans will remain installed for the current build.

For this server, acoustics are a low priority compared with sustained thermal margin and reliability. The validated result provides additional headroom for the future local-agent workload, where the CPU may simultaneously handle orchestration, tokenization, retrieval, tool execution, container services, and other supporting tasks while the GPUs are active.

GPU thermal work remains a separate track. The Radeon Pro V620 cooling hardware is being revised before additional long-duration dual-GPU stress testing.