# Roadmap dựng lại Purple-Team SOC Lab

Dựng lại lab trên một Windows host với ba VM. Hoàn thành bài demo phát hiện thay đổi đặc quyền trên máy trạm và trong Active Directory trước khi mở rộng công cụ. Kết quả của lượt dựng mới bắt đầu đo từ lúc các nguồn log được xác nhận lại.

| Phase | Target | Tiêu chí hoàn thành |
|---|---|---|
| [01 · Dựng lại lab](phases/01-rebuild-lab.md) | Nối ba VM qua mạng riêng và dựng domain | Windows client join domain, host mở được Wazuh Dashboard, thông số thực tế được ghi lại. |
| [02 · Thu telemetry](phases/02-telemetry.md) | Xác nhận Windows Security trên `WIN-01` và `DC01`, cùng Sysmon trên `WIN-01` | Có log đối chiếu từ từng máy, đúng nguồn và thời gian. Phân biệt được event đã thu với alert hiển thị. |
| [03 · Đo baseline](phases/03-privilege-baseline.md) | Cố định ca kiểm thử, cấu hình và kết quả ruleset mặc định | Hoàn thành ca cấp quyền local và đối chứng, xác nhận nguồn log nhóm AD và kiểm thử ca nhóm AD đặc quyền bằng log mẫu. |
| [04 · Detection as Code](phases/04-detection-as-code.md) | Đưa rule Wazuh XML và kiểm thử vào Git | Ca khớp và ca đối chứng có kết quả mong đợi, rule triển khai có đường hoàn tác, bảng trước/sau có bằng chứng. |
| [05 · Điều tra và demo](phases/05-investigate-and-present.md) | Đi từ alert tới báo cáo điều tra | Có ít nhất một báo cáo hoàn chỉnh và demo nêu được kết quả, giới hạn và nguồn log. |

## Thước đo cho bản đầu

- Ghi số ca có **alert đúng tiêu chí trên tổng số ca kỳ vọng có alert**. Dùng cùng bộ ca và tiêu chí giữa hai lượt.
- Ghi số **ca đối chứng phát sinh alert mục tiêu trên tổng số ca đối chứng**. Báo riêng cho ca chạy qua agent và log mẫu đưa vào bộ thử rule.
- Ghi thời gian từ event đến alert trên các ca live có đủ timestamp. Nếu mở thêm SOAR, đo riêng khoảng thời gian từ alert đến case.
- Dùng `TBD` cho kết quả chờ đo và ghi lý do cho các ca chưa có đủ dữ liệu. [Ma trận kiểm thử](attack-coverage-matrix.md)

## Sau bản đầu

- **Telemetry cấp AD** · Mở rộng theo dõi thay đổi đối tượng AD với 5136 và thêm ca đối chứng về thuộc tính thường.
- **Sigma đa nền tảng** · Thêm Sigma khi có nhu cầu dùng detection trên nền tảng thứ hai. Đối chiếu kết quả với rule Wazuh trên bộ log tương đương.
- **SOAR** · Thử một luồng từ alert vào case, sau khi báo cáo thủ công đã xác định đủ các trường cần chuyển.
- **Cảm biến mạng** · Bổ sung Suricata khi bài điều tra cần dữ liệu mạng và host còn đủ tài nguyên.
