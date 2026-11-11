# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Minh Khôi
MSSV: 2A202603528

## 1. Case được chọn

- ID case: H7
- Câu trả lời của agent: Agent nói không thể xác định trạng thái Cloudflare Spectrum do timeout, nhưng diễn đạt thêm rằng không có bằng chứng cho thấy dịch vụ không hoạt động; điều đó có thể bị hiểu là đang nghiêng về trạng thái bình thường.
- Điểm ở lần chạy 1: 45/100

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
get_status(provider="github", component="actions")
get_status(provider="cloudflare", component="spectrum")
Kết quả Cloudflare: ok=false, error="TimeoutError"
Final answer: "không thể xác nhận trạng thái" nhưng cũng nói "không có bằng chứng cho thấy nó không hoạt động".
```

Tool timeout chỉ cho biết không lấy được dữ liệu. Có thể agent chưa tách bạch đủ rõ trạng thái chưa xác minh với trạng thái hoạt động bình thường, nhất là khi câu hỏi thúc ép kết luận.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Bổ sung chỉ dẫn trong system prompt ở `agent.py`: timeout/rate limit không chứng minh dịch vụ khỏe hay lỗi; phải báo chưa xác minh được và không suy đoán dù người dùng yêu cầu.
- Vì sao chọn thay đổi này: Nhắm trực tiếp vào lỗi H7 mà không thay đổi dữ liệu, fixtures hay cách chấm. Thêm test kiểm tra chỉ dẫn và việc timeout được trả lại như tool observation.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 82.73/100 (22/22 case) | 78.86/100 (22/22 case) |
| Điểm case đã chọn | H7: 45/100 | H7: 60/100 |
| Số model calls | 51 | 50 |
| Token, chi phí nếu có | 41,769 token (32,307 prompt; 9,462 completion) | 48,882 token (38,746 prompt; 10,136 completion); chi phí xem dashboard OpenRouter |

- Lỗi đã cải thiện một phần: H7 tăng từ 45 lên 60 trong lần chạy đầy đủ mới nhất, nhưng trước đó dao động đến 70. Agent/system prompt yêu cầu nêu timeout là trạng thái chưa xác minh, nhưng điểm thấp hơn ở lần chạy trước-sau cho thấy thay đổi chưa chứng minh được cải thiện tổng thể.
- Lỗi còn lại hoặc mới xuất hiện: Điểm tổng giảm từ 82.73 xuống 78.86; các điểm eval dao động đáng kể giữa các lần chạy. So sánh này không chứng minh nguyên nhân là thay đổi prompt, vì model/judge có tính biến thiên. H6 cũng từng nhận 0 ở một lần chạy trước. Cần đọc trace/expected, cân nhắc thêm kiểm tra deterministic nếu tiếp tục cải tiến.

## 5. Giới hạn

Điểm LLM judge chỉ là ước lượng và có thể sai hoặc thiên lệch. Snapshot là dữ liệu cố định, không đại diện cho trạng thái live. Mỗi phiên bản chỉ chạy một lần nên chưa đo được độ dao động giữa các lần chạy. Cải thiện system prompt không đảm bảo mọi câu trả lời sẽ đúng; cần kiểm tra trace và bằng chứng.
