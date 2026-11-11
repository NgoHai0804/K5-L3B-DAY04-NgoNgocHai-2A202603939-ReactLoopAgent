# Day04 — Task Board: Status Investigation Agent

Mục tiêu lab: tự viết **agent loop kiểu ReAct** bằng Python thuần, dùng dữ liệu thật từ GitHub Status và Cloudflare Status để điều tra sự cố, rồi dùng **trace + eval** để kiểm tra và cải tiến agent.

> Đây là **1 vòng thực hành khép kín**: xây agent → chạy eval → phân tích lỗi → cải tiến → chạy lại.
> Thời lượng gợi ý: **~4 giờ** (gồm 10 phút nghỉ). Điểm bài chính tối đa **85**, bonus tùy chọn cộng tối đa **15** → tổng **100**.

## Cập nhật từ repo gốc (PR #1 — chuẩn hoá K5)

Repo đề vừa được cập nhật. Cần lưu ý:

- **File mới:** `CHECKPOINTS.md` (7 checkpoint theo ca), `NOTES.md` (ghi chú phân tích chính thức), `RULES.md` (quy định làm bài), `SUBMISSION.md` (cách nộp + checklist).
- **File sửa:** `GUIDE.md`, `README.md`, `RUBRIC.md`, `status_tools.py`, `bonus/tools.py`, `evals/run_evals.py`.
- **Encoding:** `status_tools.py`, `bonus/tools.py`, `evals/run_evals.py` nay đọc/ghi file với `encoding='utf-8'` → **không cần** đặt `PYTHONUTF8=1` nữa (trước đây phải bật để tránh lỗi cp1252 trên Windows).
- **Ghi chú nộp bài:** dùng `NOTES.md` (5 mục) thay cho ghi chú rời — mục 1–3 (case, evidence, thay đổi) và mục 4–5 (so sánh, giới hạn).
- **Nộp bài:** theo `SUBMISSION.md`, tên repo `K5-L3A-DAY04-[HoVaTen]-[MSSV]-ReactLoopAgent` (hoặc `L3B`).

Đối chiếu task ↔ checkpoint: 01–02 → CP1 · 03 → CP2 · 04 → CP3 · 05 → CP4 · 06 → CP5 · 07 → CP6 · 08 → CP7.

## Cách dùng bộ task này

- Làm **tuần tự theo số**, **không nhảy cóc**. Mỗi file là một task độc lập, có: mục tiêu, việc cần làm, điều kiện hoàn thành (DoD), lệnh kiểm chứng.
- Chỉ chuyển sang task kế tiếp khi **DoD của task hiện tại đã đạt** (đặc biệt là các task code phải pass test offline trước khi chạy eval).
- Task 09–12 (bonus) chỉ làm **sau khi bài chính (task 01–08) đã xong**.

## Danh sách task (theo thứ tự tiến độ)

| # | File | Nội dung | Điểm | Phụ thuộc |
|---|---|---|---|---|
| 01 | `01-chuan-bi-moi-truong.md` | Chuẩn bị Python, provider, key, chạy test baseline | — | — |
| 02 | `02-kham-pha-tools.md` | Khám phá 2 tools + dữ liệu snapshot | — | 01 |
| 03 | `03-agent-loop.md` | **Bài 1**: tự code `run_agent` | 30 | 02 |
| 04 | `04-xu-ly-loi-va-trace.md` | **Bài 2**: xử lý lỗi, `max_steps`, trace | 15 | 03 |
| 05 | `05-chay-eval-lan-1.md` | **Bài 3**: chạy eval lần 1 (`eval_before.json`) | 20 | 04 |
| 06 | `06-phan-tich-va-cai-tien.md` | **Bài 4**: phân tích lỗi + cải tiến (source validation) | 20 | 05 |
| 07 | `07-chay-eval-lan-2.md` | **Bài 5**: chạy eval lần 2 (`eval_after.json`) + so sánh | (gộp 06) | 06 |
| 08 | `08-nop-bai.md` | Đóng gói & nộp bài chính + ghi chú | — | 07 |
| 09 | `09-bonus-ket-noi-tools.md` | **Bonus**: kết nối 5 status tools + `schemas`/`system_prompt` | (bonus) | 08 |
| 10 | `10-bonus-rate-limit-va-format.md` | **Bonus**: retry 429, sửa lỗi định dạng | (bonus) | 09 |
| 11 | `11-bonus-pagination-va-context.md` | **Bonus**: pagination, dữ liệu nhiễu, injection | (bonus) | 10 |
| 12 | `12-bonus-eval-va-nop.md` | **Bonus**: chạy 10 case, `eval_bonus.json`, nộp | +15 | 11 |

## Bảng điểm cần nhớ

| Nội dung | Điểm | Tiêu chí |
|---|---:|---|
| Agent loop | 30 | gọi model + tool; lưu assistant message; trả đúng tool-call IDs; xử lý nhiều calls; dừng đúng |
| Robustness và trace | 15 | `max_steps`, invalid JSON, tool error, model dừng bất thường; trace đủ để debug |
| Eval và phân tích | 20 | chạy 2 lần; đọc case + trace; xác định lỗi; giải thích bằng evidence; nêu giới hạn của judge |
| Cải tiến và trình bày | 20 | cải tiến phù hợp; validation nguồn; tests; so sánh kết quả; giải thích lỗi còn lại |

Điểm eval mỗi case (0–100) = `factuality 40 + completeness 25 + evidence 20 + boundaries 15`, sau đó áp trần nếu vi phạm kiểm tra code (thiếu tool bắt buộc → cap 50; lặp injection marker → cap 20). Điểm tổng = trung bình **tất cả** case; nếu có case `unscored` → tổng `null`.

## Trạng thái hoàn thành (kết quả cuối)

| Hạng mục | Kết quả |
|---|---|
| Bài chính (22 case) | `eval_before.json` **79.23** → `eval_after.json` **83.64** (22/22 scored, cùng config `openrouter`/`openai/gpt-4o-mini`, `max_steps=8`) |
| Bonus (10 case, tùy chọn) | `eval_bonus.json` **84.0/100** → `bonus_points_0_15 = 12.6` (10/10 scored, không vi phạm cơ học, `max_steps=16`) |
| Tests offline | `tests` **37 OK** · `bonus` **7 OK** |
| Ghi chú phân tích | `NOTES.md` mục 1–5 (bài chính) + mục 6 (bonus) |

Chi tiết bonus: xem `tasks/12-bonus-eval-va-nop.md`. Checklist nộp: xem `tasks/08-nop-bai.md`.

## Quy tắc bất di bất dịch

- **Không sửa** `fixtures/`, `evals/cases.json`, `evals/scenarios.py`, `evals/grading.py`, hay rubric để làm case pass.
- **Không commit** API key hay trace chứa dữ liệu nhạy cảm.
- Chương trình **không tự đọc `.env`** — phải đặt biến môi trường thủ công.
- Tests offline dùng `FakeProvider`, **không cần key** và **không tốn token**; hãy tận dụng tối đa trước khi chạy eval thật.
