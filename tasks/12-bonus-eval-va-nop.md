# Task 12 (Bonus) — Chạy 10 case bonus và nộp

**Thời lượng gợi ý:** ~30 phút · **Phụ thuộc:** 11 · **Điểm:** +15 (nếu chấm đủ 10 case)

## Mục tiêu

Chạy đủ **10 case bonus** (B1–B10), tạo `eval_bonus.json`, và chuẩn bị ghi chú nộp bonus.

## Bộ bonus gồm gì (10 case, 5 API)

| Case | Kỹ năng |
|---|---|
| B1 | Tổng hợp 5 nhà cung cấp + timestamp riêng |
| B2 | Đọc nhiều trang timeline Vercel, tìm update nhắc GitHub |
| B3 | Phục hồi sau 429 giả lập một lần |
| B4 | Dừng retry khi 429 kéo dài, báo kết quả từng phần |
| B5 | Sửa lỗi định dạng và gọi lại |
| B6 | Lọc 160 mục nhiễu, so sánh Vercel và npm |
| B7 | Bỏ qua chỉ dẫn giả trong tool result |
| B8 | Đối chiếu phạm vi ảnh hưởng hai incident |
| B9 | Không bịa trạng thái/incident của provider không hỗ trợ (Linear, Slack) |
| B10 | Điều tra 5 API trong cùng yêu cầu, có 429 giả lập |

## Lệnh chạy

```powershell
# chạy full 10 case -> điểm bonus
python evals/run_evals.py --bonus --output eval_bonus.json
```

- `--max-steps` mặc định **16** khi chạy bonus; giữ nguyên giữa các lần so sánh.
- Chạy một case đơn lẻ (`--case B3`) **không** phải điểm toàn bộ bonus.

## Quy tắc điểm bonus

- Mỗi case chấm 0–100 theo **cùng rubric**; `bonus_points_0_15 = round(score_0_100 × 0.15, 2)`.
- **Chỉ** quy đổi khi **cả 10 case đã được chấm**. Nếu có case lỗi/`unscored` hoặc chỉ chạy 1 case → `bonus_points_0_15 = null`.
- Judge chỉ tham khảo; giảng viên đối chiếu trace (retry 429, lỗi định dạng, pagination, sources).

## Hồ sơ nộp bonus

- [ ] `agent.py` (bản đã hỗ trợ bonus).
- [ ] `eval_bonus.json`.
- [ ] Ghi chú: cách xử lý **retry**, **pagination**, **context** (nhiễu) và **giới hạn còn lại**.

## Điều kiện hoàn thành (DoD)

- [ ] `eval_bonus.json` có đủ **10** case được chấm (`scored_cases == 10`) → có `bonus_points_0_15`.
- [ ] `python -m unittest discover -s tests -v` và `python -m unittest discover -s bonus -p "test_*.py" -v` đều pass.
- [ ] Ghi chú nêu rõ retry/pagination/context + giới hạn.
- [ ] Không có thông tin nhạy cảm trong file nộp.

## Bẫy thường gặp

- Bỏ sót 1–2 case → bonus = `null`.
- Đổi cấu hình/max-steps giữa các lần chạy → không so sánh được.
- Quên rằng bonus **không** thay thế yêu cầu bài chính.

## Kết thúc

Hoàn tất bộ task 01–12. Xem lại `00-TONG-QUAN.md` để rà soát bảng điểm và quy tắc.
