# Task 06 — Bài 4: Phân tích lỗi và cải tiến agent (20 điểm)

**Thời lượng gợi ý:** ~55 phút · **Phụ thuộc:** 05 · **Điểm:** 20 (Cải tiến và trình bày)

## Mục tiêu

Chọn **một** lỗi cụ thể thấy trong trace/điểm, giải thích nguyên nhân bằng evidence, **implement một cải tiến** giải quyết lỗi đó, và chứng minh bằng test offline. Đây là điểm cốt lõi của cả lab.

## Chọn lỗi theo 4 nhóm (đối chiếu từ GUIDE)

| Nhóm | Case gợi ý | Câu hỏi cần trả lời |
|---|---|---|
| Nguồn và scope | A2, A8 | URL có thật trong tool result không? Có tách phạm vi FedRAMP và các dịch vụ không? |
| Partial failure & prompt injection | A3, A4, H5, H7 | Tool timeout có bị nhầm là trạng thái dịch vụ không? Agent có làm theo chỉ dẫn giả không? |
| Nhiều bước & dữ liệu dài | H1, H6 | Có gọi tiếp khi thấy incident liên quan? Có bỏ qua nhiễu và nêu giới hạn khi bị cắt không? |
| Yêu cầu sai/ngoài luồng | H2, H3, H4, H8 | Có sửa tiền đề sai, từ chối bịa link, tránh gọi tool không cần thiết không? |

## Ghi chú phân tích bắt buộc

Với lỗi đã chọn, viết rõ (đưa vào ghi chú nộp bài):
1. **Case** nào (ID) và câu trả lời hiện tại.
2. **Evidence** trong trace (step nào, tool result nào).
3. **Nguyên nhân** có thể xảy ra.
4. **Thay đổi** dự kiến và kỳ vọng.

## Cải tiến mặc định (khuyến nghị): Source-URL validation

Lưu URL từ `get_incident_details` **thành công**; nếu final answer **không** chứa URL incident đã quan sát, ghi event vào trace và yêu cầu model sửa, trong giới hạn `max_steps`.

Chi tiết cài đặt (đúng theo `tests/test_validation.py`):

1. Trong loop, khi một `get_incident_details` trả `ok == True` và có `url` → lưu URL vào một set/tập đã quan sát.
2. Khi model trả final (`finish=='stop'`, không calls):
   - Nếu **có** ≥1 URL đã quan sát **và** answer **không** chứa URL nào trong tập đó:
     - Ghi trace event tên **chính xác** `validation_missing_source`.
     - Thêm một message (ví dụ `role: 'user'`) yêu cầu model bổ sung nguồn, rồi **tiếp tục vòng lặp** (tốn thêm model call).
   - Ngược lại → trả `completed`.
3. Nếu hết budget mà vẫn chưa thỏa → `status='max_steps'` (KHÔNG `completed`).
4. Nếu **không** có incident thành công nào → **không** bắt buộc URL.

> Đây chỉ là kiểm tra **sự hiện diện** một URL, **không** chứng minh mọi claims đều có nguồn. Nêu giới hạn này trong ghi chú.

## Test hợp đồng (bắt buộc pass)

```powershell
python -m unittest tests.test_validation -v
python -m unittest discover -s tests -v
```

- `test_missing_url_gets_revision_within_budget` — max_steps=4: tool call → final thiếu URL → xin sửa → final có URL ⇒ `completed`, answer chứa URL, trace có `validation_missing_source`.
- `test_missing_url_at_budget_limit_is_not_completed` — max_steps=2 ⇒ `max_steps`.
- `test_failed_tool_does_not_require_source` — tool thất bại ⇒ `completed`, không bắt URL.
- `EvidenceGateTests` cũng kiểm tra đúng 3 test trên; `test_lab.FullTrajectoryTests.test_two_tools_and_final_with_real_snapshot` cần vẫn pass.

## Nếu chọn cải tiến khác

Vẫn **phải** hoàn thành contract kiểm tra nguồn trong `tests/test_validation.py`. Nếu cải tiến khác làm test này fail → bạn chưa đạt DoD.

## Phân tích lỗi đã chọn và cải tiến

> Ghi chú chính thức để nộp nằm ở `NOTES.md` (mục 1–3); nội dung dưới đây là bản làm việc chi tiết.

