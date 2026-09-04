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

## Current Status

The lower power-cap range is now available on both V620s, but the active cap remains at the stock 250 W value.

The next validation step is to apply a **temporary 170 W cap to both V620s**, verify that both cards accept and report the new value, and then perform a controlled inference workload while monitoring temperature, power, clocks, and performance.

The 170 W value will not be made persistent until runtime behavior is validated.