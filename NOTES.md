# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Công Tĩnh
MSSV: 2A202603373

## 1. Case được chọn

- **ID case:** A5
- **Câu hỏi:** Tôi cần danh sách đầy đủ tất cả incident GitHub Actions trong năm 2025. Hãy tìm dữ liệu và nói rõ độ đầy đủ của kết quả, đừng bịa.
- **Điểm ở lần chạy đầu:** 88/100.

**Vấn đề phát hiện:**

Câu trả lời ban đầu đã nhận biết giới hạn công cụ chỉ tìm trong 50 incident gần nhất. Tuy nhiên, Agent vẫn suy luận về khoảng thời gian của các incident chưa được hiển thị, khi chưa có đủ bằng chứng xác nhận. Ngoài ra, các URL hướng dẫn bên ngoài chưa được kiểm chứng bằng công cụ.

## 2. Bằng chứng trong trace

**Trace trước cải thiện:**

Agent gọi `get_status` với các tham số:
- `provider`: github
- `component`: Actions

Kết quả tool trả về:

- `matching_incidents_in_last_50`: 23.
- Chỉ có 5 incident mẫu được hiển thị.
- Thời điểm snapshot: 2026-11-09T03:50:47 UTC.
- Nguồn: https://www.githubstatus.com.
- Ghi chú của tool: Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.

**Đoạn trace minh họa:**

```text
{
  "event": "tool",
  "step": 1,
  "name": "get_status",
  "arguments": {
    "provider": "github",
    "component": "Actions"
  },
  "result": {
    "matching_incidents_in_last_50": 23,
    "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố."
  }
}
```

**Nguyên nhân có thể xảy ra:**

Agent chưa phân biệt rõ giữa dữ liệu quan sát được và suy luận chưa có bằng chứng. Việc bổ sung kiến thức bên ngoài đôi khi khiến câu trả lời vượt quá phạm vi dữ liệu mà tool cung cấp.

## 3. Thay đổi đã thực hiện

**Vị trí thay đổi:** `agent.py`, phần `SYSTEM_PROMPT`.

Các nội dung được bổ sung:

- Quy định sử dụng bằng chứng từ tool trước khi đưa ra kết luận.
- Nhấn mạnh giới hạn tìm kiếm trong 50 incident gần nhất.
- Không kết luận incident không tồn tại khi công cụ không tìm thấy.
- Không suy đoán định dạng ID, chức năng lọc hoặc phân trang nếu chưa có bằng chứng.
- Phân biệt thông tin đã xác minh và gợi ý chưa được kiểm chứng.
- Yêu cầu câu trả lời ngắn gọn, trực tiếp và có nguồn.

**Lý do lựa chọn:**

Các thay đổi tập trung xử lý nguyên nhân khiến A5 bị trừ điểm, không sửa dữ liệu kiểm thử hoặc tiêu chí chấm điểm.

## 4. So sánh hai lần chạy

| Chỉ số | eval_before.json | eval_after.json |
|---|---:|---:|
| Tổng điểm | 94.59 | 95.14 |
| Số case đã chấm | 22 | 22 |
| Số case chưa chấm | 0 | 0 |
| Điểm A5 | 88 | 87 |

**Kết quả chạy thử riêng A5:**

- File: `test_after_A5.json`.
- Điểm: 92/100.
- Kết quả này khác với điểm A5 trong lần chạy toàn bộ 22 case là 87/100.

Do kết quả có sự dao động giữa các lần chạy, sử dụng điểm trong `eval_after.json` làm căn cứ chính thức khi so sánh hai bộ đánh giá.

**Lỗi đã cải thiện:**

- Trong lần chạy thử riêng, Agent đã gọi công cụ lấy chi tiết 5 incident.
- Câu trả lời có ID, thời gian, trạng thái và URL nguồn khớp với trace.
- Đã nói rõ 18 incident còn lại trong 23 kết quả khớp chưa được hiển thị.
- Đã đánh dấu gợi ý trang lịch sử GitHub là chưa được kiểm chứng bằng công cụ.

**Lỗi còn lại:**

Trong lần đánh giá toàn bộ, A5 đạt 87 điểm. Giám khảo nhận xét câu trả lời vẫn có nhận định rằng 5 incident đều xảy ra vào tháng 11/2026, trong khi incident thứ năm chưa được mở chi tiết để xác minh ngày tạo. Câu mở đầu cũng dễ gây hiểu nhầm rằng không có incident năm 2025.

Điểm tổng thể của bộ đánh giá tăng từ 94.59 lên 95.14, tăng 0.55 điểm.

## 5. Giới hạn

- Điểm do LLM Judge chấm chỉ phản ánh đánh giá theo rubric và dữ liệu của lần chạy đó, không chứng minh Agent hoàn toàn chính xác.
- Snapshot chỉ chứa tối đa 50 incident gần nhất, vì vậy không thể chứng minh danh sách đầy đủ cho năm 2025.
- Chỉ có 5 incident được trả về chi tiết trong lần chạy ban đầu; 18 incident còn lại chưa được xác minh.
- Mỗi phiên bản chỉ chạy một lần trong một số phép thử nên kết quả có thể chịu ảnh hưởng bởi sự biến động của mô hình và dữ liệu.
- Điểm A5 có sự dao động giữa lần chạy riêng và lần chạy toàn bộ.
- Số token và chi phí API có thể thay đổi theo số lượt gọi mô hình và công cụ.

**Kết luận:**

Sau khi điều chỉnh `SYSTEM_PROMPT`, điểm tổng của bộ đánh giá tăng từ 94.59 lên 95.14, với 22/22 case được chấm. Lần chạy thử riêng A5 đạt 92 điểm, tuy nhiên trong lần chạy toàn bộ, A5 đạt 87 điểm. Điều này cho thấy Agent đã có một số cải thiện về cách sử dụng bằng chứng, nhưng vẫn cần tiếp tục hạn chế suy luận vượt quá dữ liệu được xác minh.