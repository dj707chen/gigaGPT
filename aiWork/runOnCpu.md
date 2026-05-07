# Run gigaGPT on CPU (macOS ARM64)

## User Ask

When I ran command:
```
GENERATING_CEREBRAS_PYTORCH_DOCS=1 uv run -p 3.12 python train.py
```
I got error message:
```
train.py: error: the following arguments are required: config_file
```
I don't have a config_file, can you create a sample / default one?

## Response

`train.py` requires a YAML config file as a positional argument. It's parsed by `HfArgumentParser` in `configuration.py` and populates three dataclasses: `GPTConfig` (model params), `RunConfig` (training params), and `ClusterConfig`.

Existing configs in `configs/` (111m, 1.3b, 13b, 70b, 175b) are all sized for CSX clusters. None are suitable for local CPU testing.

### Created: `configs/cpu_small.yaml`

A tiny ~1.6M parameter model for local CPU testing:

```yaml
# model params
width: 128
depth: 4
max_position_embeddings: 256
heads: 4
vocab_size: 50257
dropout: 0.0
bias: true

# run params
out_dir: out/cpu_small
dataset: data/shakespeare_char
batch_size: 4
sequence_length: 256
num_steps: 100
checkpoint_steps: 50
learning_rate: 3.0e-4
warmup_steps: 10
backend: "CPU"
seed: 42
```

### How to Run

```bash
# Step 1: Prepare the Shakespeare dataset
GENERATING_CEREBRAS_PYTORCH_DOCS=1 uv run -p 3.12 python data/shakespeare_char/prepare.py

# Step 2: Train
GENERATING_CEREBRAS_PYTORCH_DOCS=1 uv run -p 3.12 python train.py configs/cpu_small.yaml
```

### Notes

- The config uses `backend: "CPU"` instead of `"CSX"` since there's no Cerebras hardware on macOS.
- The `CPU` backend will require the `backends.py` import path to work, which is currently skipped with the `GENERATING_CEREBRAS_PYTORCH_DOCS=1` env var. If that causes an error at runtime, the stubs may need to be expanded.
- The Shakespeare dataset needs to be prepared first via `prepare.py` which downloads and tokenizes the text into a `train.bin` file.
