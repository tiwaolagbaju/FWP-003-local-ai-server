# Phase 21 — V620 Power-Cap Unlock

## Goal

Reduce sustained thermal and electrical load on the dual Radeon Pro V620 GPUs by enabling a lower software-selectable PPT limit under Linux.

The stock Linux `amdgpu` interface exposed a fixed 250 W range on both cards:

```text
power1_cap:         250 W
power1_cap_default: 250 W
power1_cap_min:     250 W
power1_cap_max:     250 W
```

Because the minimum and maximum were both fixed at 250 W, a lower runtime cap could not be selected through the normal hwmon interface.

## Method Selected

A V620-specific Ubuntu AMDGPU module modification from the community `v620_toolbox` project was evaluated and selected for this system.

The method:

- targets the Radeon Pro V620 PCI identity only;
- lowers the driver-visible minimum power limit to 120 W;
- leaves the stock 250 W maximum unchanged;
- does not flash or modify the physical GPU VBIOS;
- installs the modified AMDGPU module as an override while leaving Canonical's stock module on disk;
- allows the normal Linux hwmon `power1_cap` interface to be used afterward.

The source revision used during this validation was recorded before installation so the procedure is reproducible.

## Compatibility Validation

Both installed V620s reported the expected identity:

```text
vendor:            0x1002
device:            0x73a1
subsystem_vendor:  0x1002
subsystem_device:  0x0e34
```

The running Ubuntu kernel, matching headers, and kernel source were aligned before the build. Additional bootable kernels were retained as recovery options before modifying the AMDGPU load path.

The source anchors expected by the rebuild script were also inspected directly in the matching Ubuntu source archive before compilation.

## Build and Installation

The V620-specific AMDGPU module compiled successfully against the active Ubuntu kernel. The rebuild process verified that:

- the compiled module contained the V620-specific power-cap modification;
- module vermagic matched the target kernel;
- the patched module resolved from the kernel's `updates/` override path;
- the patched module was included in the target kernel's initramfs;
- Canonical's stock AMDGPU module remained in the normal kernel module tree as a rollback path.

## Reboot Validation

The system was deliberately booted back into the target kernel after installation.

On boot, the kernel log confirmed that the power-cap adjustment was applied independently to both V620s.

The resulting driver-visible power range was:

| GPU | Current | Default | Minimum | Maximum |
|---|---:|---:|---:|---:|
| V620 #1 | 250 W | 250 W | 120 W | 250 W |
| V620 #2 | 250 W | 250 W | 120 W | 250 W |

This confirms the software-side power-cap floor was successfully lowered while retaining the original 250 W maximum.

## Cooling Check

The independent ARCTIC fan controller remained fully operational after the AMDGPU change and reboot.

All four dedicated GPU cooling fans automatically returned to the previously configured ~90% PWM baseline, with observed speeds approximately in the 3,700–4,400 RPM range.

## 170 W Runtime Cap Validation

A temporary 170 W power cap was then applied independently to both V620s through the normal Linux hwmon `power1_cap` interface.

Both cards accepted the requested value and read it back successfully:

| GPU | Active Cap | Minimum | Maximum |
|---|---:|---:|---:|
| V620 #1 | 170 W | 120 W | 250 W |
| V620 #2 | 170 W | 120 W | 250 W |

This confirms that the patched driver not only exposes a lower range, but also permits a practical below-stock runtime cap to be selected on both installed GPUs.

## Controlled 170 W Load Test

A controlled dual-GPU Qwen3-Coder-Next Q4_K_M inference run was performed before considering the cap for persistent use.

The successful test used a 65,536-token context allocation with an approximately 62,000-token synthetic UPS telemetry prompt. An earlier 32,768-token attempt was rejected cleanly because the prompt exceeded the allocated context and was not treated as a hardware or driver failure.

The successful run completed normally with exit code 0 and reported:

- prompt processing: approximately **673.9 tokens/s**;
- generation: approximately **43.2 tokens/s**;
- observed high-load V620 #1: approximately **85 C junction**, **48 C memory**, **161 W PPT**;
- observed high-load V620 #2: approximately **82 C junction**, **50 C memory**, **163 W PPT**.

The two cards remained reasonably balanced, stayed below the configured thermal stop threshold, and completed the workload successfully.

These results are not a strict A/B comparison with the earlier 258K stress test because the prompt length and workload duration differ. However, they demonstrate substantially improved thermal headroom under a meaningful long-prefill workload with the 170 W cap and high fixed fan baseline in place.

## Current Status

The dual V620 configuration has now passed a meaningful controlled load at a temporary **170 W per-GPU cap** with the dedicated cooling fans at their persistent ~90% baseline.

A final post-load kernel/AMDGPU health check is the remaining validation step before deciding whether to make the 170 W cap persistent at boot. The earlier near-maximum 258K stress workload remains a separate capability test and is not required for normal operation.