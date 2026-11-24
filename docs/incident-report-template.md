# Báo cáo điều tra thay đổi đặc quyền

| Trường | Nội dung |
|---|---|
| Mã báo cáo | IR-000 |
| Analyst và ngày | |
| Mã ca trong ma trận | |
| Nguồn bằng chứng | Ca live hoặc log mẫu |
| Mức độ và trạng thái | |

## 1. Tóm tắt

Ghi alert nào mở điều tra, thay đổi quyền diễn ra ở đâu, mức ảnh hưởng và kết luận hiện tại.

## 2. Alert và phạm vi

| Trường | Giá trị |
|---|---|
| Máy hoặc domain | |
| Wazuh rule ID và Event ID | |
| Tài khoản thực hiện thay đổi | |
| Tài khoản được thêm hoặc gỡ | |
| Tên hoặc SID của nhóm | |
| Phạm vi quyền | Máy trạm hoặc domain |
| Phê duyệt | Mã yêu cầu thay đổi hoặc trạng thái chưa xác minh |

## 3. Phương pháp và timeline

Ghi bộ lọc đã dùng, khoảng thời gian UTC và liên kết tới bằng chứng. Với ca nhiều sự kiện, ghi trường dùng để đối chiếu các bản ghi và mức độ chắc chắn của mối liên hệ.

| Thời gian UTC | Sự kiện | Nguồn, Event ID hoặc liên kết |
|---|---|---|
| | Cấp quyền | |
| | Phiên đăng nhập có đặc quyền nếu xác minh được | |
| | Phát hiện và điều tra | |
| | Thu hồi quyền hoặc kết quả kiểm tra trạng thái hiện tại | |

## 4. Kết luận và phản ứng

- Ghi nhận thay đổi đã phê duyệt, thay đổi mô phỏng ngoài quy trình hoặc trạng thái chưa đủ dữ liệu.
- Nêu hành động đã thực hiện hoặc mô phỏng, cách xác nhận quyền đã được thu hồi và việc cần theo dõi tiếp.
- Liệt kê chỉ dấu, mapping MITRE ATT&CK và giới hạn điều tra nếu có bằng chứng phù hợp.

## 5. Bài học cho detection

Ghi rule bao phủ ca nào, các cảnh báo sai tìm thấy, thay đổi nên đưa vào lần kiểm thử sau và liên kết tới kết quả trước/sau.
