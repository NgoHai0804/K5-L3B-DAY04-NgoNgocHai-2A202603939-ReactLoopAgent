# Task 08 — Đóng gói và nộp bài chính

**Thời lượng gợi ý:** ~15 phút · **Phụ thuộc:** 07 · **Điểm:** — (bắt buộc để được chấm)

## Mục tiêu

Chuẩn bị đúng bộ hồ sơ nộp bài chính (tối đa 85 điểm) và ghi chú phân tích, đảm bảo đã xóa thông tin nhạy cảm.

## Hồ sơ nộp (theo GUIDE mục "Nộp bài" và `SUBMISSION.md`)

- [ ] `agent.py` — bản đã hoàn thiện và pass tests.
- [ ] `eval_before.json` — kết quả lần 1 (đủ 22 case, điểm khác `null`).
- [ ] `eval_after.json` — kết quả lần 2.
- [ ] `NOTES.md` — ghi chú phân tích đủ 5 mục:
  - Mục 1–2: case được chọn (ID) + evidence trong trace.
  - Mục 3: thay đổi đã implement.
  - Mục 4: so sánh before/after (điểm, số model calls, token/chi phí).
  - Mục 5: giới hạn (judge, snapshot, một lần chạy, kiểm tra URL chỉ là hiện diện...).
- [ ] Demo: **1 trace nhiều bước** + trình bày **1 lỗi agent**.
- [ ] Checklist theo `SUBMISSION.md`: tên repo đúng, không có `.env`/key, hai file eval cùng cấu hình.

## Kiểm tra an toàn trước khi nộp

```powershell
# xem nhanh file có chứa chuỗi nghi ngờ key không (điều chỉnh theo biến key của bạn)
Select-String -Path agent.py, eval_before.json, eval_after.json -Pattern "sk-", "AIza", "api[_-]?key", "Bearer"
```

- [ ] Không có API key thật trong bất kỳ file nào.
- [ ] Xóa `.env` nếu lỡ tạo (không commit).
- [ ] Không có trace chứa dữ liệu nhạy cảm.

## Checklist chất lượng theo rubric

| Tiêu chí | Tự kiểm |
|---|---|
| Agent loop (30) | gọi model+tool, append assistant nguyên vẹn, đúng tool-call ID, nhiều calls, dừng đúng |
| Robustness/trace (15) | `max_steps`, invalid JSON, tool error, `length`; trace debug được, không key |
| Eval & phân tích (20) | 2 lần chạy cùng cấu hình; đọc case + trace; evidence; giới hạn judge |
| Cải tiến & trình bày (20) | cải tiến hợp lý; validation nguồn; tests pass; so sánh; lỗi còn lại |

## Điều kiện hoàn thành (DoD)

- [ ] Đủ 3 file artifact (`agent.py`, `eval_before.json`, `eval_after.json`) + `NOTES.md` + demo.
- [ ] `python -m unittest discover -s tests -v` pass toàn bộ ở trạng thái nộp.
- [ ] Không có thông tin nhạy cảm trong file nộp.
- [ ] `NOTES.md` điền đủ 5 mục.

## Bước tiếp theo

→ (tùy chọn) `09-bonus-ket-noi-tools.md` — chỉ làm sau khi bài chính đã xong.
