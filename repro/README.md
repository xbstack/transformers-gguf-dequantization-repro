# Reproduction

Test target:

- Apple Silicon
- Qwen3.5-0.8B Q4_K_M GGUF
- Transformers main + kernels 0.17.1
- compare a PyTorch build without a matching published GGUF kernel against one with a matching build

## Fallback environment

In the XBSTACK test, PyTorch 2.14.0 produced:

```text
Dequantizing the whole model, because no GGUF matmul kernel is published for this device.
It will need several times the memory the file does, and load more slowly.
```

## Known-good control on 2026-09-26

```bash
python -m venv .venv
.venv/bin/python -m pip install "torch==2.13.0"
.venv/bin/python -m pip install accelerate kernels "git+https://github.com/huggingface/transformers.git"
```

Load the same GGUF with `AutoModelForCausalLM.from_pretrained(..., gguf_file=...)`.

Success criterion: the full-dequantization warning is absent. Some model-specific operator fallbacks may still occur and are a separate optimization boundary.

Do not treat PyTorch 2.13.0 as a permanent pin. Re-check the currently published Hugging Face GGUF kernel builds when reproducing this later.
