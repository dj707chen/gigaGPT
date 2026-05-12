# How to Start the PyTorch Simfab (gRPC Appliance Server)

## Question

How do I start the PyTorch simfab — the gRPC appliance server that `train.py` connects to
at `mgmt_address: localhost:9000`?

## Short answer: you can't from the SDK you have

The PyTorch appliance server requires a **Kubernetes cluster** with the Cerebras cbcore stack
deployed as pods. It is not startable as a standalone process from the `.sif` file.

## What the code reveals

Inspecting `cerebras-appliance-src` and `cerebras-pytorch-src`:

### 1. The server runs in Kubernetes

`cerebras/appliance/cluster/client.py` hard-codes the Kubernetes service account token path:

```python
KUBE_SA_TOKEN_PATH = "/var/run/secrets/kubernetes.io/serviceaccount/token"
```

The client auto-detects whether it is running inside a Kubernetes cluster. The `ClusterManagement`
and `CsCtlV1` gRPC services that `train.py` connects to are Kubernetes services — they expect to
schedule and manage jobs via the cluster control plane.

### 2. Job management via `csctl`

The appliance client communicates with a `csctl` job operator to submit, list, and cancel training
jobs. `csctl` is a Kubernetes-backed CLI tool — not something that can run standalone:

```python
# surrogate_script.py — runs inside the scheduler to proxy appliance jobs
cmd = f'csctl get job {job_id} --namespace {namespace} -ojson'
```

### 3. Two separate gRPC services (PyTorch vs CSL)

There are **two different gRPC service definitions** in the codebase:

| Service | Used by | What it does |
|---------|---------|-------------|
| `ClusterManagement` / `CsCtlV1` | `train.py` (PyTorch path) | Submit compiled PyTorch graphs to CSX hardware/simfab |
| `sdk_appliance` | `cs_python run.py` (CSL path) | Run CSL kernels on the Fabric Simulator |

The `.sif` SDK container serves the `sdk_appliance` gRPC service (CSL runtime).
The PyTorch `train.py` client uses `ClusterManagement` — a different service backed by Kubernetes.

### 4. The `.sif` is the server software, but Kubernetes is the runtime environment

The `sdk-cbcore-*.sif` contains `/cbcore/bin/entrypoint` and the full cbcore stack
(`cluster_semantic_version: 1.14.0`). This IS the software that would run the server —
but it is designed to be deployed as Kubernetes pods, not launched as a standalone binary.

## What you would actually need

To run `train.py` with `backend: "CSX"` against a local simulator:

| Component | Status |
|-----------|--------|
| Kubernetes (k3s, minikube, or full K8s) | Not installed |
| cbcore Helm chart / deployment manifests | Not provided in SDK |
| `csctl` CLI | Not provided in SDK |
| Singularity / Apptainer (to run the `.sif`) | Not installed |
| The `.sif` container itself | **Present** at `/home/weiping/Cerebras/SDK/sdk-cbcore-*.sif` |

Cerebras distributes the Helm charts and deployment manifests separately to enterprise customers.
The SDK only gives you the container image (`.sif`), not the orchestration layer needed to run it.

## Practical path forward

**Option A — Contact Cerebras**
Request access to the cbcore deployment manifests or a hosted simfab instance.
Cerebras sales/support can provide this for customers with simulation licenses.

**Option B — CPU backend (works today)**
Use `configs/cpu_small.yaml` with:
```shell
GENERATING_CEREBRAS_PYTORCH_DOCS=1 uv run -p 3.12 python train.py configs/cpu_small.yaml
```
Full local training, no appliance server needed. Proven working on this machine.

**Option C — Explore standalone cbcore launch**
Install Apptainer (`sudo apt install apptainer`) and try to invoke the `.sif` entrypoint
directly to see if a standalone server mode exists:
```shell
sudo apt install apptainer
apptainer run /home/weiping/Cerebras/SDK/sdk-cbcore-*.sif --help
```
This is experimental — there is no documented standalone mode, but the entrypoint might
expose subcommands worth exploring.
