# Phase 5 — llama.cpp Vulkan Validation

## Goal

Use `llama.cpp` as a controlled AI commissioning tool to verify that the Radeon Pro V620 can run a real local LLM through the Vulkan backend before introducing the second GPU or a full application stack.

This phase is intentionally separated from Open WebUI, Docker, ROCm, and multi-GPU inference so that any failure can be isolated to the runtime / GPU layer.

## First Build Attempt

The first Vulkan-enabled `llama.cpp` CMake configuration failed with:

```text
Could NOT find Vulkan (missing: Vulkan_LIBRARY Vulkan_INCLUDE_DIR)
```

The system already had a functioning Vulkan runtime: `vulkaninfo --summary` successfully enumerated the Radeon Pro V620 through Mesa RADV.

Therefore, this was diagnosed as a **development-package dependency issue**, not a GPU, driver, or Vulkan-runtime failure.

## Root Cause

Running Vulkan applications and **building** Vulkan applications require different package sets.

The existing Mesa/Vulkan runtime was sufficient for `vulkaninfo`, but compiling the `llama.cpp` Vulkan backend requires the Vulkan loader development library, headers, shader compiler, and SPIR-V headers.

The missing packages were installed and the build directory was recreated cleanly before recompiling.

## Corrective Action

```bash
sudo apt update
sudo apt install -y libvulkan-dev glslc spirv-headers

cd ~/llama.cpp
rm -rf build
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release -j$(nproc)
```

The Vulkan-enabled build completed successfully.

## llama.cpp Vulkan Device Enumeration

After the successful build, device enumeration was tested using:

```bash
./build/bin/llama-cli --list-devices
```

`llama.cpp` reported two usable Vulkan GPUs:

```text
Vulkan0: NVIDIA GeForce RTX 3050 (6390 MiB, 5796 MiB free)
Vulkan1: AMD Radeon Pro V620 (RADV NAVI21) (30704 MiB, 30678 MiB free)
```

This validates that `llama.cpp` can independently address the V620 through the Vulkan backend.

### Interpretation

- The NVIDIA RTX 3050 remains visible as the display / secondary compute device.
- The V620 appears as a distinct **RADV NAVI21** Vulkan accelerator.
- `llama.cpp` sees approximately **30.7GB usable VRAM** on the V620, matching the earlier kernel / `amdgpu` memory initialization results.
- Nearly all V620 VRAM was free at this checkpoint, confirming no unexpected large resident workload was consuming the accelerator before testing.

This is the first point in the project where the V620 has been validated through the **actual AI inference runtime** that will be used for the commissioning test.

## Validation Chain Completed So Far

```text
HP Z6 BIOS
  -> PCIe Gen3 x16 host link
  -> 32GB Resizable BAR
  -> Linux amdgpu driver
  -> Mesa RADV Vulkan
  -> llama.cpp Vulkan backend
  -> Radeon Pro V620 detected as usable AI device
```

This staged approach makes it possible to isolate failures at each layer rather than treating the AI software stack as one opaque system.

## Troubleshooting Lesson

This checkpoint demonstrates an important distinction in Linux GPU environments:

- **Runtime validation**: proves an installed application can use Vulkan.
- **Development environment validation**: proves headers, libraries, and shader-development tools are available to compile Vulkan software.

Because `vulkaninfo` already worked before the build failure, there was no reason to reinstall or modify the working `amdgpu` / Mesa graphics stack. Installing only the missing development dependencies resolved the build issue while preserving the known-good GPU baseline.

## Current Status

| Check | Result |
|---|---|
| V620 available through Vulkan / RADV | PASS |
| llama.cpp repository cloned | PASS |
| CMake detects compiler / CPU backend | PASS |
| Vulkan development files detected | PASS |
| Vulkan-enabled llama.cpp build | **PASS** |
| RTX 3050 visible to llama.cpp | **PASS — Vulkan0** |
| V620 visible to llama.cpp | **PASS — Vulkan1 / RADV NAVI21** |
| V620 usable VRAM reported | **PASS — 30,704 MiB** |
| First GGUF inference | Pending |
| Load thermal test | Pending |

## Next Step

The next test will load a small GGUF model entirely onto the V620 and run inference while monitoring:

- model size and quantization
- layers offloaded to GPU
- V620 VRAM utilization
- prompt processing speed
- generation speed in tokens/second
- GPU edge temperature
- junction / hotspot temperature
- memory temperature
- GPU power draw

The goal of the first model is not maximum performance. It is to establish a **stable, repeatable single-V620 AI workload baseline** before the second GPU is added.

## Security / Documentation Note

Raw terminal photographs are not published directly because the shell prompt contains local username / hostname information. Public documentation records only sanitized device names, memory figures, commands, and benchmark results required to demonstrate the build.

## Engineering Takeaway

The V620 is now validated through the complete software path needed for Vulkan-based local LLM inference. A compile-time dependency failure was isolated and corrected without disturbing the working GPU driver stack, and `llama.cpp` now recognizes the V620 as an independent 30.7GB Vulkan device.

The project is ready for its first real model inference and load-temperature benchmark.