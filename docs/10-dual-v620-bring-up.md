# Phase 7 — Second V620 Bring-Up

## Goal

Integrate the second AMD Radeon Pro V620 in Slot 5 while preserving a controlled, reversible validation process.

Planned final compute layout remains under review after PCIe fault isolation.

## First Dual-GPU Boot Attempt

The workstation completed initial power-on but stopped at:

```text
POST Error 517 — Memory configuration requires a Memory fan and this fan is not detected
```

At this point the system had 64GB of ECC Registered memory installed.

While the workstation remained on the POST warning screen, it began emitting a loud audible beep/alarm. Both passive V620 heatsinks were observed to be hot, and the system was shut down immediately rather than continuing past the warning.

## Subsequent Reboot — 928 Fatal PCIe Error

On a later reboot, firmware reported:

```text
928 — Fatal PCIe Error
PCIe Surprise Link Down error detected on Slot 0
B:20 D:0 F:0
```

A PCIe Surprise Link Down event means firmware detected that an expected PCIe link disappeared unexpectedly.

## Isolation Sequence

The configuration was progressively reduced to restore the previous known-good baseline.

### Second SSD removed

The newly installed second M.2 SSD was removed while keeping the RTX 3050 in Slot 1 and V620 #1 in Slot 2.

Result:

- **928 error remained**
- second SSD was therefore not the primary cause of the recurring fault

### RTX 3050 returned to Slot 4

The RTX 3050 was then moved from Slot 1 back to its previously proven Slot 4 position while all other isolation variables remained unchanged.

Current configuration:

- 32GB of the original temporary/test ECC Registered RAM
- RTX 3050 in **Slot 4**
- Radeon Pro V620 #1 in **Slot 2**
- Radeon Pro V620 #2 **removed**
- second M.2 SSD **removed**

Result:

- **System booted successfully**
- previous single-V620 baseline restored

## Linux PCIe Enumeration After Recovery

The restored baseline successfully enumerated both graphics devices and the V620's internal PCIe switch hierarchy:

```text
15:00.0 VGA compatible controller: NVIDIA GA107 [GeForce RTX 3050 6GB]
15:00.1 Audio device: NVIDIA GA107 High Definition Audio Controller
21:00.0 PCI bridge: AMD Navi 10 XL Upstream Port of PCI Express Switch
22:00.0 PCI bridge: AMD Navi 10 XL Downstream Port of PCI Express Switch
23:00.0 Display controller: AMD Navi 21 [Radeon Pro V620]
```

The recovery boot's kernel log provides the mapping for the earlier 928 address:

```text
20:00.0 Intel PCIe Root Port
   -> 21:00.0 AMD V620 upstream switch
      -> 22:00.0 AMD V620 downstream switch
         -> 23:00.0 Radeon Pro V620
```

The kernel reports that the V620 host-facing path is limited by the workstation to PCIe Gen3 x16, which is the expected platform behavior.

Therefore the firmware-reported `B:20 D:0 F:0` address maps to the **host-side Intel PCIe root port feeding V620 #1 in physical Slot 2**.

## Recovery Boot Driver / BAR Validation

On the current successful baseline boot, V620 #1 initializes correctly with `amdgpu`:

- V620 endpoint: `23:00.0`
- `amdgpu` kernel initialization completes successfully
- MEM ECC active
- RAS initialized
- VRAM: **30704M**
- BAR: **32768M**
- GDDR6 / 256-bit memory interface
- SMU initialized successfully
- DRM initialized successfully

The same expected non-critical messages remain present:

- ROM / VBIOS alignment warning followed by successful VBIOS fetch
- `Cannot find any crtc or sizes` because the V620 is not being used as the display GPU

## Idle Thermal Validation After Recovery

`sensors` on the restored single-V620 baseline reports:

- V620 edge: **38 C**
- V620 junction: **40 C**
- V620 memory: **36 C**
- V620 board power: **~7 W idle**
- V620 power cap reported by sensor: **250 W**
- CPU package: **~33 C**
- OS NVMe composite: **~35 C**
- chipset / PCH: **~50 C**

These are healthy idle values and provide no indication that V620 #1 is currently in a thermal-fault state.

