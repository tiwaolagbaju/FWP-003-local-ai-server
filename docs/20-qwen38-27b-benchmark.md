# Phase 17 — Qwen3.8 27B Benchmark

## Goal

Benchmark a current-generation 27B-class model on the Radeon Pro V620 platform, compare dual-GPU versus single-GPU execution, evaluate both Q4_K_M and Q8_0 quantizations, and isolate the effect of the RTX 3050 multimodal projector on text-only inference.

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

Qwen3.8-27B Q8_0 was tested on one V620 at a 4096-token context.

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

### Q8 Performance — Automatic Multimodal Projector Enabled

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

The second V620 remained essentially idle.

### Multimodal Projector Observation

The Qwen3.8 GGUF includes multimodal support. During this first Q8 run, llama.cpp automatically loaded the vision projector on the RTX 3050 (`Vulkan0`) while the 27B language model remained on the selected V620.

The vision projector was approximately **600 MiB**, with additional RTX compute-buffer allocation during warmup.

---

## Q8_0 — V620-Only Control Run (`--no-mmproj`)

The Q8 benchmark was repeated with the multimodal projector explicitly disabled using `--no-mmproj` so the RTX 3050 would not participate in the workload.

Verbose output again confirmed:

- **65/65 model layers offloaded to Vulkan1**
- Vulkan1 model buffer: **25,972.28 MiB**
- Vulkan1 KV buffer: **256.00 MiB**
- Vulkan1 recurrent-state buffer: **149.62 MiB**
- Vulkan1 compute buffer: **158.13 MiB**
- projected device-memory use: **26,536 MiB**
- projected free VRAM: **4,001 MiB**

The RTX 3050 was still enumerated as `Vulkan0`, but no vision projector/CLIP model was loaded and no RTX compute allocation occurred for the benchmark.

Measured performance:

- **Prompt processing: 119.9 tokens/s**
- **Generation: 13.6 tokens/s**

Captured active-load sensor snapshot:

### Active V620

- edge: **75 C**
- junction: **78 C**
- memory: **58 C**
- PPT: **216 W**
- sclk: **2 GHz**
- mclk: **1000 MHz**

### Second V620

- edge: **33 C**
- junction: **35 C**
- memory: **32 C**
- PPT: **7 W**
- sclk: **0 Hz**
- mclk: **96 MHz**

### Projector-On vs Projector-Off Comparison

| Metric | Q8 + automatic mmproj | Q8 `--no-mmproj` |
|---|---:|---:|
| Prompt processing | **119.4 t/s** | **119.9 t/s** |
| Generation | **13.5 t/s** | **13.6 t/s** |
| Active V620 junction | **78 C** | **78 C** |
| Active V620 memory | **58 C** | **58 C** |
| Active V620 PPT | **217 W** | **216 W** |
| 27B model layers on V620 | **65/65** | **65/65** |
| RTX 3050 vision projector | **Loaded** | **Disabled** |

The differences were approximately:

- **+0.4% prompt throughput** with the projector disabled
- **+0.7% generation throughput** with the projector disabled
- **1 W lower captured V620 PPT**
- no change in captured junction or memory temperature

These differences are small enough to be treated as normal run-to-run variation. The control run therefore confirms that the RTX 3050-hosted vision projector had **no material effect on text-only Qwen3.8-27B Q8 inference performance**.

---

## Q4_K_M vs Q8_0 — Single V620

For the cleanest text-only comparison, the Q8 `--no-mmproj` result is used below.

| Metric | Q4_K_M | Q8_0 (`--no-mmproj`) |
|---|---:|---:|
| Prompt processing | **136.7 t/s** | **119.9 t/s** |
| Generation | **19.0 t/s** | **13.6 t/s** |
| Junction snapshot | **74 C** | **78 C** |
| Memory temperature | **58 C** | **58 C** |
| PPT | **205 W** | **216 W** |
| Full model-layer GPU offload | **PASS** | **65/65 — PASS** |
| Approx. projected free VRAM at 4K | More headroom | **~4.0 GiB** |

Q8_0 remains substantially slower than Q4_K_M, but provides a higher-precision quantization while still fitting entirely on one V620 at a 4K context.

## Operational Implication

For Qwen3.8-27B:

- **Q4_K_M on one V620** is currently the best performance-oriented configuration.
- **Q8_0 on one V620** is viable when higher quantization fidelity is preferred over maximum throughput.
- For text-only Q8 use, `--no-mmproj` avoids unnecessarily loading the multimodal projector on the RTX 3050 without materially affecting text generation speed.
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
| Q8 V620-only `--no-mmproj` control | **PASS** |
| Q8 full model-layer GPU offload | **65/65 — PASS** |
| Q4 single-GPU generation | **19.0 t/s** |
| Q8 V620-only generation | **13.6 t/s** |
| Q4 single-GPU prompt processing | **136.7 t/s** |
| Q8 V620-only prompt processing | **119.9 t/s** |
| Q8 active V620 junction | **78 C** |
| Q8 active V620 PPT | **216 W** |
| Q8 projected free VRAM at 4K | **~4.0 GiB** |
| RTX 3050 effect on text-only Q8 throughput | **No material effect observed** |
| Preferred performance configuration | **Q4_K_M / single V620** |
| Higher-fidelity single-GPU option | **Q8_0 / single V620** |

## Engineering Significance

The benchmark demonstrates topology, quantization, and device-role tradeoffs on the V620 platform. Qwen3.8-27B can run entirely on a single 32 GB V620 even at Q8_0 precision, while Q4_K_M provides substantially higher generation speed and more memory headroom.

The V620-only control also proves that the RTX 3050 is not contributing materially to text-only inference performance when llama.cpp automatically loads the multimodal projector. The RTX can therefore remain reserved for display duties or be deliberately assigned to vision encoding only when multimodal input is actually needed.

This strengthens the final architecture of the local AI server: mid-size language models can be pinned to individual V620s, multi-GPU operation can be reserved for models that genuinely require more than one card's VRAM, and the RTX 3050 can be treated as an optional vision/display accelerator rather than part of the core text-inference path.

## Next Step

1. Test Q8_0 at larger context sizes to determine the practical single-V620 VRAM ceiling.
2. Benchmark additional current dense and MoE models using the most appropriate GPU topology.
3. Add a deliberate multimodal image-input benchmark to quantify the RTX 3050 + V620 split when vision is actually needed.