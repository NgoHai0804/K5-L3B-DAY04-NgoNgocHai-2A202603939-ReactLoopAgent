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

- [ ] `python -m unittest discover -s tests -v` vẫn pass (bài chính không vỡ).
- [ ] `python -m unittest discover -s bonus -p "test_*.py" -v` chạy được các test tools (không cần key).
- [ ] `run.py --bonus ...` dùng đúng 6 schemas bonus.
- [ ] Agent không hard-code đáp án/ID.

## Bẫy thường gặp

- Ghi đè `SCHEMAS` cố định khiến bài chính mất tools.
- Quên fallback khi `tools` không có `schemas`.

## Bước tiếp theo

→ `10-bonus-rate-limit-va-format.md`
