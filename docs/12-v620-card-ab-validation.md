# Phase 9 — V620 Card A/B Validation

## Goal

Validate Radeon Pro V620 #2 independently by replacing V620 #1 in the exact same known-good hardware position and keeping all other system variables unchanged.

## Test Configuration

- 32GB original temporary/test ECC Registered RAM
- RTX 3050 in Slot 4
- both 2TB NVMe drives installed
- V620 #1 removed
- **V620 #2 installed in physical Slot 2**
- same known-good Slot 2 host path and auxiliary power connection used for the test

This provides a direct A/B comparison between the two V620 cards.

## Boot Result

The workstation booted successfully with V620 #2 installed in Slot 2.

No POST 928 Fatal PCIe / Surprise Link Down condition prevented startup.

## Linux Enumeration

V620 #2 enumerated through the same PCIe hierarchy used by V620 #1:

```text
20:00.0 Intel PCIe root port for physical Slot 2
   -> 21:00.0 AMD Navi 10 XL upstream PCIe switch
      -> 22:00.0 AMD Navi 10 XL downstream PCIe switch
         -> 23:00.0 Radeon Pro V620
```

Linux reports:

```text
23:00.0 Display controller: AMD Navi 21 [Radeon Pro V620]
Kernel driver in use: amdgpu
```

RTX 3050 also continues to enumerate normally in Slot 4.

## Initial Idle Thermal Validation

On the first V620 #2 boot, `sensors` reported:

- edge: **37 C**
- junction: **40 C**
- memory: **32 C**
- board power: **~6 W**
- reported sensor power cap: **250 W**

These values were healthy and closely matched the previously validated idle behavior of V620 #1.

## First-Boot GECC State

On the first boot, `amdgpu` reported:

- MEM ECC active
- RAS initialized successfully
- VRAM RAM: **32752M**
- BAR: **32768M**
- GDDR6 / 256-bit memory interface
- SMU initialized successfully
- DRM initialized successfully

The log also stated:

```text
GECC will be enabled in next boot cycle
```

V620 #1 had already completed GECC enablement and exposed approximately **30704M** usable VRAM, so a follow-up reboot was performed to verify whether V620 #2 would converge to the same memory state.

## Second Boot — GECC Confirmation

The second clean boot completed successfully with V620 #2 still alone in Slot 2.

`amdgpu` now reports:

```text
MEM ECC is active
GECC is enabled
VRAM: 30704M
BAR: 32768M
```

This confirms the expected transition:

- V620 #2 first boot before GECC completion: **32752M usable VRAM**
- V620 #2 after GECC enablement: **30704M usable VRAM**
- V620 #1 validated GECC state: **30704M usable VRAM**

The two cards therefore expose the same usable VRAM after GECC is fully enabled.

## Second-Boot Thermal Check

After the follow-up reboot, V620 #2 reported:

- edge: **46 C**
- junction: **49 C**
- memory: **40 C**
- board power: **~7 W**

These values remain comfortably below the card's reported critical thresholds and do not indicate a thermal fault. They are higher than the first capture, so ambient conditions / elapsed idle time / airflow should continue to be monitored. Sustained compute testing remains deferred until the final directed-airflow configuration is installed.

## PCIe / BAR Validation

Direct endpoint inspection confirms:

- Region 0: **32GB 64-bit prefetchable BAR**
- Region 2: 2MB
- Region 5: 1MB
- endpoint capability: PCIe 16 GT/s x16
- endpoint link currently: **16 GT/s x16** through the V620's internal switch
- host-side system path remains PCIe Gen3 x16 through the Z6 G4 root port
- lane error status: **0**

The distinction remains important: the V620's internal endpoint/switch link can operate at PCIe Gen4 x16 while the workstation host-facing Slot 2 link is PCIe Gen3 x16.

## RTX 3050 / Dual-NVMe Lane Observation

With both NVMe drives installed, Linux reports the RTX 3050 host link at:

```text
8.0 GT/s PCIe x4
```

This matches the known Z6 G4 lane-sharing behavior where installing the second native M.2 device reduces Slot 4 from x8 to x4 electrical. Because the RTX 3050 is being retained primarily as a display GPU, this reduced link width is acceptable for the current architecture and avoids moving it back to the problematic Slot 1 test configuration.

## Error Review

No new Linux fatal PCIe error or Surprise Link Down event was observed on either V620 #2 boot.

The V620 endpoint's PCIe status continues to show latched correctable / unsupported-request indicators and an `AdvNonFatalErr` correctable status bit. However:

- the AER uncorrectable status register shows no active uncorrectable error bits
- the AER header log is zeroed
- lane error status is **0**
- no fatal link event occurred during boot

These bits are therefore being tracked as persistent non-fatal status rather than evidence of an active failed link.

The same HP ACPI/WMI firmware errors seen in previous known-good boots remain present and are not currently correlated with V620 failure.

## A/B Comparison

| Check | V620 #1 | V620 #2 |
|---|---:|---:|
| Boots in Slot 2 | PASS | **PASS** |
| `amdgpu` binding | PASS | **PASS** |
| 32GB BAR | PASS | **PASS** |
| GECC enabled | PASS | **PASS** |
| Usable VRAM after GECC | **30704M** | **30704M** |
| Idle edge temperature | ~38 C | **37–46 C** |
| Idle junction temperature | ~40 C | **40–49 C** |
| Idle memory temperature | ~36 C | **32–40 C** |
| Idle board power | ~7 W | **~6–7 W** |
| Fatal PCIe / 928 during single-card test | No | **No** |

## Current Interpretation

Both V620 cards now pass the same single-card Slot 2 baseline and converge to the same GECC-enabled usable VRAM figure.

This is strong evidence that:

- V620 #1 is functional
- V620 #2 is functional
- Slot 2 is functional
- the known-good Slot 2 power path is functional
- the Linux `amdgpu` driver stack works with either card
- both NVMe drives can remain installed
- the RTX 3050 can remain in Slot 4 at x4 for display duty

The unresolved dual-GPU problem is therefore narrowed primarily to:

- simultaneous two-V620 PCIe topology / resource allocation
- auxiliary power path used by the second V620 when both cards are installed
- passive-GPU airflow when both cards are present

## Next Step

Before reinstalling both V620s together, verify the second-card auxiliary power path and install proper directed airflow for both passive accelerators.

When dual-V620 testing resumes, keep the now-proven configuration unchanged wherever possible:

- 32GB test RAM
- RTX 3050 in Slot 4
- both SSDs installed
- V620 #1 in Slot 2
- V620 #2 in Slot 5

Do not move the RTX 3050 back to Slot 1 during the next dual-V620 attempt unless a separate test specifically requires it.

## Engineering Takeaway

The direct card-for-card swap and follow-up GECC reboot established parity between the two accelerators. Both V620s independently boot, enumerate, initialize, expose the full 32GB BAR, and settle at approximately 30.7GB usable VRAM with GECC enabled. The investigation can now move away from individual-card failure and focus on dual-card power, airflow, and PCIe topology.