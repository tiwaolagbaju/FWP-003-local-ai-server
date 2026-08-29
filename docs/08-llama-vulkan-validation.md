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

The current upstream `llama.cpp` Debian/Ubuntu build instructions specify:

```bash
sudo apt-get install libvulkan-dev glslc spirv-headers
```

## Resolution Plan

Install the missing build dependencies, remove the incomplete CMake build directory, and configure again:

```bash
sudo apt update
sudo apt install -y libvulkan-dev glslc spirv-headers

cd ~/llama.cpp
rm -rf build
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release -j$(nproc)
```

After a successful build, device enumeration will be tested with:

```bash
./build/bin/llama-cli --list-devices
```

## Troubleshooting Lesson

This checkpoint demonstrates an important distinction in Linux GPU environments:

- **Runtime validation**: proves an installed application can use Vulkan.
- **Development environment validation**: proves headers, libraries, and shader-development tools are available to compile Vulkan software.

Because `vulkaninfo` already worked before the build failure, there was no reason to reinstall or modify the working `amdgpu` / Mesa graphics stack.

## Current Status

| Check | Result |
|---|---|
| V620 available through Vulkan / RADV | PASS |
| llama.cpp repository cloned | PASS |
| CMake detects compiler / CPU backend | PASS |
| Vulkan development files detected | FAIL — missing dependency |
| Corrective action identified | PASS |
| Vulkan-enabled llama.cpp build | Pending |
| llama.cpp V620 device enumeration | Pending |
| First GGUF inference | Pending |
| Load thermal test | Pending |

## Engineering Takeaway

The failure was isolated without disturbing the known-good GPU driver stack. The distinction between runtime dependencies and compile-time development dependencies is being documented because it is common in Linux infrastructure and software integration work, and because preserving a known-good baseline reduces troubleshooting risk.
