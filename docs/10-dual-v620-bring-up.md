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

The recovery boot's kernel log provides the missing mapping for the earlier 928 address:

```text
20:00.0 Intel PCIe Root Port
   -> 21:00.0 AMD V620 upstream switch
      -> 22:00.0 AMD V620 downstream switch
         -> 23:00.0 Radeon Pro V620
```

The kernel also reports:

```text
21:00.0: 126.016 Gb/s available PCIe bandwidth,
limited by 8.0 GT/s PCIe x16 link at 20:00.0
```

Therefore the firmware-reported `B:20 D:0 F:0` address maps to the **host-side Intel PCIe root port feeding V620 #1 in Slot 2**.

This materially changes the fault interpretation: the earlier 928 event indicates that the Slot 2 / V620 #1 PCIe path experienced a Surprise Link Down event. The error was not directly reporting the second SSD or RTX 3050 endpoint.

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

## Current PCIe Error State

The recovery boot does **not** show a new Linux PCIe AER fatal, Surprise Link Down, or equivalent V620 link failure.

The kernel notes that the HP firmware does not expose OS control of AER on several root complexes:

```text
platform does not support ... AER ...
```

This means firmware-level PCIe diagnostics remain important on this workstation and the absence of Linux AER reporting should not be treated as proof that a prior firmware-level event did not occur.

Several ACPI/WMI firmware errors also appear later in boot. These are logged separately from the V620 PCIe path and are not currently being treated as evidence of a GPU link failure because the V620 subsequently initializes normally.

## Revised Interpretation

Current evidence supports the following:

1. The **928 BDF directly maps to the root port feeding V620 #1**.
2. V620 #1 and its PCIe path are currently functioning normally in the restored baseline.
3. The newly installed second SSD was not the primary cause because removing it did not clear the recurring 928 state.
4. Moving the RTX 3050 back to Slot 4 coincided with recovery, but this does **not** mean the 928 directly originated from the RTX; the actual failing link reported by firmware was the V620 #1 root-port path.
5. The earlier dual-GPU attempt involved inadequate passive-GPU cooling and a prolonged firmware halt, so transient thermal or power instability remains a plausible contributor.
6. A topology/resource interaction created by the RTX-in-Slot-1 / dual-V620 configuration also remains possible.

## Current Status

| Check | Result |
|---|---|
| Original temporary/test RAM at 32GB | **PASS / current** |
| RTX 3050 in Slot 4 | **PASS / enumerated at 15:00.0** |
| V620 #1 in Slot 2 | **PASS / enumerated at 23:00.0** |
| V620 host root port | **20:00.0 — mapped** |
| V620 onboard PCIe switch | **PASS / 21:00.0 → 22:00.0 → 23:00.0** |
| V620 host-facing link | **PCIe Gen3 x16 / 8.0 GT/s x16** |
| V620 usable VRAM | **30704M** |
| V620 BAR | **32768M** |
| V620 #2 removed | PASS — isolated |
| Second SSD removed | PASS — isolated |
| POST 517 | Not present with current 32GB configuration |
| Previous 928 BDF | **20:00.0 = root port feeding V620 #1** |
| New Linux fatal PCIe/AER error on recovery boot | **Not observed** |
| Single-V620 baseline restored | **PASS** |
| Dual-V620 testing | Paused pending cooling / controlled reintroduction |

## Recommended Next Step

Do not immediately reinstall V620 #2.

First characterize the restored single-V620 baseline for stability without sustained AI load:

```bash
sensors
lspci -vv -s 20:00.0
lspci -vv -s 21:00.0
```

Then perform several normal cold boots / reboots with the same hardware configuration and confirm that POST 928 does not recur.

If the baseline remains stable, reintroduce one component at a time. The second SSD can be re-tested independently before V620 #2. Dual-V620 testing should wait until both passive cards have proper directed airflow.

## Engineering Takeaway

The firmware's ambiguous `Slot 0` label was resolved by mapping its B:D:F address under Linux. `20:00.0` is the Intel root port directly feeding the Slot 2 V620 switch chain. The V620 path currently operates normally after returning the workstation to the known-good baseline, which points toward a transient thermal/power/topology event rather than a confirmed failed GPU.