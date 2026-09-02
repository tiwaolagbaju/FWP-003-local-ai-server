# Phase 15 — Qwen2.5 32B Dual-V620 Benchmark

## Goal

Validate that the dual Radeon Pro V620 configuration can run a substantially larger local language model across both GPUs while maintaining stable PCIe behavior and acceptable thermals.

## Model and Runtime

Model:

- **Qwen2.5-32B-Instruct-GGUF**
- Quantization: **Q4_K_M**
- Runtime: `llama.cpp`
- Backend: Vulkan
- GPU allocation: both Radeon Pro V620 accelerators

The model was loaded with all possible layers placed on the GPUs and an even tensor split across both V620s.

## Performance Result

Measured llama.cpp performance:

- **Prompt processing: 250.5 tokens/s**
- **Generation: 17.6 tokens/s**

This represents a substantial step up from the earlier 7B validation and demonstrates usable interactive generation speed with a 32B-class model.

## Thermal Result

During the 32B inference workload, observed peak GPU temperatures were approximately:

- **~45 C peak**

This was an approximate observed peak rather than a separately logged per-card maximum.

The test was performed after configuring the workstation BIOS to:

- Increase Idle Fan Speed: **80%**
- Increase PCIe Idle Fan Speed: **80%**

and after installing the permanent dual-shroud/four-fan V620 cooling solution.

## Kernel / PCIe Health Check

After the benchmark, the kernel log was filtered for:

- AMD GPU faults
- timeouts
- resets
- PCIe Bus Errors
- AER errors
- uncorrected errors
- Surprise Link Down events

The only matching output was the informational AMD message indicating that Trusted Memory Zone (TMZ) is disabled by default because it is experimental.

No GPU reset, timeout, PCIe bus error, AER error, uncorrected error, or Surprise Link Down event was observed.

## Result

| Check | Result |
|---|---|
| Qwen2.5 32B Q4_K_M loaded | **PASS** |
| Both V620s used for inference | **PASS** |
| Prompt processing | **250.5 t/s** |
| Generation | **17.6 t/s** |
| Approximate peak GPU temperature | **~45 C** |
| GPU reset / timeout errors | **None observed** |
| PCIe/AER errors | **None observed** |
| 32B dual-V620 inference | **PASS** |

## Engineering Significance

The workstation has now progressed beyond basic dual-GPU bring-up and small-model validation. A 32B-class model can run successfully across both V620 accelerators at interactive generation speeds while the permanent cooling system maintains substantial thermal headroom.

This benchmark also provides a useful intermediate reference point before attempting 70B-class models, where aggregate VRAM capacity and tensor distribution become more critical.

## Next Step

Attempt a **70B-class Q4 model** across both V620s and determine whether it can fit entirely in available GPU memory or whether partial CPU/RAM offload is required. Record generation speed, memory allocation, thermals, and post-run PCIe/AER health.