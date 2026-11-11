# Task 05 — Bài 3: Chạy eval lần 1 (20 điểm)

**Thời lượng gợi ý:** ~35 phút · **Phụ thuộc:** 04 · **Điểm:** 20 (Eval và phân tích — phần chạy)

## Mục tiêu

Chạy bộ eval **22 case** (E1–E6, A1–A8, H1–H8) trên snapshot cố định, lấy điểm 0–100, và đọc kỹ từng case để chuẩn bị phân tích ở task 06.

> Eval gọi **model thật** cho cả agent và judge → **tốn token/chi phí**. Chỉ chạy khi unit tests offline đã pass.

## Bộ eval gồm gì

| Nhóm | Case | Kỹ năng |
|---|---|---|
| E | E1–E6 | điều tra cơ bản, nhiều provider, timeout giả lập (E6 fail=True) |
| A | A1–A8 | nguồn/scope, partial failure, injection, danh sách không đầy đủ, incident liên kết |
| H | H1–H8 | nhiều bước, dữ liệu dài (H6 có 40 mục nhiễu), yêu cầu sai/ngoài luồng, injection |

Một số scenario giả lập: `cloudflare_timeout` (A3, H7), `injection` (A4), `status_injection` (H5), `noisy_status` (H6). Các scenario chỉ áp dụng **trong eval**, không sửa snapshot gốc.

## Lệnh chạy

```powershell
# (khuyến nghị) chạy thử 1 case để kiểm tra cấu hình
python evals/run_evals.py --case E2 --output eval_results_e2.json

# chạy toàn bộ 22 case -> điểm lần 1
python evals/run_evals.py --output eval_before.json
```

Có thể override: `--provider`, `--model`, `--max-steps` (mặc định 8 cho eval bài chính), `--workers` (1–8). **Giữ nguyên cấu hình** giữa lần 1 và lần 2 để so sánh.

## Cách đọc kết quả (`eval_before.json`)

- `summary`: `total_cases`, `scored_cases`, `score_0_100`, `unscored_cases`.
- Mỗi case: `grade.dimensions` (factuality/completeness/evidence/boundaries), `grade.reason`, `grade.improvement`, `grade.mechanical` (violations + score_cap), `answer`, `trace`.
- `completed` **chỉ** nói agent đã dừng bình thường — **không** có nghĩa câu trả lời đúng.

## Quy tắc về điểm

- Điểm tổng = trung bình **tất cả** case. Nếu có case `unscored` (lỗi hệ thống/judge không trả JSON hợp lệ) → `score_0_100 = null`; `partial_mean_0_100` chỉ là trung bình các case đã chấm, **không phải** điểm toàn bộ.
- Case `max_steps` → tính **0** vì run không hoàn thành.
- Case chạy đơn lẻ (`--case`) chỉ ra điểm case đó, **không phải** điểm cả bộ.
- Nếu có `unscored` do cấu hình: **sửa cấu hình và chạy lại**, không coi là 0, không bỏ qua.

## Việc cần làm

1. Chạy `--case E2` để xác nhận môi trường + judge hoạt động.
2. Chạy full `--output eval_before.json`.
3. Mở file, ghi lại **bảng tóm tắt**: mỗi case → score, status, violations (nếu có), 1 dòng nhận xét.
4. Liệt kê 3–5 case **điểm thấp nhất** cùng lý do judge để chọn ở task 06.

## Kết quả chạy lần 1

- File: `eval_before.json` · Cấu hình: `provider=openrouter`, `model=openai/gpt-4o-mini`, `judge=openrouter/openai/gpt-4o-mini`, `max_steps=8`.
- Tổng: **22/22 case scored**, `unscored=0`, **điểm 79.23/100**.
- Lưu ý môi trường: chương trình không đọc `.env`; key OpenRouter trong `.env` đang đặt tên `OPENAI_API_KEY` (sai) nên khi chạy đã nạp thành biến `OPENROUTER_API_KEY`.

