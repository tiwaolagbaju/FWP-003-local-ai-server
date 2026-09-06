# Phase 25 — ROCm 7.14 Container Validation

## Goal

Validate ROCm userspace against the existing host AMDGPU driver without installing ROCm packages directly on the host.

## Container Isolation

The ROCm container was given access only to:

- `/dev/kfd`
- the render node for V620 #1
- the render node for V620 #2

The RTX 3050 render node was intentionally excluded from the container.

Numeric supplemental group IDs were used because the container image did not contain a named `render` group matching the host. The resulting warning about the numeric group lacking a name inside the container is cosmetic and does not prevent device access.

## Results

ROCm 7.14 successfully initialized both Radeon Pro V620 GPUs.

`rocminfo` reported:

- GPU agent 1: `gfx1030` — AMD Radeon Pro V620
- GPU agent 2: `gfx1030` — AMD Radeon Pro V620

HIP was also present and reported the expected ROCm 7.14 toolchain version.

Inside the container, the visible GPU device nodes were limited to the two V620 render nodes plus `/dev/kfd`; the RTX 3050 was not exposed.

A small HIP kernel was then compiled for `gfx1030` and executed successfully on both V620s. The runtime reported two HIP devices, each identifying as AMD Radeon Pro V620 with roughly 30 GiB of usable device memory, and both completed the compute test successfully.

The development image required `/opt/rocm/lib` to be added to the container shell's library search path before the test binary could locate `libamdhip64.so.7`. This was handled only inside the disposable container environment and did not modify the host.

## Stability Incident

During the next stage of ROCm validation, the workstation experienced a complete host lockup. Remote access was lost and the local keyboard/display were also unresponsive, requiring a manual shutdown.

The first restart did not complete normally and stalled during the firmware/POST stage while the passive GPUs were warming. A second full shutdown and restart successfully restored the machine.

The preserved journal from the failed operating-system session ends abruptly without a normal shutdown sequence or a recorded AMDGPU reset, AER, watchdog, thermal, or kernel-panic event. Because the system stopped responding completely, the absence of a final kernel error in the journal does not rule out a GPU/KFD/driver-level hang; the system may have stopped before the relevant diagnostic data could be flushed to disk.

After recovery, the validated host state returned normally:

- kernel remained `7.0.0-31-generic`
- both V620s were detected by Vulkan
- the RTX 3050 remained available as the display GPU
- both V620 power caps returned to 170 W
- the dynamic GPU fan-control service returned active

## Significance

Basic ROCm 7.14 discovery and single-kernel HIP execution are validated on both V620s, but sustained or concurrent ROCm operation is **not yet considered stable** on this platform.

The failed warm restart also suggests that a GPU or PCIe device may not have returned to a clean reset state after the lockup. This is treated as an observation, not a confirmed root cause.

The known-good Vulkan inference path remains the production/control baseline.

## Next Step

Pause concurrent ROCm, RCCL, PyTorch multi-GPU, and vLLM testing until the upgraded V620 cooling fans are installed and validated.

Before resuming ROCm work:

1. verify the new cooling hardware and airflow direction
2. validate fan RPM and fail-safe behavior
3. confirm both V620s remain at the 170 W power-cap baseline
4. establish a cold-boot recovery procedure for GPU hangs
5. resume with one-GPU ROCm tests while logging temperatures, power, and kernel events locally
6. only then attempt controlled dual-GPU concurrency

No host-side ROCm installation, ECC change, P2P workaround, or PCIe tuning will be performed until stability is better understood.