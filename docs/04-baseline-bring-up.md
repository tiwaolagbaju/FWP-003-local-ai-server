# Phase 2 — Baseline Bring-Up

## Baseline Configuration

The HP Z6 G4 was powered on with a deliberately minimal configuration to establish a clean troubleshooting baseline before introducing either Radeon Pro V620 GPU.

Installed for the first POST test:

- Intel Xeon Platinum 8168
- 4×16GB SK hynix DDR4-2133 ECC Registered RDIMMs — 64GB total
- WD Blue SN5000 2TB NVMe
- NVIDIA RTX 3050 6GB display GPU
- No Radeon Pro V620 GPUs installed

## First POST Result

The workstation successfully reached POST, confirming that the platform was able to initialize with the temporary CPU, memory, storage, and display configuration.

POST then stopped with the following message:

> **517 — Memory configuration requires a Memory fan and this fan is not detected.**

This was an expected integration risk identified during the pre-build inspection because HP specifies additional memory cooling for Z6 G4 configurations above 32GB total system memory.

## BIOS Baseline Validation

After acknowledging the POST warning, the system successfully entered HP Computer Setup / BIOS.

The BIOS reported:

| Item | Detected Value | Result |
|---|---|---|
| Platform | HP Z6 G4 Workstation | PASS |
| CPU | Intel Xeon Platinum 8168 @ 2.70GHz | PASS |
| Memory | 64GB ECC DDR4 | PASS |
| BIOS | P60 v03.00 | Recorded |
| BIOS date | 04/15/2026 | Recorded |
| Display output | NVIDIA RTX 3050 | PASS |

This confirms that all four temporary 16GB RDIMMs are being recognized as ECC DDR4 and that the Xeon Platinum 8168 is correctly identified by firmware.

The BIOS version is sufficiently recent for baseline testing, so no firmware update will be performed merely to match settings shown in the HP Z4 reference video. Z6-specific settings will be inspected on the actual platform before changes are made.

## Diagnosis of POST 517

The firmware is detecting the installed 64GB memory configuration and enforcing the platform's memory-cooling requirement.

This is **not** being treated as a RAM-compatibility failure. The system identifies the full 64GB ECC memory configuration correctly; the unresolved issue is the required memory fan / cooling assembly detection.

## Immediate Decision

The warning will **not** be disabled or ignored for sustained operation.

Because the system can enter BIOS with the 64GB configuration, hardware and BIOS inventory can continue temporarily while the cooling requirement is resolved.

Two safe options remain for longer testing:

1. Temporarily reduce the configuration to **32GB (2×16GB)** if extended operation is required before the memory-cooling solution is installed.
2. Install / reconnect the correct HP Z6 G4 memory-cooling hardware before continuing with 64GB or the final 96GB configuration.

The final build will use 96GB and therefore requires a proper resolution rather than suppressing the warning.

## Why This Matters

This is a useful example of why the build is being performed incrementally rather than installing every component at once.

Because the V620 GPUs were not yet installed, the POST warning could immediately be isolated to the memory/cooling configuration rather than being confused with GPU power, PCIe resource allocation, or driver issues.

## Troubleshooting Record

| Item | Result |
|---|---|
| System reaches POST | PASS |
| System enters BIOS | PASS |
| CPU correctly detected | PASS |
| 64GB ECC DDR4 correctly detected | PASS |
| Display output from RTX 3050 | PASS |
| BIOS version recorded | PASS |
| Memory cooling requirement satisfied | FAIL / OPEN |
| V620 GPUs involved | No |

## Next Actions

- [ ] Verify whether the HP Z6 G4 Memory Cooling Solution is physically present but disconnected, or absent
- [ ] Inspect motherboard memory-fan header / cooling assembly connection
- [ ] If cooling hardware is absent, source the correct Z6 G4 memory-cooling solution
- [ ] Inspect BIOS System Options and PCIe / Slot configuration pages
- [ ] Record NVMe detection
- [ ] Identify Z6-specific large-PCIe-resource / MMIO settings
- [ ] Confirm PCIe generation settings for planned GPU slots
- [ ] Do not install V620 GPUs until the baseline system is stable and the required BIOS settings are understood

## Engineering Takeaway

The first POST produced a useful validation result rather than a failed build. The workstation successfully initialized the baseline platform, correctly detected the processor and all 64GB of ECC memory, and exposed a platform-specific thermal requirement before any high-power accelerator hardware was added.

This staged validation reduces fault-isolation time: introduce one class of change at a time, verify system behavior, document the result, and resolve exceptions before increasing complexity.
