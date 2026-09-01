# Phase 12 — 96GB Six-Channel Memory Upgrade

## Goal

Upgrade the HP Z6 G4 from the temporary 32GB test configuration to the planned 96GB memory configuration using six matching 16GB DDR4 DIMMs on CPU0.

## Installed Configuration

- CPU0-DIMM1: 16GB DDR4 @ 2666 MT/s
- CPU0-DIMM2: 16GB DDR4 @ 2666 MT/s
- CPU0-DIMM3: 16GB DDR4 @ 2666 MT/s
- CPU0-DIMM4: 16GB DDR4 @ 2666 MT/s
- CPU0-DIMM5: 16GB DDR4 @ 2666 MT/s
- CPU0-DIMM6: 16GB DDR4 @ 2666 MT/s

Total installed capacity: **96GB**.

All six CPU0 memory channels are populated with one DIMM each, which is the preferred layout for maximizing memory bandwidth on the single-socket Xeon platform.

## Linux Validation

`free -h` reports approximately **90 GiB** of usable system memory. This is expected for a nominal 96GB installed configuration because Linux reports memory in binary GiB while DIMM capacities are marketed in decimal GB.

The system also retained the configured 8GB swap device.

`dmidecode` confirmed that all six CPU0 DIMM locations are populated and operating at **2666 MT/s configured memory speed**.

## Memory Cooling / Firmware Warning

At capacities above the previous 32GB test configuration, the workstation firmware presented its memory-fan warning. The build is currently using a **non-OEM hardware workaround that suppresses the fan-detection warning** rather than the HP memory-cooling accessory.

The exact jumper/pinout procedure is intentionally not documented here or presented as a recommended modification.

Important distinction: suppressing the firmware warning does **not** add airflow or establish that the DIMMs are adequately cooled. The current configuration therefore remains subject to controlled thermal and memory-stability validation under realistic workloads.

## Baseline Sensor Snapshot

Before memory stress testing, the system was allowed to idle and a baseline sensor capture was taken.

Observed values included:

- CPU package: **38 C**
- CPU cores: approximately **33–38 C**
- PCH: **53 C**
- V620 #1: **44 C edge / 46 C junction / 42 C memory / ~7 W**
- V620 #2: **44 C edge / 46 C junction / 44 C memory / ~7 W**
- NVMe #1 composite: **~25 C**
- NVMe #2 composite: **~36 C**

Linux does **not** currently expose individual DIMM temperature sensors through `lm-sensors`, so DIMM thermal behavior cannot be measured directly from this sensor set.

## EDAC / ECC Baseline

The Linux EDAC subsystem initialized successfully for both Skylake memory controllers:

- Socket 0 IMC #0 detected by `skx_edac`
- Socket 0 IMC #1 detected by `skx_edac`

The baseline kernel-log review showed **no corrected or uncorrected system-memory errors**.

Both Radeon Pro V620 accelerators also continued to report MEM ECC active and GECC enabled.

## Controlled Memory Stress Test

A short-duration validation was run with `stress-ng` using approximately **70GB** of memory across six VM workers:

```text
6 VM workers
70GB total allocation
5 minute runtime
--verify enabled
```

Results:

- workers passed: **6/6**
- failed: **0**
- skipped: **0**
- untrustworthy metrics: **0**
- successful runtime: **~5 minutes**

During the test:

- CPU package reached approximately **79 C**
- CPU reported high threshold: **91 C**
- CPU reported critical threshold: **101 C**
- PCH remained approximately **53 C**

The system remained stable for the duration of the test.

## Post-Test EDAC Review

After the stress test, the kernel log was checked again for:

- EDAC events
- corrected memory errors
- uncorrected memory errors
- MCE events
- other memory-error indicators

No new corrected or uncorrected **host-memory** errors were observed.

The only ECC-related entries remained the expected initialization messages for the host EDAC controllers and the two V620 GPU ECC/GECC states.

## Result

| Check | Result |
|---|---|
| Six DIMMs detected | **PASS** |
| Capacity per DIMM | **16GB** |
| Total installed capacity | **96GB** |
| CPU0-DIMM1 through DIMM6 populated | **PASS** |
| Configured memory speed | **2666 MT/s** |
| Linux usable memory | **~90 GiB** |
| Balanced six-channel layout | **PASS** |
| Firmware fan-warning suppression | **Working — non-OEM workaround** |
| EDAC controllers initialized | **PASS** |
| 70GB / 5-minute stress-ng verification | **PASS** |
| stress-ng workers | **6/6 passed** |
| Corrected host-memory errors after test | **None observed** |
| Uncorrected host-memory errors after test | **None observed** |
| Peak observed CPU package temperature | **79 C** |
| Direct DIMM temperature telemetry | **Not exposed** |
| Short-duration 96GB stability | **PASS** |
| Long-duration / production thermal validation | Pending |

## Engineering Significance

The 96GB six-channel memory configuration passed a controlled 70GB memory workload with verification enabled and no observed host-memory ECC errors. This provides a strong short-duration stability baseline for the upgraded system.

Because individual DIMM temperatures are not exposed, this result should not be interpreted as full thermal qualification of the non-OEM memory-fan bypass. Longer workloads and real AI/CPU-offload use should continue to be monitored for stability and EDAC events.

## Next Validation Step

With the 96GB short-duration memory test complete, the project can return to dual-V620 AI validation. The next useful milestone is a larger model that makes meaningful use of the system's approximately 61.4GB aggregate usable GPU VRAM and 96GB host RAM, while continuing to monitor GPU power, thermals, PCIe stability, and host-memory EDAC status.
