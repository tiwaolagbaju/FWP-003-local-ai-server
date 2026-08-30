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

A PCIe Surprise Link Down event means firmware detected that an expected PCIe link disappeared unexpectedly. The literal `Slot 0` text is not assumed to correspond directly to a physical expansion-slot number; the bus/device/function address is the more useful diagnostic identifier.

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

This confirms:

- RTX 3050 is enumerating normally at `15:00.0`
- V620 #1 is enumerating normally at `23:00.0`
- the V620's onboard PCIe switch chain is present at `21:00.0` and `22:00.0`

The earlier firmware error referenced `B:20 D:0 F:0`. Because the filtered enumeration starts the visible V620 switch hierarchy at bus 21, the device at `20:00.0` should be mapped directly under the working baseline before attributing the previous Surprise Link Down event to a specific card or slot.

Recommended mapping commands:

```bash
lspci -s 20:00.0 -nnk
lspci -t
```

## Interpretation

This is strong evidence that the RTX 3050 relocation to Slot 1, or the PCIe topology created by that configuration, contributed to the 928 event.

It does **not yet prove** Slot 1 is defective. Possible explanations include:

- RTX 3050 / Slot 1 compatibility or link-training behavior
- physical seating or mechanical fit in Slot 1
- a topology/resource interaction introduced by the three-card layout
- a transient error generated during the earlier hot dual-V620 attempt

Because the system returned to stable operation when the display GPU was restored to its previous Slot 4 location, Slot 4 is retained as the current known-good display-GPU position.

## Current Status

| Check | Result |
|---|---|
| Original temporary/test RAM at 32GB | **PASS / current** |
| RTX 3050 in Slot 4 | **PASS / enumerated at 15:00.0** |
| V620 #1 in Slot 2 | **PASS / enumerated at 23:00.0** |
| V620 onboard PCIe switch | **PASS / 21:00.0 → 22:00.0 → 23:00.0 chain visible** |
| V620 #2 removed | PASS — isolated |
| Second SSD removed | PASS — isolated |
| POST 517 | Not present with current 32GB configuration |
| 928 PCIe Surprise Link Down | **Not preventing current successful boot** |
| Previous error BDF | `20:00.0` |
| Current device at 20:00.0 mapped | Pending direct `lspci` query |
| RTX 3050 / Slot 1 configuration | Primary suspect from current evidence |
| Single-V620 baseline restored | **PASS** |
| Dual-V620 testing | Paused pending fault mapping / cooling readiness |

## Recommended Next Step

Before reintroducing any hardware, map `20:00.0` under the known-good baseline and inspect the PCIe tree. Then confirm `amdgpu` binding, temperatures, and absence of new PCIe/AER errors.

Once the baseline is fully characterized, reintroduce one component at a time. The final placement of the display-only RTX 3050 should be reconsidered if Slot 1 repeatedly reproduces the 928 error.

## Engineering Takeaway

Returning the RTX 3050 to its previously proven Slot 4 position restored a successful boot, and Linux now confirms normal enumeration of the RTX 3050 and the complete V620 switch/GPU hierarchy. Mapping the firmware-reported `20:00.0` address against this working topology is the next high-value diagnostic step.