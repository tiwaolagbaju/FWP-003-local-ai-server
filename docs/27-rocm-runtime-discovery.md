# Phase 23 — ROCm / Multi-GPU Runtime Discovery

## Goal

Establish a read-only baseline before adding ROCm, Docker, or a second inference runtime. The purpose of this phase is to understand the current PCIe topology and software state without disturbing the validated Vulkan configuration.

## Current Baseline

- Ubuntu 26.04.1 LTS
- validated kernel: 7.0.0-31-generic
- 2× Radeon Pro V620 32 GB
- RTX 3050 6 GB retained as the display / auxiliary GPU
- Vulkan llama.cpp remains functional
- no host ROCm stack installed
- Docker not yet installed
- CPU governor currently set to `powersave`
- system exposes a single NUMA node

## GPU Enumeration

The current Vulkan layout remains stable:

- Vulkan0: RTX 3050
- Vulkan1: Radeon Pro V620
- Vulkan2: Radeon Pro V620

Both V620s expose approximately 30.7 GB usable VRAM with ECC active.

## PCIe Topology

Each V620 is attached to a separate CPU PCIe root port. The GPU-side link is capable of PCIe 4.0 x16, while the host-facing root link negotiates at PCIe 3.0 x16, which is the expected limit of the Xeon Scalable platform used in this workstation.

Observed path for each V620:

```text
Xeon PCIe root port
    |
    | PCIe 3.0 x16
    v
GPU upstream bridge
    |
    | PCIe 4.0 x16 internally
    v
GPU downstream bridge
    |
    v
Radeon Pro V620
```

Both cards show the same host-side link width and speed, so there is no obvious asymmetric PCIe bottleneck between the two V620s.

Because the cards sit under separate root-port branches, GPU-to-GPU peer traffic should be treated as a benchmark question rather than assumed to be beneficial. P2P / RCCL tuning will be evaluated later with controlled A/B testing instead of enabling kernel or PCIe workarounds preemptively.

## Why This Matters for Agentic Workloads

The planned architecture uses both V620s as the shared inference backend for a concurrent OpenAI-compatible server. The important characteristics at this stage are:

- both cards have full x16 host links
- both paths are symmetric
- the system has one NUMA node, simplifying host memory placement
- the existing Vulkan backend remains available as a known-good control
- ROCm can be introduced as an isolated userspace/container experiment before changing the host runtime

## Next Step

Install Docker Engine from Docker's official Ubuntu repository, then validate that the existing kernel, fan-control service, NVIDIA display driver, and Vulkan llama.cpp device mapping remain unchanged.

After Docker is validated, the first ROCm work will be done inside a container. The initial container tests will focus on:

1. V620 enumeration as `gfx1030`
2. HIP / PyTorch device visibility
3. basic compute on each V620 independently
4. simultaneous dual-GPU compute
5. RCCL / tensor-parallel viability

Host-side ROCm packages, P2P modifications, ECC changes, and CPU-governor changes are intentionally deferred until the container baseline is validated.