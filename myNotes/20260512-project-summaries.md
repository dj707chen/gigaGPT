# Project Summaries: gigaGPT, cerebras-pytorch-src, cerebras-appliance-src

**Date:** 2026-05-12  
**Model:** Claude Opus 4.6 (1M context)

---

## Question

Summarize what each of the following projects does and the relationships between them:
- gigaGPT
- cerebras-pytorch-src
- cerebras-appliance-src

---

## Answer

### 1. gigaGPT

**Purpose:** A minimal, clean implementation for training GPT-class large language models (111M to 175B parameters) on Cerebras hardware. Inspired by Karpathy's nanoGPT but scaled to GPT-3 size.

**Key highlights:**
- Only ~565 lines of Python (vs. ~20,500 for Nvidia Megatron)
- 6 core files: `train.py`, `model.py`, `data.py`, `eval.py`, `sample.py`, `configuration.py`
- Supports CPU, GPU, and CSX (Cerebras) backends
- Trains on OpenWebText with 2048-token sequences
- Includes predefined configs for 111M, 13B, 70B, and 175B parameter models
- HuggingFace model conversion for inference

### 2. cerebras-pytorch-src (v2.10.0)

**Purpose:** A PyTorch extension library that enables PyTorch models to compile and execute on Cerebras Wafer-Scale Cluster (WSC) hardware. It acts as the **framework-level integration layer**.

**Key components:**
- **Backend system** — pluggable backends for CPU, GPU, and CSX
- **Compilation pipeline** — traces PyTorch models, converts to Torch-MLIR intermediate representation, then optimizes for Cerebras hardware via `torch-cirh-opt`
- **Distributed training** — cluster configuration, data parallelism, worker state management
- **13+ optimizers** — Adam, AdamW, SGD, Lamb, Lion, Muon, etc.
- **AMP (automatic mixed precision)** — including Cerebras-specific CBFloat16
- **Sparsity** — static, dynamic, GMP, RigL, SET algorithms
- **Metrics** — accuracy, perplexity, AUC, F-beta, precision/recall, etc.
- **Checkpointing** — save/load with multiple storage backends

### 3. cerebras-appliance-src (v2.10.0)

**Purpose:** A framework-agnostic Python client SDK for communicating with and managing jobs on Cerebras Wafer-Scale Cluster hardware. It is the **lowest-level client layer**.

**Key components:**
- **ApplianceClient** — low-level gRPC client that talks to the Appliance Coordinator server; handles message serialization, retry logic, heartbeats, and bidirectional tensor streaming (max 2GB messages, 256MB chunks)
- **ApplianceManager** — high-level orchestrator for job lifecycle (compile, load, execute, finalize); manages checkpoints and power profiles
- **ClusterManagementClient** — job scheduling, resource allocation, node management, volume mounting, JWT authentication
- **Storage backends** — HDF5 and S3 for checkpoint persistence
- **Protocol Buffers** — 22 protobuf definitions for appliance services, workflow orchestration, and runtime communication
- **Error handling** — corruption, deadlock, NaN, compilation failure, resource exhaustion, version mismatch detection

---

## Relationships (Layered Architecture)

The three projects form a **layered stack**:

```
┌─────────────────────────────────────────────┐
│              gigaGPT (User Code)            │  ← Application layer
│  Minimal GPT training scripts (~565 lines)  │     Uses cerebras_pytorch API
├─────────────────────────────────────────────┤
│         cerebras-pytorch (v2.10.0)          │  ← Framework integration layer
│  PyTorch backend, compilation, distributed  │     Depends on cerebras_appliance
│  training, optimizers, AMP, sparsity        │     for hardware communication
├─────────────────────────────────────────────┤
│        cerebras-appliance (v2.10.0)         │  ← Hardware client layer
│  gRPC client, cluster management, storage,  │     Talks to Cerebras WSC
│  protobuf services, job orchestration       │     via gRPC
├─────────────────────────────────────────────┤
│     Cerebras Wafer-Scale Cluster (WSC)      │  ← Hardware
└─────────────────────────────────────────────┘
```

**Execution flow when training a model in gigaGPT:**

1. `train.py` calls `cstorch.backend("CSX")` from **cerebras-pytorch**
2. **cerebras-pytorch** traces the model, converts it to Torch-MLIR, and optimizes for Cerebras hardware
3. **cerebras-pytorch** delegates hardware operations to **cerebras-appliance**
4. **cerebras-appliance's** `ClusterManagementClient` allocates compute pods on the k8s cluster
5. **cerebras-appliance's** `ApplianceClient` opens a gRPC channel to the Coordinator
6. The model is compiled, loaded, and executed on the Wafer-Scale Engine

**Key dependency direction:** gigaGPT → cerebras-pytorch → cerebras-appliance → WSC Hardware
