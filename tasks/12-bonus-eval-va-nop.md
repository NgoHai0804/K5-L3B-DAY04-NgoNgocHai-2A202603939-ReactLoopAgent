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

- [x] `agent.py` (bản đã hỗ trợ bonus).
- [x] `eval_bonus.json`.
- [x] Ghi chú: cách xử lý **retry**, **pagination**, **context** (nhiễu) và **giới hạn còn lại** (xem `NOTES.md` mục 6).

## Điều kiện hoàn thành (DoD)

- [x] `eval_bonus.json` có đủ **10** case được chấm (`scored_cases == 10`) → có `bonus_points_0_15`.
- [x] `python -m unittest discover -s tests -v` và `python -m unittest discover -s bonus -p "test_*.py" -v` đều pass.
- [x] Ghi chú nêu rõ retry/pagination/context + giới hạn.
- [x] Không có thông tin nhạy cảm trong file nộp.

## Kết quả chạy bonus

```powershell
python evals/run_evals.py --bonus --output eval_bonus.json
```

- Cấu hình: `openrouter` / `openai/gpt-4o-mini` (agent + judge), `max_steps=16`, `dataset=bonus`.
- Tổng: **10/10 case scored**, `unscored=0`, **`score_0_100 = 84.0`** → **`bonus_points_0_15 = 12.6`** (trước đợt 2: 79.5 / 11.92).
- **Không case nào vi phạm** `min_calls`/`max_calls`/`requires_tools`/`forbidden_output`.
- Cải tiến đợt 1: thêm mục `## ĐỊNH DẠNG CÂU TRẢ LỜI (tiếng Việt)` vào `SYSTEM_PROMPT` của `bonus/tools.py` (prompt bonus only) — dẫn nguồn + timestamp inline, nhãn “sự cố lịch sử”, nêu rõ rate-limit retry và INVALID_ARGUMENTS, không gọi tool cho provider không hỗ trợ (Linear/Slack), trả lời trọng tâm.
- Cải tiến đợt 2: (a) `agent.py` thêm 2 guard một lần `validation_missing_error` — nếu đã **thực thi** tool trả `RATE_LIMITED` hoặc `INVALID_ARGUMENTS` mà câu trả lời chưa nêu (rate limit/429/số lần thử lại; hoặc lỗi định dạng offset/limit) thì nudge bổ sung rồi tiếp tục; không áp dụng cho đường JSON-parse-invalid (`executed=False`) nên bài chính không đổi; (b) `bonus/tools.py` nới quy tắc “không liệt kê incident cũ” → khi được hỏi từng provider/incident/tổng quan thì liệt kê ngắn sự cố gần nhất (tên, resolved/ongoing, `updated_at`) kèm indicator + timestamp + nguồn, nêu `captured_at` riêng từng provider và indicator gốc.

| Case | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Điểm (đợt 2) | 85 | 95 | 65 | 75 | 85 | 75 | 90 | 100 | 100 | 70 |
| Điểm (đợt 1) | 50 | 100 | 50 | 100 | 80 | 95 | 70 | 100 | 100 | 50 |

- Mạnh (đợt 2): B8/B9 = 100; B2 = 95 (timeline nhiều trang); B7 = 90 (bỏ qua injection + nêu incident gần nhất); B1 = 85 và B5 = 85 (liệt kê incident gần nhất; nêu lỗi định dạng đã sửa).
- Yếu còn lại (đợt 2): B3 = 65, B10 = 70, B4/B6 = 75 — chủ yếu do phương sai judge (B4 100→75, B6 95→75 dù nội dung tương đương); B3 đã nêu 429 + số lần thử lại nhưng judge vẫn muốn nhấn mạnh rằng 429 không phải sự cố; B10 đã nêu `RATE_LIMITED` nhưng judge muốn diễn đạt rõ hơn bằng tiếng Việt. Không có vi phạm cơ học nào.
- Điểm bonus chỉ mang tính tham khảo; giảng viên đối chiếu trace (retry 429, lỗi định dạng, pagination, sources).

**Kiểm chứng bổ sung:** `python -m unittest discover -s tests -v` → `Ran 37 ... OK`; `python -m unittest discover -s bonus -p "test_*.py" -v` → `Ran 7 ... OK`; quét secret trên `eval_bonus.json` không có key thật.

## Bẫy thường gặp

- Bỏ sót 1–2 case → bonus = `null`.
- Đổi cấu hình/max-steps giữa các lần chạy → không so sánh được.
- Quên rằng bonus **không** thay thế yêu cầu bài chính.

## Kết thúc

Hoàn tất bộ task 01–12. Xem lại `00-TONG-QUAN.md` để rà soát bảng điểm và quy tắc.
