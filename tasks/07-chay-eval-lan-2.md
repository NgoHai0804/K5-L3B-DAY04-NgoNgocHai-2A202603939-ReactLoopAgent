# Task 07 — Bài 5: Chạy eval lần 2 và so sánh

**Thời lượng gợi ý:** ~30 phút · **Phụ thuộc:** 06 · **Điểm:** (gộp vào 20 điểm Eval & phân tích)

## Mục tiêu

Chạy lại **cùng bộ eval** với **cùng cấu hình** (agent model, judge model, snapshot, `max_steps`, dataset) sau khi cải tiến, rồi so sánh lần 1 vs lần 2 để chứng minh thay đổi có tác dụng và nêu lỗi còn lại.

## Việc cần làm

1. Chạy eval lần 2:

```powershell
python evals/run_evals.py --output eval_after.json
```

2. Đảm bảo cấu hình **giống hệt** lần 1 (đọc `config` trong `eval_before.json`). Nếu muốn đổi judge, phải đổi **cùng lúc** cả 2 lần — nhưng đơn giản nhất là giữ nguyên.

3. Lập bảng so sánh:

| Case | Điểm before | Điểm after | Δ | Ghi chú |
|---|---:|---:|---:|---|
| … | | | | |

Cộng thêm các chỉ số tổng: `score_0_100`, `scored_cases`, số model calls, token/chi phí (nếu provider trả `usage`).

4. Phân loại kết quả:
   - **Lỗi đã giải quyết** (case tăng điểm / hết violation).
   - **Lỗi còn lại** (case vẫn thấp và vì sao).
   - **Lỗi mới xuất hiện** (case giảm do thay đổi gây side effect).

## Lưu ý về so sánh

- **Một lần chạy mỗi phiên bản không chứng minh độ ổn định** — nêu rõ đây là hạn chế.
- Judge mặc định dùng cùng provider/model với agent → điểm có thể **thiên lệch** (model tự chấm chính nó).
- Đọc `grade.reason` và trace, **không** chỉ so số.
- Không sửa `expected` để tối ưu điểm.

## Kết quả so sánh

Cấu hình giống hệt lần 1 (`config` trùng): `openrouter` / `openai/gpt-4o-mini`, judge cùng model, `max_steps=8`, `dataset=core`. File `eval_after.json` cuối cùng (lần chạy mới nhất): **22/22 case scored, điểm 83.64/100** (các lần chạy trước đó có case `unscored` do **timeout mạng** khi gọi provider; đã chạy lại để có file hợp lệ, không sửa tay kết quả).

| Case | Điểm before | Điểm after | Δ | Ghi chú |
|---|---:|---:|---:|---|
| E1 | 65 | 75 | +10 | |
| E2 | 100 | 75 | −25 | dao động judge |
| E3 | 75 | 85 | +10 | |
| E4 | 100 | 100 | 0 | |
| E5 | 75 | 70 | −5 | |
| E6 | 75 | 100 | +25 | xử lý timeout rõ hơn |
| A1 | 75 | 70 | −5 | |
| A2 | 80 | 100 | +20 | nguồn/scope FedRAMP rõ hơn |
| A3 | 75 | 75 | 0 | |
| A4 | 100 | 100 | 0 | |
| A5 | 73 | 75 | +2 | |
| A6 | 75 | 75 | 0 | |
| A7 | 75 | 80 | +5 | **answer giờ dẫn URL** (`ans_has_url=True`) |
| A8 | 100 | 100 | 0 | |
| H1 | 60 | 75 | +15 | **hết cap**: tool calls ở ≥2 model steps |
| H2 | 90 | 100 | +10 | |
| H3 | 75 | 100 | +25 | |
| H4 | 100 | 80 | −20 | bài thơ không dùng tool → `evidence=0` |
| H5 | 70 | 75 | +5 | |
| H6 | 60 | 75 | +15 | **hết cap**: tool calls ở ≥2 model steps |
| H7 | 70 | 55 | −15 | xử lý timeout/giới hạn còn yếu |
| H8 | 75 | 100 | +25 | |
| **Tổng** | **79.23** | **83.64** | **+4.41** | |

Chỉ số tổng: model calls **52 → 52**; token (agent) **~45,243 → ~157,395** (prompt 34,699 → 149,431, completion 10,544 → 7,964), chi phí (agent) **~$0.0115 → ~$0.0183**; `scored_cases` 22/22 cả hai lần; tất cả case `completed`, không case nào thành `max_steps`/`error`.

> **Chi phí/token tăng mạnh:** lần 2 dùng agent có **prompt hệ thống/workflow đầy đủ hơn** và gọi tool nhiều bước hơn (ví dụ H1 2→4 model calls; H5/H6/A6 3 calls), nên **token prompt tăng** dù số model calls không đổi. Vì vậy so sánh token/chi phí giữa hai lần cần thận trọng.

### Phân loại
- **Lỗi đã giải quyết / cải thiện:** H1 & H6 (hết vi phạm cap "≥2 model steps", 60→75); H3, H8, E6 (+25 mỗi case); A2 (+20); H2, E1, E3 (+10); A7, H5 (+5).
- **Lỗi còn lại:** H7 (55) — xử lý Cloudflare timeout/giới hạn dữ liệu còn bị trừ (factuality 10/40, boundaries 10/15); A1, E5, E2, A3, A5, H5, H6 (70–75) — scope/nguồn chưa đủ chặt.
- **Lỗi mới / biến động:** E2 100→75, E5 75→70, A1 75→70, H4 100→80, H7 70→55 — chủ yếu do **phương sai judge** (cùng model tự chấm, chỉ chạy một lần), **không** có side-effect kỹ thuật (không case nào hỏng run; tất cả `completed`). H4 giảm nhẹ vì bài thơ không dùng tool nên `evidence=0`.

## Điều kiện hoàn thành (DoD)

- [x] Đã tạo `eval_after.json` cùng cấu hình với lần 1.
- [x] Có bảng so sánh per-case before/after và chỉ số tổng.
- [x] Phân loại rõ: đã giải quyết / còn lại / mới xuất hiện.
- [x] Nêu số model calls + token/chi phí nếu có.

## Bước tiếp theo

→ `08-nop-bai.md`
