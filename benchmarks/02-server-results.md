# 02 — Server Load Test Results

Summary of Locust load tests against `llama-server` (Vulkan backend).

## Load Test Parameters
- **Duration**: 60s per run
- **Model**: Qwen2.5-1.5B-Instruct (Q4_K_M)
- **Engine**: llama-server (native)

## Results

| Concurrency | Total RPS | E2E P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|------------:|----------:|-------------:|-------------:|-------------:|---------:|
| 10          | 0.20      | 34000        | 47000        | 47000        | 0        |
| 50          | 0.15      | 20000        | 54000        | 54000        | 0        |

## KV-Cache Observation
- **Peak Usage Ratio**: 0.0168457 (~1.68%)
- **Observation**: Memory was not a bottleneck during the test.
