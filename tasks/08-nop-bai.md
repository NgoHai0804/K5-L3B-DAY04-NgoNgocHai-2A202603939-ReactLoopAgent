# Task 08 — Đóng gói và nộp bài chính

**Thời lượng gợi ý:** ~15 phút · **Phụ thuộc:** 07 · **Điểm:** — (bắt buộc để được chấm)

## Mục tiêu

Chuẩn bị đúng bộ hồ sơ nộp bài chính (tối đa 85 điểm) và ghi chú phân tích, đảm bảo đã xóa thông tin nhạy cảm.

## Hồ sơ nộp (theo GUIDE mục "Nộp bài" và `SUBMISSION.md`)

- [x] `agent.py` — bản đã hoàn thiện và pass tests.
- [x] `eval_before.json` — kết quả lần 1 (đủ 22 case, điểm khác `null`).
- [x] `eval_after.json` — kết quả lần 2.
- [x] `NOTES.md` — ghi chú phân tích đủ 5 mục:
  - Mục 1–2: case được chọn (ID) + evidence trong trace.
  - Mục 3: thay đổi đã implement.
  - Mục 4: so sánh before/after (điểm, số model calls, token/chi phí).
  - Mục 5: giới hạn (judge, snapshot, một lần chạy, kiểm tra URL chỉ là hiện diện...).
- [x] Demo: **1 trace nhiều bước** + trình bày **1 lỗi agent** (xem mục "Demo" bên dưới).
- [x] Checklist theo `SUBMISSION.md`: tên repo đúng, không có `.env`/key, hai file eval cùng cấu hình.

## Kiểm tra an toàn trước khi nộp

```powershell
# xem nhanh file có chứa chuỗi nghi ngờ key không (điều chỉnh theo biến key của bạn)
Select-String -Path agent.py, eval_before.json, eval_after.json -Pattern "sk-", "AIza", "api[_-]?key", "Bearer"
```

- [x] Không có API key thật trong bất kỳ file nào. (Các hit `sk-`/`Bearer` chỉ là false positive: `ta**sk-**related` trong câu trả lời và tên biến môi trường `*_API_KEY` trong `providers.py`.)
- [x] `.env` cục bộ **không** được track: `git ls-files` chỉ thấy `.env.example`. Cần xóa/rotate key trước khi push (xem ghi chú cuối task).
- [x] Không có trace chứa dữ liệu nhạy cảm.

## Checklist chất lượng theo rubric

| Tiêu chí | Tự kiểm |
|---|---|
| Agent loop (30) | gọi model+tool, append assistant nguyên vẹn, đúng tool-call ID, nhiều calls, dừng đúng |
| Robustness/trace (15) | `max_steps`, invalid JSON, tool error, `length`; trace debug được, không key |
| Eval & phân tích (20) | 2 lần chạy cùng cấu hình; đọc case + trace; evidence; giới hạn judge |
| Cải tiến & trình bày (20) | cải tiến hợp lý; validation nguồn; tests pass; so sánh; lỗi còn lại |

## Điều kiện hoàn thành (DoD)

- [x] Đủ 3 file artifact (`agent.py`, `eval_before.json`, `eval_after.json`) + `NOTES.md` + demo.
- [x] `python -m unittest discover -s tests -v` pass toàn bộ ở trạng thái nộp (`Ran 37 tests ... OK`).
- [x] Không có thông tin nhạy cảm trong file nộp.
- [x] `NOTES.md` điền đủ 5 mục + header (Họ và tên, MSSV).

## Trạng thái đóng gói (đã kiểm tra)

| Hạng mục | Kết quả | Bằng chứng |
|---|---|---|
| Unit tests | Pass | `python -m unittest discover -s tests -v` → `Ran 37 tests in 0.240s OK` |
| `eval_before.json` | Hợp lệ | 22/22 scored, `unscored=0`, `score_0_100=79.23` |
| `eval_after.json` | Hợp lệ | 22/22 scored, `unscored=0`, `score_0_100=83.64` |
| Cùng cấu hình 2 file eval | Khớp | `config` trùng: `openrouter` / `openai/gpt-4o-mini` (agent + judge), `max_steps=8`, `dataset=core` |
| Secret trong file nộp | Không có | `Select-String` trên `agent.py`, `eval_before.json`, `eval_after.json`, `NOTES.md`: chỉ false positive |
| `.env` | Không track | `git ls-files` chỉ thấy `.env.example` |

## Demo

**1) Trace nhiều bước — case A1** (`eval_after.json`, `completed`, 70/100):

```text
step 1 MODEL finish=tool_calls
step 1 TOOL get_status(provider=github,     component=Actions)
step 1 TOOL get_status(provider=cloudflare, component=Spectrum)
step 2 MODEL finish=tool_calls
step 2 TOOL get_incident_details(provider=github,     incident_id=t89g2q1800fr)  -> url=https://stspg.io/lv4q3xm9r0rk
step 2 TOOL get_incident_details(provider=cloudflare, incident_id=c60k3g4vsoc1)  -> url=https://www.cloudflarestatus.com/incidents/c60k3g4vsoc1
step 3 MODEL finish=stop  -> final answer kèm URL cả hai dịch vụ
```

→ Agent gọi nhiều tool qua nhiều bước, giữ nguyên assistant message, trả đúng `tool_call_id` cho mỗi call và chèn nguồn.

**2) Lỗi agent — case A7 (lần 1)** (`eval_before.json`, 75/100, `evidence 15/20`):

- `step 2` gọi `get_incident_details` trả `ok=True`, `url=https://www.cloudflarestatus.com/incidents/c60k3g4vsoc1`; nhưng final answer **không** chứa URL nào đã quan sát (`ans_has_url=False`) → judge trừ `evidence`.
- Nguyên nhân: thiếu ràng buộc dẫn nguồn ở tầng agent loop. Đã khắc phục bằng **source-URL validation**; lần 2 A7 đã dẫn URL (`ans_has_url=True`) và tăng 75→80.

## Việc còn lại (cần bạn thực hiện)

- **Đổi tên repo** về đúng quy ước: `K5-L3A-DAY04-NgoNgocHai-2A202603939-ReactLoopAgent` (dùng `L3B` nếu bạn thuộc lớp L3B). Repo hiện tại: `NgoHai0804/K5-L3-Day4-ReactLoopAgent`.
- **Commit & push** các thay đổi: `agent.py`, `NOTES.md`, `tasks/06-phan-tich-va-cai-tien.md`, `tasks/07-chay-eval-lan-2.md`, `tasks/08-nop-bai.md`, `eval_after.json`.
- **Xóa/rotate key** trong `.env` (đang ở plaintext cục bộ; đã gitignore nên không vào repo, nhưng key thật thì nên thu hồi).
- **Dán link repo lên LMS** (https://aithucchien.vinuni.edu.vn) trước hạn.

## Bước tiếp theo

→ (tùy chọn) `09-bonus-ket-noi-tools.md` — chỉ làm sau khi bài chính đã xong.
