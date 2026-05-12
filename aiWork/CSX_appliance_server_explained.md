# CSX Appliance Server — What It Is and How to Get It

## What the SDK at `/home/weiping/Cerebras/SDK/` Is

This is the **CSL (Cerebras Systems Language) SDK** — tools for writing low-level code directly
in the hardware programming language for the Wafer-Scale Engine:

| Tool | Purpose |
|------|---------|
| `cslc` | CSL compiler |
| `csdb` | CSL debugger |
| `cs_python` | Python wrapper inside the container |
| `sdk_debug_shell` | Interactive shell inside the container |
| `sdk-cbcore-*.sif` | Singularity container holding all of the above |

All tools run inside the `.sif` Singularity container image. This is a **separate layer** from
PyTorch training — it targets bare-metal kernel development on the WSE, not the Python ML
training path.

## What `train.py` Actually Needs

`train.py` with `backend: "CSX"` connects to a **CSX Appliance Server** — a gRPC service
(default port 9000, configured via `mgmt_address` in the YAML) that:

1. Accepts a compiled model graph (torch → CIRH lowering, done locally by `torch-cirh-opt`)
2. Runs the computation on simulated or real CSX hardware
3. Streams results (activations, gradients, losses) back to the training client

The `cerebras-appliance` Python package in `cerebras-appliance-src/` is the **client** side of
this protocol. The `main.py` in that package is a stub:

```python
def main():
    print("Hello from cerebras-appliance!")
```

You have the client library. You do not have the server.

## How the CSX Appliance Server Is Deployed

Cerebras distributes the appliance server as a proprietary software stack to enterprise customers.
It is **not publicly downloadable**. Typical deployment forms:

- **Real CSX cluster** — Kubernetes-managed pods running on CS-2/CS-3 hardware, provisioned by
  Cerebras at a customer data center or Cerebras cloud
- **Simfab (software simulator)** — a VM image or container that emulates CSX hardware at the
  software level, also distributed by Cerebras under a separate agreement

Neither is included in the CSL SDK.

## Options

| Option | What you get |
|--------|-------------|
| Contact Cerebras (sales/support) | Request simfab access or a cloud trial |
| Use `configs/cpu_small.yaml` | Fully local CPU training — proven working on this machine |
| CSL SDK simulator | Can simulate CSL kernels directly, but does not serve the PyTorch gRPC protocol |

## Local Compilation Does Work

The torch→CIRH lowering (model graph compilation) **runs successfully** on this machine with:

```shell
PATH="/home/weiping/Cerebras/cerebras-pytorch-src/cerebras_pytorch-2.10.0/cerebras/pytorch/lib:$PATH" \
  uv run -p 3.11 python train.py configs/csx_sim_small.yaml
```

The only blocker is the gRPC connection to `localhost:9000`. Everything up to and including
compilation is functional — Python 3.11, `torch-cirh-opt`, and `cerebras.appliance` client
are all working.
