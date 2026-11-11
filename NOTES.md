# Ghi chú phân tích · Day 04

Họ và tên: Ngô Ngọc Hải
MSSV: 2A202603939

## 1. Case được chọn

- ID case: **A7** (nhóm "Nguồn và scope")
- Câu trả lời của agent (trích): "… Cloudflare Spectrum đang gặp vấn đề hiệu suất … hiện tại có một sự cố được theo dõi ở Phoenix … không có thông tin cụ thể nào cho thấy sự cố này ảnh hưởng trực tiếp đến Việt Nam … Bước kiểm tra tiếp theo: theo dõi thêm cập nhật từ Cloudflare."
- Điểm ở lần chạy 1: **75/100** (`factuality 30/40`, `completeness 20/25`, **`evidence 15/20`**, `boundaries 10/15`).

## 2. Bằng chứng trong trace

`step 1` gọi `get_status`; `step 2` gọi `get_incident_details` trả `ok=True` và `url=https://www.cloudflarestatus.com/incidents/c60k3g4vsoc1`. Answer **không** chứa URL này (kiểm tra tự động: `obs_urls=1`, `ans_has_url=False`).

```text
step 1  tool get_status            -> ...
step 2  tool get_incident_details  -> ok=True  url=https://www.cloudflarestatus.com/incidents/c60k3g4vsoc1
final   answer                     -> không chứa URL nào đã quan sát
```

**Nguyên nhân có thể xảy ra:** agent lưu đúng dữ liệu incident vào observation nhưng không có bước bắt buộc dẫn nguồn ở final answer, nên model bỏ qua `url` dù đã thấy trong tool result. Judge cũng nêu "chưa giải thích rõ ràng", và `evidence` bị trừ còn 15/20.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: thêm **source-URL validation** trong `agent.py`:
  - Lưu `observed_urls` từ mọi `get_incident_details` trả `ok == True` và có `url`.
  - Khi final answer không chứa URL nào đã quan sát → ghi trace event `validation_missing_source`, thêm message yêu cầu bổ sung nguồn, rồi tiếp tục vòng lặp trong `max_steps`.
  - Hết budget khi vẫn chờ sửa nguồn → `status='max_steps'` (không báo `completed`); không bắt URL khi tool thất bại.
- Vì sao chọn thay đổi này: lỗi nằm ở tầng agent loop (thiếu ràng buộc dẫn nguồn), không phải model hay dữ liệu; đây là cải tiến mặc định trong GUIDE và được kiểm chứng bởi `tests/test_validation.py` (cả 5 test pass).

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 79.23 | 83.64 |
| Điểm case đã chọn (A7) | 75 | 80 |
| Số model calls | 52 (A7: 3) | 52 (A7: 3) |
| Token (agent) | ~45,243 (prompt 34,699 + completion 10,544) | ~157,395 (prompt 149,431 + completion 7,964) |
| Chi phí (agent) nếu có | ~$0.0115 | ~$0.0183 |

- Lỗi đã giải quyết: A7 giờ **đã dẫn URL nguồn** đã quan sát (answer chứa `https://www.cloudflarestatus.com/incidents/c60k3g4vsoc1`, `ans_has_url=True`; guard `validation_missing_source` không cần kích hoạt). H1/H6 **hết vi phạm cap** "≥2 model steps" (60→75). Các case tăng điểm khác: H3 75→100, H8 75→100, E6 75→100, A2 80→100, H2 90→100, E1 65→75, E3 75→85, A7 75→80, H5 70→75, A5 73→75.
- Lỗi còn lại hoặc mới xuất hiện: **H7 70→55** (xử lý Cloudflare timeout và nêu giới hạn dữ liệu còn yếu). Một số case giảm (E2 100→75, E5 75→70, A1 75→70, H4 100→80) — chủ yếu do **phương sai của judge** (cùng model tự chấm, chỉ chạy một lần), không phải side-effect kỹ thuật: **tất cả 22 case đều `completed`, không case nào chuyển sang `max_steps`/`error`**. H4 giảm nhẹ vì bài thơ không dùng tool nên `evidence=0`.

Cấu hình dùng cho cả hai lần: `provider=openrouter`, `model=openai/gpt-4o-mini`, `judge=openrouter/openai/gpt-4o-mini`, `max_steps=8`, `dataset=core`.

> Lần 2 dùng agent có prompt hệ thống/workflow đầy đủ hơn và gọi tool nhiều bước hơn, nên **token prompt và chi phí tăng mạnh** dù số model calls không đổi.

**Kết luận:** cải tiến source-URL validation được chứng minh bằng test hợp đồng (`tests/test_validation.py` 5/5 pass). Điểm eval lần 2 cao hơn (83.64 so với 79.23, **+4.41**) và H1/H6 hết bị cap; tuy nhiên **một lần chạy mỗi phiên bản không đủ kết luận**, phần tăng/giảm lẻ tẻ nằm trong biên độ dao động của judge.

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.

- **Judge:** là LLM chấm xấp xỉ, có thể nhầm/thiên lệch; mặc định judge dùng cùng model với agent nên có thể tự chấm điểm cao. Điểm cần đối chiếu lại với `expected` và trace.
- **Cải tiến source-URL chỉ kiểm tra sự hiện diện của một URL** đã quan sát, không chứng minh mọi claim đều có nguồn; và chỉ áp dụng cho `get_incident_details` (có `url`), không áp dụng cho `get_status`.
- **Snapshot:** dữ liệu lưu ngày 09–10/11/2026, không phản ánh trạng thái thực tế hiện tại; tool chỉ tìm trong 50 incident gần nhất.
- **So sánh chi phí/token:** cấu hình model/judge giống nhau nhưng agent lần 2 có prompt hệ thống đầy đủ hơn và gọi tool nhiều bước hơn, nên token prompt và chi phí cao hơn hẳn; chênh lệch này phản ánh thay đổi ở agent, **không** phải thay đổi cấu hình, và chỉ mang tính tham khảo.
- **Một lần chạy mỗi phiên bản** không chứng minh độ ổn định; kết quả có thể thay đổi giữa các lần chạy (điểm lần 2 tăng +4.41 nhưng nhiều case dao động ±20–25 chỉ do judge).
