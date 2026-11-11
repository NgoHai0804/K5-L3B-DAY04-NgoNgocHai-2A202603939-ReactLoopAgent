# Task 02 — Khám phá tools và dữ liệu

**Thời lượng gợi ý:** 15 phút · **Phụ thuộc:** 01 · **Điểm:** — (nền tảng cho Bài 1)

## Mục tiêu

Hiểu rõ hai tools bài chính **trước khi** viết agent loop, để biết chúng trả gì, giới hạn ở đâu, và lỗi được biểu diễn thế nào. Không cần LLM key cho task này.

## Kiến thức cần nắm (từ `status_tools.py`)

- `SCHEMAS` (import từ `status_tools`): 2 tool
  - `get_status(provider, component='')` — `provider` ∈ {`github`, `cloudflare`}, `component` là **chuỗi tìm gần đúng** (substring), không phải semantic search.
  - `get_incident_details(provider, incident_id)` — đọc chi tiết một incident có trong **50 incident gần nhất**.
- `StatusTools.execute(name, args)` — registry thực thi tool, luôn trả **dict**; lỗi dự kiến được chuyển thành `{ok: false, error: ...}` để model quan sát.
- `get_status` trả: `provider`, `captured_at`, `overall`, `components`, `incidents` (tối đa 5), `matching_incidents_in_last_50`, `note`, `source`.
- `get_incident_details` trả: `ok`, `id`, `name`, `status`, `created_at`, `updated_at`, `components`, `updates` (2 update mới nhất + update đầu tiên, mỗi body ≤ 900 ký tự), `updates_total`, `url`.

## Giới hạn quan trọng (đừng suy luận quá mức)

- Tool chỉ tìm trong **50 incident gần nhất**: **tìm không thấy KHÔNG chứng minh không có sự cố**.
- `component` là khớp chuỗi: `cache purge` có thể khớp nhưng `xóa bộ nhớ đệm` có thể **không**.
- Status tổng quan (`overall`) **không** đồng nghĩa mọi component/khu vực đều ổn.
- `truncated` / `updates_total` báo dữ liệu có thể đã bị lược bỏ.

## Việc cần làm

Chạy trực tiếp (không cần LLM key):

```powershell
python -c "import json; from status_tools import StatusTools; print(json.dumps(StatusTools().get_status('github', 'Actions'), ensure_ascii=False, indent=2))"
python -c "import json; from status_tools import StatusTools; print(json.dumps(StatusTools().get_incident_details('cloudflare', 'wp49bp8bqo6c'), ensure_ascii=False, indent=2))"
```

Quan sát thêm các trường hợp:

```powershell
# component rỗng -> chỉ tìm theo title/nội dung
python -c "from status_tools import StatusTools; print(StatusTools().get_status('github', 'Copilot Mission Control'))"
# tool không tồn tại
python -c "from status_tools import StatusTools; print(StatusTools().execute('missing', {}))"
# lỗi timeout giả lập
python -c "from status_tools import StatusTools; print(StatusTools(fail=True).execute('get_status', {'provider':'github'}))"
```

## Câu hỏi cần trả lời được (viết ngắn vào ghi chú)

1. `get_status` và `get_incident_details` khác nhau ở điểm nào về mục đích?
2. Vì sao một kết quả tìm kiếm rỗng **không** đủ để kết luận "không có incident"?
3. Khi tool lỗi, dữ liệu trả về có dạng gì? Agent nên làm gì với nó?
4. `url` của incident quan trọng thế nào với yêu cầu "dẫn nguồn"?

### Ghi chú trả lời

1. **Mục đích khác nhau:** `get_status` là điểm vào/khám phá — trả trạng thái tổng quan + tối đa 5 incident khớp chuỗi (chỉ id/name/status/updated). `get_incident_details` đọc sâu *một* incident cụ thể: components, timeline updates, `updates_total`, `truncated`, `url` — dùng để lấy bằng chứng chi tiết.
2. **Search rỗng không đủ kết luận "không có incident":** tool chỉ quét **50 incident gần nhất** và khớp **substring** trên title/component/body. Không khớp chỉ nghĩa là "không có trong cửa sổ/nội dung đó", không chứng minh sự cố không tồn tại.
3. **Khi tool lỗi:** luôn trả **dict** dạng `{ok: false, error: <mã>, message: ...}` (vd `UNKNOWN_TOOL`, `TimeoutError`), không crash. Agent nên đưa lại làm observation cho model để báo thiếu dữ liệu/thử cách khác — không coi lỗi tool là outage dịch vụ.
4. **`url` và yêu cầu dẫn nguồn:** `url` là link chuẩn, có thật trong tool result để trích dẫn bằng chứng. Phải dùng đúng URL này (không bịa link); chỉ nêu URL của incident đã quan sát thành công.

## Điều kiện hoàn thành (DoD)

- [x] Đã chạy được 2 lệnh đầu tiên và đọc được JSON trả về.
- [x] Xác nhận tổng quan GitHub Actions trong snapshot (overall `none`, incident Actions 05/11 `resolved`).
- [x] Xác nhận incident `wp49bp8bqo6c` có update nêu **"Non-fedramp customers are not impacted"**.
- [x] Trả lời được 4 câu hỏi ở trên.

## Bẫy thường gặp

- Coi `overall == none` là "mọi thứ chắc chắn ổn".
- Kết luận "không có incident" khi search rỗng.
- Quên rằng tool đã **truncate** body và chỉ giữ vài update.

## Bước tiếp theo

→ `03-agent-loop.md`