This remains an **idle-only** validation. Sustained V620 compute testing remains deferred until the final directed-airflow hardware is installed.

## Direct PCIe Bridge Validation

Direct inspection of `20:00.0` confirms:

- device: Intel Skylake-E PCI Express Root Port A
- **Physical Slot: 2**
- secondary bus: 21
- subordinate bus: 23
- **32GB prefetchable memory window** allocated behind the bridge
- kernel driver: `pcieport`

Direct inspection of `21:00.0` confirms:

- AMD Navi 10 XL upstream switch port
- downstream buses 22–23
- the same **32GB prefetchable memory window** propagated through the V620 switch hierarchy
- kernel driver: `pcieport`

The initial non-root `lspci -vv` commands showed `Capabilities: <access denied>`. This is a permissions limitation, not a PCIe fault. A root-level query can be used if detailed link capability/status registers are needed.

## Current PCIe Error State

The recovery boot does **not** show a new Linux PCIe AER fatal, Surprise Link Down, or equivalent V620 link failure.

The kernel notes that the HP firmware does not expose OS control of AER on several root complexes, so firmware-level PCIe diagnostics remain important on this workstation.

Several ACPI/WMI firmware errors also appear later in boot. These are logged separately from the V620 PCIe path and are not currently being treated as evidence of a GPU link failure because the V620 subsequently initializes normally.

## Revised Interpretation

Current evidence supports the following:

1. The **928 BDF directly maps to the root port feeding V620 #1 in physical Slot 2**.
2. V620 #1 and its PCIe path are currently functioning normally in the restored baseline.
3. Current V620 idle thermals are healthy.
4. The newly installed second SSD was not the primary cause because removing it did not clear the recurring 928 state.
5. Moving the RTX 3050 back to Slot 4 coincided with recovery, but this does **not** mean the 928 directly originated from the RTX; the reported failing link was the V620 #1 root-port path.
6. The earlier dual-GPU attempt involved inadequate passive-GPU cooling and a prolonged firmware halt, so transient thermal or power instability remains a plausible contributor.
7. A topology/resource interaction created by the RTX-in-Slot-1 / dual-V620 configuration also remains possible.

## Current Status

| Check | Result |
|---|---|
| Original temporary/test RAM at 32GB | **PASS / current** |
| RTX 3050 in Slot 4 | **PASS / enumerated at 15:00.0** |
| V620 #1 in Slot 2 | **PASS / enumerated at 23:00.0** |
| V620 host root port | **20:00.0 / Physical Slot 2 — mapped** |
| V620 onboard PCIe switch | **PASS / 21:00.0 → 22:00.0 → 23:00.0** |
| V620 host-facing link | **PCIe Gen3 x16** |
| V620 usable VRAM | **30704M** |
| V620 BAR / bridge aperture | **32768M / 32GB** |
| V620 idle edge / junction / memory | **38 C / 40 C / 36 C** |
| V620 idle board power | **~7 W** |
| V620 #2 removed | PASS — isolated |
| Second SSD removed | PASS — isolated |
| POST 517 | Not present with current 32GB configuration |
| Previous 928 BDF | **20:00.0 = root port feeding V620 #1** |
| New Linux fatal PCIe/AER error on recovery boot | **Not observed** |
| Single-V620 baseline restored | **PASS** |
| Dual-V620 testing | Paused pending cooling / controlled reintroduction |

## Recommended Next Step

Keep the present hardware configuration unchanged and perform several normal cold boots / reboots. Confirm that POST 928 does not recur.

If the baseline remains stable across repeated boots, reintroduce the second SSD as the next single variable and verify a clean boot again. Only after storage is independently validated should V620 #2 be reconsidered.

Dual-V620 sustained testing remains deferred until both passive cards have proper directed airflow.

## Engineering Takeaway

The restored baseline now passes enumeration, driver initialization, BAR allocation, PCIe bridge mapping, and idle thermal checks. The original 928 address is confirmed as the Intel root port for physical Slot 2, but that link is currently stable. This points toward a transient thermal/power/topology event during the earlier dual-GPU configuration rather than a confirmed failed V620.