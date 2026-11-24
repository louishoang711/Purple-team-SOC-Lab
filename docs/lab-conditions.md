# Điều kiện lab

Điền thông số thực tế của lượt dựng lại và cập nhật mỗi khi điều kiện đo thay đổi. Trạng thái ban đầu là `Chưa xác nhận`.

## Máy và mạng

| Hạng mục | Giá trị mẫu hoặc cần ghi | Giá trị thực tế |
|---|---|---|
| `HOST` | Phiên bản Windows, VMware, CPU, RAM và dung lượng trống | Chưa xác nhận |
| `WAZUH01` | Ubuntu Server, `192.168.149.20/24`, phiên bản Wazuh, RAM và đĩa | Chưa xác nhận |
| `DC01` | Windows Server, `192.168.149.10/24`, phiên bản agent | Chưa xác nhận |
| `WIN-01` | Windows Pro, `192.168.149.30/24`, phiên bản agent và Sysmon | Chưa xác nhận |
| Domain và DNS | `soclab.local`, Windows client dùng địa chỉ DC làm DNS | Chưa xác nhận |
| VMnet | Tên mạng host-only, subnet và địa chỉ adapter host | Chưa xác nhận |
| Khả năng truy cập | Host vào được Dashboard, client truy cập được DNS của DC, ba VM ở mạng host-only trong lúc kiểm thử | Chưa xác nhận |
| Trạng thái khôi phục | Tên snapshot nếu có, hoặc điểm khôi phục thủ công và ghi chú dựng lại | Chưa xác nhận |

## Telemetry và điều kiện đo

| Hạng mục | Cần ghi | Giá trị thực tế |
|---|---|---|
| Windows Security | Kênh log, audit policy hiệu lực trên máy trạm và DC, Event ID đã kiểm tra | Chưa xác nhận |
| Sysmon | Máy đã cài, phiên bản, nguồn và bản cấu hình, Event ID đã kiểm tra | Chưa xác nhận |
| Lưu event | Trạng thái lưu archive, lưu ở đâu và khoảng thời gian giữ dữ liệu nếu bật | Chưa xác nhận |
| Wazuh | Ruleset mặc định khi đo baseline, ngưỡng alert, phiên bản custom rule khi đo lại | Chưa xác nhận |
| Thời gian | Múi giờ, trạng thái đồng bộ và quy ước dùng UTC để đối chiếu | Chưa xác nhận |
| Bộ ca | Phiên bản bộ log mẫu hoặc commit, ID ca, nguồn và nhãn kỳ vọng | Chưa xác nhận |
| Hoạt động nền | Khung giờ đo, thay đổi quản trị đã biết và nguồn log lành tính dùng làm đối chứng | Chưa xác nhận |

Lưu cấu hình và log minh họa đã ẩn dữ liệu nhạy cảm. Nếu thay tên hoặc SID trong log mẫu, dùng cùng giá trị thay thế xuyên suốt các sự kiện cần ghép. Ghi vị trí bằng chứng vào [ma trận kiểm thử](attack-coverage-matrix.md).