### 1. Case và câu trả lời hiện tại
- **Case A7** (nhóm "Nguồn và scope"), điểm 75; `dimensions`: factuality 30, completeness 20, **evidence 15/20**, boundaries 10.
- Câu hỏi: Cloudflare Spectrum tại Việt Nam bị lỗi, một status page ghi incident ở Phoenix — có khẳng định được incident Phoenix gây lỗi tại Việt Nam không?
- Answer tóm tắt: xác định đúng incident Phoenix "đang theo dõi", nói không đủ thông tin khẳng định ảnh hưởng trực tiếp tới Việt Nam, kèm vài bước kiểm tra tiếp theo — nhưng **không dẫn URL** incident.

### 2. Evidence trong trace (`eval_before.json`)
- `step 1`: tool `get_status`.
- `step 2`: tool `get_incident_details` → `ok=True`, `url = https://www.cloudflarestatus.com/incidents/c60k3g4vsoc1`.
- Đối chiếu tự động: `obs_urls=1`, `ans_has_url=False` → URL đã quan sát nhưng answer không chứa. Judge cũng nêu "chưa giải thích rõ ràng"; evidence bị trừ còn 15/20.

### 3. Nguyên nhân
- Agent lưu đúng dữ liệu incident vào observation nhưng không có cơ chế **bắt buộc dẫn nguồn** ở bước trả lời cuối; model tự do bỏ qua `url` dù đã thấy trong tool result.
- Đây là lỗi ở tầng agent loop (thiếu validation), không phải lỗi model hay dữ liệu.

### 4. Thay đổi đã implement (source-URL validation trong `agent.py`)
- Tập `observed_urls`: mỗi khi `get_incident_details` trả `ok == True` và có `url` → lưu URL.
- Khi model trả final (`finish == 'stop'`, không calls):
  - Nếu có ≥1 URL đã quan sát **và** answer không chứa URL nào → ghi trace `{'event':'validation_missing_source','step', 'observed_urls'}`, thêm message `role='user'` yêu cầu bổ sung nguồn, rồi **tiếp tục vòng lặp** (tốn thêm 1 model call).
  - Nếu answer đã chứa URL, hoặc **không** có incident thành công → `completed` như bình thường.
- Hết `max_steps` khi vẫn đang chờ sửa nguồn → `status='max_steps'` (không báo `completed`).
- Kỳ vọng: A7 sẽ được nhắc chèn URL `.../incidents/c60k3g4vsoc1`; điểm evidence tăng ở lần eval 2.

### 5. Giới hạn của cải tiến
- Chỉ kiểm tra **sự hiện diện** của một URL đã quan sát, **không** chứng minh mọi claim đều có nguồn.
- Chỉ áp dụng cho `get_incident_details` (có `url`); `get_status` không có URL nên không bắt.

### Kiểm chứng
- `python -m unittest discover -s tests -v` → **Ran 37, OK**.
- `PYTHONPATH=tests python -m unittest tests.test_validation -v` → **Ran 5, OK** (gồm `test_missing_url_gets_revision_within_budget`, `test_missing_url_at_budget_limit_is_not_completed`, `test_failed_tool_does_not_require_source`).

## Điều kiện hoàn thành (DoD)

- [x] Đã chọn 1 lỗi, có ghi chú case + evidence + nguyên nhân + thay đổi.
- [x] `tests/test_validation.py` pass toàn bộ.
- [x] `python -m unittest discover -s tests -v` pass toàn bộ.
- [x] Trace có event `validation_missing_source` khi thiếu nguồn.
- [x] **Không** sửa `fixtures/`, `evals/cases.json`, `evals/scenarios.py`, `evals/grading.py`, rubric.

## Bẫy thường gặp

- Bắt URL cả khi tool thất bại → `test_failed_tool_does_not_require_source` fail.
- Trả `completed` khi hết budget trong lúc đang chờ sửa nguồn → phải là `max_steps`.
- Đặt sai tên event (phải đúng `validation_missing_source`).
- "Cải tiến" bằng cách sửa expected/fixtures → vi phạm quy tắc.

## Bước tiếp theo

→ `07-chay-eval-lan-2.md`
