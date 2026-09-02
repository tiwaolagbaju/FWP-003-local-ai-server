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
- Maximum context: **262,144 tokens**
- GPU layers: all available layers
- Tensor split: 1:1
- Multimodal projector: disabled for text-only benchmarking

## Model Architecture Confirmed

llama.cpp reports:

- architecture: **qwen3next**
- model type: **80B.A3B**
- total parameters: **79.67B**
- transformer blocks: **48**
- experts: **512**
- experts selected per token: **10**
- native context: **262,144 tokens**

Across all completed tests, model-weight placement remained constant:

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

- Context: **4,096**
- Prompt: **102.5 t/s**
- Generation: **63.4 t/s**
- Projected GPU use: **46,419 MiB**
- Projected aggregate VRAM headroom: **~14.5 GiB**
- Junction snapshots: **38 / 40 C**
- Combined PPT snapshot: **117 W**

---

## 32K Context — PASS

- Context: **32,768**
- Prompt: **202.7 t/s**
- Generation: **63.5 t/s**
- Combined KV cache: **768 MiB**
- Projected GPU use: **47,315 MiB**
- Projected aggregate VRAM headroom: **~13.7 GiB**
- Junction snapshots: **40 / 42 C**
- Combined PPT snapshot: **115 W**

---

## 128K Context — PASS

- Context: **131,072**
- Prompt: **203.2 t/s**
- Generation: **64.7 t/s**
- Combined KV cache: **3,072 MiB (3 GiB)**
- Projected GPU use: **50,387 MiB**
- Projected aggregate VRAM headroom: **~10.7 GiB**
- Junction snapshots: **41 / 42 C**
- Combined PPT snapshot: **118 W**

---

## 262K Native Maximum Context — PASS

The final replication run requested the model's full native context:

- **n_ctx = 262,144**
- **n_ctx_train = 262,144**

Unlike the smaller-context runs, llama.cpp did not report that the allocated context was below the model's training context. The full native context allocation was accepted without silent reduction.

### Full GPU Offload

The maximum-context run retained complete GPU offload:

- output layer offloaded to GPU
- 47 repeating layers offloaded to GPU
- **49/49 total loadable layers offloaded to GPU**
- Vulkan1 model buffer: **23,764.29 MiB**
- Vulkan2 model buffer: **22,231.40 MiB**
- CPU_Mapped model buffer: **166.92 MiB**

No transformer layers were moved to system RAM.

### 262K Memory Allocation

llama.cpp projected:

- Vulkan1 device use: **28,002 MiB**
- Vulkan2 device use: **26,481 MiB**
- combined projected V620 use: **54,483 MiB**
- combined available device memory: **61,298 MiB**
- aggregate projected headroom: **6,815 MiB (~6.7 GiB)**

Runtime allocations:

- Vulkan1 KV cache: **3,072 MiB**
- Vulkan2 KV cache: **3,072 MiB**
- combined KV cache: **6,144 MiB (6 GiB)**
- Vulkan1 recurrent-state buffer: **39.78 MiB**
- Vulkan2 recurrent-state buffer: **35.59 MiB**
- Vulkan1 compute buffer: **1,126.23 MiB**
- Vulkan2 compute buffer: **1,126.23 MiB**
- Host compute buffer: **1,032.05 MiB**
- pipeline parallelism: **enabled**
- Flash Attention: **enabled**

### 262K Performance

Measured performance:

- **Prompt processing: 202.5 tokens/s**
- **Generation: 64.3 tokens/s**

Generation throughput remained remarkably stable across context allocations:

- 4K: **63.4 t/s**
- 32K: **63.5 t/s**
- 128K: **64.7 t/s**
- 262K: **64.3 t/s**

The 262K generation result is approximately **17% higher** than the video's reported ~55 t/s result.

This is still not a strict apples-to-apples throughput comparison because the current command allocates a 262K context window but supplies only a short prompt. It does, however, reproduce the video's maximum-context configuration and confirms that the full context can coexist with complete dual-V620 model offload.

### 262K Thermal / Power Snapshot

#### V620 #1

- edge: **39 C**
- junction: **39 C**
- memory: **38 C**
- PPT: **52 W**
- sclk: **827 MHz**
- mclk: **1000 MHz**

#### V620 #2

- edge: **37 C**
- junction: **42 C**
- memory: **40 C**
- PPT: **61 W**
- sclk: **873 MHz**
- mclk: **1000 MHz**

Captured combined PPT was approximately **113 W**.

### 262K Result

| Check | Result |
|---|---|
| Native 262,144 context allocated | **PASS** |
| Silent context reduction | **None observed** |
| Full transformer/output GPU offload | **49/49 — PASS** |
| Prompt processing | **202.5 t/s** |
| Generation | **64.3 t/s** |
| V620 junction snapshots | **39 / 42 C** |
| Combined PPT snapshot | **113 W** |
| Combined KV cache | **6 GiB** |
| Projected aggregate VRAM headroom | **~6.7 GiB** |

---

## Context-Scaling Comparison

| Metric | Source Video | Brain-Box 4K | Brain-Box 32K | Brain-Box 128K | Brain-Box 262K |
|---|---:|---:|---:|---:|---:|
| Model | Qwen3 Coder Next | Q4_K_M | Q4_K_M | Q4_K_M | **Q4_K_M** |
| Allocated context | ~262K | 4K | 32K | 128K | **262,144** |
| GPU offload | all blocks | **49/49** | **49/49** | **49/49** | **49/49** |
| Prompt processing | Not captured | 102.5 | 202.7 | 203.2 | **202.5** |
| Generation | ~55 t/s | 63.4 | 63.5 | 64.7 | **64.3** |
| Junction snapshot | ~44–45 C | 38 / 40 C | 40 / 42 C | 41 / 42 C | **39 / 42 C** |
| Combined PPT snapshot | ~230–270 W | 117 W | 115 W | 118 W | **113 W** |
| Projected VRAM headroom | Not stated | ~14.5 GiB | ~13.7 GiB | ~10.7 GiB | **~6.7 GiB** |

## Replication Status

The configured maximum-context replication target is **PASS**:

- same broad model family and size class
- same dual-V620 64 GB physical-VRAM concept
- full native 262,144-token context allocated
- all model layers offloaded across the V620s
- generation measured at **64.3 t/s**
- thermals remained in the low-40 C range in the captured snapshot

The remaining validation item is a post-run kernel/PCIe/AER health check.

## Important Benchmark Note

Allocating `-c 262144` reserves the model's full context capacity but does not itself populate the context with 262,144 prompt tokens. A true full-context throughput test would require feeding a prompt close to that size. This should be treated as a separate long-context stress benchmark rather than conflated with the video's configured-context replication.

## Remaining Validation Step

Run the post-test AMD GPU / PCIe / AER kernel-health check and record the result.