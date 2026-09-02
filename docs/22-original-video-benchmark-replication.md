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
- GPU offload: **all 48 layers**
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
- Published model size: approximately **48.4 GB**

Because the video calls its model “47 gig” without naming the quantization, this test should be described as a **near-exact benchmark replication**, not a byte-for-byte identical model-file comparison.

## Brain-Box Replication Configuration

Planned configuration:

- Host: HP Z6 G4
- GPU backend: Vulkan
- Compute GPUs: 2× Radeon Pro V620 32 GB
- Display GPU excluded from text inference
- Model: `Qwen/Qwen3-Coder-Next-GGUF:Q4_K_M`
- Context target: **262,144 tokens**
- GPU layers: all available layers
- Tensor split: 1:1
- Sampling parameters aligned with Qwen recommendations where practical

## Measurements to Capture

- successful model load
- actual layer offload count
- model buffer allocation per V620
- KV / recurrent / compute buffer allocation
- whether llama.cpp preserves the full 262,144-token context or must reduce it
- prompt-processing throughput
- generation throughput
- V620 junction temperatures
- V620 memory temperatures
- V620 PPT / power
- post-run AMD GPU / PCIe / AER kernel health

## Comparison Table

| Metric | Source Video | Brain-Box |
|---|---:|---:|
| Model | Qwen3 Coder Next | Qwen3 Coder Next |
| Approx. GGUF size | ~47 GB | Q4_K_M ~48.4 GB |
| Context | ~262K | 262,144 target |
| GPU layers | 48/48 | Pending |
| GPUs | 2× V620 | 2× V620 |
| Generation | ~55 t/s | Pending |
| GPU temperature | ~44–45 C | Pending |
| GPU power | ~230–270 W total | Pending |

## Acceptance Criteria

The benchmark will be considered successfully reproduced if:

1. Qwen3 Coder Next loads across both V620s.
2. All model layers are GPU-offloaded.
3. The requested 262,144-token context is retained without silent reduction.
4. Generation throughput, thermals, and power are recorded under the same broad class of workload.
5. Any difference from the video is documented with likely variables such as quantization, runtime version, LM Studio vs direct llama.cpp execution, CPU/platform differences, and cooling configuration.