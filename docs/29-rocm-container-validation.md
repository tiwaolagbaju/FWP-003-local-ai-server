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

## Significance

This confirms that the current host AMDGPU kernel driver and ROCm 7.14 userspace are compatible with both V620 cards at both the HSA/HIP discovery and basic compute layers. No architecture override was required: both cards identify natively as `gfx1030`.

The validated host Vulkan stack remains untouched and can continue to serve as the known-good control path.

## Next Step

Perform a short concurrent dual-GPU HIP test to verify that both V620s can execute work at the same time. Because the replacement GPU cooling fans have not yet been installed, this validation should remain brief and should not be treated as a sustained thermal or performance benchmark.

After short concurrent compute is validated, move to PyTorch ROCm device discovery and basic tensor operations. Sustained RCCL, tensor-parallel, and vLLM benchmarking will wait for the upgraded GPU cooling hardware.