# Transformers GGUF dequantization regression on Apple Silicon

This lab reproduces the exact Transformers warning:

`Dequantizing the whole model, because no GGUF matmul kernel is published for this device.`

The important finding is that this warning was **not** an M1 Pro hardware limitation in our test. On the same Apple M1 Pro, same GGUF file, same Transformers commit and same `kernels` version, changing PyTorch from 2.14.0 to 2.13.0 restored the packed GGUF path.

## Environment

- Apple M1 Pro, 16 GB unified memory
- macOS 26.6.2, arm64
- Python 3.10.2
- Model: Qwen3.5-0.8B Q4_K_M
- Transformers: 5.18.0.dev0, commit 27166ea...
- kernels: 0.17.1
- llama-cpp-python: 0.3.35 with Metal GPU offload

## Result

| Runtime | PyTorch | Packed GGUF | Load | Median generation | Measured MPS allocation |
| --- | --- | --- | ---: | ---: | ---: |
| Transformers | 2.14.0 | No, full dequantization | 9.08 s | 29.24 tok/s | 2879.97 MB |
| Transformers | 2.13.0 | Yes | 8.60 s | 80.55 tok/s | 509.47 MB |
| llama.cpp binding | n/a | Native GGUF | 0.43 s | 104.95 tok/s | n/a |

Each timed generation produced 128 new tokens after a warm-up. Three different short prompts were used. The Transformers and llama.cpp measurements both include prompt processing, so this lab is **not** directly comparable to Hugging Face's published `llama-bench` decode-only table.

Moving from the incompatible PyTorch 2.14 path to 2.13 improved median Transformers throughput by about **2.75x** and reduced measured MPS allocation by about **82.3%**. With the packed path restored, llama.cpp was still about **1.30x** faster on this specific workload.

## Reproduce the compatibility check

Use a clean environment. Install a PyTorch version for which the Hugging Face GGUF kernel build is actually published, then install current Transformers and kernels:

```bash
python -m venv .venv
.venv/bin/python -m pip install "torch==2.13.0"
.venv/bin/python -m pip install accelerate kernels "git+https://github.com/huggingface/transformers.git"
```

Download the same Q4_K_M GGUF and load it through `AutoModelForCausalLM.from_pretrained(..., gguf_file=...)`.

The fastest verification is the log:

- **Bad path:** the full-dequantization warning appears.
- **Packed path:** that warning is absent. Some Qwen3.5 operators can still report separate reference-PyTorch fallbacks; those are different from dequantizing the whole GGUF.

## Practical fix

Do not treat `torch==2.13.0` as a permanent universal pin. The durable rule is:

1. check which PyTorch versions have published `ggml-quantization` kernels;
2. use one of those versions;
3. verify the dequantization warning is gone;
4. benchmark your own model and workload;
5. use llama.cpp when inference efficiency is the main objective.

## Evidence

Raw measurements are in `results/` and the tested matrix is in `version-matrix.md`.

Official background:

- Hugging Face: Transformers now runs llama.cpp quants
- Transformers GGUF quantization documentation

The repository intentionally excludes virtual environments and downloaded model weights.
