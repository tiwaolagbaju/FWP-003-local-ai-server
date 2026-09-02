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

## Single-V620 Run

The same model, context size, prompt, and generation length were then run on a single V620.

Measured performance:

- **Prompt processing: 136.7 tokens/s**
- **Generation: 19.0 tokens/s**

Captured active-load sensor snapshot:

### Active V620

- edge: **72 C**
- junction: **74 C**
- memory: **58 C**
- PPT: **205 W**
- sclk: **2 GHz**
- mclk: **1000 MHz**

### Second V620

- edge: **43 C**
- junction: **44 C**
- memory: **40 C**
- PPT: **7 W**
- sclk: **0 Hz**
- mclk: **96 MHz**

The second card remained essentially idle, confirming the workload was isolated to one V620 as intended.

## Single vs Dual Comparison

| Metric | 2× V620 | 1× V620 |
|---|---:|---:|
| Prompt processing | **85.6 t/s** | **136.7 t/s** |
| Generation | **17.5 t/s** | **19.0 t/s** |
| Active-card peak junction snapshot | **47 / 52 C** | **74 C** |
| Captured GPU PPT | **70 + 87 W** | **205 W active + 7 W idle** |

For this model and configuration, the single-V620 run was faster for both prompt processing and token generation.

The result indicates that splitting a model that already fits comfortably on one 32 GB V620 can introduce enough inter-GPU pipeline/communication overhead to outweigh the benefit of using a second accelerator.

The tradeoff is that the single active V620 runs substantially harder: the card reached 205 W and 74 C junction in the captured snapshot, compared with a more distributed and cooler dual-GPU workload.

## Operational Implication

For models that fit within a single V620's usable VRAM, the default deployment strategy should be to test single-GPU execution first.

This can provide two advantages:

1. higher per-model performance when multi-GPU overhead would otherwise dominate
2. the second V620 remains available for another model, another user, embeddings/reranking, or a concurrent inference workload

Dual-GPU execution remains valuable for models that exceed one card's VRAM capacity, such as the previously validated Qwen2.5 72B Q4_K_M workload.

## Result

| Check | Result |
|---|---|
| Qwen3.8 27B Q4_K_M dual-GPU run | **PASS** |
| Qwen3.8 27B Q4_K_M single-GPU run | **PASS** |
| Dual-GPU generation | **17.5 t/s** |
| Single-GPU generation | **19.0 t/s** |
| Dual-GPU prompt processing | **85.6 t/s** |
| Single-GPU prompt processing | **136.7 t/s** |
| Single active V620 junction | **74 C** |
| Single active V620 PPT | **205 W** |
| Second V620 confirmed idle | **PASS** |
| Preferred configuration for this model | **Single V620** |

## Engineering Significance

This benchmark demonstrates that adding GPUs does not automatically improve inference performance. For a 27B-class Q4 model that fits on one 32 GB accelerator, single-GPU execution produced better prompt and generation throughput than an even two-GPU split.

This is useful for the final architecture of the local AI server: smaller and mid-size models can be pinned to individual V620s, while both cards can be combined only when the model requires additional VRAM. That creates the possibility of serving two substantial models or two users concurrently without sacrificing the ability to run 70B-class workloads when needed.

## Next Step

Continue the current-model benchmark suite using the preferred GPU topology for each model size. For models that fit on one V620, benchmark single-GPU first; for larger models, use both V620s and capture full-layer offload, generation speed, thermals, and power.