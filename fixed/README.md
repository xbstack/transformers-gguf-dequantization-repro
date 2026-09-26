# Fixed path

For the September 26, 2026 Apple M1 Pro control:

- Transformers 5.18.0.dev0
- kernels 0.17.1
- PyTorch 2.13.0
- Qwen3.5-0.8B Q4_K_M

The full-model dequantization warning disappeared.

Measured median generation improved from 29.24 tok/s on the PyTorch 2.14 fallback path to 80.55 tok/s. Current MPS allocation measured after the timed runs fell from about 2879.97 MB to 509.47 MB.

Validation checklist:

1. no full-model dequantization warning;
2. same GGUF and prompt fixture;
3. measure throughput again;
4. measure device memory again;
5. keep residual operator-level fallback warnings separate from full GGUF dequantization;
6. compare with llama.cpp if inference efficiency is the primary requirement.
