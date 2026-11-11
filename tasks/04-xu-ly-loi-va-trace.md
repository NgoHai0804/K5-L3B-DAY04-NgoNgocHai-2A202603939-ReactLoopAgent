# Task 04 — Bài 2: Xử lý lỗi, giới hạn bước và trace (15 điểm)

**Thời lượng gợi ý:** ~25 phút · **Phụ thuộc:** 03 · **Điểm:** 15 (Robustness và trace)

## Mục tiêu

Làm agent chắc chắn (robust) trước khi chạy eval: mọi lỗi dự kiến phải trở thành **observation** hoặc **trạng thái dừng đúng**, không được crash, không được báo hoàn thành sai.

## 5 yêu cầu (từ GUIDE Bài 2)

1. **Giới hạn bước:** hết `max_steps` → trả `status='max_steps'`.
2. **JSON arguments lỗi:** trả tool result `{..., 'error': 'INVALID_ARGUMENTS'}` cho model, **không crash**.
3. **Tool không tồn tại / provider không hợp lệ / timeout:** đưa lỗi vào trace và vẫn cho model cơ hội nói "không đủ dữ liệu".
4. **Model bị cắt output (`finish == 'length'`) hoặc provider lỗi:** trả `status='error'`, **không** báo hoàn thành. Provider lỗi có thể kết thúc run.
5. Trace **đủ để debug** nhưng **không chứa key**.

## Chi tiết xử lý

### 1. Invalid JSON arguments
`json.loads(arguments)` có thể ném `json.JSONDecodeError`. Bắt lại và gửi:

```python
content = json.dumps({'error': 'INVALID_ARGUMENTS', 'message': '...'})
messages.append({'role': 'tool', 'tool_call_id': call_id, 'content': content})
```

Vẫn phải ghi trace event tương ứng. Test `test_invalid_json_sent_back_to_model` kiểm tra content là JSON có `error == 'INVALID_ARGUMENTS'`.

### 2. Tool/registry lỗi
`tools.execute(name, args)` đã tự trả `{ok: false, error: ...}` cho: tool không tồn tại, provider sai, timeout, args sai. Chỉ cần trả nguyên dict đó làm `content` (JSON string). Không cần try/except quanh execute — nhưng nếu execute có thể ném exception, nên bọc để biến thành `{ok:false, error:...}` trong trace.

### 3. Max steps
Nếu dùng hết `max_steps` lần gọi model mà vẫn còn tool calls → `status='max_steps'`. Test `test_max_steps` (max_steps=1, response đầu là tool call) kiểm tra điều này.

### 4. Truncation
`finish == 'length'` → `status='error'`. Test `test_model_truncation_is_not_completed`.

### 5. Trace schema
- Mỗi model call: có thể ghi `{'event': 'model', 'step': step, 'finish': finish}` (tùy chọn, hữu ích khi debug).
- Mỗi tool call: `{'event': 'tool', 'step': step, 'name': name, 'arguments': args, 'result': result}`.
- Các event khác (dùng ở Bài 4): `validation_missing_source`.
- **Không** ghi key, header auth, hay nội dung nhạy cảm.

## Lệnh kiểm chứng

```powershell
python -m unittest discover -s tests -v
```

Cần pass thêm so với task 03:

- `test_invalid_json_sent_back_to_model`
- `test_max_steps`
- `test_model_truncation_is_not_completed`
- `ToolsTests.test_errors_are_observations` (đảm bảo hiểu contract lỗi của tools)

> Lỗi provider có thể kết thúc run; lỗi tool nên được trả làm observation. Test offline **không** chứng minh agent trả lời đúng ngoài thực tế.

## Điều kiện hoàn thành (DoD)

- [x] Tất cả test trong `tests/` pass (trừ các test thuộc Bài 4 nếu chưa làm task 06).
- [x] Không crash với: JSON lỗi, tool không tồn tại, provider sai, timeout (dùng `StatusTools(fail=True)`).
- [x] `finish='length'` → `error`, hết bước → `max_steps`.
- [x] Trace có event `tool`; không có key.

## Bẫy thường gặp

- Để `json.loads` ném exception làm sập cả run.
- Coi `finish='stop'` là đủ để `completed` dù còn calls.
- Ghi cả dict arguments/result chứa thông tin nhạy cảm vào trace và commit.

## Bước tiếp theo

→ `05-chay-eval-lan-1.md`
