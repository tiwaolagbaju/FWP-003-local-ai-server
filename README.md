# FWP-003 — Cost-Optimized Local AI Server

I started this project with a simple question: **how much useful local-AI capability can I get without buying a brand-new high-end workstation?**

The build uses repurposed enterprise/workstation hardware with a strong focus on cost efficiency, practical performance, and learning through hands-on testing.

My goal is not just to assemble the hardware. I want to understand the full stack: Linux, GPU compute, multi-GPU inference, cooling, power, remote administration, model performance, and the tradeoffs that come with using older hardware in a modern AI workload.

[← Back to Fun Weekend Projects](https://github.com/tiwaolagbaju/fun-weekend-projects)

> Public documentation is intentionally sanitized. Credentials, addresses, device identifiers, remote-access details, and other unnecessary operational information are not published.

## What I’m Building

The system is based on an HP Z6 G4 workstation with two AMD Radeon Pro V620 GPUs.

| Component | Configuration |
|---|---|
| Workstation | HP Z6 G4 |
| CPU | Intel Xeon Platinum 8168 — 24 cores / 48 threads |
| AI GPUs | 2× AMD Radeon Pro V620 32GB |
| Aggregate GPU VRAM | **64GB** |
| Secondary GPU | NVIDIA RTX 3050 6GB |
| System memory | 96GB ECC planned |
| Storage | 2× 2TB NVMe |
| PSU | 1000W |
| GPU cooling | Custom active cooling for the passive V620 cards |

The hardware is intentionally unconventional for a local-AI workstation. That is part of the project: figuring out where used enterprise hardware provides real value and where the compromises start to outweigh the savings.

## Main Goals

- run larger local language models that benefit from 64GB of aggregate VRAM
- test single-GPU and multi-GPU inference
- compare Vulkan and ROCm support where practical
- experiment with CPU/RAM offload for workloads that exceed GPU memory
- provide browser-based access to local models
- manage the system remotely without exposing unnecessary services
- benchmark performance, thermals, stability, and power behavior
- compare total spend against the AI capability achieved

## Cost Focus

A big part of this project is **AI capability per dollar**.

Current documented new spending is approximately **$1,822**, with some components reused from hardware I already owned. The final evaluation will look at what that investment can actually run rather than relying on specifications alone.

I plan to compare:

- model size and quantization
- usable VRAM
- tokens per second
- single- vs. dual-GPU behavior
- system RAM usage
- thermals and stability
- power consumption where measurable
- total project cost
- cost per GB of GPU VRAM
- overall usefulness compared with newer alternatives

## High-Level Architecture

```text
Client Workstation
      |
      | Secure remote administration / browser access
      v
Local Network
      |
      v
Linux AI Server
      |
      +-- Local model runtime
      +-- Containerized services
      +-- Web interface
      |
      +-- Radeon Pro V620 — 32GB
      +-- Radeon Pro V620 — 32GB
      +-- Secondary display / compute GPU
```

The public architecture is intentionally simplified. Network addressing, hostnames, device identifiers, authentication details, and remote-access configuration are kept private.

## Software I’m Exploring

- Ubuntu Linux
- llama.cpp
- Vulkan
- ROCm where supported
- GGUF models
- Docker / Docker Compose
- Open WebUI
- SSH / remote development tools

I may test additional platforms as the build evolves, but I want the stack to stay understandable and easy to reproduce in my own lab rather than adding tools just for the sake of complexity.

## Build Approach

I’m validating the system in stages instead of installing everything at once. That makes troubleshooting much easier when working with older workstation hardware and unusual GPU configurations.

The main phases are:

1. baseline workstation validation
2. BIOS and PCIe configuration
3. power and cooling validation
4. single-GPU bring-up
5. dual-GPU bring-up
6. Linux GPU configuration
7. local model deployment
8. multi-GPU testing
9. remote access and service deployment
10. benchmarking and cost/performance analysis

## Why This Project Matters to Me

Most of my professional experience has been around mission-critical electrical infrastructure, where reliability, troubleshooting, and understanding how systems interact are essential.

This project lets me apply that same engineering mindset to IT and AI infrastructure: isolate variables, validate changes, watch thermals and power, troubleshoot methodically, document what worked, and avoid assuming that a configuration is stable just because it boots.

## Skills Demonstrated

- Linux administration
- local AI infrastructure
- GPU compute
- multi-GPU configuration
- Docker and service deployment
- Vulkan / ROCm experimentation
- PCIe resource planning
- enterprise workstation hardware
- power and thermal management
- remote administration
- benchmarking and performance analysis
- cost/performance engineering
- technical troubleshooting
- security-conscious documentation

## Current Status

**In progress.** The build is being documented at validated milestones rather than treating planned work as completed work.

Supporting notes in the `docs/` folder cover project planning and hardware/cost tracking. Additional benchmark and troubleshooting documentation will be added as the system is brought online.

---

**FWP-003** is part of my [Fun Weekend Projects](https://github.com/tiwaolagbaju/fun-weekend-projects) portfolio.