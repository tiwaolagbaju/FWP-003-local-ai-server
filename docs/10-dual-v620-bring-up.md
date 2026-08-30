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

At this stage the exact device associated with `B:20 D:0 F:0` has not yet been mapped under a known-good Linux boot, so the error is not being assigned to a specific V620 or to the RTX 3050 without additional evidence.

## Current Interpretation

Likely classes of causes include:

- a PCIe device becoming unstable due to inadequate cooling
- device auxiliary power loss / transient
- an improperly seated card or power connector
- PCIe link / signal-integrity instability
- a card fault

Because both V620s are passive server accelerators and the final directed-airflow shrouds have not yet been installed, thermal instability is considered a significant possibility. The cards may remain powered while the system is halted in firmware even though the normal Linux `amdgpu` runtime / power-management stack has not loaded.

The system also now contains multiple high-power PCIe devices, so the second-card auxiliary power path remains a separate item that must be verified before dual-GPU testing resumes.

## Corrective Action / Isolation Plan

Dual-V620 testing is paused.

Before another dual-GPU boot:

1. Allow the workstation and both V620s to cool completely.
2. Resolve the Z6 G4 memory-fan requirement or temporarily restore the known-good 32GB memory baseline so POST is not intentionally held at error 517 during PCIe troubleshooting.
3. Return to the previously proven single-V620 hardware baseline.
4. Verify that the known-good baseline boots without a recurring 928 event.
5. Verify the intended auxiliary power cable/path for V620 #2 before reconnecting it.
6. Do not perform sustained dual-GPU testing until directed cooling is installed for both passive V620s.

If the known-good single-V620 baseline is stable, introduce the second V620 again as a single variable and re-run enumeration/PCIe checks before any AI workload.

## Current Status

| Check | Result |
|---|---|
| RTX 3050 moved to Slot 1 for display role | Installed during dual-GPU attempt |
| V620 #1 in Slot 2 | Installed during dual-GPU attempt |
| V620 #2 in Slot 5 | Installed during dual-GPU attempt |
| Initial system power-on | PASS |
| POST Error 517 with 64GB / no memory fan | **BLOCKED** |
| Passive V620s observed hot at firmware halt | **Observed** |
| Audible alarm while halted at POST | **Observed** |
| Subsequent 928 PCIe Surprise Link Down | **FAIL / requires isolation** |
| Error BDF | **B:20 D:0 F:0** |
| Physical device mapped to BDF | Pending |
| Dual-V620 Linux enumeration | Not validated |
| Second V620 amdgpu initialization | Not validated |
| Second V620 BAR / PCIe validation | Not validated |
| Dual-V620 Vulkan enumeration | Not validated |
| Dual-V620 llama.cpp enumeration | Not validated |

## Engineering Takeaway

The dual-GPU bring-up has exposed two distinct platform prerequisites: required memory cooling for the >32GB configuration and an unexpected PCIe link-down event. The correct troubleshooting response is to return to the known-good single-GPU baseline and reintroduce variables one at a time rather than continuing to boot a thermally and electrically unvalidated dual-passive-GPU configuration.