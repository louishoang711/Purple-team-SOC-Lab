# Phase 05 · Điều tra và đóng gói

## Mục tiêu

Từ một alert thay đổi đặc quyền, dựng được timeline có bằng chứng và trình bày kết quả của lab. Mốc tối thiểu gồm một báo cáo điều tra và một demo ngắn.

## Đầu vào

Chuẩn bị rule, kết quả kiểm thử, ma trận trước/sau và alert `LOC-01` đã xác nhận trong Dashboard. Chuẩn bị dữ liệu tổng hợp `AD-01` nếu muốn trình bày thêm case Domain Admins. [Phase 04](04-detection-as-code.md) · [Ma trận coverage](../attack-coverage-matrix.md)

## 1. Mở case từ alert

1. Chọn alert thật trong lab của `LOC-01` để thực hiện case đầu tiên. Tạo báo cáo Markdown dưới `05-incident-reports/`, ví dụ `IR-001-local-admin.md`. Ghi UTC xuất hiện alert, mức độ, tên máy, Event ID, Wazuh rule ID và mã ca kiểm thử. [Mẫu báo cáo](../incident-report-template.md)
2. Trong Wazuh, lấy bản ghi của alert và log gốc hoặc mã tham chiếu. Ghi ba thành phần riêng gồm người thực hiện, tài khoản bị thay đổi và nhóm. Kiểm tra tên máy sinh event và ghi rõ nguồn của từng dữ kiện.
3. So sánh dữ kiện với lịch hoặc phiếu thay đổi của lab. Đối chiếu nhóm, tài khoản và thời gian trước khi kết luận. Ghi kết quả là thay đổi được phê duyệt, thay đổi ngoài quy trình theo kịch bản, hoặc chưa đủ bằng chứng để xác định.

## 2. Dựng timeline và xác nhận kết quả

1. Với `LOC-01`, ghép 4732 và 4733 theo cùng máy, SID nhóm và SID thành viên. Lưu UTC của thao tác thêm và thu hồi. Kiểm tra lại trạng thái thành viên sau khi thu hồi.
2. Nếu điều tra `AD-01`, đánh dấu toàn bộ case là **tổng hợp**. Dùng 4728 và 4729 để xác định thao tác trên nhóm global. Khi có 4624 và 4672, đối chiếu phiên đăng nhập theo Logon ID trên cùng hệ thống và khung thời gian. Đối chiếu SID tài khoản đăng nhập với SID thành viên của thay đổi nhóm. 4672 mô tả đặc quyền gán cho phiên đăng nhập. Bằng chứng sử dụng một đặc quyền cụ thể cần event phù hợp với hành động đó. [Microsoft Event 4624](https://learn.microsoft.com/windows/security/threat-protection/auditing/event-4624) · [Microsoft Event 4672](https://learn.microsoft.com/windows/security/threat-protection/auditing/event-4672)
3. Ghi tác động dựa trên dữ liệu trong case. Nếu thiếu log đăng nhập hay thu hồi, ghi khoảng trống và nêu dữ liệu cần kiểm tra thêm. Đối với ca mô phỏng ngoài quy trình, ghi phương án xử lý như xác nhận với chủ sở hữu nhóm, thu hồi quyền theo quy trình và kiểm tra lại membership. Đánh dấu rõ hành động nào đã thực hiện trong lab và hành động nào còn đề xuất.
4. Gắn từng phát hiện chính với file log đã ẩn thông tin, trích dẫn alert hoặc ảnh Dashboard. Giữ timestamp và ID nhất quán để người đọc có thể đối chiếu. Ghi cách xử lý đối chứng `LOC-02` hoặc `AD-02` trong báo cáo.

Mẫu timeline để điền trong case.

| Thời điểm UTC | Nguồn và Event ID | Máy | Quan sát | Bằng chứng |
| --- | --- | --- | --- | --- |
| … | Windows Security 4732 | … | Tài khoản test được thêm vào nhóm được giám sát | … |
| … | Wazuh alert | … | Rule phát hiện thay đổi nhóm đặc quyền | … |
| … | Windows Security 4733 | … | Tài khoản test được gỡ khỏi cùng nhóm | … |

## 3. Đóng gói demo

1. Cập nhật README với sơ đồ ba VM, phase đã chạy, phiên bản Wazuh và liên kết đến rule, lần chạy CI, ma trận trước/sau và báo cáo. Ghi kết quả live và kiểm thử trên log mẫu theo đúng nguồn dữ liệu.
2. Quay demo 3–5 phút theo trình tự mở log gốc, xem rule và ca đối chứng, mở alert, rồi giải thích timeline và kết luận. Nếu baseline đã phát hiện, chỉ ra cải tiến thực tế như tiêu đề phân biệt cấp và thu hồi quyền hoặc kiểm thử hồi quy.
3. Nếu bổ sung SOAR, dùng case đầu tiên làm chuẩn để kiểm thử việc tự tạo hồ sơ và gắn đúng alert. Nếu bổ sung Sigma, dùng lại fixture và kỳ vọng để đối chiếu đầu ra của backend đã chọn. Đánh dấu các hạng mục đó trong roadmap theo kết quả thực tế. [Roadmap](../ROADMAP.md)

## Xong khi

- [ ] Có ít nhất một báo cáo điều tra hoàn chỉnh, từ alert đến kết luận và bằng chứng.
- [ ] Có ma trận trước/sau đã điền và phân biệt dữ liệu live với dữ liệu tổng hợp.
- [ ] README dẫn tới rule, kết quả CI, báo cáo và demo 3–5 phút.
