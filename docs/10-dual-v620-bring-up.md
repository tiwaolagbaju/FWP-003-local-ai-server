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

At this stage the exact device associated with `B:20 D:0 F:0` has not been mapped conclusively, so the error is not being assigned to a specific component without controlled testing.

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
- recurring 928 PCIe Surprise Link Down error did not prevent boot
- previous single-V620 baseline has been restored

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
| RTX 3050 in Slot 4 | **PASS / known-good** |
| V620 #1 in Slot 2 | **PASS / known-good baseline** |
| V620 #2 removed | PASS — isolated |
| Second SSD removed | PASS — isolated |
| POST 517 | Not present with current 32GB configuration |
| 928 PCIe Surprise Link Down | **Cleared sufficiently for successful boot after RTX returned to Slot 4** |
| Second SSD identified as primary cause | No |
| RTX 3050 / Slot 1 configuration | **Primary suspect from current evidence** |
| Single-V620 baseline restored | **PASS** |
| Dual-V620 testing | Paused pending next controlled step |

## Recommended Next Step

Do not immediately rebuild the full three-GPU configuration.

First verify the restored baseline in Linux, including:

- RTX 3050 enumeration
- V620 #1 enumeration and `amdgpu` binding
- V620 temperature / power telemetry
- absence of new PCIe/AER errors

Once the baseline is confirmed stable, reintroduce **one component at a time**. The second M.2 SSD can be re-tested independently before attempting V620 #2 again.

The final placement of the display-only RTX 3050 should be reconsidered if Slot 1 repeatedly reproduces the 928 error.

## Engineering Takeaway

Returning the RTX 3050 to its previously proven Slot 4 position restored a successful boot after the second SSD had already been ruled out. This demonstrates the value of one-variable-at-a-time fault isolation and provides a clean baseline before any further dual-accelerator integration work.