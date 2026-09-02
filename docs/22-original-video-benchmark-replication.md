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

The exact quantization name is not exposed in the available video transcript. The closest official Qwen GGUF currently available is:

- Repository: `Qwen/Qwen3-Coder-Next-GGUF`
- Quantization: **Q4_K_M**
- Tested GGUF size reported by llama.cpp: **45.08 GiB** (approximately 48.4 GB decimal)

This is therefore documented as a **near-exact benchmark replication**, not a byte-for-byte identical model-file comparison.

## Brain-Box Replication Configuration

- Host: HP Z6 G4
- GPU backend: Vulkan
- Compute GPUs: **2× Radeon Pro V620 32 GB**
- Display GPU excluded from text inference
- Model: `Qwen/Qwen3-Coder-Next-GGUF:Q4_K_M`
- Final context target: **262,144 tokens**
- GPU layers: all available layers
- Tensor split: 1:1
- Multimodal projector: disabled for text-only benchmarking

The benchmark is being scaled progressively through smaller contexts before the final 262K replication target.

## Model Architecture Confirmed

llama.cpp reports:

- architecture: **qwen3next**
- model type: **80B.A3B**
- total parameters: **79.67B**
- transformer blocks: **48**
- experts: **512**
- experts selected per token: **10**
- native context: **262,144 tokens**

Across all completed tests so far, model-weight placement has remained constant:

- output layer offloaded to GPU
- 47 repeating layers offloaded to GPU
- **49/49 total loadable layers offloaded to GPU**
- Vulkan1 model buffer: **23,764.29 MiB**
- Vulkan2 model buffer: **22,231.40 MiB**
- CPU_Mapped model buffer: **166.92 MiB**
- pipeline parallelism: **enabled**
- Flash Attention: **enabled**

The 49/49 count includes the output layer in addition to the model's 48 transformer blocks.

---

## 4K Baseline — PASS

Context: **4,096 tokens**

Runtime allocations:

- Vulkan1 KV cache: **48 MiB**
- Vulkan2 KV cache: **48 MiB**
- Vulkan1 recurrent-state buffer: **39.78 MiB**
- Vulkan2 recurrent-state buffer: **35.59 MiB**
- Vulkan1 compute buffer: **118.23 MiB**
- Vulkan2 compute buffer: **118.23 MiB**
- projected combined device use: **46,419 MiB**
- projected aggregate VRAM headroom: **~14.5 GiB**

Performance:

- Prompt processing: **102.5 t/s**
- Generation: **63.4 t/s**

Captured sensor snapshot:

- V620 #1: **38 C junction / 36 C memory / 53 W**
- V620 #2: **40 C junction / 38 C memory / 64 W**
- combined PPT snapshot: **117 W**

---

## 32K Context — PASS

Context: **32,768 tokens**

Runtime allocations:

- Vulkan1 KV cache: **384 MiB**
- Vulkan2 KV cache: **384 MiB**
- combined KV cache: **768 MiB**
- Vulkan1 compute buffer: **230.23 MiB**
- Vulkan2 compute buffer: **230.23 MiB**
- projected combined device use: **47,315 MiB**
- projected aggregate VRAM headroom: **~13.7 GiB**

Performance:

- Prompt processing: **202.7 t/s**
- Generation: **63.5 t/s**

Captured sensor snapshot:

- V620 #1: **40 C junction / 38 C memory / 52 W**
- V620 #2: **42 C junction / 40 C memory / 63 W**
- combined PPT snapshot: **115 W**

Generation throughput was effectively unchanged from the 4K baseline.

---

## 128K Context — PASS

The third validation increased the allocated context to **131,072 tokens**, exactly half the model's native 262,144-token context.

### Full GPU Offload

The model again remained completely GPU-offloaded:

- **49/49 total loadable layers offloaded to GPU**
- Vulkan1 model buffer: **23,764.29 MiB**
- Vulkan2 model buffer: **22,231.40 MiB**
- CPU_Mapped model buffer: **166.92 MiB**

No model layers were moved back to system memory.

### 128K Memory Allocation

llama.cpp projected:

- Vulkan1 device use: **25,954 MiB**
- Vulkan2 device use: **24,433 MiB**
- combined projected V620 use: **50,387 MiB**
- combined available device memory: **61,298 MiB**
- aggregate projected headroom: **10,911 MiB (~10.7 GiB)**

