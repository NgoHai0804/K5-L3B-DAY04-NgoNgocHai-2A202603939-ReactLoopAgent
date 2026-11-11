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

## Điều kiện hoàn thành (DoD)

- [ ] Đã tạo `eval_after.json` cùng cấu hình với lần 1.
- [ ] Có bảng so sánh per-case before/after và chỉ số tổng.
- [ ] Phân loại rõ: đã giải quyết / còn lại / mới xuất hiện.
- [ ] Nêu số model calls + token/chi phí nếu có.

## Bước tiếp theo

→ `08-nop-bai.md`
