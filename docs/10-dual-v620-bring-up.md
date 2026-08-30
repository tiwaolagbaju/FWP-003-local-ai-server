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

The system has now been reduced to:

- 32GB ECC Registered memory
- RTX 3050 in Slot 1, display only
- V620 #1 in Slot 2
- V620 #2 removed
- second M.2 SSD installed in the system's SSD0 position

This removes the previous >32GB memory-cooling warning and the second accelerator from the immediate test configuration.

## New PCIe Variable — Second M.2 SSD

The second SSD is now considered a significant troubleshooting variable. On the Z6 G4, both native M.2 sockets are PCIe devices connected through CPU PCIe resources. A failing, poorly seated, or otherwise unstable NVMe device can therefore generate a PCIe Surprise Link Down event in the same general way as an expansion card.

HP Z6 G4 field reports also document 928 Surprise Link Down events associated with M.2/NVMe changes, so the SSD should be isolated before concluding that the remaining V620 is faulty.

The second M.2 device also changes PCIe resource allocation on the Z6 G4 by reducing Slot 4 from x8 to x4 electrical. The RTX 3050 is currently in Slot 1, so this particular lane-sharing behavior does not directly constrain the current display card, but it confirms that the second M.2 device is an active part of the workstation's PCIe topology.

## Current Interpretation

Possible causes now include:

- the newly installed second M.2 SSD or its socket/seating
- V620 #1 or its Slot 2 link
- V620 auxiliary power instability
- RTX 3050 after relocation to Slot 1
- residual firmware/AER error state following the previous dual-GPU event
- PCIe link / signal-integrity instability
- thermal damage or instability from the earlier inadequate-airflow event

The exact `B:20 D:0 F:0` device should not be inferred solely from the bus number because PCIe bus numbering can change with installed topology.

## Corrective Action / Isolation Plan

Use one-variable-at-a-time testing:

1. Power down completely and disconnect AC.
2. Keep the current 32GB memory configuration.
3. Keep RTX 3050 in Slot 1 and V620 #1 in Slot 2 unchanged.
4. Remove **only the newly added non-boot second SSD** from SSD0.
5. Boot and observe whether 928 recurs.
6. If 928 disappears, investigate the second SSD, its seating, firmware/health, and M.2 socket before reinstalling it.
7. If 928 persists, restore the RTX 3050 to its previously proven display slot as the next single-variable test.
8. If still present, isolate V620 #1 by reseating it and its known-good auxiliary power path, then if necessary test the workstation without the V620.

Do not reintroduce V620 #2 until the current single-V620 baseline is again stable.

## Current Status

| Check | Result |
|---|---|
| Memory restored to 32GB baseline | PASS |
| RTX 3050 in Slot 1 | Current |
| V620 #1 in Slot 2 | Current |
| V620 #2 removed | PASS — isolated |
| Second SSD installed in SSD0 | **New suspect / pending isolation** |
| POST 517 in current 32GB config | Expected cleared |
| Previous 928 PCIe Surprise Link Down | **FAIL / requires isolation** |
| Error BDF | **B:20 D:0 F:0** |
| Physical device mapped to BDF | Pending |
| Single-V620 known-good baseline restored | Pending SSD isolation test |
| Dual-V620 testing | Paused |

## Engineering Takeaway

The troubleshooting process is now focused on reproducing the last known-good single-V620 state one variable at a time. Because native NVMe storage is itself part of the PCIe fabric, the newly installed second SSD must be ruled out before assigning the 928 event to a GPU. This avoids replacing or condemning hardware based only on an ambiguous firmware slot label.