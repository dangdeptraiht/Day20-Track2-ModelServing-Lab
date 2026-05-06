# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** _Dang Nguyen_
**Cohort:** _A20-K1_
**Ngày submit:** _2026-05-06_

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

> Paste output của `python 00-setup/detect-hardware.py` vào đây, hoặc điền thủ công:

- **OS:** macOS 15.0 (Apple Silicon)
- **CPU:** Apple M3 Pro
- **Cores:** 11 physical / 11 logical
- **CPU extensions:** NEON
- **RAM:** 18.0 GB
- **Accelerator:** Apple Metal
- **llama.cpp backend đã chọn:** Metal
- **Recommended model tier:** Llama-3.2-3B-Instruct (Q4_K_M)

**Setup story** (≤ 80 chữ): The `Llama-3.2-3B-Instruct-Q2_K.gguf` model was missing from the Hugging Face repository, causing a 404 error during `make setup`. I updated the download script to use the `Q3_K_L` quantization instead. Also, `uvicorn` and other server dependencies were missing from the initial installation, so I had to install `llama-cpp-python[server]` to enable the OpenAI-compatible API.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

> Paste bảng từ `benchmarks/01-quickstart-results.md` xuống đây (auto-generated bởi `python 01-llama-cpp-quickstart/benchmark.py`).

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|--:|--:|--:|--:|--:|
| Llama-3.2-3B-Instruct-Q4_K_M.gguf | 9132 | 63 / 156 | 19.1 / 19.6 | 1281 / 1333 / 1354 | 52.2 |
| Llama-3.2-3B-Instruct-Q3_K_L.gguf | 1042 | 68 / 142 | 23.2 / 24.1 | 1532 / 1629 / 1648 | 43.1 |

**Một quan sát** (≤ 50 chữ): Surprisingly, the larger Q4_K_M quantization showed a higher decode rate (52.2 tok/s) compared to Q3_K_L (43.1 tok/s) on this M3 Pro. This might be due to Metal kernels being more optimized for 4-bit quantizations on Apple Silicon. Q4 is the "sweet spot" for this hardware.

---

## 3. Track 02 — llama-server load test

> Chạy 2 lần locust ở concurrency 10 và 50, paste tóm tắt bên dưới.

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.60 | 14000 | 18000 | 19000 | 0 |
| 50 | 0.63 | 20000 | 35000 | 36000 | 0 |

**KV-cache observation**: peak `llamacpp:kv_cache_usage_ratio` ở concurrency 50 = _0.85_, nghĩa là hệ thống đang tận dụng tối đa cache để xử lý batching, nhưng latency tăng cao do contention trên GPU.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: localhost only
- **N17 (Data pipeline):** stub: in-memory dict
- **N18 (Lakehouse):** stub: SQLite
- **N19 (Vector + Feature Store):** stub: TOY_DOCS

**Nơi tốn nhiều ms nhất** trong pipeline:

- embed: 0.0 ms (stub)
- retrieve: 0.0 ms (stub)
- llama-server: 1200.0 ms

**Reflection** (≤ 60 chữ): Bottleneck rõ ràng nằm ở llama-server (phần inference). Điều này đúng với kỳ vọng vì retrieval hiện tại chỉ là stub trên RAM, trong khi LLM inference là tác vụ nặng nhất yêu cầu tính toán GPU liên tục.

---

## 5. Bonus — The single change that mattered most

> **Most important section.** Pick **một** thay đổi từ bonus track (build flag, thread sweep, quant pick, GPU offload, KV-cache quantization, speculative decoding, bất cứ challenge nào trong `BONUS-llama-cpp-optimization/CHALLENGES.md`) đã tạo ra speedup lớn nhất trên máy bạn.

**Change:** Bật Metal acceleration (GPU offload) trên chip M3 Pro.

**Before vs after**:

```
before: 5.2 tok/s (CPU only)
after:  52.2 tok/s (Metal enabled)
speedup: ~10.0×
```

**Tại sao nó work**:
Việc chuyển từ CPU sang Metal (GPU) giúp tận dụng băng thông bộ nhớ cực lớn của Unified Memory trên chip Apple Silicon. LLM inference là tác vụ memory-bandwidth bound; GPU của M3 Pro có khả năng đọc các weight của model nhanh hơn nhiều lần so với CPU truyền thống, giúp giảm TPOT đáng kể.

---

## 6. (Optional) Điều ngạc nhiên nhất

_(1–2 câu — không bắt buộc, nhưng người grader đọc tất cả)_

_Answer here._

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit (hoặc paste path snapshot vào section 1)
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [x] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [ ] Ít nhất 6 screenshots trong `submission/screenshots/` (xem `submission/screenshots/README.md`)
- [x] `make verify` exit 0 (chạy ngay trước khi push)
- [ ] Repo trên GitHub ở chế độ **public**
- [ ] Đã paste public repo URL vào VinUni LMS

---

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Nếu private, grader không xem được → 0 điểm.
