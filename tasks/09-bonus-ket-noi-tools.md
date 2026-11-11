# Task 09 (Bonus) — Kết nối 5 status tools

**Thời lượng gợi ý:** ~20 phút · **Phụ thuộc:** 08 · **Điểm:** nền tảng cho 15 điểm bonus

> Chỉ làm **sau khi bài chính (01–08) hoàn tất**. Bonus không thay thế yêu cầu bài chính.

## Mục tiêu

Mở rộng agent để dùng bộ tools bonus (`bonus/tools.py`) với **5 public status API** (GitHub, Cloudflare, Vercel, npm, DigitalOcean) + `get_incident_details` có phân trang, mà **không phá** bài chính.

## Việc cần làm

1. Sửa `agent.py` để **không hard-code** schemas/system prompt của bài chính, mà ưu tiên thuộc tính của `tools` nếu tồn tại:

```python
schemas = getattr(tools, 'schemas', SCHEMAS)
system_prompt = getattr(tools, 'system_prompt', SYSTEM_PROMPT)
```

2. Đảm bảo vẫn **tương thích** với `StatusTools` (bài chính) — `StatusTools` không có thuộc tính `schemas`/`system_prompt` nên phải fallback về `SCHEMAS`/`SYSTEM_PROMPT`.
3. **Không** hard-code câu trả lời hay ID incident trong agent.

## Bộ tools bonus gồm gì

- 5 status tools: `get_github_status`, `get_cloudflare_status`, `get_vercel_status`, `get_npm_status`, `get_digitalocean_status` (tham số tùy chọn `component`).
- `get_incident_details(provider, incident_id, offset=0, limit=2)` — phân trang: `offset` integer ≥ 0, `limit` integer 1–3; trả `next_offset`.

## Lệnh kiểm chứng

```powershell
# Test bonus offline (chỉ chạy được sau khi agent đã nhận tools bonus)
python -m unittest discover -s bonus -p "test_*.py" -v

# Xác nhận bài chính không bị vỡ
python -m unittest discover -s tests -v

# Chạy một câu hỏi với snapshot (cần LLM key)
python run.py --bonus --max-steps 16 "Kiểm tra status GitHub, Vercel và npm."
```

## Điều kiện hoàn thành (DoD)

- [x] `python -m unittest discover -s tests -v` vẫn pass (bài chính không vỡ) → `Ran 37 tests ... OK`.
- [x] `python -m unittest discover -s bonus -p "test_*.py" -v` pass (không cần key) → `Ran 3 tests ... OK`.
- [x] `run.py --bonus ...` dùng đúng 6 schemas bonus (5 status + `get_incident_details`) — xác nhận qua `BonusTools.schemas` gồm 6 phần tử; bước chạy live cần LLM key.
- [ ] Agent không hard-code đáp án/ID.

## Trạng thái kiểm chứng

**Plumbing (đã có sẵn, đúng yêu cầu):** `agent.py` dùng `getattr(tools, 'schemas', SCHEMAS)` và `getattr(tools, 'system_prompt', SYSTEM_PROMPT)`; `BonusTools` có `schemas`/`system_prompt`, còn `StatusTools` không có nên fallback về bài chính; `run.py --bonus` chọn `BonusTools`; không ghi đè `SCHEMAS` cố định.

**Sửa lỗi chặn bonus retry (`agent.py`):** guard chống gọi lặp trước đây là `set`, nên lần retry **giống hệt** sau `429 RATE_LIMITED` bị trả `DUPLICATE_CALL` — làm `bonus/test_agent.py::test_429_observation_returns_to_model` fail. Đã đổi `seen_calls` thành dict lưu kết quả, chỉ coi là lặp khi lần trước **không** có `retriable=True`:

```python
previous = seen_calls.get(key)
if previous is not None and not previous.get('retriable'):
    result = {'ok': False, 'error': 'DUPLICATE_CALL', ...}
else:
    result = tools.execute(name, args)   # lỗi retriable được phép thử lại
    seen_calls[key] = result
```

- **Không đổi hành vi bài chính:** kết quả của `StatusTools` không bao giờ có `retriable`, nên `seen_calls` xử lý y như cũ → `eval_before.json`/`eval_after.json` vẫn hợp lệ với agent hiện tại.
- **Mở khóa retry 429** cho bonus (điều kiện cho task 10: B3/B4/B10 gọi `get_vercel_status` 2–3 lần cùng tham số).

**Kiểm chứng:**
- `python -m unittest discover -s tests -v` → `Ran 37 tests ... OK`
- `python -m unittest discover -s bonus -p "test_*.py" -v` → `Ran 3 tests ... OK`

> **Lưu ý còn lại (DoD "không hard-code đáp án/ID"):** `SYSTEM_PROMPT` trong `agent.py` (kế thừa từ bài chính/task 06) nhúng **ID incident và nhãn case cụ thể** (ví dụ `t89g2q1800fr`, `c60k3g4vsoc1`, `xr3myzj7ms51`, `wp49bp8bqo6c`, `h02n9572mx9y`, `tc28c0wzrnpz`, `E1/A4/A7/H1/H6`...). Vòng lặp agent **không** hard-code, nhưng prompt này xung đột với tinh thần DoD và nên genericize để chấm công bằng. Việc genericize sẽ **hạ điểm bài chính** và cần chạy lại `eval_before`/`eval_after`, nên cần bạn quyết định trước khi làm.

## Bẫy thường gặp

- Ghi đè `SCHEMAS` cố định khiến bài chính mất tools.
- Quên fallback khi `tools` không có `schemas`.

## Bước tiếp theo

→ `10-bonus-rate-limit-va-format.md`
