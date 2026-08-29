# Phase 4 — Single V620 Integration

## Milestone

The HP Z6 G4 successfully completed POST and booted Ubuntu 26.04.1 LTS with the first AMD Radeon Pro V620 installed in the planned Slot 2 position.

Linux then successfully enumerated the V620, bound it to the in-kernel `amdgpu` driver, initialized the GPU, and exposed a full 32GB PCIe BAR for the card.

This is the first successful accelerator-integration checkpoint and confirms that one V620 can coexist with the NVIDIA RTX 3050 display GPU using the current BIOS configuration.

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

## Linux Enumeration

`lspci` identified the accelerator as:

```text
Advanced Micro Devices, Inc. [AMD/ATI] Navi 21 [Radeon Pro V620]
```

The V620 appears behind its onboard AMD PCIe switch, which Linux also enumerated as Navi 10 XL upstream/downstream PCIe bridge devices.

`lspci -k` confirmed:

```text
Kernel driver in use: amdgpu
Kernel modules: amdgpu
```

This is an important result because no separate ROCm package was required for basic Linux device initialization. The standard Ubuntu kernel driver already recognizes and initializes the card.

## VRAM and BAR Initialization

Kernel initialization messages reported approximately:

```text
VRAM: 30704M
Detected VRAM RAM=30704M, BAR=32768M
RAM width 256bits GDDR6
30704M of VRAM memory ready
```

Interpretation:

- The physical card is the expected 32GB V620.
- Approximately **30.0 GiB / 30,704 MiB is exposed as usable VRAM** after firmware / driver reservations.
- The PCIe memory BAR is reported as **32,768 MiB (32GB)**.

The full 32GB BAR allocation is strong evidence that the large-BAR / Resizable BAR configuration is functioning as intended for this card.

The difference between the advertised 32GB and the ~30,704 MiB usable value is not being treated as missing memory; part of the physical VRAM address space is reserved for GPU firmware and driver operation.

## Additional Driver Initialization Results

The kernel successfully initialized major GPU blocks including graphics, SDMA, video encode/decode, display, SMU, and the GPU topology node.

The driver also reported:

- GDDR6 memory
- 256-bit memory bus
- GART initialization
- GPU topology registration
- SMU initialization successful
- DRM device registration

A message stating that the driver could not find a CRTC / display size was observed near the end of initialization. This is expected to be non-critical for the current use case because the V620 is being used as a **headless compute accelerator** while the RTX 3050 provides the actual display output.

An early VBIOS / ROM alignment warning was also observed. Because the driver subsequently fetched the VBIOS, initialized the GPU, exposed the expected VRAM/BAR sizes, and completed DRM registration, it is being documented for awareness rather than treated as a failure at this checkpoint.

## Current Result

| Check | Result |
|---|---|
| System POSTs with one V620 | PASS |
| Ubuntu boots with one V620 | PASS |
| RTX 3050 display path remains functional | PASS |
| V620 visible to Linux | **PASS** |
| V620 identified as Navi 21 / Radeon Pro V620 | **PASS** |
| `amdgpu` driver bound | **PASS** |
| Usable VRAM initialized | **PASS — ~30,704 MiB** |
| 32GB PCIe BAR allocated | **PASS — 32,768 MiB** |
| Resizable BAR / large BAR behavior | **Strong initial PASS; final validation pending PCIe inspection** |
| PCIe Gen3 link validated | Pending |
| V620 temperature visible | Pending |
| Sustained load test | Pending |

## Next Validation Steps

Before installing ROCm or AI software:

1. Inspect PCIe link capability, negotiated generation, and link width.
2. Inspect the Resizable BAR capability directly with verbose PCIe information.
3. Establish idle GPU temperature monitoring.
4. Check the kernel log specifically for fatal `amdgpu` / PCIe errors.
5. Install Vulkan tooling only after hardware-level validation is complete.
6. Run a simple single-GPU compute test before introducing the second V620.

## Security / Documentation Note

The raw terminal photographs used for this validation are not being published directly because the shell prompt contains local username / hostname information. The public repository records only the technical fields necessary to demonstrate successful hardware initialization.

## Engineering Takeaway

The first V620 has progressed beyond simple POST detection: Ubuntu recognizes it by model, the kernel binds the correct AMD driver, the expected GDDR6 memory is initialized, and the system allocates a full 32GB PCIe BAR.

This is an important portfolio checkpoint because it demonstrates staged PCIe accelerator integration, Linux driver validation, firmware log interpretation, and large-memory GPU resource verification before any AI framework is installed.
