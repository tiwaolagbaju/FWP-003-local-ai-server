# Phase 16 — Qwen2.5 72B Dual-V620 Benchmark

## Goal

Validate that the dual Radeon Pro V620 workstation can run a 70B-class local language model at usable speed while maintaining stable thermals.

## Model and Runtime

Model:

- **Qwen2.5-72B-Instruct-GGUF**
- Quantization: **Q4_K_M**
- Runtime: `llama.cpp`
- Backend: Vulkan
- GPU allocation target: both Radeon Pro V620 accelerators

The benchmark was run with both V620s selected and an even tensor split.

## Performance Result

Measured llama.cpp performance:

- **Prompt processing: 82.3 tokens/s**
- **Generation: 8.5 tokens/s**

This establishes that a 72B-class Q4 model is usable on the dual-V620 system at interactive generation speed.

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

## Current Status

| Check | Result |
|---|---|
| Qwen2.5 72B Q4_K_M launched | **PASS** |
| Both V620s active | **PASS** |
| Prompt processing | **82.3 t/s** |
| Generation | **8.5 t/s** |
| V620 #1 junction | **68 C** |
| V620 #2 junction | **77 C** |
| V620 #1 memory | **56 C** |
| V620 #2 memory | **64 C** |
| V620 #1 PPT | **111 W** |
| V620 #2 PPT | **110 W** |
| 72B inference functionality | **PASS** |
| Post-run GPU/PCIe kernel health | Pending |
| Exact VRAM / CPU-offload allocation confirmation | Pending |

## Engineering Significance

This is the first successful 70B-class inference result for the workstation. The system is generating at **8.5 tokens/s** with Qwen2.5 72B Q4_K_M, demonstrating that the dual-V620 platform can reach the model-size range that motivated the 64 GB physical VRAM design.

The result should not yet be described publicly as a fully GPU-resident 72B workload until the model-load allocation output is captured and confirms whether all model layers and relevant runtime buffers remained in GPU memory or whether any CPU/RAM offload occurred.

Thermals are higher than the 32B benchmark but still controlled. A longer-duration run should be used later to determine whether the 77 C junction snapshot represents a steady-state maximum or a transient value.

## Next Validation Step

1. Review the kernel log for AMD GPU resets, timeouts, PCIe Bus Errors, AER errors, uncorrected errors, and Surprise Link Down events.
2. Capture model-load memory/allocation output on the next 72B run to determine the exact GPU-resident versus CPU-offloaded configuration.
3. Run a longer 72B workload while recording per-card temperature and power over time.