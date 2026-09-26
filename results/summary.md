# Result summary

The same Qwen3.5-0.8B Q4_K_M GGUF was tested on an Apple M1 Pro with 16 GB unified memory.

- Transformers + PyTorch 2.14.0 fell back to full dequantization. Median generation throughput: **29.24 tok/s**. MPS current allocation after the run: **2879.97 MB**.
- Transformers + PyTorch 2.13.0 kept the packed GGUF path. Median generation throughput: **80.55 tok/s**. MPS current allocation after the run: **509.47 MB**.
- llama.cpp through llama-cpp-python 0.3.35 with Metal GPU offload reached **104.95 tok/s** median.
- Moving from the incompatible Torch 2.14 path to Torch 2.13 improved the Transformers median by **2.75x** and cut measured MPS allocation by about **82.3%**.
- In the supported Torch 2.13 configuration, llama.cpp was still about **1.30x** faster on this specific 0.8B, 128-token, end-to-end workload.

These numbers are device- and workload-specific. They are not a universal runtime ranking. The experiment exists to verify the exact dequantization warning and its practical effect.
