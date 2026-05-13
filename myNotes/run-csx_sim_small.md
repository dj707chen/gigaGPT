# Run configs/csx_sim_small.yaml on MB14 (Linux x86_64)

### Session 2026-05-11

```shell
# Attempt 1: original command — fails on Python 3.12
GENERATING_CEREBRAS_PYTORCH_DOCS=1 uv run -p 3.12 python train.py configs/csx_sim_small.yaml
# ModuleNotFoundError: No module named '...._mlir_libs._mlir'
# Root cause: MLIR .so files compiled for Python 3.11, not 3.12.

# Attempt 2: switch to Python 3.11 (uv python install 3.11 first)
GENERATING_CEREBRAS_PYTORCH_DOCS=1 uv run -p 3.11 python train.py configs/csx_sim_small.yaml
# AttributeError: module 'cerebras.pytorch' has no attribute 'backends'
# Root cause: GENERATING_CEREBRAS_PYTORCH_DOCS=1 skips `from .backends import backends`.
# CSX LTC backend needs cstorch.backends.csx.debug (ltc_backend.py:98).
# This flag was designed for CPU doc generation, not CSX runs.

# Attempt 3: drop the env var (cerebras.appliance IS available from cerebras-appliance-src)
uv run -p 3.11 python train.py configs/csx_sim_small.yaml
# RuntimeError: torch-to-cirh lowering failed — sh: 1: torch-cirh-opt: not found
# Fix: chmod +x cerebras-pytorch-src/.../pytorch/lib/torch-cirh-opt, add to PATH.

# Attempt 4: add torch-cirh-opt to PATH — compilation succeeds, then hits network
cd ${HOME}/Cerebras/gigaGPT

PATH="${HOME}/Cerebras/cerebras-pytorch-src/cerebras_pytorch-2.10.0/cerebras/pytorch/lib:$PATH"
uv run -p 3.11 python train.py configs/csx_sim_small_macos.yaml
uv run -p 3.11 python train.py configs/csx_sim_small.yaml
# grpc._channel._InactiveRpcError: Connection refused ipv4:127.0.0.1:9000
# Torch→CIRH compilation ran successfully. Blocked by: no CSX appliance server at localhost:9000.

# How do I start the CSX appliance server? I have Cerebras SDK installed at /home/weiping/Cerebras/SDK
```
