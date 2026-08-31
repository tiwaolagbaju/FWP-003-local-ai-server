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

## Inference Result

The model loaded and completed inference successfully.

Generated response:

```text
A UPS (Uninterruptible Power Supply) in a data center ensures continuous power by switching to battery power when the main supply fails. It protects sensitive equipment from power outages and voltage fluctuations. This stability is crucial for maintaining data integrity and system uptime.
```

Measured llama.cpp performance:

- prompt processing: **150.0 tokens/s**
- generation: **62.8 tokens/s**

No application crash or obvious PCIe failure occurred during this run.

## Verified Dual-GPU Load Capture

A follow-up run was monitored live with Linux `sensors` while inference was active. Both V620s clearly left their idle states simultaneously.

### V620 #1 — `amdgpu-pci-2300`

Observed during active compute:

- edge: **50 C**
- junction: **51 C**
- memory: **44 C**
- PPT: **71 W**
- shader clock: **884 MHz**
- memory clock: **1000 MHz**

### V620 #2 — `amdgpu-pci-2f00`

Observed during active compute:

- edge: **46 C**
- junction: **51 C**
- memory: **48 C**
- PPT: **82 W**
- shader clock: **971 MHz**
- memory clock: **1000 MHz**

For comparison, both cards had previously idled around **7 W**, `sclk` 0 Hz, and memory clock 96 MHz. The simultaneous increase in board power and clocks confirms that both accelerators were actively participating in compute.

The captured temperatures remained low during this short run, with both junction temperatures at **51 C** in the observed sample.

## Status

| Validation item | Result |
|---|---|
| Model download / load | **PASS** |
| llama.cpp inference | **PASS** |
| Prompt processing | **150.0 t/s** |
| Generation | **62.8 t/s** |
| Application stability during short run | **PASS** |
| Confirmed simultaneous load on both V620s | **PASS** |
| V620 #1 observed active PPT | **71 W** |
| V620 #2 observed active PPT | **82 W** |
| V620 #1 observed junction temp | **51 C** |
| V620 #2 observed junction temp | **51 C** |
| Short-duration dual-GPU compute | **PASS** |
| Second-card power path under short application load | **PASS — provisional** |
| Sustained / high-power adapter validation | Pending |
| Sustained dual-GPU inference | Pending |

## Interpretation

This is the first **verified simultaneous dual-V620 compute load** in the project.

Both accelerators moved well above their approximately 7 W idle state, raised shader and memory clocks, and remained thermally controlled while llama.cpp inference was running. This establishes that the application stack can execute work on both V620 devices at the same time.

The second-card power path also remained functional during this short application workload. However, the observed load of approximately 82 W on that GPU is substantially below the card's possible maximum power level, so this does **not** establish long-duration or full-power adapter safety.

## Next Step

The next milestone should increase duration before increasing model size or power aggressively.

Recommended progression:

1. Repeat the current model with a longer generation while monitoring both V620s.
2. Capture the highest observed junction temperature and PPT for each card over several minutes.
3. Check for new PCIe / AER / Surprise Link Down events after the run.
4. Physically inspect the second-card adapter/cable and connectors for abnormal heat, odor, discoloration, or softening.
5. Only after that passes move to larger models that make meaningful use of the approximately 61.4GB aggregate usable VRAM.

## Engineering Takeaway

The workstation has progressed from dual-GPU enumeration to **verified simultaneous dual-GPU inference**. Both Radeon Pro V620 accelerators were observed performing compute concurrently through llama.cpp/Vulkan, with a short-run sample of 71 W and 82 W respectively and both junction temperatures at 51 C. This is a successful controlled-load milestone, while sustained thermal and power-path qualification remains intentionally separate.