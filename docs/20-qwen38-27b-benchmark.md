# Phase 17 — Qwen3.8 27B Benchmark

## Goal

Benchmark a current-generation 27B-class model on the Radeon Pro V620 platform, compare dual-GPU versus single-GPU execution, and evaluate both Q4_K_M and Q8_0 quantizations.

## Model

- **Qwen3.8-27B-GGUF**
- Runtime: `llama.cpp`
- Backend: Vulkan

Tested quantizations:

- **Q4_K_M**
- **Q8_0**

---

## Q4_K_M — Dual-V620 Baseline

The first Q4 benchmark used both Radeon Pro V620 accelerators with an even tensor split.

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

---

## Q4_K_M — Single-V620 Run

The same Q4 model, context size, prompt, and generation length were then run on a single V620.

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

### Q4 Single vs Dual

| Metric | 2× V620 | 1× V620 |
|---|---:|---:|
| Prompt processing | **85.6 t/s** | **136.7 t/s** |
| Generation | **17.5 t/s** | **19.0 t/s** |
| Junction snapshot | **47 / 52 C** | **74 C** |
| Captured GPU PPT | **70 + 87 W** | **205 W active + 7 W idle** |

For Q4_K_M, the single-V620 run was faster for both prompt processing and token generation. This demonstrates that splitting a model that already fits on one 32 GB accelerator can introduce enough inter-GPU pipeline/communication overhead to outweigh the benefit of using a second GPU.

---

## Q8_0 — Single-V620 Run

Qwen3.8-27B Q8_0 was then tested on one V620 at the same 4096-token context.

### Full GPU-Offload Confirmation

Verbose llama.cpp output confirmed:

- output layer offloaded to GPU
- 63 repeating layers offloaded to GPU
- **65/65 total layers offloaded to GPU**

Memory allocation:

- model file size: **26.62 GiB**
- Vulkan1 model buffer: **25,972.28 MiB**
- CPU_Mapped model buffer: **1,288.28 MiB**
- Vulkan1 KV buffer: **256.00 MiB**
- Vulkan1 recurrent-state buffer: **149.62 MiB**
- Vulkan1 compute buffer: **158.13 MiB**

`llama.cpp` projected approximately **26,536 MiB of V620 memory use** from approximately **30,537 MiB free**, leaving about **4,001 MiB** of projected free VRAM at a 4096-token context.

The CPU-mapped buffer does not indicate CPU layer evaluation; llama.cpp explicitly reported **65/65 model layers offloaded to the V620**.

### Q8 Performance

Measured performance:

- **Prompt processing: 119.4 tokens/s**
- **Generation: 13.5 tokens/s**

Captured active-load sensor snapshot:

### Active V620

- edge: **75 C**
- junction: **78 C**
- memory: **58 C**
- PPT: **217 W**
- sclk: **2 GHz**
- mclk: **1000 MHz**

### Second V620

- edge: **35 C**
- junction: **36 C**
- memory: **34 C**
- PPT: **7 W**
- sclk: **0 Hz**
- mclk: **96 MHz**

The second V620 again remained essentially idle.

### Multimodal Projector Observation

The Qwen3.8 GGUF includes multimodal support. During model initialization, llama.cpp automatically loaded the vision projector on the RTX 3050 (`Vulkan0`) while the 27B language model remained on the selected V620.

The vision projector was approximately **600 MiB**, with additional RTX compute-buffer allocation during warmup. This did not change the full-layer offload status of the language model, but it is useful for future multimodal benchmarking because the RTX 3050 can serve as the vision-encoder device while a V620 hosts the language model.

---

## Q4_K_M vs Q8_0 — Single V620

| Metric | Q4_K_M | Q8_0 |
|---|---:|---:|
| Prompt processing | **136.7 t/s** | **119.4 t/s** |
| Generation | **19.0 t/s** | **13.5 t/s** |
| Junction snapshot | **74 C** | **78 C** |
| Memory temperature | **58 C** | **58 C** |
| PPT | **205 W** | **217 W** |
| Full model-layer GPU offload | **PASS** | **65/65 — PASS** |
| Approx. projected free VRAM at 4K | More headroom | **~4.0 GiB** |

Relative to Q4_K_M on the same single V620, Q8_0 delivered approximately:

- **12.7% lower prompt-processing throughput**
- **28.9% lower generation throughput**
- **12 W higher captured GPU power**
- **4 C higher captured junction temperature**

The performance cost buys a substantially higher-precision quantization while still fitting entirely on one V620 at a 4K context.

## Operational Implication

For Qwen3.8-27B:

- **Q4_K_M on one V620** is currently the best performance-oriented configuration.
- **Q8_0 on one V620** is viable when higher quantization fidelity is preferred over maximum throughput.
- Splitting Q4 across both V620s reduced performance and is not the preferred topology for this model size.
- The second V620 can remain available for another model or user while one card runs Qwen3.8-27B.
- The Q8 configuration has substantially less VRAM headroom than Q4, so larger-context testing should be performed before assuming very long context operation on a single card.

Dual-GPU execution remains valuable for models that exceed one card's VRAM capacity, such as the previously validated Qwen2.5 72B Q4_K_M workload.

## Result

| Check | Result |
|---|---|
| Qwen3.8 27B Q4_K_M dual-GPU run | **PASS** |
| Qwen3.8 27B Q4_K_M single-GPU run | **PASS** |
| Qwen3.8 27B Q8_0 single-GPU run | **PASS** |
| Q8 full model-layer GPU offload | **65/65 — PASS** |
| Q4 single-GPU generation | **19.0 t/s** |
| Q8 single-GPU generation | **13.5 t/s** |
| Q4 single-GPU prompt processing | **136.7 t/s** |
| Q8 single-GPU prompt processing | **119.4 t/s** |
| Q8 active V620 junction | **78 C** |
| Q8 active V620 PPT | **217 W** |
| Q8 projected free VRAM at 4K | **~4.0 GiB** |
| Preferred performance configuration | **Q4_K_M / single V620** |
| Higher-fidelity single-GPU option | **Q8_0 / single V620** |

## Engineering Significance

The benchmark demonstrates both topology and quantization tradeoffs on the V620 platform. Qwen3.8-27B can run entirely on a single 32 GB V620 even at Q8_0 precision, while Q4_K_M provides substantially higher generation speed and more memory headroom.

This strengthens the final architecture of the local AI server: mid-size models can be pinned to individual V620s, reserving multi-GPU operation for models that genuinely require more than one card's VRAM. The separate RTX 3050 can also participate as a vision-encoder device for multimodal Qwen3.8 workloads.

## Next Step

1. Test Q8_0 at larger context sizes to determine the practical single-V620 VRAM ceiling.
2. Benchmark additional current dense and MoE models using the most appropriate GPU topology.
3. Add multimodal image-input testing to quantify the RTX 3050 + V620 split architecture.