# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** Nguyễn Duy Hưng

**Cohort:** A20

**Ngày submit:** 2026-05-06

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

- **OS:** Windows 10
- **CPU:** Intel Core i5-10500H
- **Cores:** 6 physical / 12 logical
- **CPU extensions:** AVX2
- **RAM:** 15.8 GB
- **Accelerator:** NVIDIA GeForce GTX 1650 (4 GB VRAM)
- **llama.cpp backend đã chọn:** Vulkan / CPU
- **Recommended model tier:** Qwen2.5-1.5B

**Setup story** (≤ 80 chữ): Do hạn chế của script trên Windows, tôi đã phải cài đặt cấu hình thủ công trong môi trường ảo (.venv). Lựa chọn chạy bản `llama-server.exe` dạng Native thay vì bọc qua thư viện Python để có thể dùng GPU (Vulkan) offload và đồng thời lấy được số liệu /metrics từ Prometheus (mà bản Python trên Windows đang thiếu).

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|--:|--:|--:|--:|--:|
| (Q4_K_M) | 1192 | 117 / 136 | 36.8 / 39.2 | 2402 / 2527 / 2528 | 27.2 |
| (Q2_K)   | 412 | 174 / 202 | 27.9 / 29.8 | 1949 / 2034 / 2068 | 35.8 |

**Một quan sát** (≤ 50 chữ): Bản Q2_K có tốc độ load nhanh gấp 3 lần và Decode rate cao hơn hẳn (35.8 so với 27.2 tok/s) nhờ tiêu tốn ít memory bandwidth hơn. Đánh đổi quality lấy Q2_K là hoàn toàn xứng đáng với cấu hình máy bị giới hạn (4GB VRAM).

---

## 3. Track 02 — llama-server load test

| Concurrency | Total RPS | E2E P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.20 | 34000 | 47000 | 47000 | 0 |
| 50 | 0.15 | 20000 | 54000 | 54000 | 0 |

**KV-cache observation** (từ `record-metrics.py`): peak `llamacpp:kv_cache_usage_ratio` ở concurrency 50 = 0.0168457 (khoảng ~1.68%), nghĩa là số lượng token mà hệ thống quản lý trong Context Size hiện tại vẫn còn rất dư dả, chưa hề chạm tới mức phải swap (RAM/Disk) hay gây ra bottleneck bộ nhớ.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: localhost only
- **N17 (Data pipeline):** stub: in-memory dict
- **N18 (Lakehouse):** stub: SQLite / local
- **N19 (Vector + Feature Store):** stub: TOY_DOCS

**Nơi tốn nhiều ms nhất** trong pipeline (đo bằng `time.perf_counter` trong `pipeline.py`):

- embed: 0 ms (stub)
- retrieve: ~0.1 ms
- llama-server: ~8251.1 ms

**Reflection** (≤ 60 chữ): Rõ ràng LLM Inference chiếm gần như 100% (bottleneck) thời gian xử lý toàn hệ thống. Điều này hoàn toàn khớp với dự kiến vì các tác vụ retrieve hiện tại chỉ chạy bằng từ khóa trong bộ nhớ tạm (TOY_DOCS).

---

## 5. Bonus — The single change that mattered most

**Change:** Sử dụng GPU Offloading với backend Vulkan (thông số `-ngl 99`) thay vì chỉ chạy bằng CPU.

**Before vs after** (từ bảng benchmark thread sweep `llama-bench.exe`):

```
before (chạy 1 thread CPU): 45.35 tok/s
after (chạy 12 thread CPU): 45.46 tok/s
speedup: 1.0x (Tốc độ không phụ thuộc CPU nữa)
```

**Tại sao nó work**:
Khi offload toàn bộ 99 layers lên GPU (NVIDIA GTX 1650 4GB) bằng Vulkan, quá trình tính toán ma trận và memory bandwidth đều do VRAM gánh 100%. Nhờ vậy, việc thay đổi số lượng luồng (threads) của CPU không còn tạo ra bất kỳ sự khác biệt nào về tốc độ nữa (luôn ổn định ở mức ~45.5 tok/s). 
Thay đổi này quan trọng nhất vì nó chứng minh tôi đã loại bỏ hoàn toàn CPU bottleneck (nút thắt cổ chai) ra khỏi pipeline, ép sức mạnh Inference dồn toàn bộ vào GPU, tận dụng triệt để kiến trúc phần cứng hiện tại.

---

## 6. (Optional) Điều ngạc nhiên nhất

Sự chênh lệch gần như không có của các mức độ threads khi đã offload sang GPU cho thấy trong Model Serving, Memory Bandwidth của phần cứng xử lý đóng vai trò sống còn hơn là chỉ phụ thuộc vào số lượng nhân xử lý vật lý.

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit (Dùng ảnh minh chứng 03-server-running.png)
- [x] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep) (Dùng ảnh minh chứng 06-bonus-sweep.png)
- [x] Ít nhất 6 screenshots trong `submission/screenshots/`
- [x] `make verify` exit 0
- [x] Repo trên GitHub ở chế độ **public**
- [x] Đã paste public repo URL vào VinUni LMS
