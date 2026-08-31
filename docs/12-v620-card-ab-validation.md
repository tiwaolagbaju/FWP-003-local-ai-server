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

## Idle Thermal Validation

`sensors` reports the following for V620 #2 at idle:

- edge: **37 C**
- junction: **40 C**
- memory: **32 C**
- board power: **~6 W**
- reported sensor power cap: **250 W**

These values are healthy and closely match the previously validated idle behavior of V620 #1.

## Driver / VRAM Initialization

`amdgpu` initializes successfully and reports:

- MEM ECC active
- RAS initialized successfully
- VRAM RAM: **32752M**
- BAR: **32768M**
- GDDR6 / 256-bit memory interface
- SMU initialized successfully
- DRM initialized successfully

The log additionally reports:

```text
GECC will be enabled in next boot cycle
```

This is an important difference from the earlier V620 #1 validation. V620 #1 had already completed GECC enablement and exposed approximately **30704M** usable VRAM, while V620 #2 currently exposes **32752M** before the requested next boot cycle.

The working hypothesis is that V620 #2 may expose a smaller usable VRAM figure after GECC is fully enabled on the next boot, potentially aligning with V620 #1. This should be verified rather than assumed.

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

## Error Review

No new Linux fatal PCIe error or Surprise Link Down event was observed during the V620 #2 boot.

The V620 endpoint's PCIe status does contain latched correctable / unsupported-request indicators and an `AdvNonFatalErr` correctable status bit. The Advanced Error Reporting uncorrectable status register itself shows no active uncorrectable error bits, the header log is zeroed, and lane error status is zero.

These bits should be re-checked after the next clean reboot before being interpreted as a persistent hardware fault; they may represent boot-time / latched status rather than an active unstable link.

The same HP ACPI/WMI firmware errors seen in previous known-good boots remain present and are not currently correlated with V620 failure.

## A/B Comparison

| Check | V620 #1 | V620 #2 |
|---|---:|---:|
| Boots in Slot 2 | PASS | **PASS** |
| `amdgpu` binding | PASS | **PASS** |
| 32GB BAR | PASS | **PASS** |
| Idle edge temperature | ~38 C | **37 C** |
| Idle junction temperature | ~40 C | **40 C** |
| Idle memory temperature | ~36 C | **32 C** |
| Idle board power | ~7 W | **~6 W** |
| Usable VRAM at captured boot | ~30704M | **32752M** |
| GECC state | enabled | **enable requested for next boot** |
| Fatal PCIe / 928 during test | No | **No** |

## Current Interpretation

V620 #2 successfully passes the same Slot 2 baseline that V620 #1 passes. This is strong evidence that V620 #2 itself is functional and that Slot 2, the known-good power path, Linux driver stack, and current dual-SSD system configuration can operate normally with either card individually.

This substantially narrows the unresolved dual-GPU problem. The remaining high-priority variables are now:

- simultaneous two-V620 PCIe topology / resource allocation
- auxiliary power path for V620 #2 when both cards are installed
- passive-GPU airflow when both cards are present
- RTX 3050 final slot placement in the three-GPU configuration

## Next Step

Perform one clean reboot with V620 #2 still alone in Slot 2, then re-check:

```bash
sensors
sudo dmesg -T | grep -Ei 'pcie|aer|amdgpu|error|fatal|surprise|gecc'
sudo lspci -vv -s 23:00.0
```

Confirm whether GECC completes and whether usable VRAM changes from 32752M.

Do not proceed to sustained compute load or reinstall both V620s until directed cooling and the second-card auxiliary power path are ready for validation.

## Engineering Takeaway

A direct card-for-card swap isolated the accelerator itself as a variable. V620 #2 boots, enumerates, initializes, exposes its full 32GB BAR, and idles normally in the exact same Slot 2 environment previously used by V620 #1. Both accelerators have therefore passed independent single-card bring-up.