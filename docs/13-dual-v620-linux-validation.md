# Phase 10 — Dual V620 Linux Validation

## Goal

Validate both Radeon Pro V620 accelerators installed simultaneously in the HP Z6 G4 after completing independent single-card A/B testing.

## Test Configuration

- 32GB original temporary/test ECC Registered RAM
- RTX 3050 in Slot 4
- both 2TB NVMe drives installed
- V620 #1 in Slot 2
- V620 #2 in Slot 5
- V620 #1 on the previously validated power path
- V620 #2 powered through a third-party adapter/cable path that has now passed POST and idle boot validation

The second-card adapter is **not yet considered load validated**. A successful boot confirms basic functionality only; sustained current capability and thermal behavior still require controlled testing.

## Boot Result

The workstation booted successfully with both V620s installed simultaneously.

No POST 928 Fatal PCIe / Surprise Link Down condition prevented startup.

## Linux Enumeration

Linux sees both V620 devices and both internal PCIe switch hierarchies:

```text
20:00.0 Intel root port (Slot 2)
   -> 21:00.0 AMD upstream switch
      -> 22:00.0 AMD downstream switch
         -> 23:00.0 Radeon Pro V620

2c:00.0 Intel root port (Slot 5)
   -> 2d:00.0 AMD upstream switch
      -> 2e:00.0 AMD downstream switch
         -> 2f:00.0 Radeon Pro V620
```

Both endpoints are present as `1002:73a1` devices and both bind to the `amdgpu` kernel driver.

## Driver / VRAM State

Both V620s initialize successfully with:

- MEM ECC active
- GECC enabled
- RAS initialized successfully
- usable VRAM: **30704M per card**
- BAR: **32768M per card**
- GDDR6 / 256-bit memory interface
- SMU initialized successfully
- DRM initialized successfully

This yields approximately **61.4GB total usable VRAM across two independent devices** after GECC overhead. The two cards remain separate memory devices rather than a single monolithic VRAM pool.

## PCIe / BAR Validation

### V620 #1 — `23:00.0`

- Region 0: **32GB 64-bit prefetchable BAR**
- endpoint link: **16 GT/s x16** through the V620 internal switch
- host-facing root path: **8 GT/s x16 (PCIe Gen3 x16)**
- lane error status: **0**

### V620 #2 — `2f:00.0`

- Region 0: **32GB 64-bit prefetchable BAR**
- endpoint link: **16 GT/s x16** through the V620 internal switch
- host-facing root path: **8 GT/s x16 (PCIe Gen3 x16)**
- lane error status: **0**

The Z6 host platform limits each accelerator to PCIe Gen3 x16, while each card's internal endpoint/switch segment runs at PCIe Gen4 x16.

## Idle Thermal Validation

At idle, Linux reports:

### V620 #1

- edge: **38 C**
- junction: **39 C**
- memory: **36 C**
- board power: **~7 W**

### V620 #2

- edge: **42 C**
- junction: **44 C**
- memory: **40 C**
- board power: **~7 W**

Both cards are currently well below their reported critical thresholds. V620 #2 is a few degrees warmer, which is expected to remain under observation as airflow is finalized.

## Error Review

The dual-card boot does **not** show:

- a new POST 928 condition
- a Linux Surprise Link Down event
- an active fatal PCIe error
- an active uncorrectable AER status bit on either V620 endpoint
- lane errors on either V620 endpoint

Both endpoints continue to show the same persistent correctable / unsupported-request status indicators observed during the single-card tests, including `AdvNonFatalErr`. The AER uncorrectable status registers are clear, header logs are zero, and lane error status is zero, so these are being tracked as non-fatal latched status rather than evidence of an active failed link.

The same HP ACPI/WMI firmware errors seen in prior known-good boots remain present and are not currently correlated with V620 failure.

## RTX 3050 / Storage Topology

With both native M.2 drives installed, the RTX 3050 in Slot 4 negotiates at **PCIe Gen3 x4** due to the Z6 G4 lane-sharing behavior. This remains acceptable for its display-only role.

## Vulkan Visibility

Initial Vulkan enumeration over SSH reported permission errors opening the two AMD render nodes. The render devices were owned by the Linux `render` group, while the login user was not yet a member of that group.

After adding the user to the `render` and `video` groups and reconnecting the session, Vulkan enumeration succeeded.

`vulkaninfo --summary` now sees:

- **GPU0 — NVIDIA GeForce RTX 3050** using the NVIDIA proprietary Vulkan driver
- **GPU1 — AMD Radeon Pro V620 (RADV NAVI21)**
- **GPU2 — AMD Radeon Pro V620 (RADV NAVI21)**
- **GPU3 — llvmpipe CPU software renderer**

Both V620s use Mesa RADV and report the expected AMD `0x73a1` device ID. Their Vulkan device UUIDs also correspond to the two distinct PCIe endpoints (`23:00.0` and `2f:00.0`).

The headless SSH warning about `DISPLAY` not being set is expected and does not prevent compute-device enumeration.

This confirms that the dual-V620 Vulkan compute stack is operational at the device-discovery level.

## Result

| Check | Result |
|---|---|
| V620 #1 enumerates | **PASS** |
| V620 #2 enumerates | **PASS** |
| Both bind to `amdgpu` | **PASS** |
| GECC enabled on both | **PASS** |
| Usable VRAM per card | **30704M** |
| 32GB BAR per card | **PASS** |
| Slot 2 host link | **PCIe Gen3 x16** |
| Slot 5 host link | **PCIe Gen3 x16** |
| V620 #1 idle thermals | **PASS** |
| V620 #2 idle thermals | **PASS** |
| POST 928 / Surprise Link Down | **Not reproduced** |
| Dual-V620 Linux enumeration | **PASS** |
| Dual-V620 Vulkan visibility | **PASS** |
| Second-card power adapter | **POST/idle validated only** |
| llama.cpp dual-device visibility | Pending |
| Sustained dual-GPU load | Pending |

## Interpretation

This is the first successful dual-V620 Linux and Vulkan validation in the project.

The earlier fault is no longer reproducible under the current configuration. Both accelerators are functional, both PCIe x16 host paths train correctly, both 32GB BARs are allocated simultaneously, both cards initialize with GECC and `amdgpu`, and both are visible to the Vulkan userspace stack through Mesa RADV.

The remaining validation work is now focused on:

- final directed airflow for both passive accelerators
- controlled load testing of the second-card power adapter/path
- llama.cpp visibility with both V620s
- short-duration single-GPU and dual-GPU inference tests
- sustained thermal / power monitoring

## Next Step

Verify that the Vulkan-enabled llama.cpp build sees both Radeon Pro V620 devices.

Do **not** jump directly to a long model run. After llama.cpp device discovery passes, perform a short, low-risk workload while continuously monitoring both cards. The second-card power adapter should remain classified as provisional until it passes controlled current and thermal validation under load.

## Engineering Takeaway

The system now successfully boots with two independently validated Radeon Pro V620 accelerators installed at the same time. Both expose full 32GB BAR apertures, settle at approximately 30.7GB usable VRAM with GECC enabled, operate over separate PCIe Gen3 x16 host links, and enumerate successfully through the Vulkan RADV stack. This marks the transition from hardware bring-up into application-level dual-GPU compute validation.