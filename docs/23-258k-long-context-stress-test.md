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

During sustained prompt prefill, both V620 GPUs tracked at broadly similar temperatures for most of the run and eventually approached the sensor-reported critical junction temperature.

First captured high-load snapshot:

| Metric | V620 #1 | V620 #2 |
|---|---:|---:|
| Edge temperature | 93 C | 91 C |
| Junction temperature | **99 C** | **99 C** |
| Memory temperature | 52 C | 56 C |
| PPT | 179 W | 162 W |
| Core clock | 2 GHz | 2 GHz |

The Linux sensor output reported a **100 C critical junction threshold**.

The run was stopped before completion. The last reported llama.cpp progress line was:

```text
cached n_tokens = 94208, memory_seq_rm [94208, end)
```

This indicates that llama.cpp had reached/cached approximately **94,208 tokens** in the prompt-processing workflow when the run was interrupted. Relative to the 258,018-token test document, that is roughly 36% of the intended prompt. This log line is a context/cache-management message, not by itself evidence of a thermal shutdown.

## Late-Run / Post-Stop Thermal Observation

A later sensor snapshot showed a highly asymmetric operating state:

| Metric | V620 #1 | V620 #2 |
|---|---:|---:|
| Edge temperature | 73 C | **104 C** |
| Junction temperature | 74 C | **109 C** |
| Memory temperature | 58 C | 76 C |
| PPT | 17 W | 88 W |
| Core clock | 0 | 505 MHz |
| Memory clock | 96 MHz | 1000 MHz |

The important context is that the two V620s had remained relatively even through most of the sustained prefill. The late divergence is therefore **more consistent with V620 #1 having throttled or dropped into a low-power state while V620 #2 continued processing/remaining active** than with immediately concluding that V620 #2 has a fundamentally worse cooling assembly.

V620 #1 had fallen to 17 W with a 0 MHz reported core clock and 96 MHz memory clock, while V620 #2 was still drawing 88 W with a 505 MHz core clock and 1000 MHz memory clock. That operating-state difference can explain why card #1 cooled rapidly while card #2 continued accumulating or retaining heat.

The available snapshots do **not** prove the exact throttling mechanism or why the two cards behaved differently. Possible areas for investigation include per-GPU thermal/power management, workload scheduling/pipeline behavior, driver state, and card-specific cooling. A physical cooling fault should not be assumed solely from the late asymmetric snapshot.

## Interpretation

This is **not** classified as a model-capability failure. The test exposed a thermal limitation during sustained long-context prefill.

The previous configured-context replication remained cool because the 262K KV/cache capacity was allocated but not populated through a quarter-million-token prefill operation. The actual long-context prefill drove both GPUs to sustained high clocks and substantially higher power.

The key new observation is an apparent difference in late-run power/clock behavior: V620 #1 backed down dramatically while V620 #2 remained active. The next diagnostic priority is therefore to determine **whether both cards' thermal/power-management behavior is functioning consistently under sustained load**, while also confirming the physical cooling path.

No prompt-throughput, generation-throughput, or retrieval result is reported because the test did not complete.

## Status

**258K actual-prompt long-context test: ABORTED — sustained thermal behavior and asymmetric GPU throttling/power-state behavior require investigation before another near-full-context attempt.**

## Required Next Actions

Before another sustained near-full-context inference test:

1. Allow both V620s to cool fully and confirm normal idle temperatures/power after a cold boot.
2. Perform a post-test AMD GPU / PCIe kernel-health check.
3. Confirm both cards' fans/shrouds and airflow paths are physically comparable and unobstructed.
4. During the next controlled workload, record both cards' junction temperature, PPT, core clock, and memory clock continuously so the point where either card begins backing down can be identified.
5. Use a smaller actual-prompt workload first rather than immediately repeating 258K.
6. Consider a conservative power cap for sustained-prefill testing if supported and verified on this platform.

The completed 262K configured-context benchmark remains valid; this is a separate sustained long-context stress validation.