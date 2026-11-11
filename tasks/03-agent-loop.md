# Task 03 — Bài 1: Tự viết agent loop (30 điểm)

**Thời lượng gợi ý:** ~55 phút · **Phụ thuộc:** 02 · **Điểm:** 30 (Agent loop)

## Mục tiêu

Implement `run_agent` trong `agent.py` để hoàn thành vòng lặp: **gọi model → thực thi tool → đưa kết quả về model → lặp → trả lời hoặc hết giới hạn**. Không cần sửa provider adapter hay tool implementations.

## Contract chính xác

```python
run_agent(question, provider, tools, max_steps=6) -> dict
```

- `provider.complete(messages, SCHEMAS)` trả `{'message', 'calls', 'finish', 'usage'}`.
- `message` là assistant message — phải append **nguyên vẹn** vào `messages` **trước** khi thêm tool results.
- Mỗi `call` có `id`, `name`, `arguments` (JSON string). Parse `arguments` rồi gọi `tools.execute(name, args)`.
- Mỗi tool result gửi lại bằng `{'role': 'tool', 'tool_call_id': call_id, 'content': json_string}`.
- Phải xử lý **MỌI** call trong một response **trước** model call tiếp theo.
- Thành công **chỉ khi** `finish == 'stop'` **và** không còn calls.
- Kết quả trả về **tối thiểu**: `status` (`completed` / `max_steps` / `error`), `answer` (string), `trace` (list), `usage` (list).

## Gợi ý cấu trúc (khung, không copy mù)

1. `messages = [{'role': 'system', 'content': SYSTEM_PROMPT}, {'role': 'user', 'content': question}]`
2. `trace = []`, `usage = []`, `answer = ''`
3. Lặp `for step in range(1, max_steps + 1):`
   - `reply = provider.complete(messages, SCHEMAS)`
   - `messages.append(reply['message'])` (nguyên vẹn)
   - `usage.append(reply.get('usage', {}))`
   - Nếu **không** có `calls`:
     - Nếu `finish == 'stop'` → lấy `answer`, `return {'status': 'completed', ...}`
     - Ngược lại (`length` / lý do khác) → `return {'status': 'error', ...}` (KHÔNG báo hoàn thành)
   - Nếu **có** `calls`:
     - Với mỗi call: parse args → execute → thêm tool message → ghi trace event
     - Sau khi xử lý hết calls, tiếp tục vòng lặp (gọi model tiếp)
4. Nếu vòng lặp kết thúc mà chưa `return` → `return {'status': 'max_steps', ...}`

> Quan trọng: mỗi **model call** là một step, **không** phải mỗi tool call.

## Trace — schema tối thiểu

Ghi cho mỗi tool call một event có thể chấm được bởi `evals/grading.py`:

```python
{'event': 'tool', 'step': step, 'name': name, 'arguments': args, 'result': result}
```

- **Không đưa API key** hay dữ liệu nhạy cảm vào trace.
- Không yêu cầu model in chain-of-thought; trace chỉ cần step, finish, tool name, arguments, result.

## Checkpoint (tự hỏi sau khi code)

- Sau tool call đầu tiên, model có thực sự nhận được tool result gắn **đúng ID** không?
- Nếu có **hai** calls trong một response thì **cả hai** kết quả đã được trả chưa?

## Lệnh kiểm chứng

```powershell
# Chạy offline, dùng FakeProvider, không tốn token
python -m unittest tests.test_lab.LoopTests -v
python -m unittest discover -s tests -v
```

Trong `tests/test_lab.py` (LoopTests) cần pass:

- `test_tool_result_is_returned_with_id` — tool message có đúng `tool_call_id` và content đúng provider.
- `test_multiple_tool_calls` — hai calls → hai tool message đúng thứ tự ID `['a','b']`.
- `test_model_truncation_is_not_completed` — `finish='length'` → `status='error'` (test này đạt khi bạn xử lý đúng ở task 04).
- `FullTrajectoryTests.test_two_tools_and_final_with_real_snapshot` — 2 tools + final với snapshot thật.

## Điều kiện hoàn thành (DoD)

- [x] `run_agent` không còn `raise NotImplementedError`.
- [x] `test_tool_result_is_returned_with_id` pass.
- [x] `test_multiple_tool_calls` pass.
- [x] `FullTrajectoryTests.test_two_tools_and_final_with_real_snapshot` pass.
- [x] Kết quả trả về đủ 4 trường `status`, `answer`, `trace`, `usage`.

## Bẫy thường gặp

- Append assistant message **sau** tool results (sai thứ tự) → provider lỗi.
- Chỉ xử lý call đầu tiên rồi gọi model tiếp → mất tool result thứ hai.
- Đếm step theo tool call thay vì model call.
- Trả `completed` chỉ vì `finish='stop'` nhưng vẫn còn calls.

## Bước tiếp theo

→ `04-xu-ly-loi-va-trace.md`
