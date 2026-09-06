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

## Post-Cooling Single-GPU Validation

After the replacement V620 cooling fans were installed and thermally validated under the known-good Vulkan workload, ROCm testing resumed conservatively with one V620 exposed to the container at a time.

### V620 #1

- the container exposed `/dev/kfd` and only the first V620 render node
- `rocminfo` reported a single `gfx1030` AMD Radeon Pro V620 GPU agent
- HIP reported exactly one visible GPU
- a small HIP compute workload completed 20 out of 20 synchronized iterations successfully
- no AMDGPU, KFD, timeout, reset, PCIe, or AER faults were recorded in the kernel log after the test

### V620 #2

The same isolated test was repeated with only the second V620 render node exposed to the container.

- `rocminfo` again reported a single `gfx1030` AMD Radeon Pro V620 GPU agent
- HIP reported exactly one visible GPU
- the same HIP compute workload completed 20 out of 20 synchronized iterations successfully
- no AMDGPU, KFD, timeout, reset, PCIe, or AER faults were recorded after the test

The persistent one-second telemetry logger captured only brief power changes because both smoke workloads completed faster than the logging interval. These runs therefore validate device isolation and short single-GPU HIP execution on each card independently, but they are not sustained thermal or stability tests.

The only lines returned by the broad post-test kernel grep were normal Docker virtual-Ethernet interface teardown messages. They matched the broad expression because `unregistering` contains the substring `ring`; they were not GPU ring faults.

## Significance

Both Radeon Pro V620 GPUs are now independently validated for isolated ROCm 7.14 / HIP execution after the cooling upgrade.

This narrows the remaining stability question to multi-device ROCm behavior rather than basic HIP functionality on either individual GPU.

Sustained or concurrent ROCm operation is **not yet considered fully stable** on this platform.

The failed warm restart after the earlier lockup also suggests that a GPU or PCIe device may not have returned to a clean reset state. This is treated as an observation, not a confirmed root cause.

The known-good Vulkan inference path remains the production/control baseline.

## Next Step

Continue the staged ROCm validation sequence:

1. perform a carefully controlled short dual-GPU asynchronous/concurrent scheduling test with both V620s exposed and persistent telemetry running
2. verify both GPUs complete and kernel logs remain clean
3. if stable, move to a modest-duration dual-GPU HIP workload before introducing higher-level frameworks
4. defer RCCL, PyTorch multi-GPU, P2P experimentation, and vLLM tensor parallelism until the dual-device HIP stage is stable

No host-side ROCm installation, ECC change, P2P workaround, or PCIe tuning will be performed until stability is better understood.