# Task 01 — Chuẩn bị môi trường

**Thời lượng gợi ý:** 10–15 phút · **Phụ thuộc:** không · **Điểm:** —

## Mục tiêu

Có một môi trường chạy được: Python 3.10+, một provider có tool calling, key hợp lệ, và chạy được bộ unit test offline để thấy trạng thái "ban đầu một số test fail vì `agent.py` chưa implement".

## Việc cần làm

1. Kiểm tra Python và vị trí làm việc (thư mục repo lab `K5-L3-Day4-ReactLoopAgent`).
2. Chọn **một** provider trong 4: `openai`, `gemini`, `anthropic`, `openrouter`. Chọn model có hỗ trợ **tool calling**.
3. Đặt biến môi trường đúng tên biến key của provider (xem bảng dưới). Không cần cả 4 key.
4. Chạy bộ unit test offline để xác nhận baseline (một số test sẽ fail — đó là bình thường).

## Bảng cấu hình provider

| `PROVIDER` | Biến chứa key | `MODEL` |
|---|---|---|
| `openai` | `OPENAI_API_KEY` | tên model OpenAI có tool calling |
| `gemini` | `GEMINI_API_KEY` | model Gemini hỗ trợ function calling |
| `anthropic` | `ANTHROPIC_API_KEY` | model Claude hỗ trợ tool use |
| `openrouter` | `OPENROUTER_API_KEY` | model ID đầy đủ, dạng `provider/model` |

## Lệnh (Windows PowerShell)

```powershell
python --version

# Ví dụ chọn OpenAI (đổi theo provider bạn chọn)
$env:PROVIDER = "openai"
$env:MODEL    = "<ten-model-co-tool-calling>"
$env:OPENAI_API_KEY = "<api-key-cua-ban>"

# Kiểm tra baseline tests (KHÔNG cần key, dùng FakeProvider)
python -m unittest discover -s tests -v
```

> Ghi chú: tài liệu gốc viết lệnh dạng `python3` và `export` (bash). Trên máy này shell là PowerShell nên dùng `python` và `$env:...`.
>
> Ghi chú (sau update repo PR #1): `status_tools.py`, `bonus/tools.py`, `evals/run_evals.py` đã đọc/ghi file với `encoding='utf-8'`, nên **không cần** đặt `$env:PYTHONUTF8="1"` nữa (trước đây cần để tránh lỗi codec cp1252 trên Windows).

Tùy chọn: giảng viên có thể yêu cầu đặt User-Agent có liên hệ cho mode live:

```powershell
$env:STATUS_USER_AGENT = "StatusAgentLab/1.0 (contact: email-lop@example.edu)"
```

## Điều kiện hoàn thành (DoD)

- [x] `python --version` trả về **3.10+**.
- [x] Đã đặt `PROVIDER`, `MODEL`, và đúng biến key của provider đó.
- [x] `python -m unittest discover -s tests -v` chạy được; các test liên quan `run_agent` **fail** vì chưa implement (đúng như dự kiến).
- [x] Xác nhận bạn **không** dựa vào `.env` (chương trình không tự đọc file này).

## Bẫy thường gặp

- Dùng model không hỗ trợ tool calling → provider trả lỗi hoặc không sinh `tool_calls`.
- Quên đặt `MODEL` → lỗi "Cần đặt MODEL là model có hỗ trợ tool calling."
- Đặt key sai tên biến → `Provider` báo "Thiếu <KEY_NAME>".
- Kỳ vọng tất cả test pass ngay: **sai**, một số test phải fail trước khi bạn code.

## Bước tiếp theo

→ `02-kham-pha-tools.md`
