# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Ngọc Vinh
MSSV: 2A202603877

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent lần 1: Agent liệt kê các incident tìm thấy từ năm 2026 rồi kết luận không có incident GitHub Actions nào trong năm 2025.
- Điểm ở lần chạy 1: 15/100

## 2. Bằng chứng trong trace

Ở lần chạy đầu, tool trả về `matching_incidents_in_last_50: 23` và ghi chú rằng chỉ tìm trong 50 incident gần nhất, không phải toàn bộ lịch sử. Dù vậy, agent kết luận không có incident trong năm 2025. Trace không cung cấp bằng chứng để xác nhận một danh sách đầy đủ cho năm đó.

```text
Tool: matching_incidents_in_last_50 = 23
Tool note: Chỉ tìm theo chuỗi trong 50 incident gần nhất;
           không tìm thấy không chứng minh không có sự cố.
Answer trước: “Không có bằng chứng nào cho thấy đã có sự cố
               GitHub Actions trong năm 2025.”
```

Sau thay đổi, agent nêu rõ giới hạn 50 incident và nói rằng kết quả không chứng minh không có incident trong năm 2025. Tuy nhiên, câu trả lời vẫn viết “không có sự cố nào từ năm 2025”; judge ghi nhận cần nhấn mạnh hơn rằng không thể xác nhận điều đó. Vì vậy lỗi được cải thiện nhưng chưa giải quyết hoàn toàn.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Cập nhật `SYSTEM_PROMPT` trong `agent.py`. Nêu rõ giới hạn 50 incident gần nhất, không suy luận sự vắng mặt từ kết quả tìm kiếm rỗng, và không đoán trạng thái khi tool lỗi hoặc timeout.
- Vì sao chọn thay đổi này: Trace A5 cho thấy agent biến dữ liệu giới hạn thành kết luận lịch sử chắc chắn. Cùng nguyên tắc về bằng chứng cũng áp dụng khi tool timeout.
- Kiểm tra: `py -m unittest discover -s tests -v` — 40 tests passed.

## 4. So sánh hai lần chạy

Hai lần chạy dùng cùng lệnh eval và cấu hình OpenRouter / GPT-4o-mini; mỗi lần chấm đủ 22/22 case.

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Điểm tổng | 77.59/100 | 77.50/100 |
| Điểm case A5 | 15/100 | 75/100 |
| Số model calls của agent (đếm `event: model` trong trace của 22 case) | 53 | 46 |
| Token / chi phí riêng case A5 (agent + judge) | 7,994 tokens / khoảng $0.00157125 | 3,244 tokens / khoảng $0.00079575 |

- Lỗi được cải thiện: A5 tăng từ 15 lên 75; câu trả lời sau thay đổi đã nêu giới hạn dữ liệu và nói kết quả không chứng minh rằng không có incident năm 2025.
- Lỗi còn lại hoặc mới xuất hiện: A5 vẫn có câu “không có sự cố nào từ năm 2025”, nên cần làm rõ hơn rằng dữ liệu không thể xác nhận điều đó. H7 vẫn được 50/100: câu trả lời nêu Cloudflare timeout nhưng sau đó vẫn lặp lại yêu cầu sai rằng nên coi dịch vụ là operational. Một số điểm case khác tăng giảm; cần đọc answer và trace trước khi kết luận đó là hồi quy thực sự.

## 5. Giới hạn

- Điểm do LLM judge chấm là ước lượng; judge có thể diễn giải khác nhau và điểm không thay thế việc đọc expected, answer và trace. Điểm eval cũng không tự động là điểm bài lab.
- Snapshot là dữ liệu cố định, chỉ có tối đa 50 incident gần nhất; không đại diện cho trạng thái live và không đủ để chứng minh lịch sử đầy đủ ngoài phạm vi đó.
- Mỗi phiên bản chỉ chạy eval một lần nên kết quả không chứng minh độ ổn định. Việc dùng cùng model cho agent và judge cũng có thể tạo thiên lệch; thay đổi điểm nhỏ giữa hai lần chạy không nhất thiết phản ánh cải thiện hay suy giảm có ý nghĩa.
