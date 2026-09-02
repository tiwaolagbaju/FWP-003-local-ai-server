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

- **Prompt processing: 102.5 tokens/s**
- **Generation: 63.4 tokens/s**

### Thermal / Power Snapshot

- V620 #1: **38 C junction / 36 C memory / 53 W**
- V620 #2: **40 C junction / 38 C memory / 64 W**
- captured combined PPT: **117 W**

### 4K Result

| Check | Result |
|---|---|
| Full transformer/output GPU offload | **49/49 — PASS** |
| Context | **4096** |
| Prompt processing | **102.5 t/s** |
| Generation | **63.4 t/s** |
| V620 junction snapshots | **38 / 40 C** |
| Captured combined PPT | **117 W** |
| Aggregate projected VRAM headroom | **~14.5 GiB** |

---

## 32K Context — PASS

The second validation increased the allocated context from 4096 to **32,768 tokens** while keeping the same model, two-GPU topology, quantization, prompt, and generation length.

### GPU Offload

The 32K run retained the exact same model-weight split:

- **49/49 total loadable layers offloaded to GPU**
- Vulkan1 model buffer: **23,764.29 MiB**
- Vulkan2 model buffer: **22,231.40 MiB**
- CPU_Mapped model buffer: **166.92 MiB**

No model layers were moved back to CPU memory as context increased.

### 32K Memory Allocation

llama.cpp projected:

- Vulkan1 device use: **24,418 MiB**
- Vulkan2 device use: **22,897 MiB**
- combined projected V620 use: **47,315 MiB**
- combined available device memory: **61,298 MiB**
- aggregate projected headroom: approximately **13.7 GiB**

Context/runtime allocations increased to:

- Vulkan1 KV cache: **384.00 MiB**
- Vulkan2 KV cache: **384.00 MiB**
- combined KV cache: **768.00 MiB**
- Vulkan1 recurrent-state buffer: **39.78 MiB**
- Vulkan2 recurrent-state buffer: **35.59 MiB**
- Vulkan1 compute buffer: **230.23 MiB**
- Vulkan2 compute buffer: **230.23 MiB**
- pipeline parallelism: **enabled**
- Flash Attention: **enabled**

### 32K Performance

Measured performance:

- **Prompt processing: 202.7 tokens/s**
- **Generation: 63.5 tokens/s**

Generation throughput was effectively unchanged from the 4K baseline:

- 4K: **63.4 t/s**
- 32K: **63.5 t/s**

The higher prompt-processing number should not be interpreted as evidence that larger context inherently improves prompt speed. The actual supplied prompt remained short, and short-prompt timing can vary substantially between runs.

### 32K Thermal / Power Snapshot

Captured sensor snapshot:

#### V620 #1

- edge: **39 C**
- junction: **40 C**
- memory: **38 C**
- PPT: **52 W**
- sclk: **792 MHz**
- mclk: **1000 MHz**

#### V620 #2

- edge: **37 C**
- junction: **42 C**
- memory: **40 C**
- PPT: **63 W**
- sclk: **1 GHz**
- mclk: **1000 MHz**

Captured combined PPT was approximately **115 W**.

### 32K Result

| Check | Result |
|---|---|
| Full transformer/output GPU offload | **49/49 — PASS** |
| Context allocation | **32,768 — PASS** |
| Prompt processing | **202.7 t/s** |
| Generation | **63.5 t/s** |
| V620 junction snapshots | **40 / 42 C** |
| Captured combined PPT | **115 W** |
| Aggregate projected VRAM headroom | **~13.7 GiB** |

### Important Benchmark Note

Setting `-c 32768` allocates a 32K-capable context window, but the benchmark prompt itself does **not** contain 32K tokens. This test therefore validates **32K context capacity and memory allocation**, not performance with a fully populated 32K prompt. The same distinction will apply to the 128K and 262K replication steps unless a long prompt is deliberately supplied.

---

## Comparison Table

| Metric | Source Video | Brain-Box 4K | Brain-Box 32K | Brain-Box 262K |
|---|---:|---:|---:|---:|
| Model | Qwen3 Coder Next | Qwen3 Coder Next Q4_K_M | Same | Pending |
| Approx. GGUF size | ~47 GB | 45.08 GiB / ~48.4 GB decimal | Same | Pending |
| Allocated context | ~262K | 4K | **32K** | 262,144 target |
| Transformer blocks | 48 | 48 | 48 | Pending |
| llama.cpp offload count | N/A | **49/49** | **49/49** | Pending |
| GPUs | 2× V620 | 2× V620 | 2× V620 | 2× V620 |
| Prompt processing | Not captured | **102.5 t/s** | **202.7 t/s** | Pending |
| Generation | ~55 t/s | **63.4 t/s** | **63.5 t/s** | Pending |
| GPU junction | ~44–45 C | **38 / 40 C** | **40 / 42 C** | Pending |
| GPU power | ~230–270 W total | **117 W snapshot** | **115 W snapshot** | Pending |

## Remaining Validation Steps

1. Run a **128K context** validation.
2. Attempt the full **262,144-token context** target.
3. Record post-run GPU/PCIe/AER health after the heavy-context tests.
4. Compare the final 262K generation result directly to the video's approximately 55 t/s result.

## Acceptance Criteria

The replication will be considered successful if:

1. Qwen3 Coder Next loads across both V620s.
2. All model layers remain GPU-offloaded.
3. The requested 262,144-token context is retained without silent reduction.
4. Generation throughput, thermals, and power are recorded under the same broad class of workload.
5. Any difference from the video is documented with likely variables such as quantization, runtime version, LM Studio vs direct llama.cpp execution, CPU/platform differences, and cooling configuration.