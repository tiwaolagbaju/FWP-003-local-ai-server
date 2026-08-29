# FWP-003 — Cost-Optimized Local AI / Homelab Server

## Project Overview

FWP-003 is part of my **Fun Weekend Project** series: hands-on technical projects designed to build practical skills and document the engineering process from planning through validation.

The goal of this project is to build a **powerful, cost-effective local AI workstation/server using used enterprise and workstation hardware**, then measure the AI capability achieved for the money spent.

Rather than buying a new high-end AI workstation, this project explores a different question:

> **How much practical local-AI capability can be achieved per dollar by carefully selecting and repurposing used enterprise hardware?**

The finished system will run as a basement homelab server and will be administered remotely from an office workstation.

## Target Outcome

- **64GB aggregate GPU VRAM** from 2× AMD Radeon Pro V620 32GB GPUs
- **96GB ECC system memory** planned
- 30B–70B class local LLM inference
- 70B Q4-class model testing across both GPUs
- Multi-GPU inference experimentation on AMD hardware
- Vulkan and ROCm testing where supported
- CPU/RAM offload experiments for larger models and MoE workloads
- Browser-based AI access using Open WebUI
- Remote administration using SSH and VS Code Remote
- Docker-based services where practical
- Secure remote connectivity using Tailscale where appropriate
- Benchmarking focused on **AI capability per dollar**

## Core Hardware

| Component | Configuration |
|---|---|
| Workstation | HP Z6 G4 |
| CPU | Intel Xeon Platinum 8168 — 24C/48T |
| AI GPUs | 2× AMD Radeon Pro V620 32GB |
| Aggregate AI VRAM | **64GB** |
| Display / secondary GPU | NVIDIA RTX 3050 6GB |
| Planned system RAM | 96GB DDR4-2666 ECC Registered |
| Primary NVMe | 2TB WD Blue SN5000 |
| Secondary NVMe | 2TB SanDisk Optimus 5100 |
| PSU | HP 1000W |
| V620 cooling | 2 custom shrouds + 4× Noctua NF-A4x20 PWM fans |

## Current Documented Spend

| Component | Cost |
|---|---:|
| 2× Radeon Pro V620 32GB | $850.00 |
| HP Z6 G4 + Xeon Platinum 8168 | $366.88 |
| A-Tech ECC RAM purchased to date | $253.81 |
| 4× Noctua NF-A4x20 fans | $38.03 |
| 2× V620 cooling shrouds | $50.84 |
| SanDisk Optimus 5100 2TB | $262.87 |
| RTX 3050 6GB | Already owned |
| WD Blue SN5000 2TB | Already owned |
| **Current documented new spend** | **$1,822.43** |

The cost figure will be updated as the build progresses. The final analysis will compare total project spend against the model sizes, inference performance, VRAM capacity, thermals, and overall capability achieved.

## High-Level Architecture

```text
Office Workstation
      |
      | Browser / SSH / VS Code Remote
      v
Home Ethernet
      |
      v
HP Z6 G4 Local AI Server
      |
      +-- Ubuntu Linux
      +-- Docker / Open WebUI
      +-- llama.cpp / Vulkan / ROCm testing
      |
      +-- Radeon Pro V620 #1 — 32GB
      +-- Radeon Pro V620 #2 — 32GB
      +-- RTX 3050 — display / secondary compute
```

The public architecture is intentionally simplified. Sensitive addressing, device identifiers, credentials, and remote-access details are not published.

## Planned PCIe Layout

| Slot | Device |
|---|---|
| Slot 2 | Radeon Pro V620 #1 |
| Slot 4 | RTX 3050 |
| Slot 5 | Radeon Pro V620 #2 |

Final layout will be validated against physical clearance, PCIe resource allocation, NVMe interactions, thermals, and system stability.

## Planned Software Stack

- Ubuntu Linux
- llama.cpp
- Vulkan
- ROCm where supported
- GGUF models
- Docker / Docker Compose
- Open WebUI
- SSH
- VS Code Remote
- Tailscale
- Potential future Proxmox experimentation

## Build Strategy

The build will be validated incrementally rather than installing every component at once:

1. Project planning and documentation
2. Baseline Z6 hardware validation
3. BIOS and PCIe configuration
4. GPU power solution validation
5. V620 cooling modification
6. Single-V620 bring-up and thermal testing
7. Dual-V620 bring-up
8. Linux GPU / Vulkan / ROCm configuration
9. Local AI software deployment
10. Multi-GPU inference testing
11. Remote administration and browser access
12. Benchmarking and cost/performance analysis
13. Final architecture, troubleshooting summary, lessons learned, and portfolio write-up

## Benchmarking Goals

The final benchmark dataset will capture, where practical:

- Model family and parameter count
- Quantization
- Model file size
- Backend used
- Single vs dual GPU
- VRAM utilization
- System RAM utilization
- Prompt-processing and generation performance
- Tokens per second
- GPU temperatures
- Power consumption
- Stability observations
- Total build cost
- Cost per GB of GPU VRAM
- AI capability achieved per dollar

The intent is to **measure value rather than claim it**. Comparisons with newer hardware will be benchmarked or sourced before conclusions are made.

## Reference Build

A key inspiration for this project is a Country Boy Computers dual-V620 budget AI workstation build demonstrating 64GB aggregate VRAM using used Radeon Pro V620 GPUs:

**YouTube:** https://www.youtube.com/watch?v=8zHUvixZAtg

FWP-003 adapts the concept to an HP Z6 G4 platform and expands it into a remotely administered Linux homelab AI server with deeper documentation, thermal validation, multi-GPU testing, and cost/performance analysis.

## Security & Documentation Policy

This is a public technical portfolio. Documentation will demonstrate architecture, configuration decisions, troubleshooting methodology, testing, and results without publishing information that creates unnecessary risk.

The repository will intentionally omit or sanitize:

- Public IP addresses
- Credentials and passwords
- API keys and authentication tokens
- Tailscale identifiers and sensitive remote-access details
- MAC addresses
- Hardware serial and asset identifiers
- Personally identifying hostnames
- Wi-Fi credentials
- Exact private-network information where it adds no technical value
- Screenshots containing sensitive information

## Skills Demonstrated

This project is intended to demonstrate practical experience with:

- Linux administration
- Local AI infrastructure
- GPU compute
- Multi-GPU configuration
- AMD Vulkan / ROCm experimentation
- Docker and service deployment
- Remote administration
- SSH and secure connectivity
- PCIe resource planning
- ECC memory and enterprise workstation hardware
- GPU power planning
- Thermal management for passive server GPUs
- Networking and homelab design
- Benchmarking and performance analysis
- Technical troubleshooting
- Cost/performance engineering
- Security-conscious technical documentation

## Project Status

**Current phase:** Phase 1 — Planning and baseline documentation

**Next milestone:** Validate the HP Z6 G4 in a known-good baseline configuration before installing the V620 GPUs.

## Documentation

- [Project Plan](docs/01-project-plan.md)
- [Hardware & Cost Tracker](docs/02-hardware-and-cost.md)

---

*FWP-003 is an evolving build. Documentation and benchmark results will be updated at validated project checkpoints.*