# Phase 16 — Qwen2.5 72B Dual-V620 Benchmark

## Goal

Validate that the dual Radeon Pro V620 workstation can run a 70B-class local language model at usable speed while maintaining stable thermals.

## Model and Runtime

Model:

- **Qwen2.5-72B-Instruct-GGUF**
- Quantization: **Q4_K_M**
- Runtime: `llama.cpp`
- Backend: Vulkan
- GPU allocation: both Radeon Pro V620 accelerators

The benchmark was run with both V620s selected and an even tensor split.

## Performance Result

Measured llama.cpp performance:

- **Prompt processing: 82.3 tokens/s**
- **Generation: 8.5 tokens/s**

This establishes that a 72B-class Q4 model is usable on the dual-V620 system at interactive generation speed.

## Full GPU-Offload Confirmation

A verbose model load confirmed:

- output layer offloaded to GPU
- 79 repeating layers offloaded to GPU
- **81/81 total layers offloaded to GPU**

Model-weight allocation:

- **Vulkan1 model buffer: 20,635.16 MiB**
- **Vulkan2 model buffer: 20,662.52 MiB**
- **CPU_Mapped model buffer: 668.25 MiB**

The small CPU-mapped buffer does not indicate that transformer layers are being evaluated on the CPU. `llama.cpp` explicitly reported all 81 model layers as GPU-offloaded.

At a 4096-token context, additional GPU allocations were:

- Vulkan1 KV cache: **656 MiB**
- Vulkan2 KV cache: **624 MiB**
- Vulkan1 compute buffer: **286.04 MiB**
- Vulkan2 compute buffer: **286.04 MiB**

The runtime projected approximately **43,213 MiB of total device-memory use** across both V620s versus approximately **61,375 MiB of free device memory**, leaving substantial headroom at this context size.

`llama.cpp` also enabled pipeline parallelism across the two V620s.

## Thermal / Power Snapshot

A captured active-load sensor snapshot during the 72B run showed:

### V620 #1

- edge: **68 C**
- junction: **68 C**
- memory: **56 C**
- PPT: **111 W**
- sclk: **257 MHz**
- mclk: **673 MHz**

### V620 #2

- edge: **72 C**
- junction: **77 C**
- memory: **64 C**
- PPT: **110 W**
- sclk: **2.0 GHz**
- mclk: **1000 MHz**

Both GPUs were under active load. The hotter card reached **77 C junction temperature** in the captured snapshot, remaining below the 100 C critical threshold reported by the sensors.

The card-to-card clock difference in this single snapshot should not be interpreted as a persistent imbalance without time-series data, since layer scheduling and instantaneous utilization can vary during inference.

## Post-Run Kernel / PCIe Health Check

After the 72B benchmark, the kernel log was filtered for:

- AMD GPU faults
- timeouts
- resets
- PCIe Bus Errors
- AER errors
- uncorrected errors
- Surprise Link Down events

The only matching entries were the informational AMD messages stating that Trusted Memory Zone (TMZ) is disabled by default because it is experimental.

No GPU reset, timeout, PCIe bus error, AER error, uncorrected error, or Surprise Link Down event was observed.

## Current Status

| Check | Result |
|---|---|
| Qwen2.5 72B Q4_K_M launched | **PASS** |
| Both V620s active | **PASS** |
| Model layers offloaded to GPU | **81/81 — PASS** |
| Vulkan1 model buffer | **20,635.16 MiB** |
| Vulkan2 model buffer | **20,662.52 MiB** |
| CPU_Mapped model buffer | **668.25 MiB** |
| Prompt processing | **82.3 t/s** |
| Generation | **8.5 t/s** |
| V620 #1 junction | **68 C** |
| V620 #2 junction | **77 C** |
| V620 #1 memory | **56 C** |
| V620 #2 memory | **64 C** |
| V620 #1 PPT | **111 W** |
| V620 #2 PPT | **110 W** |
| Post-run GPU/PCIe kernel health | **PASS** |
| 72B full-layer GPU offload | **PASS** |

## Engineering Significance

This benchmark confirms that the workstation can run a 72.96-billion-parameter Qwen2.5 model at Q4_K_M quantization with **all 81 model layers offloaded across two Radeon Pro V620 accelerators**.

The two GPUs hold roughly **41.3 GiB of model-weight buffers combined**, with KV cache and compute buffers also resident on the V620s. The small CPU-mapped model buffer is ancillary and does not change the explicit full-layer GPU-offload result reported by `llama.cpp`.

Generation at **8.5 tokens/s** demonstrates practical interactive 70B-class inference on the dual-V620 platform while retaining significant VRAM headroom at a 4096-token context.

This validates the central capability target of the build: cost-effective 70B-class local inference using two used 32 GB enterprise GPUs rather than a single high-cost accelerator.

## Next Validation Step

1. Run a longer 72B workload while recording per-card temperature and power over time.
2. Test progressively larger context sizes to determine practical VRAM headroom and performance impact.
3. Compare additional 70B-class models and quantizations for quality, speed, and memory use.