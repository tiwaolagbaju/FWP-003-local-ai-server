# Phase 7 — Second V620 Bring-Up

## Goal

Integrate the second AMD Radeon Pro V620 in Slot 5 while preserving a controlled, reversible validation process.

Planned layout:

- Slot 1: NVIDIA RTX 3050 (display only)
- Slot 2: Radeon Pro V620 #1
- Slot 5: Radeon Pro V620 #2

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

At this stage the exact device associated with `B:20 D:0 F:0` has not yet been mapped under a known-good Linux boot, so the error is not being assigned to a specific V620, RTX 3050, or M.2 SSD without additional evidence.

## Current Isolation Configuration

The system has been reduced to:

- 32GB ECC Registered memory
- RTX 3050 in Slot 1, display only
- V620 #1 in Slot 2
- V620 #2 removed
- second M.2 SSD removed

Removing the second SSD did **not** clear the 928 PCIe Surprise Link Down error, so the new SSD is no longer the leading suspect.

## New Primary Variable — RTX 3050 Relocation to Slot 1

The RTX 3050 had previously operated successfully in Slot 4 before being relocated to Slot 1 for the dual-V620 layout.

HP documents Slot 1 as PCIe Gen3 x4, CPU-connected, with an open-ended connector that can physically accept a wider PCIe card. This means the RTX 3050 can fit there, but the relocation is still a meaningful topology change and must be isolated experimentally.

A Z6 G4 field report documents a similar `PCIe Link Down` / `Slot 0` symptom after changing a consumer GPU, with slot/card isolation used as the troubleshooting path. Because the SSD has now been ruled out and the RTX relocation is one of the remaining changes from the previous stable state, Slot 1 / RTX 3050 is now a high-priority test variable.

## Corrective Action / Isolation Plan

Use one-variable-at-a-time testing:

1. Power down completely and disconnect AC.
2. Keep the 32GB memory configuration unchanged.
3. Keep V620 #1 in Slot 2 unchanged.
4. Keep V620 #2 removed.
5. Keep the newly added second SSD removed.
6. Move **only the RTX 3050 from Slot 1 back to its previously proven Slot 4 position**.
7. Boot and observe whether 928 recurs.
8. If 928 disappears, investigate the Slot 1 / RTX 3050 combination before using Slot 1 in the final layout.
9. If 928 persists, the next isolation step is V620 #1 / Slot 2 / auxiliary power, including a boot with V620 #1 removed if necessary.

Do not reintroduce the second SSD or V620 #2 until the current baseline is stable.

## Current Status

| Check | Result |
|---|---|
| Memory restored to 32GB baseline | PASS |
| V620 #1 in Slot 2 | Current |
| V620 #2 removed | PASS — isolated |
| Second SSD removed | PASS — **928 still present; SSD not primary cause** |
| RTX 3050 in Slot 1 | **Current high-priority suspect** |
| RTX 3050 previously stable in Slot 4 | Historical known-good state |
| POST 517 in current 32GB config | Cleared / no longer active variable |
| 928 PCIe Surprise Link Down | **Still present** |
| Error BDF | **B:20 D:0 F:0** |
| Physical device mapped to BDF | Pending |
| Next test | Move RTX 3050 Slot 1 → Slot 4 only |
| Dual-V620 testing | Paused |

## Engineering Takeaway

The second M.2 SSD was isolated and the fault persisted, so troubleshooting has advanced to the next topology change: relocating the RTX 3050 from its previously stable Slot 4 position to Slot 1. Returning one device at a time to the known-good configuration provides stronger evidence than interpreting the ambiguous firmware `Slot 0` label.