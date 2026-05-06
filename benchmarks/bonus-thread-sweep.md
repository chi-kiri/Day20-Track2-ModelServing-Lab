# 06 — Bonus: Thread Sweep Results

Model: `qwen2.5-1.5b-instruct-q4_k_m.gguf`
Backend: `Vulkan (GTX 1650)`

| threads | tg64 (tok/s) |
|---:|---:|
| 1 | 45.35 |
| 2 | 45.62 |
| 4 | 45.48 |
| 6 | 45.38 |
| 8 | 45.33 |
| 12 | 45.46 |

**Best**: `-t 2` at 45.6 tok/s (statistical noise).

**Observation**: Since the model is 100% offloaded to the GPU via Vulkan, the CPU thread count has negligible impact on the performance.
