# Is the SDK `.sif` the Container That Emulates CSX Hardware?

## Question

Is `/home/weiping/Cerebras/Cerebras-SDK-2.10.0-sdk-202604101435-4586d3f0d8/sdk-cbcore-2.10.0-sdk-202604101435-4586d3f0d8.sif`
the VM image or container that emulates CSX hardware (i.e., the simfab that `train.py` connects to)?

## Answer: No — not for the PyTorch training path

The `.sif` does contain a CSX hardware emulator, but for a completely different programming layer.

| | This SDK `.sif` | What `train.py` needs |
|--|--|--|
| **Language** | CSL (Cerebras Systems Language) — low-level hardware kernels | PyTorch — high-level ML training |
| **Simulator** | Cerebras Fabric Simulator (simulates individual PEs on the WSE) | PyTorch simfab — a gRPC appliance server |
| **Workflow** | Write `.csl` → compile with `cslc` → run with `cs_python run.py` | Write PyTorch model → compile CIRH via `torch-cirh-opt` → connect to `localhost:9000` |
| **Who uses it** | CSL kernel developers writing hardware-level code | ML engineers training models via the PyTorch API |

## Evidence

From the csl-extras tutorial README:

> The device used may be an actual CS system, or it may be simulated without access to an
> actual CS system using the **Cerebras Fabric Simulator**.

The example workflow in `commands_wse2.sh`:
```bash
cslc --arch=wse2 ./layout.csl --fabric-dims=8,3 --fabric-offsets=4,1 -o out --memcpy --channels 1
cs_python run.py --name out
```

This is the CSL compile-and-simulate path, not the PyTorch training path.

## What the `.sif` actually contains

- **3.46 GB** SquashFS filesystem, 178,799 inodes
- Container label: `net.cerebras.cluster_semantic_version: 1.14.0`
- Entrypoint: `/cbcore/bin/entrypoint`
- Internal PATH: `/cbcore/bin`, CSL toolchains (`cslang`, `llvm`), RDMA tools
- Internal PYTHONPATH: `/cbcore/py_root`
- Tools exposed by the SDK wrappers: `cslc`, `csdb`, `cs_python`, `sdk_debug_shell`

Despite the large size, gRPC strings, and `cluster_semantic_version` label, this is the
**Cerebras Fabric Simulator + CSL toolchain** for low-level kernel development — not the
PyTorch appliance gRPC server.

## Naming confusion

"Simfab" appears in both contexts but means different things:

- **Cerebras Fabric Simulator** (simfab) → simulates CSL fabric/PE execution → **this SDK**
- **PyTorch simfab** → gRPC service at `mgmt_address:9000` that accepts compiled PyTorch
  model graphs → **separate enterprise software, not in this SDK**

## Bottom line

You have the CSL SDK (bare-metal hardware programming layer). The PyTorch appliance server
is a separate Cerebras product distributed to enterprise customers with CSX hardware or
simulation licenses.  
