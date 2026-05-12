# Change Log: Local cerebras-pytorch for macOS ARM64

**Date**: 2026-05-06

## Goal

Make gigaGPT run on macOS ARM64 by switching the `cerebras_pytorch` dependency from PyPI (Linux x86_64 only) to a local unpacked source at `../cerebras-pytorch-src`.

## Investigation & Thinking Process

### 1. Explored the dependency landscape

- gigaGPT pinned `cerebras_pytorch==2.5.0` with a platform guard: `sys_platform == 'linux' and platform_machine == 'x86_64'`
- The local source in `../cerebras-pytorch-src` was version **2.10.0**, unpacked from a Linux wheel
- The unpacked wheel at `cerebras_pytorch-2.10.0/` already had a valid `pyproject.toml` using hatchling as the build backend

### 2. Discovered the `cerebras.appliance` blocker

`cerebras.pytorch` imports from `cerebras.appliance` pervasively — not just in a few files, but across **40+ import sites** spanning storage, distributed, backends, optim, utils, and more. These imports fire at module load time, so even `import cerebras.pytorch` would crash without it.

`cerebras-appliance` is on PyPI but only as a Linux x86_64 wheel — same problem as `cerebras-pytorch` itself.

### 3. Evaluated approaches

| Approach | Pros | Cons |
|----------|------|------|
| Full appliance stub | Works without env vars | Huge surface area (~50+ symbols across 28 files) |
| `GENERATING_CEREBRAS_PYTORCH_DOCS=1` env var | Skips `backends.py` (heaviest appliance consumer) | Still need stubs for remaining top-level imports |
| Patch cerebras-pytorch to lazy-import appliance | Clean, no stubs needed | Invasive changes to upstream code |

**Chosen**: Stub package + env var (hybrid approach). The env var skips the `backends.py` import chain, and a minimal stub package handles everything else.

### 4. Identified Python version constraints

- gigaGPT required `>=3.8,<3.9`, cerebras-pytorch 2.10.0 required `>=3.11`
- torch 2.4.0 CPU wheels for macOS ARM64 are available for cp310–cp312
- System Python was 3.14 (too new for torch 2.4.0)
- **Resolution**: Set `requires-python = ">=3.11,<3.13"` and use Python 3.12

## Changes Made

### A. Created `cerebras.appliance` stub package

**Location**: `../cerebras-pytorch-src/cerebras/appliance/` (~25 files)

Symlinked into the build directory: `cerebras_pytorch-2.10.0/cerebras/appliance` → `../cerebras/appliance`

Key stubs and their purposes:

| Stub module | Key exports | Why needed |
|-------------|-------------|------------|
| `utils/descriptor.py` | `Descriptor` base class | Used by `core/compile.py`, `core/device.py`, `utils/_flags.py`, `utils/pol.py` |
| `log.py` | `ClassLogger`, `named_class_logger` | Used by data executors, tensorboard writer, backends |
| `storage/__init__.py` | `StorageReader`, `register_serializer` | Used by `storage/serializers.py` as a decorator at module scope |
| `utils/memory.py` | `with_memory_info_logged` | Used as a decorator on `cstorch.load()` at module scope |
| `utils/_contexts.py` | `ValueContext`, `BooleanContext` | Used by distributed module and storage serializers |
| `cluster_config.py` | `ClusterConfig` | Directly imported by gigaGPT's `configuration.py` |
| `utils/typing.py` | `check_type`, `type_hint_to_string` | Used by `_flags.py` for runtime type validation |
| `utils/classes.py` | `retrieve_all_subclasses` | Used at module scope in `optim/scheduler.py` to build `__all__` |

### B. Updated `gigaGPT/pyproject.toml`

```diff
- requires-python = ">=3.8,<3.9"
+ requires-python = ">=3.11,<3.13"

- "cerebras_pytorch==2.5.0; sys_platform == 'linux'  and platform_machine == 'x86_64'",
- # "cerebras_pytorch==2.5.0; sys_platform == 'darwin' and platform_machine == 'arm64'",
+ "cerebras_pytorch>=2.5.0",

+ cerebras-pytorch = { path = "../cerebras-pytorch-src/cerebras_pytorch-2.10.0", editable = true }
```

### C. Patched Linux-specific code in cerebras-pytorch source

`cerebras/pytorch/utils/benchmark/utils/dataloader.py`: Removed `psutil._pslinux` and `psutil._pslinux.pio` type annotations (Linux-only, crash on macOS at class definition time).

### D. Updated `requirements.txt`

Added comment noting local source dependency.

## How to Run

```bash
GENERATING_CEREBRAS_PYTORCH_DOCS=1 uv run -p 3.12 python train.py [args...]
```

The env var `GENERATING_CEREBRAS_PYTORCH_DOCS=1` is required to skip the `backends.py` import chain, which has deep dependencies on `cerebras.appliance` internals that aren't practical to stub (appliance client, protobuf definitions, gRPC services, etc.).

## Verification Results

All gigaGPT modules import successfully:
- `configuration.py` ✓
- `data.py` ✓
- `model.py` ✓
- `train.py` ✓ (including `--help` output)
- `eval.py` ✓
- `sample.py` ✓ (exits on missing `--checkpoint_path` arg, which is normal runtime behavior)

## Known Limitations

1. **No CSX backend**: The `backends.py` module is skipped, so `cstorch.backend("CSX", ...)` won't work. CPU backend instantiation may also need additional testing.
2. **Native extensions**: The `.so` files in `cerebras/pytorch/lib/` are for Linux x86_64. The code has a graceful fallback (mock class) so import succeeds, but any code path that actually calls into the native library will raise a `RuntimeError`.
3. **Storage serialization**: The stub `StorageReader`/`StorageWriter` have no real implementation. Loading/saving Cerebras-format checkpoints won't work, but standard PyTorch checkpoint operations should be fine.
4. **`cstorch.__version__`**: Returns `None` when `GENERATING_CEREBRAS_PYTORCH_DOCS=1` is set (version import is skipped along with backends).
