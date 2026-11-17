# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.80 | Có (cả ba ngưỡng) | dùng |
| helpful | 100 | 0.375 | 1.0 | 0.30 | Không (κ và R dưới ngưỡng) | loại |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | H-081 | FAIL | PASS | Judge đúng về nội dung, người đúng về tone | Câu trả lời khẳng định đơn O-107 "chưa đủ điều kiện", trong khi claim C-001 (cùng đơn) phải được hoàn 35.000đ. Lỗi là sai thông tin, không phải giọng điệu. Nhãn người ghi chú "đọc nhầm ticket". Tiêu chí tone không nên chấm nội dung đúng sai. |
| 2 | H-042 | FAIL | PASS | Người đúng | Câu ngắn, lịch sự, đủ thông tin số tiền và thời gian. Judge phạt vì "thiếu xin lỗi", nhưng định nghĩa tone trong repo không yêu cầu xin lỗi. Rubric chưa nói rõ điều này. |
| 3 | H-008 | PASS | FAIL | Người đúng | "Em đã hoàn tiền cho anh rồi nha" xưng hô suồng sã với khách. Judge bỏ sót vì không coi xưng hô là vi phạm tone. |
| 4 | H-029 | PASS | FAIL | Người đúng | "không cần nhắn thêm" mang giọng ra lệnh với khách đang chờ. Judge chỉ thấy phần lịch sự đầu câu. |
| 5 | H-070 | PASS | FAIL | Người đúng | Emoji trong câu trả lời khiếu nại. Judge không phạt vì không có quy tắc về emoji. |

Ghi chú về phần còn lại: 4 trường hợp judge báo nhầm (FP) và 6 trường hợp bỏ sót (FN) của `tone_ok` cho thấy tiêu chí chưa đủ cụ thể về xưng hô, emoji và lời hứa ngoài hệ thống. Với `helpful`, phần lớn 21 trường hợp FN là sai số tiền hoặc sai xưng hô; judge chỉ đọc câu trả lời, không đọc ledger, nên không thể phát hiện sai số tiền.

## Kết luận

- **Tiêu chí bị loại và lý do.** `helpful`: ma trận tp 9, fp 0, fn 21, tn 70. Precision đạt 1.0 nhưng recall chỉ 0.30 và κ = 0.375, dưới ngưỡng. Judge gần như không báo lỗi, nên bỏ sót 21/30 trường hợp người đánh FAIL. Tiêu chí này cũng yêu cầu đúng số tiền, việc judge không đọc được ledger nên không phù hợp.
- **Tiêu chí giữ lại.** `tone_ok`: ma trận tp 24, fp 4, fn 6, tn 66. P = 0.857, R = 0.80, κ = 0.757, đạt cả ba ngưỡng. Tuy vậy, các case audit cho thấy rubric cần làm rõ hơn (xem hành động bên dưới).
- **Rủi ro còn lại.**
  - Judge chỉ được kiểm trên 100 mẫu replay công khai, chưa có tập held-out thật.
  - Nhãn người chưa rõ đã được kiểm bởi bao nhiêu người. Case H-081 cho thấy nhãn cũng có thể sai.
  - Hiệu chỉnh Rogan-Gladen cho `helpful` ra tỉ lệ fail 0.3 trong khi tỉ lệ quan sát là 0.09. Con số này không đáng tin vì recall thấp.
- **Hành động sửa rubric.**
  1. `helpful`: loại khỏi bộ đo. Kiểm tra số tiền chuyển sang code grader `RefundAmountCorrect` và metric `reply_ledger_mismatch` (FM-08), đã thêm trong M3.
  2. `tone_ok`: bổ sung định nghĩa về xưng hô với khách, emoji trong ngữ cảnh khiếu nại, và lời hứa ngoài kết quả hệ thống (liên quan đến case 3, 4, 5).
  3. `tone_ok`: không phạt câu ngắn chỉ vì thiếu lời xin lỗi (case 2), trừ khi rubric yêu cầu rõ điều đó.
  4. `tone_ok`: tách việc đúng sai thông tin ra khỏi tone (case 1). Tone không nên chấm nội dung sai.
  5. Nhãn: kiểm lại H-081 đối chiếu với ticket trước khi giữ hay sửa nhãn.
