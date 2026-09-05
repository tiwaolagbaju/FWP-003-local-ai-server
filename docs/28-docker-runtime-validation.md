# Phase 24 — Docker Runtime Validation

## Goal

Add a container runtime for ROCm and vLLM experimentation without disturbing the known-good host inference stack.

## Validation

Docker Engine was installed successfully and validated with the standard `hello-world` container.

Post-install checks confirmed that the existing host configuration remained intact:

- validated kernel remained `7.0.0-31-generic`
- the dynamic Radeon Pro V620 fan-control service remained enabled and active
- the fan policy still initializes at the expected normal and full-speed PWM values
- the existing Vulkan llama.cpp installation continued to enumerate all three GPUs correctly
- GPU ordering remained stable: RTX 3050 first, followed by the two Radeon Pro V620 cards

## Why Docker Is Being Used

ROCm and vLLM experimentation will initially be containerized rather than installed directly into the host operating system. This preserves the current Vulkan-based inference environment as a known-good recovery and comparison path while allowing ROCm userspace, PyTorch, RCCL, and vLLM versions to be tested independently.

The host will continue to provide the validated AMDGPU kernel driver. Containers will receive GPU access through the Linux KFD and DRI device interfaces.

## Next Step

Validate the host KFD/DRI device nodes, then launch a ROCm 7.14 RDNA container and confirm that both Radeon Pro V620 GPUs enumerate as `gfx1030` before attempting PyTorch or multi-GPU inference.