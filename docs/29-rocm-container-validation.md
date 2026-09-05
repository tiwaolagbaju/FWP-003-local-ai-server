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

## Significance

This confirms that the current host AMDGPU kernel driver and ROCm 7.14 userspace are compatible with both V620 cards at the basic HSA/HIP discovery layer. No architecture override was required: both cards identify natively as `gfx1030`.

The validated host Vulkan stack remains untouched and can continue to serve as the known-good control path.

## Next Step

Validate actual HIP compute independently on each V620, then run simultaneous compute on both cards. Only after basic compute passes will PyTorch, RCCL, tensor parallelism, and vLLM be introduced.