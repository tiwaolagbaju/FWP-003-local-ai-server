# Phase 11 — First Inference Validation

## Goal

Perform the first real language-model inference after completing dual-V620 hardware, Linux, Vulkan, and llama.cpp device-discovery validation.

## Model / Build

- llama.cpp build: `b10680-d7bd3bfca`
- model: `Qwen/Qwen2.5-7B-Instruct-GGUF:Q4_K_M`
- quantization: **Q4_K_M**
- modality: text

The llama.cpp build was rebuilt with HTTPS/OpenSSL support after the first Hugging Face invocation failed because HTTPS support was not compiled into the binary.

## Test Prompt

```text
In three short sentences, explain what a UPS does in a data center.
```

## Result

The model loaded and completed inference successfully.

Generated response:

```text
A UPS (Uninterruptible Power Supply) in a data center ensures continuous power by switching to battery power when the main supply fails. It protects sensitive equipment from power outages and voltage fluctuations. This stability is crucial for maintaining data integrity and system uptime.
```

Measured llama.cpp performance:

- prompt processing: **150.0 tokens/s**
- generation: **62.8 tokens/s**

No application crash or obvious PCIe failure occurred during this run.

## Status

| Validation item | Result |
|---|---|
| Model download / load | **PASS** |
| llama.cpp inference | **PASS** |
| Prompt processing | **150.0 t/s** |
| Generation | **62.8 t/s** |
| Application stability during short run | **PASS** |
| Confirmed simultaneous load on both V620s | Pending verification |
| V620 peak thermals during run | Pending capture |
| V620 peak board power during run | Pending capture |
| Second-card adapter load validation | Pending |
| Sustained dual-GPU inference | Pending |

## Important Qualification

The inference command was configured to target both V620 devices, but no simultaneous utilization / thermal capture was preserved from the run. Therefore this checkpoint documents a successful application-level inference test, not yet a fully proven dual-GPU load test.

Before classifying the power adapter or dual-GPU compute path as load validated, repeat a short inference while monitoring both V620s and confirm that both cards show increased power / clocks and remain thermally stable.

## Next Step

Repeat a short controlled model run while recording both `amdgpu` sensor blocks. Confirm:

- both V620s leave idle power state
- temperatures rise in a controlled manner
- no PCIe / AER / Surprise Link Down event occurs
- the second-card power path remains stable
- no adapter or connector shows abnormal heating

Only after that should the project move to larger-model dual-GPU benchmarking.