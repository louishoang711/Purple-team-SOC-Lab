# Purple-Team SOC Lab · Detection as Code

Lab SOC cá nhân để thực hành phát hiện thay đổi đặc quyền trên Windows và Active Directory. Mục tiêu là theo dõi được **ai cấp quyền, tài khoản nào nhận quyền, phiên đăng nhập nào có đặc quyền và khi nào quyền được thu hồi**. Rule và bộ log kiểm thử được quản lý bằng Git để so sánh kết quả trước và sau khi tinh chỉnh.

**Trạng thái hiện tại** · Đang bắt đầu dựng lại lab sau khi reset máy host. Kết quả telemetry của lần dựng cũ nằm trong lịch sử Git. Baseline, rule và báo cáo của lần dựng mới sẽ được cập nhật sau khi kiểm chứng.

## Kiến trúc

| Máy | Vai trò |
|---|---|
| `HOST` | Máy Windows vật lý chạy VMware và truy cập Wazuh Dashboard |
| `WAZUH01` | Ubuntu Server VM chạy Wazuh Manager, Indexer và Dashboard |
| `DC01` | Windows Server VM chạy Active Directory và DNS, cài Wazuh Agent |
| `WIN-01` | Windows Pro VM tham gia domain, cài Wazuh Agent và Sysmon |

Ba VM dùng cùng một mạng lab riêng. Xem sơ đồ, địa chỉ mẫu và luồng dữ liệu trong [Kiến trúc lab](docs/architecture.md).

## Bắt đầu

| Phase | Mục tiêu | Trạng thái |
|---|---|---|
| [01 · Dựng lại lab](docs/phases/01-rebuild-lab.md) | Dựng ba VM, nối mạng riêng, join domain và mở Dashboard từ host | Bắt đầu ở đây |
| [02 · Thu telemetry](docs/phases/02-telemetry.md) | Xác nhận Windows Security từ hai máy Windows và Sysmon từ máy trạm | Chờ Phase 01 |
| [03 · Đo baseline](docs/phases/03-privilege-baseline.md) | Đo khả năng nhận diện thay đổi nhóm đặc quyền bằng ruleset mặc định | Chờ Phase 02 |
| [04 · Detection as Code](docs/phases/04-detection-as-code.md) | Kiểm thử rule Wazuh XML, triển khai vào lab và đo lại trên cùng bộ ca | Chờ Phase 03 |
| [05 · Điều tra và demo](docs/phases/05-investigate-and-present.md) | Viết báo cáo có timeline và chuẩn bị demo trước/sau | Chờ Phase 04 |

## Kịch bản chính

- **Máy trạm** · Kiểm tra tín hiệu khi quản trị viên cấp rồi thu hồi quyền Administrators cục bộ cho tài khoản test. Đối chứng bằng thay đổi nhóm thường.
- **Active Directory** · Kiểm tra rule với log đã ẩn thông tin hoặc log tổng hợp mô tả thay đổi nhóm đặc quyền như Domain Admins. Dùng sự kiện đăng nhập có đặc quyền để bổ sung ngữ cảnh và kiểm tra bằng chứng thu hồi quyền.
- **Mở rộng** · Giám sát thay đổi thuộc tính nhạy cảm của đối tượng AD bằng event 5136 sau khi hai ca chính có kết quả.

Ghi trạng thái môi trường tại [Điều kiện lab](docs/lab-conditions.md), kết quả tại [Ma trận kiểm thử](docs/attack-coverage-matrix.md) và dùng [Mẫu báo cáo điều tra](docs/incident-report-template.md) để đối chiếu alert với bằng chứng và phê duyệt. [Roadmap](docs/ROADMAP.md) ghi phạm vi và tiêu chí hoàn thành của toàn project.

## Vị trí lưu kết quả

| Thư mục | Nội dung |
|---|---|
| `01-infrastructure/` | Ghi chú dựng lab và cấu hình mẫu đã bỏ thông tin nhạy cảm |
| `02-attack-simulation/` | Log kiểm thử, nguồn dữ liệu và nhãn của từng ca |
| `03-detection-engineering/` | Rule Wazuh, ca kiểm thử và kết quả trước/sau |
| `04-automation-soar/` | Workflow tạo case sau khi quy trình điều tra thủ công ổn định |
| `05-incident-reports/` | Báo cáo điều tra và timeline đã ẩn dữ liệu nhạy cảm |

Giấy phép MIT. [LICENSE](LICENSE)
