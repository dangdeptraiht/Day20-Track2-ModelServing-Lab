# Bonus — GPU-offload sweep

Model: `Llama-3.2-3B-Instruct-Q4_K_M.gguf`  ·  threads: `11`

| -ngl | tg128 (tok/s) |
|--:|--:|
| 0 | 0.0 |
| 8 | 0.0 |
| 16 | 0.0 |
| 24 | 0.0 |
| 32 | 0.0 |
| 99 | 0.0 |

When the model fits in VRAM, `-ngl 99` (full offload) is fastest. When it doesn't, partial offload (`-ngl 16` or `-ngl 24`) keeps the most compute on the GPU while spilling weights to RAM — usually still beats CPU-only (`-ngl 0`). Watch for the curve flattening: after the layer count covers the model's actual depth, more `-ngl` does nothing.
