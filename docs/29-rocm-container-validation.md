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

A GPU/KFD/driver lifecycle, reset, PCIe, SVM/HMM, or platform interaction remains possible, but the current evidence does not identify a confirmed root cause.

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

This created a controlled kernel-only A/B path while keeping the GPU power, cooling, display stack, and Vulkan control baseline unchanged.

### ROCm Discovery-Only Lifecycle Test

On the fallback kernel, a ROCm 7.14 container was launched with only one V620 exposed. Discovery tools were used without launching a HIP compute kernel, and the container was then exited.

The host remained responsive for the observation period after container teardown. Persistent telemetry showed both V620s holding normal idle conditions at roughly 29–30 C junction temperature and 6–7 W board power. The post-test kernel review showed normal boot-time AMDGPU/KFD initialization and no new ring timeout, GPU reset, page fault, watchdog, machine-check, PCIe/AER, or hardware-error event associated with the discovery test.

This stage passed the ROCm discovery/container-lifecycle check on the fallback kernel.

### Tiny HIP Compute Lifecycle Test

The next fallback-kernel test exposed only one V620 and launched a deliberately tiny HIP kernel against 1,024 float elements. The runtime reported one visible HIP device, `hipDeviceSynchronize` returned no error, and the test exited with code 0.

After the container exited normally, the workstation remained responsive through a ten-minute post-workload observation window. Persistent telemetry showed both V620s back at idle conditions around 29–31 C junction temperature and 6–7 W board power. A focused kernel-event review returned no new AMDGPU/KFD fault, ring timeout, GPU reset, page fault, PCIe/AER event, watchdog/lockup report, machine-check, or hardware-error message.

This stage passed the tiny single-GPU HIP compute + teardown + idle observation checkpoint on the fallback kernel.

### Fallback-Kernel Hard Lock During Repeated ROCm Container Use

The fallback kernel did not remain stable when ROCm testing continued.

After the earlier discovery-only and tiny HIP tests had both completed successfully, another single-V620 ROCm container was launched in preparation for a five-iteration HIP test. Persistent telemetry showed both V620s still at idle immediately beforehand, around 29–30 C and 6–7 W. The journal records the new ROCm container launch, but no normal container teardown or later system activity was preserved before the host became completely unresponsive and required a power cycle.

Because the journal and telemetry stop essentially at the point where the new ROCm container is launched, there is no evidence that the planned five-iteration HIP kernel actually began executing. This means the failure cannot be attributed specifically to the larger compute workload. The reproduced instability may instead involve repeated ROCm/KFD initialization or teardown, HSA/KFD process lifecycle, SVM/HMM state, or another platform/driver interaction.

No pstore record or vmcore was recovered after reboot, and the preserved kernel log again contains no ring timeout, GPU reset, PCIe/AER fault, watchdog panic, MCE, or hardware-error event immediately before the freeze.

The fallback kernel therefore **does not resolve the ROCm hard-lock problem**. The earlier successful tiny test should be treated only as a limited smoke-test pass, not evidence of overall ROCm stability.

### Post-Reboot Hard Lock During Read-Only Inspection

After the fallback-kernel failure, the workstation rebooted into the newer kernel. The captured terminal sequence on that boot shows read-only inspection of kernel, IOMMU/KFD state, AMDGPU parameters, and PCIe capabilities. Both V620 endpoints reported normal active links and exposed no fatal or non-fatal AER status, although their PCIe device-status fields had correctable-error and unsupported-request bits latched. Both endpoints also reported that PCIe function-level reset is not supported.

The read-only PCIe command completed and returned to the shell prompt. The SSH session was then reset when the host became unresponsive again. No additional ROCm userspace launch is shown in the captured terminal sequence before this failure.

This incident is important because the immediate trigger was not a HIP kernel or an active ROCm benchmark. It suggests the workstation may remain vulnerable to a later platform/driver lock even after a reboot, and further narrows the investigation toward AMDGPU/KFD initialization, device reset/recovery behavior, PCIe/IOMMU interaction, or residual platform state rather than GPU temperature or compute load alone.

The read-only `lspci` inspection itself is not considered proven causal because it had completed before the SSH disconnect.

## Current Test Policy

ROCm compute and repeated ROCm container testing are paused.

Do not proceed to:

- additional single-GPU HIP stress testing
- dual-GPU HIP
- RCCL
- PyTorch ROCm
- PCIe P2P experimentation
- vLLM tensor parallelism

The next investigation should focus on the ROCm/KFD lifecycle and Linux 7.0 AMDGPU/KFD interaction rather than thermal tuning or simply increasing/decreasing HIP workload size. The Vulkan inference path remains the known-good control baseline.

No host-side ROCm installation, ECC change, P2P workaround, VBIOS modification, or aggressive PCIe tuning will be performed at this stage.