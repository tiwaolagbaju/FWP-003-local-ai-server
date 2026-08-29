# FWP-003 Project Plan

## Objective

Build and validate a highly cost-effective local AI workstation/server using used enterprise and workstation hardware, then document the engineering process and measure the AI capability achieved per dollar.

This project is intended to function as both a practical homelab system and a technical portfolio piece demonstrating infrastructure planning, Linux administration, GPU compute, networking, troubleshooting, benchmarking, and security-conscious documentation.

## Success Criteria

The project will be considered successful when the system can reliably:

- Operate as a remotely administered Linux server
- Enumerate and use both Radeon Pro V620 GPUs
- Provide 64GB aggregate GPU VRAM for supported AI workloads
- Run large local language models, including 30B–70B class models where practical
- Demonstrate stable multi-GPU inference
- Provide browser-based access through Open WebUI or a comparable interface
- Be administered remotely using secure methods such as SSH and VS Code Remote
- Maintain safe GPU temperatures under sustained load
- Produce repeatable benchmark results
- Document total cost and compare it against the capability achieved

## Engineering Approach

The system will be built incrementally using known-good checkpoints. Each major hardware or software change will be validated before the next change is introduced.

This approach improves troubleshooting by limiting the number of variables changed at one time.

### Phase 1 — Planning & Documentation

- Define technical and cost goals
- Record current hardware inventory
- Establish public documentation structure
- Define security/redaction policy
- Identify the reference build and expected differences

### Phase 2 — Baseline Workstation Validation

Initial configuration:

- HP Z6 G4
- Intel Xeon Platinum 8168
- Temporary known-good ECC RDIMMs
- WD Blue SN5000 NVMe
- RTX 3050 display GPU
- No V620 GPUs installed

Validation targets:

- Successful POST and boot
- Firmware/BIOS review
- CPU and memory detection
- NVMe detection
- Display output
- Baseline system stability

### Phase 3 — BIOS / PCIe Preparation

Review and document the Z6-specific settings required for large multi-GPU configurations.

Potential areas of interest include:

- PCIe MMIO/resource assignment
- UEFI boot configuration
- PCIe generation settings
- Resizable BAR or related resource features where available and applicable
- Fast Boot behavior

Settings from reference builds on other HP workstation models will not be copied blindly; Z6-specific behavior will be validated first.

### Phase 4 — GPU Power Validation

- Identify available OEM PSU connectors
- Verify the purpose and pinout of any undocumented or unused connector before use
- Determine a safe method to supply the required PCIe GPU power
- Confirm cable and connector current capacity
- Avoid SATA-to-PCIe and Molex-to-PCIe adapters

### Phase 5 — Passive GPU Cooling Adaptation

Each Radeon Pro V620 is designed for high-airflow server chassis and therefore requires active airflow modification in the Z6 workstation.

Planned solution:

- 1 custom shroud per V620
- 2× Noctua NF-A4x20 PWM 40mm fans per GPU

Validation:

- Fan operation
- Idle temperatures
- Sustained inference temperatures
- Thermal stability
- Noise observations

### Phase 6 — Single-V620 Bring-Up

Install only the first V620 and validate:

- BIOS/PCIe enumeration
- Linux enumeration
- Correct PCIe link characteristics
- Vulkan visibility
- Temperature monitoring
- Initial AI inference

### Phase 7 — Dual-V620 Bring-Up

Add the second V620 and validate:

- Both GPUs enumerate reliably
- 64GB aggregate VRAM is visible to the intended AI runtime
- PCIe resource allocation remains stable
- No thermal or power-related instability
- Display GPU remains functional

### Phase 8 — Linux AI Stack

Deploy and document:

- Ubuntu Linux
- GPU drivers
- Vulkan tools/runtime
- ROCm where supported
- llama.cpp
- GGUF model workflow

### Phase 9 — AI Services

Deploy:

- Docker / Docker Compose where practical
- Open WebUI
- Local model backend

### Phase 10 — Remote Administration

Provide secure remote access from the office workstation using:

- Browser-based AI UI
- SSH
- VS Code Remote
- Tailscale where appropriate

Public documentation will use generalized diagrams and sanitized examples rather than exposing sensitive addressing or authentication details.

### Phase 11 — Benchmarking

Test a representative set of local models and configurations.

Benchmark dimensions may include:

- Model family
- Parameter count
- Quantization
- Model file size
- Backend
- Context configuration
- Single vs dual GPU
- VRAM usage
- System RAM usage
- Tokens/second
- GPU temperatures
- Power consumption where measurable
- Stability

### Phase 12 — Cost / Capability Analysis

The project will conclude with a value analysis focused on **AI capability per dollar**.

Examples:

- Total out-of-pocket build cost
- Cost per GB of GPU VRAM
- Largest model successfully loaded
- Performance of representative models
- Thermals and power compromises
- Comparison with newer alternatives where equivalent benchmark data or reliable sources are available

The project will avoid unsupported claims and will distinguish measured results from estimates.

## Public Documentation Principles

The repository is intentionally employer-facing and portfolio-oriented.

Documentation should emphasize:

- Why design decisions were made
- What constraints were identified
- How risks were mitigated
- How changes were validated
- What failed and how it was troubleshot
- What measurable result was achieved
- What skill the work demonstrates

Sensitive operational details will be omitted or redacted.

## Portfolio Outcome

The completed project should support:

- A polished GitHub repository
- Resume project bullets
- A LinkedIn project post
- Recruiter conversations about Linux, AI infrastructure, homelab engineering, networking, troubleshooting, cost optimization, and technical documentation
