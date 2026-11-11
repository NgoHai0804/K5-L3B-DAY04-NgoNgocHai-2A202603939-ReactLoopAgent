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

- [ ] Case B3/B4/B5/B10 không vượt `max_calls` và đạt `min_calls`.
- [ ] Agent không tuyên bố Vercel outage khi nhận 429.
- [ ] Khi 429 kéo dài, agent báo partial failure, không gọi vô hạn.
- [ ] Khi `INVALID_ARGUMENTS`, agent gọi lại với `offset`/`limit` đúng integer.

## Bẫy thường gặp

- Retry vô hạn → vượt `max_calls`, hết bước, hoặc tốn token.
- Coi 429 là "dịch vụ đang lỗi".
- Không sửa kiểu tham số (gửi lại string `"one"` thay vì int).

## Bước tiếp theo

→ `11-bonus-pagination-va-context.md`
