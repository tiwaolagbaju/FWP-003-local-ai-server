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

## Engineering Significance

The workstation now has the planned 96GB system-memory configuration in addition to the two V620 accelerators. This provides substantially more host memory for model loading, CPU/GPU offload, larger context windows, containers, and future experimentation with models that exceed aggregate GPU VRAM.

The next validation step should confirm system stability under memory load and verify that the previously validated dual-V620 configuration remains unaffected by the memory upgrade.