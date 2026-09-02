# Phase 17 — Qwen3.8 27B Benchmark

## Goal

Benchmark a current-generation 27B-class model on the Radeon Pro V620 platform and compare dual-GPU versus single-GPU execution when the model is small enough to fit on one 32 GB V620.

## Model

- **Qwen3.8-27B-GGUF**
- Quantization: **Q4_K_M**
- Runtime: `llama.cpp`
- Backend: Vulkan

## Dual-V620 Baseline

The first benchmark used both Radeon Pro V620 accelerators with an even tensor split.

Measured performance:

- **Prompt processing: 85.6 tokens/s**
- **Generation: 17.5 tokens/s**

Captured active-load sensor snapshot:

### V620 #1

- edge: **46 C**
- junction: **47 C**
- memory: **46 C**
- PPT: **70 W**
- sclk: **116 MHz**
- mclk: **456 MHz**

### V620 #2

- edge: **48 C**
- junction: **52 C**
- memory: **50 C**
- PPT: **87 W**
- sclk: **1 GHz**
- mclk: **1000 MHz**

Total captured GPU PPT was approximately **157 W**.

Both cards remained thermally comfortable, with junction temperatures at or below **52 C** in the captured snapshot.

## Initial Observation

The snapshot shows a substantial instantaneous clock difference between the two V620s: the second card was operating at a much higher core and memory clock than the first. This single sample does not prove persistent imbalance, because pipeline scheduling can cause utilization to vary between cards during inference.

However, the model is small enough to fit on a single V620, making this an ideal workload for measuring whether dual-GPU splitting provides a performance advantage or instead introduces communication/pipeline overhead.

## Current Result

| Check | Result |
|---|---|
| Qwen3.8 27B Q4_K_M dual-GPU run | **PASS** |
| Prompt processing | **85.6 t/s** |
| Generation | **17.5 t/s** |
| V620 #1 junction | **47 C** |
| V620 #2 junction | **52 C** |
| V620 #1 PPT | **70 W** |
| V620 #2 PPT | **87 W** |
| Approx. combined GPU PPT | **157 W** |
| Single-V620 comparison | Pending |

## Next Step

Run the exact same model, context, prompt, and generation length on a single V620. Compare:

- prompt processing speed
- generation speed
- GPU power
- temperature
- whether model layers fully fit on one card

If single-GPU performance is equal to or better than dual-GPU performance, models of this size can be assigned to one V620 while the second accelerator remains available for another model, user, or concurrent workload.