Runtime allocations:

- Vulkan1 KV cache: **1,536 MiB**
- Vulkan2 KV cache: **1,536 MiB**
- combined KV cache: **3,072 MiB (3 GiB)**
- Vulkan1 recurrent-state buffer: **39.78 MiB**
- Vulkan2 recurrent-state buffer: **35.59 MiB**
- Vulkan1 compute buffer: **614.23 MiB**
- Vulkan2 compute buffer: **614.23 MiB**
- Host compute buffer: **520.05 MiB**
- pipeline parallelism: **enabled**
- Flash Attention: **enabled**

### 128K Performance

Measured performance:

- **Prompt processing: 203.2 tokens/s**
- **Generation: 64.7 tokens/s**

Generation throughput remained extremely stable across the allocated-context tests:

- 4K: **63.4 t/s**
- 32K: **63.5 t/s**
- 128K: **64.7 t/s**

The supplied prompt remains short, so these tests validate context allocation and available-memory behavior rather than performance with a fully populated long prompt.

### 128K Thermal / Power Snapshot

Captured sensor snapshot:

#### V620 #1

- edge: **40 C**
- junction: **41 C**
- memory: **38 C**
- PPT: **54 W**
- sclk: **1 GHz**
- mclk: **1000 MHz**

#### V620 #2

- edge: **38 C**
- junction: **42 C**
- memory: **40 C**
- PPT: **64 W**
- sclk: **1 GHz**
- mclk: **1000 MHz**

Captured combined PPT was approximately **118 W**.

### 128K Result

| Check | Result |
|---|---|
| Full transformer/output GPU offload | **49/49 — PASS** |
| Context allocation | **131,072 — PASS** |
| Prompt processing | **203.2 t/s** |
| Generation | **64.7 t/s** |
| V620 junction snapshots | **41 / 42 C** |
| Captured combined PPT | **118 W** |
| Aggregate projected VRAM headroom | **~10.7 GiB** |

---

## Context-Scaling Comparison

| Metric | Source Video | Brain-Box 4K | Brain-Box 32K | Brain-Box 128K | Brain-Box 262K |
|---|---:|---:|---:|---:|---:|
| Model | Qwen3 Coder Next | Q4_K_M | Q4_K_M | Q4_K_M | Pending |
| Allocated context | ~262K | 4K | 32K | 128K | 262,144 target |
| GPU offload | all blocks | **49/49** | **49/49** | **49/49** | Pending |
| Prompt processing | Not captured | **102.5** | **202.7** | **203.2** | Pending |
| Generation | ~55 t/s | **63.4** | **63.5** | **64.7** | Pending |
| Junction snapshot | ~44–45 C | **38 / 40 C** | **40 / 42 C** | **41 / 42 C** | Pending |
| Combined PPT snapshot | ~230–270 W | **117 W** | **115 W** | **118 W** | Pending |
| Projected VRAM headroom | Not stated | **~14.5 GiB** | **~13.7 GiB** | **~10.7 GiB** | Pending |

## Important Benchmark Note

Allocating a context window with `-c` does not fill it with prompt tokens. The current runs establish that llama.cpp can reserve the requested context while retaining full GPU offload and measure short-prompt generation throughput under that allocation.

A true long-context throughput benchmark would require feeding tens or hundreds of thousands of actual prompt tokens. That is a separate test from reproducing the video's configured maximum-context setup.

## Remaining Validation Steps

1. Attempt the full **262,144-token context** target.
2. Confirm full 49/49 layer offload at maximum context.
3. Capture projected VRAM use, KV/cache buffers, performance, thermals, and power.
4. Run the post-test GPU/PCIe/AER kernel-health check.
5. Compare the final maximum-context result directly with the video's approximately **55 t/s** reported generation figure.

## Acceptance Criteria

The replication will be considered successful if:

1. Qwen3 Coder Next loads across both V620s.
2. All model layers remain GPU-offloaded.
3. The requested **262,144-token context** is retained without silent reduction.
4. Generation throughput, thermals, and power are recorded under the same broad class of workload.
5. Any difference from the video is documented with likely variables such as quantization, runtime version, LM Studio vs direct llama.cpp execution, CPU/platform differences, and cooling configuration.