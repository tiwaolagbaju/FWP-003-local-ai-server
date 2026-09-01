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

Important distinction: suppressing the firmware warning does **not** add airflow or establish that the DIMMs are adequately cooled. HP offers a dedicated Z6 G4 memory-cooling solution for this platform. The current configuration therefore remains subject to controlled thermal and memory-stability validation under realistic workloads.

The workstation is intended to operate in a cool environment, but ambient temperature alone is not sufficient evidence of DIMM thermal margin under sustained memory traffic or CPU-offload workloads.

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

Both Radeon Pro V620 accelerators also continued to report:

- MEM ECC active
- GECC enabled

These GPU ECC messages are expected and separate from host-memory EDAC monitoring.

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
| Corrected host-memory errors at baseline | **None observed** |
| Uncorrected host-memory errors at baseline | **None observed** |
| Direct DIMM temperature telemetry | **Not exposed** |
| Sustained memory stability test | Pending |

## Engineering Significance

The workstation now has the planned 96GB system-memory configuration in addition to the two V620 accelerators. This provides substantially more host memory for model loading, CPU/GPU offload, larger context windows, containers, and future experimentation with models that exceed aggregate GPU VRAM.

The clean EDAC baseline gives a useful before-test reference. Because DIMM temperatures are not directly exposed, stability and ECC behavior under controlled memory load will be used as the primary validation signals for the current cooling configuration.

## Next Validation Step

Before treating the 96GB configuration as production-stable:

1. Run a controlled memory stress test while monitoring available sensors.
2. Review kernel logs afterward for ECC/EDAC/MCE/memory errors.
3. Confirm that the dual-V620 configuration remains stable after the RAM change.
4. Revisit dedicated memory airflow if corrected ECC events, instability, or other thermal symptoms appear.
