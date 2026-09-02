# Phase 18 — Original Dual-V620 Video Benchmark Replication

## Goal

Reproduce as closely as practical the main AI inference benchmark shown in the Country Boy Computers video:

**“Can it be: 64gb Home AI-Server for under $1000?”**

The source video uses two Radeon Pro V620 32 GB GPUs and demonstrates Qwen3 Coder Next at maximum context in LM Studio over Vulkan.

## Source-Backed Video Target

The transcript-backed benchmark target is:

- Model: **Qwen3 Coder Next**
- Architecture: **Mixture of Experts (MoE)**
- Total parameters: approximately **80B**
- Active parameters: approximately **10B**
- Model size described in video: approximately **47 GB**
- Context: approximately **262,000 tokens**
- GPU offload: **all 48 transformer blocks**
- GPUs: **2× Radeon Pro V620 32 GB**
- Backend: **Vulkan / LM Studio**
- Reported generation speed: approximately **55 tokens/s**
- Reported GPU temperatures: approximately **44–45 C**
- Reported GPU power: approximately **230–270 W** total during the demonstrated workload

## Quantization Matching Limitation

The available transcript confirms the model size and benchmark conditions but does not expose the exact GGUF quantization name selected in LM Studio.

The closest official Qwen GGUF currently available is:

- Repository: `Qwen/Qwen3-Coder-Next-GGUF`
- Quantization: **Q4_K_M**
- Tested GGUF size reported by llama.cpp: **45.08 GiB** (approximately 48.4 GB decimal)

Because the video calls its model “47 gig” without naming the quantization, this test is described as a **near-exact benchmark replication**, not a byte-for-byte identical model-file comparison.

## Brain-Box Replication Configuration

- Host: HP Z6 G4
- GPU backend: Vulkan
- Compute GPUs: 2× Radeon Pro V620 32 GB
- Display GPU excluded from text inference
- Model: `Qwen/Qwen3-Coder-Next-GGUF:Q4_K_M`
- Final context target: **262,144 tokens**
- GPU layers: all available layers
- Tensor split: 1:1
- Multimodal projector: disabled for text-only benchmarking

The benchmark is being scaled progressively through smaller contexts before the final 262K replication target.

---

## 4K Baseline — PASS

The first validation run used a **4096-token context** to establish model compatibility, full GPU offload, thermals, and baseline throughput before increasing context size.

### Model Architecture Confirmed

llama.cpp reported:

- architecture: **qwen3next**
- model type: **80B.A3B**
- total parameters: **79.67B**
- transformer blocks: **48**
- experts: **512**
- experts selected per token: **10**
- native context: **262,144 tokens**

### Full GPU-Offload Confirmation

llama.cpp reported:

- output layer offloaded to GPU
- 47 repeating layers offloaded to GPU
- **49/49 total loadable layers offloaded to GPU**

The 49/49 count includes the output layer in addition to the model's 48 transformer blocks.

Model buffers:

- **Vulkan1: 23,764.29 MiB**
- **Vulkan2: 22,231.40 MiB**
- **CPU_Mapped: 166.92 MiB**

Combined V620 model buffers were approximately **45,995.69 MiB (~44.92 GiB)**.

### 4K Context / Runtime Buffers

- Vulkan1 KV cache: **48.00 MiB**
- Vulkan2 KV cache: **48.00 MiB**
- Vulkan1 recurrent-state buffer: **39.78 MiB**
- Vulkan2 recurrent-state buffer: **35.59 MiB**
- Vulkan1 compute buffer: **118.23 MiB**
- Vulkan2 compute buffer: **118.23 MiB**
- pipeline parallelism: **enabled**
- Flash Attention: **enabled**

Projected total device-memory use was approximately **46,419 MiB** from approximately **61,298 MiB free**, leaving roughly **14.5 GiB aggregate headroom** at 4K context.

### Performance

Measured llama.cpp performance:

- **Prompt processing: 102.5 tokens/s**
- **Generation: 63.4 tokens/s**

The 4K generation result is approximately **15% faster** than the video's reported ~55 t/s figure. This is not yet a direct apples-to-apples performance comparison because the source video used approximately 262K context while this baseline used only 4K.

### Thermal / Power Snapshot

Captured active-load sensor snapshot:

#### V620 #1

- edge: **38 C**
- junction: **38 C**
- memory: **36 C**
- PPT: **53 W**
- sclk: **997 MHz**
- mclk: **1000 MHz**

#### V620 #2

- edge: **36 C**
- junction: **40 C**
- memory: **38 C**
- PPT: **64 W**
- sclk: **1 GHz**
- mclk: **1000 MHz**

Captured combined PPT was approximately **117 W**. These are snapshot values rather than recorded peak or time-averaged power measurements.

### 4K Result

| Check | Result |
|---|---|
| Qwen3 Coder Next Q4_K_M loaded | **PASS** |
| Model architecture | **79.67B MoE / 10 experts active** |
| Full transformer/output GPU offload | **49/49 — PASS** |
| Both V620s used | **PASS** |
| Pipeline parallelism | **Enabled** |
| Context | **4096** |
| Prompt processing | **102.5 t/s** |
| Generation | **63.4 t/s** |
| V620 #1 junction snapshot | **38 C** |
| V620 #2 junction snapshot | **40 C** |
| Captured combined PPT | **117 W** |
| Aggregate projected VRAM headroom | **~14.5 GiB** |

---

## Comparison Table

| Metric | Source Video | Brain-Box 4K | Brain-Box 262K |
|---|---:|---:|---:|
| Model | Qwen3 Coder Next | Qwen3 Coder Next Q4_K_M | Pending |
| Approx. GGUF size | ~47 GB | 45.08 GiB / ~48.4 GB decimal | Pending |
| Context | ~262K | 4K | 262,144 target |
| Transformer blocks | 48 | 48 | Pending |
| llama.cpp offload count | N/A | **49/49** | Pending |
| GPUs | 2× V620 | 2× V620 | 2× V620 |
| Prompt processing | Not captured | **102.5 t/s** | Pending |
| Generation | ~55 t/s | **63.4 t/s** | Pending |
| GPU temperature | ~44–45 C | **38 / 40 C junction snapshot** | Pending |
| GPU power | ~230–270 W total | **117 W snapshot** | Pending |

## Remaining Validation Steps

1. Run a **32K context** validation.
2. Run a **128K context** validation.
3. Attempt the full **262,144-token context** target.
4. Record post-run GPU/PCIe/AER health after the heavy-context tests.
5. Compare the final 262K generation result directly to the video's approximately 55 t/s result.

## Acceptance Criteria

The replication will be considered successful if:

1. Qwen3 Coder Next loads across both V620s.
2. All model layers remain GPU-offloaded.
3. The requested 262,144-token context is retained without silent reduction.
4. Generation throughput, thermals, and power are recorded under the same broad class of workload.
5. Any difference from the video is documented with likely variables such as quantization, runtime version, LM Studio vs direct llama.cpp execution, CPU/platform differences, and cooling configuration.