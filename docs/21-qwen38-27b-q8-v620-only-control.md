# Qwen3.8-27B Q8_0 — V620-Only Control Benchmark

## Goal

Repeat the Qwen3.8-27B Q8_0 benchmark with the multimodal projector disabled so the RTX 3050 does not participate in the workload.

## Test Configuration

- Model: **Qwen3.8-27B-GGUF**
- Quantization: **Q8_0**
- Runtime: `llama.cpp`
- Backend: Vulkan
- Language-model GPU: **1× Radeon Pro V620**
- Multimodal projector: **disabled with `--no-mmproj`**
- Context: **4096 tokens**

The RTX 3050 remained visible to Vulkan but was not used by the text-only benchmark.

## Full GPU-Offload Confirmation

Verbose llama.cpp output confirmed:

- output layer offloaded to GPU
- 63 repeating layers offloaded to GPU
- **65/65 total model layers offloaded to Vulkan1**

Memory allocation:

- Vulkan1 model buffer: **25,972.28 MiB**
- CPU_Mapped model buffer: **1,288.28 MiB**
- Vulkan1 KV buffer: **256.00 MiB**
- Vulkan1 recurrent-state buffer: **149.62 MiB**
- Vulkan1 compute buffer: **158.13 MiB**
- projected device-memory use: **26,536 MiB**
- projected free V620 memory: **4,001 MiB**

The CPU-mapped buffer does not represent CPU transformer-layer execution; llama.cpp explicitly reported all **65/65 layers** offloaded to the V620.

## Performance

Measured performance:

- **Prompt processing: 119.9 tokens/s**
- **Generation: 13.6 tokens/s**

## Thermal / Power Snapshot

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

The second V620 remained effectively idle.

## Comparison to Previous Q8 Run

The earlier Q8 run allowed llama.cpp to automatically load the model's multimodal projector on the RTX 3050. That run produced:

- Prompt: **119.4 t/s**
- Generation: **13.5 t/s**
- Active V620 junction: **78 C**
- PPT: **217 W**

The V620-only control produced:

- Prompt: **119.9 t/s**
- Generation: **13.6 t/s**
- Active V620 junction: **78 C**
- PPT: **216 W**

The difference is negligible and consistent with normal run-to-run variation.

## Result

| Check | Result |
|---|---|
| Multimodal projector disabled | **PASS** |
| RTX 3050 excluded from workload | **PASS** |
| Q8 model fully offloaded to one V620 | **65/65 layers — PASS** |
| Prompt processing | **119.9 t/s** |
| Generation | **13.6 t/s** |
| Active V620 junction | **78 C** |
| Active V620 PPT | **216 W** |
| Projected VRAM headroom at 4K | **~4.0 GiB** |
| Material RTX 3050 effect on text-only inference | **None observed** |

## Engineering Significance

This control run confirms that the RTX 3050-hosted vision projector had no meaningful effect on text-only Qwen3.8-27B Q8_0 throughput. For clean text-only benchmarking, `--no-mmproj` should be used so the RTX 3050 remains outside the inference path.

The V620 alone is sufficient to host the full Q8 language model at a 4096-token context while retaining approximately 4 GiB of projected VRAM headroom.