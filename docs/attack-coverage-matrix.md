# Ma trận kiểm thử phát hiện thay đổi đặc quyền

Điền kết quả baseline trước khi thêm custom rule. Lượt đo lại dùng cùng ca, phạm vi log và tiêu chí alert. Dùng cột bằng chứng để gắn Event ID, rule ID, timestamp UTC và vị trí log đã ẩn dữ liệu nhạy cảm.

| Ca | Nguồn thử | Hành vi và telemetry kỳ vọng | Alert mục tiêu kỳ vọng | Baseline | Sau tuning | Bằng chứng |
|---|---|---|---|---|---|---|
| `LOC-01` | Live · `WIN-01` | Thêm tài khoản test vào nhóm Administrators cục bộ (4732), sau đó thu hồi (4733) | Có cảnh báo cấp quyền nhóm được giám sát. Sự kiện thu hồi bổ sung timeline. | TBD | TBD | |
| `LOC-02` | Live · `WIN-01` | Thay đổi thành viên nhóm bảo mật cục bộ thông thường (4732/4733) | Không | TBD | TBD | |
| `AD-AUDIT` | Live · `DC01` | Kiểm tra luồng log bằng thay đổi thành viên nhóm global thông thường (4728/4729) | Không | TBD | TBD | |
| `AD-01` | Log mẫu · AD | Tài khoản được thêm vào nhóm global đặc quyền (4728). Có thể bổ sung log đăng nhập và thu hồi (4624/4672, 4729). | Có cảnh báo cấp quyền nhóm được giám sát. | TBD | TBD | |
| `AD-02` | Log mẫu · AD | Thay đổi thành viên nhóm global thông thường (4728/4729) | Không | TBD | TBD | |
| `CTX-01` | Log mẫu · tùy chọn | Đăng nhập có đặc quyền (4672) | Không có cảnh báo thay đổi thành viên nhóm chỉ từ 4672. | TBD | TBD | |
| `AD-03` | Live hoặc log mẫu · tùy chọn | Thay đổi thuộc tính của đối tượng AD đã cấu hình audit (5136). Ghi rõ thuộc tính và ca đối chứng. | Xác định trước khi thử theo thuộc tính được giám sát. | TBD | TBD | |

## Cách ghi kết quả

- Với ca **live**, ghi ba tín hiệu độc lập: đã thu nhận event, có alert chung và có alert khớp tiêu chí. Dùng nguồn log gốc và bằng chứng từ Wazuh để xác nhận đã thu nhận.
- Với **log mẫu**, ghi bộ thử nhận mẫu hay báo lỗi, rule ID khớp và kết quả đối chiếu tiêu chí. Nếu thử nhiều mẫu trong một ca, giữ cùng điều kiện chạy cho cả hai lượt.
- Dùng `Chưa chạy` hoặc `Chưa xác minh` khi thiếu bằng chứng và ghi lý do. Tổng hợp kết quả theo tập ca có đủ dữ liệu ở cả hai lượt, kèm số ca bỏ trống. Đếm theo ca để một ca có nhiều alert vẫn chỉ tính một lần.
- Tỷ lệ phát hiện là số ca kỳ vọng có alert đã khớp tiêu chí chia cho tổng số ca kỳ vọng có alert. Tỷ lệ báo sai trên bộ ca đối chứng là số ca đối chứng kích hoạt alert mục tiêu chia cho tổng số ca đối chứng. Báo kết quả live và log mẫu ở hai dòng riêng.
- Alert thay đổi nhóm đặc quyền đánh dấu sự kiện cần xem xét. Ghi bối cảnh phê duyệt trong hồ sơ điều tra.

Xem cách chạy từng ca ở [Phase 03](phases/03-privilege-baseline.md) và cách đo lại ở [Phase 04](phases/04-detection-as-code.md).