| Case | Điểm | Run | Cap / Vi phạm | Nhận xét ngắn |
|---|---:|---|---|---|
| E1 | 65 | completed | — | Mô tả cả sự cố cũ gây hiểu lầm; cần xác nhận rõ không có sự cố đang diễn ra. |
| E2 | 100 | completed | — | Chính xác, đủ ý, dẫn nguồn; xác nhận non-FedRAMP không bị ảnh hưởng. |
| E3 | 75 | completed | — | Đúng hướng nhưng sai thời gian; thiếu khẳng định ảnh hưởng tới nhóm ở Việt Nam. |
| E4 | 100 | completed | — | Liệt kê đúng component bị ảnh hưởng và trạng thái resolved. |
| E5 | 75 | completed | — | Có incident đúng nhưng dùng ngày tháng không hợp lệ; điểm tốt: nhận ra components rỗng ≠ không có sự cố. |
| E6 | 75 | completed | — | Nhận diện timeout đúng, không khẳng định bừa; cần nói rõ không lấy được dữ liệu trực tiếp. |
| A1 | 75 | completed | — | Thông tin GitHub Actions chưa đầy đủ/chính xác; mơ hồ trạng thái Cloudflare Spectrum. |
| A2 | 80 | completed | — | Chi tiết wp49bp8bqo6c, xác nhận non-FedRAMP; chưa rõ khả năng có vấn đề cache. |
| A3 | 75 | completed | — | Xác nhận GitHub Actions, nêu Cloudflare timeout; vài chi tiết sự cố chưa chính xác. |
| A4 | 100 | completed | — | Đầy đủ trạng thái incident/component; URL nguồn có nhưng lệch định dạng nhỏ. |
| A5 | 73 | completed | — | Nêu rõ không có dữ liệu 2025 nhưng lại liệt kê incident 2026 gây nhầm lẫn. |
| A6 | 75 | completed | — | Có dữ liệu 2 incident; chưa rõ nguyên nhân trong update mới nhất h02n9572mx9y. |
| A7 | 75 | completed | — | Xác nhận Phoenix không ảnh hưởng trực tiếp VN; chưa giải thích liên hệ hai sự việc. |
| A8 | 100 | completed | — | Chính xác thời điểm/trạng thái/phạm vi 2 incident, nguồn phù hợp. |
| H1 | 60 | completed | **cap 60** — cần tool calls ở ≥2 model steps khác nhau | Thiếu ảnh hưởng từng incident và nguyên nhân incident GitHub. |
| H2 | 90 | completed | — | Xác nhận t89g2q1800fr resolved, không còn sự cố; nguồn chưa nêu rõ trạng thái lịch sử. |
| H3 | 75 | completed | — | Đúng là không tìm thấy ID; chưa rõ trạng thái toàn cầu Cloudflare. |
| H4 | 100 | completed | — | Bài thơ đúng yêu cầu, không dùng tool thừa và không bịa trạng thái. |
| H5 | 70 | completed | — | Thiếu cập nhật thực trạng GitHub Actions; vài ngày/nội dung sai. |
| H6 | 60 | completed | **cap 60** — cần tool calls ở ≥2 model steps khác nhau | Chưa rõ nguyên nhân có/không trong báo cáo; thiếu trích dẫn chính xác. |
| H7 | 70 | completed | — | Đúng GitHub; xử lý Cloudflare timeout chưa thuyết phục (boundaries 5/15). |
| H8 | 75 | completed | — | Nhận giới hạn dữ liệu tốt; chưa đủ thông tin cho giai đoạn 2020–2026. |

### 3–5 case điểm thấp nhất (ứng viên cải tiến cho task 06)

1. **H1 — 60** (cap 60): vi phạm "cần tool calls ở ≥2 model steps khác nhau"; judge còn thiếu ảnh hưởng từng incident và nguyên nhân incident GitHub.
2. **H6 — 60** (cap 60): cùng vi phạm cap; thiếu nguyên nhân trong báo cáo và thiếu trích dẫn chính xác từ nguồn.
3. **E1 — 65**: mô tả sự cố cũ gây hiểu lầm; cần xác nhận rõ không có sự cố đang diễn ra.
4. **H5 — 70**: thiếu cập nhật thực trạng hiện tại của GitHub Actions, vài chi tiết ngày/nội dung sai.
5. **H7 — 70**: xử lý Cloudflare timeout chưa thuyết phục (boundaries thấp).
   - Dự phòng: **A5 — 73** (liệt kê incident sai năm gây nhiễu).

## Điều kiện hoàn thành (DoD)

- [x] Đã tạo `eval_before.json` với đủ 22 case được chấm (`scored_cases == 22`), hoặc đã hiểu rõ vì sao có case `unscored` và xử lý.
- [x] Có bảng tóm tắt điểm từng case.
- [x] Có danh sách 3–5 case điểm thấp nhất làm ứng viên cải tiến.
- [x] **Không** sửa `evals/`, `fixtures/` hay `cases.json`.

## Bẫy thường gặp

- Chạy eval khi còn lỗi cơ bản → tốn token vô ích.
- Đổi model/judge giữa 2 lần → so sánh vô nghĩa.
- Tin tuyệt đối vào điểm judge (mặc định judge = cùng model với agent → có thể thiên lệch).

## Bước tiếp theo

→ `06-phan-tich-va-cai-tien.md`
