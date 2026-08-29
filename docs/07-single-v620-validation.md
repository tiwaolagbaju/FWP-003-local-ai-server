# Phase 4 — Single V620 Integration

## Milestone

The HP Z6 G4 successfully completed POST and booted Ubuntu 26.04.1 LTS with the first AMD Radeon Pro V620 installed in the planned Slot 2 position.

This is the first successful accelerator-integration checkpoint and validates that the workstation can at least initialize one V620 alongside the NVIDIA RTX 3050 display GPU with the current BIOS configuration.

## Configuration at This Checkpoint

- HP Z6 G4 Workstation
- Intel Xeon Platinum 8168
- Temporary 32GB ECC Registered memory
- WD Blue SN5000 2TB NVMe
- NVIDIA RTX 3050 6GB display GPU
- **AMD Radeon Pro V620 #1 installed in Slot 2**
- V620 #2 not installed

## Relevant BIOS State

- PCIe MMIO Assignment Mode: 32 Bit
- Slot 2 speed limit: Gen3
- Slot 2 Resizable BAR: Enabled
- Fast Boot: Disabled

These settings are not yet considered fully validated. Successful POST is the first data point; Linux device enumeration, driver binding, BAR allocation, link width/speed, temperature monitoring, and load stability still need to be verified.

## Current Result

| Check | Result |
|---|---|
| System POSTs with one V620 | PASS |
| Ubuntu boots with one V620 | PASS |
| RTX 3050 display path remains functional | PASS |
| V620 visible to Linux | Pending |
| `amdgpu` driver bound | Pending |
| PCIe Gen3 link validated | Pending |
| Resizable BAR / BAR allocation validated | Pending |
| V620 temperature visible | Pending |
| Sustained load test | Pending |

## Next Validation Steps

The next checks will be performed from Ubuntu **before installing ROCm or AI software**:

1. Confirm that Linux enumerates the V620.
2. Confirm which kernel driver is bound.
3. Review kernel messages for BAR/resource or `amdgpu` errors.
4. Inspect PCIe link capability and negotiated state.
5. Inspect Resizable BAR / memory-region allocation.
6. Establish idle temperature monitoring.
7. Only after those checks pass, install Vulkan tooling and perform the first compute test.

## Engineering Takeaway

The staged approach continues to reduce troubleshooting scope. Because the system was already documented as stable before the V620 was added, successful POST and Ubuntu boot demonstrate that the first accelerator can coexist with the existing platform without immediately introducing a firmware or PCIe initialization failure.
