# Phase 4 — Single V620 Integration

## Milestone

The HP Z6 G4 successfully completed POST and booted Ubuntu 26.04.1 LTS with the first AMD Radeon Pro V620 installed in the planned Slot 2 position.

Linux successfully enumerated the V620, bound it to the in-kernel `amdgpu` driver, initialized the GPU, exposed a full 32GB PCIe BAR, provided working thermal / power telemetry, and exposed the accelerator to Vulkan through Mesa RADV.

This confirms that one V620 can coexist with the NVIDIA RTX 3050 display GPU using the current BIOS and Linux configuration without requiring ROCm for basic Vulkan access.

## Configuration at This Checkpoint

- HP Z6 G4 Workstation
- Intel Xeon Platinum 8168
- Temporary 32GB ECC Registered memory
- WD Blue SN5000 2TB NVMe
- NVIDIA RTX 3050 6GB display GPU
- **AMD Radeon Pro V620 #1 installed in Slot 2**
- V620 #2 not installed

## Temporary Cooling Configuration

The first V620 is **not yet using the planned final cooling assembly**.

Current temporary setup:

- One Noctua 40×20 mm fan
- Fan temporarily secured directly to the V620 with a zip tie
- Ordered dual-fan shroud has not yet arrived
- Planned final configuration remains two 40×20 mm fans per V620 with a dedicated shroud that directs airflow through the passive heatsink

This distinction is important because the V620 is a passive server accelerator designed around directed chassis airflow. A single small fan can provide enough airflow for BIOS, Linux enumeration, Vulkan detection, and idle validation, but it has **not** been demonstrated to provide adequate cooling under sustained compute load.

Accordingly, no prolonged AI inference, stress test, or thermal benchmark will be performed with the temporary single-fan arrangement.

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

No separate ROCm package was required for basic Linux device initialization. The standard Ubuntu kernel driver recognizes and initializes the card.

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

A message stating that the driver could not find a CRTC / display size was observed near the end of initialization. This is expected to be non-critical because the V620 is being used as a **headless compute accelerator** while the RTX 3050 provides display output.

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

These are healthy **idle** temperatures with the temporary one-fan setup. They confirm that basic airflow is present, but they do **not** validate the cooling arrangement for sustained compute.

Because the card exposes a 250W power cap and was designed for directed server airflow, the final shroud / dual-fan arrangement must be installed and tested before thermal performance under AI load is considered safe or complete.

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

The word `downgraded` is expected here. The V620's onboard switch supports PCIe Gen4, while the HP Z6 G4 host platform provides PCIe Gen3. The link therefore negotiates to the fastest common generation while retaining the full x16 width.

## Vulkan Validation

Vulkan tooling was installed using Ubuntu's `vulkan-tools` and Mesa Vulkan packages.

`vulkaninfo --summary` successfully enumerated three Vulkan devices:

| Vulkan Device | Driver Path | Result |
|---|---|---|
| NVIDIA GeForce RTX 3050 | NVIDIA proprietary Vulkan driver | PASS |
| **AMD Radeon Pro V620 (RADV NAVI21)** | **Mesa RADV** | **PASS** |
| llvmpipe | Mesa software / CPU renderer | Present as expected |

The V620 was reported as a **discrete GPU** using the `radv` driver, confirming that the open-source Mesa Vulkan stack can address the accelerator directly.

This is important for the project because **llama.cpp can use Vulkan as the initial AMD inference backend**, allowing useful AI testing before deciding whether ROCm support is necessary or beneficial for this specific card.

`vulkaninfo` also emitted display-plane warnings for a selected physical device. These are not being treated as a compute failure: the V620 is a headless accelerator with no display output in this configuration, while Vulkan still enumerated the device correctly through RADV.

No unique Vulkan device UUIDs or raw terminal screenshots are being published in the public documentation.

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
| Idle thermals | **PASS — temporary single-fan setup only** |
| Idle GPU power telemetry | **PASS — ~7 W** |
| V620 endpoint link | **PASS — 16GT/s x16** |
| Z6 host-facing PCIe link | **PASS — 8GT/s / Gen3 x16** |
| V620 available through Vulkan / RADV | **PASS** |
| Final dual-fan shroud installed | **Pending** |
| Actual sustained AI compute workload | **Deferred pending final cooling** |
| Sustained load thermal test | **Deferred pending final cooling** |

## Next Validation Steps

While waiting for the cooling shroud, software-only checks that do not meaningfully load the GPU can continue. Sustained inference and thermal benchmarking are deferred.

Next hardware steps:

1. Install the dedicated V620 shroud when it arrives.
2. Install two 40×20 mm fans in the intended airflow direction.
3. Verify the shroud seals / directs airflow through the passive heatsink rather than around it.
4. Re-record idle temperatures with the final cooling configuration.
5. Run the first controlled inference workload while continuously monitoring `sensors`.
6. Record edge, junction, memory temperature, and power during load.
7. Only after cooling validation passes, introduce the second V620.

## Security / Documentation Note

The raw terminal photographs used for validation are not being published directly because the shell prompt contains local username / hostname information and Vulkan output contains device-specific identifiers. The public repository records only the technical fields necessary to demonstrate successful hardware initialization.

## Engineering Takeaway

The first V620 is validated from firmware through the Vulkan API layer, but thermal validation is intentionally being kept separate from software validation. The temporary single-fan arrangement is sufficient for low-power bring-up and enumeration; it is not being treated as a production cooling solution.

Deferring sustained load until the engineered dual-fan shroud is installed preserves hardware safety and creates a more defensible benchmark methodology for the final portfolio documentation.