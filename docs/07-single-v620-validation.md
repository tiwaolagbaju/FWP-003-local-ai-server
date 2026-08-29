# Phase 4 — Single V620 Integration

## Milestone

The HP Z6 G4 successfully completed POST and booted Ubuntu 26.04.1 LTS with the first AMD Radeon Pro V620 installed in the planned Slot 2 position.

Linux then successfully enumerated the V620, bound it to the in-kernel `amdgpu` driver, initialized the GPU, exposed a full 32GB PCIe BAR for the card, and provided working thermal / power telemetry through `lm-sensors`.

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

Verbose PCIe output independently confirmed the large memory window:

```text
Region 0: Memory ... (64-bit, prefetchable) [size=32G]
Region 2: Memory ... (64-bit, prefetchable) [size=2M]
Region 5: Memory ... (32-bit, non-prefetchable) [size=1M]
```

Interpretation:

- The physical card is the expected 32GB V620.
- Approximately **30,704 MiB is exposed as usable VRAM** after firmware / driver reservations.
- The primary GPU PCIe memory region is a full **32GB prefetchable BAR**.

Because Slot 2 Resizable BAR was enabled in BIOS and Linux is receiving a full 32GB BAR instead of a legacy small aperture, the large-BAR / Resizable-BAR configuration is considered **validated for the first V620**.

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

## Thermal and Power Telemetry

`lm-sensors` successfully exposed the first V620 as an `amdgpu` hwmon device.

Idle readings observed during validation:

| Sensor | Idle Reading |
|---|---:|
| GPU edge | **43°C** |
| GPU junction / hotspot | **45°C** |
| Memory | **42°C** |
| GPU power (PPT) | **~7 W** |
| Reported power cap | **250 W** |
| Memory clock | **~96 MHz** |

These are healthy idle temperatures for the modified passive V620 cooling setup. The small difference between edge and junction temperature also indicates that the card is not showing an obvious localized hotspot at idle.

These readings validate only the **idle state**. The fan shroud and airflow design will not be considered fully validated until temperatures are monitored under sustained GPU compute load.

## PCIe Link Validation

The V620 GPU endpoint at `23:00.0` reported:

```text
LnkCap: Speed 16GT/s, Width x16
LnkSta: Speed 16GT/s, Width x16
```

This is the internal connection between the V620's onboard PCIe switch and the Navi 21 GPU endpoint.

The switch's **upstream / host-facing port** at `21:00.0` reported:

```text
LnkCap: Speed 16GT/s, Width x16
LnkSta: Speed 8GT/s (downgraded), Width x16
```

This confirms the actual workstation-to-card connection is:

- **PCIe Gen3** — 8 GT/s
- **x16 link width**

The word `downgraded` is expected here. The V620's onboard switch supports PCIe Gen4, while the HP Z6 G4 host platform provides PCIe Gen3. The link therefore negotiates down to the fastest common generation while retaining the full x16 width.

This is the exact target for Slot 2 and confirms that the manually configured Gen3 limit is functioning correctly.

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
| 32GB PCIe BAR allocated | **PASS — 32GB** |
| Resizable BAR / large BAR behavior | **PASS** |
| V620 temperature telemetry visible | **PASS** |
| Idle thermals | **PASS — edge 43°C / junction 45°C / memory 42°C** |
| Idle GPU power telemetry | **PASS — ~7 W** |
| V620 endpoint link | **PASS — 16GT/s x16** |
| Z6 host-facing PCIe link | **PASS — 8GT/s / Gen3 x16** |
| Sustained load thermal test | Pending |

## Next Validation Steps

The hardware-level checks for one V620 are now substantially complete. Next:

1. Install Vulkan tooling.
2. Confirm the V620 appears as a Vulkan-capable device.
3. Run a simple single-GPU Vulkan / compute workload.
4. Monitor `sensors` during the workload.
5. Record load edge, junction, memory temperature, and power draw.
6. If the single-card cooling test passes, introduce the second V620 and repeat enumeration, BAR, link, and thermal checks.

## Security / Documentation Note

The raw terminal photographs used for this validation are not being published directly because the shell prompt contains local username / hostname information. The public repository records only the technical fields necessary to demonstrate successful hardware initialization.

## Engineering Takeaway

The first V620 is now validated at the hardware / kernel level: Ubuntu recognizes it, `amdgpu` binds correctly, the expected VRAM is initialized, a full 32GB PCIe BAR is allocated, the Z6 host connection negotiates correctly at PCIe Gen3 x16, and healthy idle telemetry is available.

This creates a strong single-GPU baseline before software acceleration testing and before introducing the second 32GB accelerator.