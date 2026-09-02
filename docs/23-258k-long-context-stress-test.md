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

During sustained prompt prefill, both V620 GPUs initially approached the sensor-reported critical junction temperature.

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

## Post-Stop Thermal Observation

A subsequent sensor snapshot showed a highly asymmetric condition:

| Metric | V620 #1 | V620 #2 |
|---|---:|---:|
| Edge temperature | 73 C | **104 C** |
| Junction temperature | 74 C | **109 C** |
| Memory temperature | 58 C | 76 C |
| PPT | 17 W | 88 W |
| Core clock | 0 | 505 MHz |
| Memory clock | 96 MHz | 1000 MHz |

V620 #2 remained extremely hot even after V620 #1 had already dropped substantially in temperature and power. The second card exceeded the sensor-reported 100 C critical junction threshold in this snapshot.

This asymmetric cooldown behavior is more concerning than the earlier symmetric 99 C snapshot and points to a **card-specific cooling/airflow issue on V620 #2 or a card-specific operating-state problem requiring investigation**. The available data does not identify the exact physical cause.

## Interpretation

This is **not** classified as a model-capability failure. The test exposed a thermal limitation during sustained long-context prefill.

The previous configured-context replication remained cool because the 262K KV/cache capacity was allocated but not populated through a quarter-million-token prefill operation. The actual long-context prefill drove the GPUs to sustained high clocks and substantially higher power.

The later asymmetric temperature behavior means the next diagnostic priority is V620 #2 cooling and operating state rather than simply increasing overall chassis airflow.

No prompt-throughput, generation-throughput, or retrieval result is reported because the test did not complete.

## Status

**258K actual-prompt long-context test: ABORTED — V620 #2 thermal condition requires investigation before further sustained GPU testing.**

## Required Next Actions

Before another sustained inference test:

1. Power the workstation down normally and allow both V620s to cool completely.
2. Inspect V620 #2's fan/shroud assembly, fan operation, airflow direction, obstructions, seating, and power connections, comparing it directly with V620 #1.
3. After a cold boot, record idle temperatures and power for both cards before running any model.
4. Run a kernel-health check for AMD GPU resets/faults and PCIe/AER errors.
5. Only after V620 #2 behaves normally at idle should a short, low-risk workload be used to compare its thermal ramp against V620 #1.
6. Do not retry 64K/128K/258K sustained-prefill testing until the asymmetric thermal behavior has been resolved.

The completed 262K configured-context benchmark remains valid; this is a separate sustained long-context stress validation.