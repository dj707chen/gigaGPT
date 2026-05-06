# Local Cerebras PyTorch Dependency Notes

## Timing

- Work window: 2026-05-06 10:31-10:38 CDT
- Environment: macOS arm64, `uv 0.11.9`, Python 3.11.15 for the macOS sync

## What Changed

- Updated `pyproject.toml` so `gigaGPT` resolves `cerebras-pytorch==2.10.0` from `../cerebras-pytorch-src/cerebras_pytorch-2.10.0` on macOS arm64 with Python 3.11 or newer.
- Kept the original `cerebras_pytorch==2.5.0` dependency for Linux x86_64 with Python 3.8.
- Regenerated `uv.lock` with both resolution paths.
- Updated `requirements.txt` as a simple macOS fallback that installs the local Cerebras PyTorch source in editable mode.
- Updated `README.md` to show the macOS arm64 `uv sync --python 3.11` setup path.

## Verification

- `uv lock` completed successfully and recorded the local editable source.
- `uv build` completed successfully in `../cerebras-pytorch-src/cerebras_pytorch-2.10.0`.
- `uv sync --python 3.11` completed successfully on macOS arm64 and installed `cerebras-pytorch==2.10.0` from the local path.
- `uv lock --check` passed after regenerating the lockfile.
- `uv run python` confirmed the active platform is Darwin arm64 and the installed `cerebras-pytorch` package version is `2.10.0`.

## Remaining Runtime Blocker

Importing `cerebras.pytorch` still fails because the local package expects `cerebras.appliance`, but `cerebras-appliance==2.10.0` only has a `manylinux2014_x86_64` wheel available. `uv pip install 'cerebras-appliance==2.10.0' --dry-run` reports that there is no compatible macOS arm64 wheel.

So the dependency resolution now points at the local copy as requested, but running the Cerebras-backed code on macOS arm64 still needs a compatible `cerebras-appliance` source/package or code changes that avoid the Cerebras runtime imports for CPU-only workflows.
