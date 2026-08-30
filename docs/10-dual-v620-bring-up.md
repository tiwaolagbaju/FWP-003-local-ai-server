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

While the workstation remained on the POST warning screen, it began emitting a loud audible beep/alarm. The system was shut down immediately rather than continuing past the warning.

## Interpretation

The 517 message is a platform memory-cooling requirement and is not being attributed to the second V620 at this stage.

Because the system had already demonstrated the same 517 warning with greater-than-32GB memory configurations before dual-GPU testing, the second-GPU bring-up is considered **halted by an unresolved memory-cooling prerequisite**, not failed.

The audible alert is documented as part of the POST event. No assumption is being made that it represents a GPU fault without a distinct HP diagnostic code or repeatable GPU-specific symptom.

## Corrective Action

Do not continue dual-GPU initialization with the 64GB configuration until the required HP Z6 G4 memory fan / cooling assembly is installed and detected.

No sustained workload testing will be performed while this condition is present.

## Current Status

| Check | Result |
|---|---|
| RTX 3050 moved to Slot 1 for display role | Installed |
| V620 #1 in Slot 2 | Installed |
| V620 #2 in Slot 5 | Installed |
| System powers on | PASS |
| POST reaches firmware checks | PASS |
| 64GB memory detected without required memory fan | **BLOCKED — POST 517** |
| Audible POST alert observed | Documented |
| Dual-V620 Linux enumeration | Not tested |
| Second V620 amdgpu initialization | Not tested |
| Second V620 BAR / PCIe validation | Not tested |
| Dual-V620 Vulkan enumeration | Not tested |
| Dual-V620 llama.cpp enumeration | Not tested |

## Next Step

Resolve POST 517 with the proper HP Z6 G4 memory-cooling hardware before continuing the second-GPU validation path.

After the memory fan is installed and POST completes cleanly, resume with Linux enumeration, amdgpu binding, BAR allocation, PCIe link checks, thermal telemetry, Vulkan enumeration, and finally llama.cpp device enumeration.

## Engineering Takeaway

The second accelerator was not treated as the cause of an unrelated platform prerequisite. The bring-up was stopped at the firmware warning, power was removed, and the unresolved cooling requirement was isolated before any additional variables or sustained load were introduced.