# Task 10 (Bonus) — Retry 429 và sửa lỗi định dạng

**Thời lượng gợi ý:** ~25 phút · **Phụ thuộc:** 09 · **Điểm:** nền tảng bonus (case B3, B4, B5, B10)

## Mục tiêu

Agent phải **phục hồi có kiểm soát** trước `RATE_LIMITED` (429) và `INVALID_ARGUMENTS`, biết dừng đúng lúc và báo rõ phần không kiểm tra được — **không** coi 429 là outage.

## Kiến thức từ `bonus/tools.py`

- Tool có thể trả:
  - `{'ok': False, 'error': 'RATE_LIMITED', 'http_status': 429, 'retriable': True, ...}` (429 giả lập/live; live có `retry_after` từ header `Retry-After`).
  - `{'ok': False, 'error': 'INVALID_ARGUMENTS', 'message': ...}` khi sai kiểu `offset`/`limit` hoặc truyền tham số lạ.
  - `{'ok': False, 'error': 'TimeoutError'/'URLError'/...}` khi lỗi mạng.

## Yêu cầu

1. **Tool lỗi → trả lại model làm observation**, không crash.
2. **Retry có giới hạn:** sửa định dạng khi cần; giới hạn số lần retry. Với HTTP 429 thật: đọc `Retry-After` nếu có — chờ hợp lý trong budget hoặc **dừng**, **không gọi liên tục**.
3. **Báo phần không kiểm tra được** rõ ràng khi hết retry (partial failure).

## Ràng buộc số lần gọi (rất quan trọng cho điểm)

| Case | Ràng buộc |
|---|---|
| B3 `rate_limit_once` | `get_vercel_status` min 2, max 3 (lần đầu 429, lần sau OK) |
| B4 `rate_limit_always` | `get_vercel_status` **max 3**; không bịa status Vercel; vẫn báo GitHub |
| B5 `format_once` | `get_incident_details` min 2 (lần đầu INVALID_ARGUMENTS, gọi lại đúng kiểu) |
| B10 `rate_limit_once` | `get_vercel_status` min 2, max 3 |

`mechanical_checks` sẽ **cap 50 điểm** nếu vượt `max_calls`, và cap 50 nếu thiếu `min_calls`.

## Lệnh kiểm chứng

```powershell
python -m unittest discover -s bonus -p "test_*.py" -v

# chạy riêng vài case để quan sát (cần key)
python evals/run_evals.py --bonus --case B3 --output bonus_b3.json
python evals/run_evals.py --bonus --case B4 --output bonus_b4.json
```

## Điều kiện hoàn thành (DoD)

- [x] Case B3/B4/B5/B10 không vượt `max_calls` và đạt `min_calls`.
- [x] Agent không tuyên bố Vercel outage khi nhận 429.
- [x] Khi 429 kéo dài, agent báo partial failure, không gọi vô hạn.
- [x] Khi `INVALID_ARGUMENTS`, agent gọi lại với `offset`/`limit` đúng integer.

## Trạng thái kiểm chứng

**Cơ chế đã thêm (`agent.py`):**
- **Bounded retry:** hằng `RETRIABLE_RETRY_LIMIT = 3`. Chỉ truy vấn trả `retriable=True` (HTTP 429) được thử lại **cùng tham số**; mỗi cặp `(name, args)` thực thi tối đa 3 lần. Lần vượt hạn trả `RETRY_EXHAUSTED` và **không** gọi tool nữa (không gọi liên tục).
- **Đếm call đúng:** trace chỉ ghi `event='tool'` khi tool **thực sự chạy**. Các call bị chặn (JSON sai, `DUPLICATE_CALL`, `RETRY_EXHAUSTED`) ghi `event='tool_blocked'` nên **không** bị tính vào `min_calls`/`max_calls`.
- **Retry-nudge:** nếu model định kết thúc mà còn một truy vấn retriable chưa đạt tối thiểu 2 lần thử, loop chèn một message yêu cầu thử lại (tối đa một lần cho mỗi truy vấn) — đảm bảo `min_calls` khi 429 xảy ra ở lần gọi đầu.
- **Prompt bonus (`bonus/tools.py`):** retry 429 tối đa 3 lần + honor `Retry-After`, bắt buộc nêu rõ đã gặp rate limit và đã thử lại, báo partial failure khi hết retry, sửa `INVALID_ARGUMENTS` rồi gọi lại.
- **Không đổi hành vi bài chính:** `StatusTools` không bao giờ trả `retriable` → nhánh retry/nudge không kích hoạt.

**Kết quả chạy live** (`openrouter` / `openai/gpt-4o-mini`, snapshot, `max_steps=16`):

| Case | Ràng buộc | Thực tế | Vi phạm | Ghi chú |
|---|---|---|---|---|
| B3 | vercel 2–3 | 2 (`429` → OK) | none | retry chạy đúng |
| B4 | vercel ≤ 3 | 3 (đều 429) + 1 blocked | none | dừng đúng, báo partial failure |
| B5 | details ≥ 2 | 2 (`INVALID_ARGUMENTS` → OK) | none | sửa tham số rồi gọi lại |
| B10 | vercel 2–3 | 2 (`429` → OK) | none | retry-nudge kích hoạt ở bước 3 |

- Điểm judge tham khảo dao động (B3 ~55, B4 ~75, B5 ~75, B10 ~50) do nội dung câu trả lời/chất lượng judge, **không** phải lỗi cơ chế retry.
- Một lần chạy B4 đầu tiên bị `IncompleteRead` (lỗi mạng của provider) → `unscored`; đã chạy lại và đạt OK. Lỗi hạ tầng không tính là lỗi agent.

**Tests offline:** `python -m unittest discover -s tests -v` → `Ran 37 ... OK`; `python -m unittest discover -s bonus -p "test_*.py" -v` → `Ran 6 ... OK` (thêm `test_retriable_retry_is_bounded`, `test_blocked_calls_do_not_count_as_tool_calls`, `test_rate_limit_final_is_nudged_to_retry`).

## Bẫy thường gặp

- Retry vô hạn → vượt `max_calls`, hết bước, hoặc tốn token.
- Coi 429 là "dịch vụ đang lỗi".
- Không sửa kiểu tham số (gửi lại string `"one"` thay vì int).

## Bước tiếp theo

→ `11-bonus-pagination-va-context.md`
