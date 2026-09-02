# Phase 19 — 258K Actual-Prompt Long-Context Stress Test

## Goal

Move beyond allocating a 262,144-token context window and test the system with an actually populated near-maximum prompt.

A synthetic long-context retrieval document was created and measured with the model's tokenizer at **258,018 input tokens**, approximately 98.4% of the model's 262,144-token native context.

The document placed a known reference code near the beginning, followed by repeated benign UPS telemetry records and retrieval questions at the end. The objective was to measure sustained prompt-prefill behavior and basic long-range retrieval.

## Test Configuration

- Model: Qwen3 Coder Next Q4_K_M
- Compute GPUs: 2× Radeon Pro V620 32 GB
- Backend: Vulkan / llama.cpp
- Context allocation: 262,144
- Actual tokenized prompt: 258,018 tokens
- Output target: 256 tokens
- GPU offload target: all model layers
- Prompt cache disabled for one-pass testing

## Result — ABORTED FOR THERMAL LIMIT

During sustained prompt prefill, both V620 GPUs reached junction temperatures immediately adjacent to the sensor-reported critical threshold.

Captured high-load snapshot:

| Metric | V620 #1 | V620 #2 |
|---|---:|---:|
| Edge temperature | 93 C | 91 C |
| Junction temperature | **99 C** | **99 C** |
| Memory temperature | 52 C | 56 C |
| PPT | 179 W | 162 W |
| Core clock | 2 GHz | 2 GHz |

The Linux sensor output reported a **100 C critical junction threshold**. The run was manually terminated before completion.

No prompt-throughput, generation-throughput, or retrieval result is reported because the test did not complete.

## Interpretation

This is **not** classified as a model-capability failure or GPU failure. It demonstrates that an actually populated ~258K-token prompt creates a substantially more demanding sustained compute workload than the earlier maximum-context allocation test with a short prompt.

The previous configured-context replication remained cool because the 262K KV/cache capacity was allocated but not populated through a quarter-million-token prefill operation. The actual long-context prefill drove both GPUs to sustained high clocks and materially higher power.

Memory temperatures remained comparatively moderate in the captured snapshot; the limiting metric was GPU junction/core temperature.

## Status

**258K actual-prompt long-context test: ABORTED — cooling configuration requires improvement or workload/power limiting before retry.**

## Next Actions

Before attempting the full 258K prompt again:

1. Allow the system to cool fully and perform a post-reboot GPU/PCIe kernel-health check.
2. Confirm all GPU cooling fans are operating and shrouds have not shifted or developed an airflow obstruction.
3. Improve V620 airflow and/or use a conservative GPU power limit before another sustained-prefill attempt.
4. Validate the revised setup first with a smaller actual prompt such as 64K, then 128K, while recording peak junction temperatures.
5. Retry 258K only after sustained smaller-context tests demonstrate adequate thermal margin.

The completed 262K configured-context benchmark remains valid; this test is a separate sustained long-context stress validation.