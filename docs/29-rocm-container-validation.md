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

## Stability Incidents

### Initial ROCm Lockup

During the next stage of ROCm validation, the workstation experienced a complete host lockup. Remote access was lost and the local keyboard/display were also unresponsive, requiring a manual shutdown.

The first restart did not complete normally and stalled during the firmware/POST stage while the passive GPUs were warming. A second full shutdown and restart successfully restored the machine.

The preserved journal from the failed operating-system session ended abruptly without a normal shutdown sequence or a recorded AMDGPU reset, AER, watchdog, thermal, or kernel-panic event. Because the system stopped responding completely, the absence of a final kernel error does not rule out a GPU/KFD/driver-level hang.

### Post-Cooling Isolated-GPU Lockup

After replacement V620 cooling fans were installed and validated under the known-good Vulkan workload, ROCm testing resumed with one V620 exposed to the container at a time.

V620 #1 completed 20 out of 20 synchronized HIP iterations successfully. The container exited normally and the host remained responsive.

V620 #2 then completed the same 20 out of 20 synchronized HIP iterations successfully. The container also shut down normally. Approximately 14 seconds after container teardown, the host journal stopped, and the workstation subsequently became completely unresponsive and required a power cycle.

Persistent telemetry continued after container shutdown and showed both V620s back at idle conditions immediately before the lockup, around 29–31 C junction temperature and 6–7 W board power. This strongly argues against GPU overheating or sustained power load as the immediate cause of this incident.

No AMDGPU, KFD, GPU reset, ring timeout, PCIe/AER, watchdog, OOM, or kernel-panic fault was preserved in the journal before the freeze. The last normal journal activity included the user's post-test kernel-log inspection. The lack of a recorded final fault is consistent with a complete kernel or hardware-level hang that prevented diagnostic data from being flushed.

Docker/containerd teardown for both isolated tests appeared normal in the journal. This means the HIP kernel itself can complete successfully while the overall ROCm/KFD lifecycle remains unstable afterward.

The broad kernel grep also returned Docker virtual-Ethernet `unregistering` messages because the search term `ring` matches that word; these were not GPU ring errors.

## Current Interpretation

The two Radeon Pro V620 GPUs can each execute short isolated ROCm 7.14 / HIP workloads successfully. However, **post-workload ROCm/KFD stability is not validated**, and isolated execution must not be described as stable.

The second lockup occurred after a single-GPU test with the other V620 not exposed to the container, so simultaneous dual-GPU execution is not required to reproduce the system-level instability.

The new cooling solution substantially improved Vulkan thermal behavior, and telemetry before the most recent lockup showed both GPUs cool and idle. Cooling is therefore not the leading explanation for the ROCm lockups.

A GPU/KFD/driver lifecycle, reset, PCIe, or platform interaction remains possible, but the current evidence does not identify a confirmed root cause.

The known-good Vulkan inference path remains the production/control baseline.

## Kernel A/B Baseline

To isolate the kernel as a variable, the retained fallback kernel was booted and validated before any further ROCm activity.

The fallback baseline passed the following checks:

- kernel booted successfully
- patched AMDGPU module matched the running kernel
- both V620s exposed the expected 120–250 W power-cap range
- both V620s were automatically capped to 170 W
- the ARCTIC fan-controller module matched the running kernel and its service was active
- the RTX 3050 loaded with the expected NVIDIA driver
- Vulkan enumeration remained unchanged: RTX 3050 first, followed by the two V620s

This creates a controlled kernel-only A/B path while keeping the GPU power, cooling, display stack, and Vulkan control baseline unchanged.

### ROCm Discovery-Only Lifecycle Test

On the fallback kernel, a ROCm 7.14 container was launched with only one V620 exposed. Discovery tools were used without launching a HIP compute kernel, and the container was then exited.

The host remained responsive for the observation period after container teardown. Persistent telemetry showed both V620s holding normal idle conditions at roughly 29–30 C junction temperature and 6–7 W board power. The post-test kernel review showed normal boot-time AMDGPU/KFD initialization and no new ring timeout, GPU reset, page fault, watchdog, machine-check, PCIe/AER, or hardware-error event associated with the discovery test.

This stage therefore **passes the ROCm discovery/container-lifecycle check on the fallback kernel**. It does not yet validate HIP compute stability, but it shows that simply initializing ROCm/KFD and tearing down the container did not reproduce the earlier hard lockup during this observation window.

### Tiny HIP Compute Lifecycle Test

The next fallback-kernel test exposed only one V620 and launched a deliberately tiny HIP kernel against 1,024 float elements. The runtime reported one visible HIP device, `hipDeviceSynchronize` returned no error, and the test exited with code 0.

After the container exited normally, the workstation remained responsive through a ten-minute post-workload observation window. Persistent telemetry showed both V620s back at idle conditions around 29–31 C junction temperature and 6–7 W board power. A focused kernel-event review returned no new AMDGPU/KFD fault, ring timeout, GPU reset, page fault, PCIe/AER event, watchdog/lockup report, machine-check, or hardware-error message.

This stage therefore **passes the tiny single-GPU HIP compute + teardown + idle observation checkpoint on the fallback kernel**. The result is encouraging because it did not reproduce the post-ROCm hard lock observed on the newer kernel, but it is still too small a sample to claim full ROCm stability or establish the kernel as the confirmed root cause.

## Next Step

Increase the fallback-kernel ROCm workload gradually while preserving the same single-variable A/B approach:

1. run five synchronized HIP iterations on V620 #1 only
2. exit the container and observe the host for ten minutes with telemetry and kernel-event monitoring
3. if stable, repeat the identical five-iteration test on V620 #2 only
4. only after both isolated cards pass should the single-GPU workload be increased further
5. do not proceed to dual-GPU HIP, RCCL, PyTorch ROCm, P2P experimentation, or vLLM tensor parallelism until the fallback-kernel path remains stable under progressively larger isolated workloads

No host-side ROCm installation, ECC change, P2P workaround, VBIOS modification, or aggressive PCIe tuning will be performed at this stage.