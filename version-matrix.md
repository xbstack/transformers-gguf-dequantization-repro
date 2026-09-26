# Version matrix

| Path | PyTorch | Transformers / runtime | kernels | Packed GGUF | Median tok/s | MPS allocated |
| --- | --- | --- | --- | --- | ---: | ---: |
| Transformers | 2.14.0 | 5.18.0.dev0 | 0.17.1 | No: full dequantization fallback | 29.24 | 2879.97 MB |
| Transformers | 2.13.0 | 5.18.0.dev0 | 0.17.1 | Yes | 80.55 | 509.47 MB |
| llama.cpp binding | n/a | llama-cpp-python 0.3.35 | n/a | Native GGUF | 104.95 | n/a |

Hardware: Apple M1 Pro, 16 GB unified memory, macOS 26.6.2, arm64.
Model: Qwen3.5-0.8B Q4_K_M. Each timed run generated 128 tokens after warm-up. Three prompts were used to avoid identical-prompt cache reuse.
