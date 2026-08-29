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

## Diagnosis

The firmware is detecting the installed 64GB memory configuration and enforcing the platform's memory-cooling requirement.

This is not currently being treated as a RAM-compatibility failure. The machine reached POST and the error specifically identifies the missing or undetected memory fan.

HP documentation and support material for the Z6 G4 state that configurations with greater than 32GB total system memory require the HP Z6 Memory Cooling Solution.

## Immediate Decision

The warning will **not** be disabled or ignored for sustained operation.

Two safe paths are available for continued bring-up:

1. Temporarily reduce the baseline configuration to **32GB (2×16GB)** so the system can be tested without triggering the >32GB memory-cooling requirement.
2. Install / reconnect the correct HP Z6 G4 memory-cooling hardware before continuing with 64GB or the final 96GB configuration.

For a quick baseline validation, temporarily reducing memory to 32GB is acceptable. The final build will require the correct cooling solution because the target configuration is 96GB.

## Why This Matters

This is a useful example of why the build is being performed incrementally rather than installing every component at once.

Because the V620 GPUs were not yet installed, the POST warning could immediately be isolated to the memory/cooling configuration rather than being confused with GPU power, PCIe resource allocation, or driver issues.

## Troubleshooting Record

| Item | Result |
|---|---|
| System reaches POST | PASS |
| Display output from RTX 3050 | PASS |
| 64GB memory configuration initializes far enough for firmware validation | PASS |
| Memory cooling requirement satisfied | FAIL / OPEN |
| V620 GPUs involved | No |

## Next Actions

- [ ] Verify whether the HP Z6 G4 Memory Cooling Solution is physically present but disconnected, or absent
- [ ] Inspect motherboard memory-fan header / cooling assembly connection
- [ ] If cooling hardware is absent, source the correct Z6 G4 memory-cooling solution
- [ ] Optionally reduce temporary bring-up memory to 2×16GB (32GB) and continue baseline BIOS validation
- [ ] Record BIOS version, CPU detection, memory detection, and NVMe detection
- [ ] Do not install V620 GPUs until the baseline system is stable

## Engineering Takeaway

The first POST produced a useful validation result rather than a failed build. The workstation successfully initialized the baseline platform and exposed a platform-specific thermal requirement before any high-power accelerator hardware was added.

This is exactly the type of staged validation that reduces fault isolation time in production infrastructure work: introduce one class of change at a time, verify system behavior, document the result, and resolve exceptions before increasing complexity.
