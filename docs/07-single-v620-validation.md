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

## PCIe Endpoint Link Observation

Verbose PCIe inspection of the **V620 GPU endpoint** at `23:00.0` reported:

```text
LnkCap: Speed 16GT/s, Width x16
LnkSta: Speed 16GT/s, Width x16
```

This corresponds to a PCIe 4.0 x16-capable link and shows that the endpoint-to-switch connection is operating at its full advertised width and speed.

However, the V620 contains an onboard PCIe switch. Because the Z6 G4 host slot itself is a PCIe Gen3 platform, the `23:00.0` reading should **not** be interpreted as proof that the workstation slot is operating at PCIe Gen4.

The host-facing link still needs to be checked on the switch's **upstream port** (`21:00.0`). The expected host-side result is Gen3 / 8GT/s at x16 if Slot 2 is negotiating correctly with the Z6.

This distinction is intentionally documented so the final project does not overstate the platform's PCIe capability.

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
| Resizable BAR / large BAR behavior | **Strong initial PASS** |
| V620 temperature telemetry visible | **PASS** |
| Idle thermals | **PASS — edge 43°C / junction 45°C / memory 42°C** |
| Idle GPU power telemetry | **PASS — ~7 W** |
| V620 endpoint link | **PASS — 16GT/s x16** |
| Z6 host-facing PCIe Gen3 link | Pending upstream-port check |
| Sustained load thermal test | Pending |

## Next Validation Steps

Before installing ROCm or a full AI stack:

1. Inspect the V620 switch upstream port at `21:00.0` to verify the **host-facing** link is Gen3 x16.
2. Confirm the 32GB memory region directly in verbose PCIe output.
3. Install Vulkan tooling.
4. Confirm the V620 appears as a Vulkan-capable device.
5. Run a simple single-GPU compute / inference workload while monitoring `sensors`.
6. Record load temperature, junction temperature, memory temperature, and power draw.
7. Only after the single-card cooling and compute tests pass, introduce the second V620.

## Security / Documentation Note

The raw terminal photographs used for this validation are not being published directly because the shell prompt contains local username / hostname information. The public repository records only the technical fields necessary to demonstrate successful hardware initialization.

## Engineering Takeaway

The first V620 has progressed beyond simple POST detection: Ubuntu recognizes it by model, the kernel binds the correct AMD driver, the expected GDDR6 memory is initialized, the system allocates a full 32GB PCIe BAR, hardware telemetry is accessible from Linux, and the GPU endpoint link itself is healthy at x16.

The remaining PCIe question is specifically the Z6-to-card host link, which is being verified independently because the V620's onboard PCIe switch can otherwise make the endpoint speed easy to misinterpret.
