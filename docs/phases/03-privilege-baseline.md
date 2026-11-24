# Phase 03 — Đo baseline phát hiện thay đổi đặc quyền

Đo khả năng của ruleset mặc định trên hai lớp quyền của lab. Lớp máy trạm dùng sự kiện thật từ thao tác quản trị được phê duyệt. Lớp đặc quyền domain dùng log mẫu để kiểm thử rule và dựng timeline.

Trước khi đo, xác nhận Wazuh đã nhận Windows Security từ máy trạm và `DC01`. [Phase 02 — Thu telemetry](02-telemetry.md)

## Bước 1 — Cố định điều kiện đo

1. Ghi phiên bản Wazuh manager, phiên bản agent, phiên bản Windows, chính sách audit hiện hành và ngưỡng ghi alert của manager. Đo trên cấu hình mặc định của ruleset. Nếu lab đã nạp custom rule, dùng bản cấu hình lab sạch để đo. [Điều kiện lab](../lab-conditions.md)
2. Ghi mã lượt chạy và mốc giờ UTC. Trong Dashboard, tìm trên `wazuh-alerts-*`, lọc theo agent, Event ID và thời gian thao tác. Mở bản ghi để lưu `rule.id`, mô tả, mức độ, thời gian và ID bản ghi Windows nếu có.
3. Kiểm tra sự kiện gốc trên **Event Viewer → Windows Logs → Security**. Khi cần xác nhận manager đã nhận sự kiện dưới ngưỡng alert, bật lưu trữ JSON theo hướng dẫn Wazuh cho cửa sổ kiểm thử và kiểm tra `archives.json` trên manager. Sau khi thu đủ bằng chứng, đưa cấu hình lưu trữ về mức trước bài kiểm thử. [Wazuh về event logging](https://documentation.wazuh.com/current/user-manual/manager/event-logging.html) · [Wazuh về chỉ mục alert](https://documentation.wazuh.com/current/user-manual/wazuh-indexer/wazuh-indexer-indices.html)

## Bước 2 — Tạo sự kiện thật trên máy trạm và DC

| Ca | Nơi thực hiện | Thao tác | Kỳ vọng |
|---|---|---|---|
| `LOC-01` | `WIN-01` | Dùng tài khoản quản trị lab để thêm tài khoản test vào nhóm local **Administrators**, xác nhận thành viên và gỡ ngay sau khi lấy log. | `4732` cho lần thêm và `4733` cho lần gỡ. Lần thêm là ca dương tính của rule giám sát cấp quyền local admin. |
| `LOC-02` | `WIN-01` | Thêm rồi gỡ cùng tài khoản test trong một nhóm bảo mật local thông thường do bạn tạo cho lab. | Windows vẫn ghi `4732` và `4733`. Rule dành riêng cho local Administrators phải bỏ qua ca này. |
| `AD-AUDIT` | `DC01` | Dùng Active Directory Users and Computers để thêm rồi gỡ tài khoản test trong nhóm bảo mật **Global** thông thường của lab. | `4728` và `4729` phải xuất hiện trong Security log trên DC. Dùng ca này để xác minh đường thu log và lấy mẫu định dạng Wazuh. |

1. Tạo tài khoản test riêng. Trên Windows client, mở **Computer Management → Local Users and Groups → Groups** bằng tài khoản quản trị lab để thực hiện hai ca local. Chạy từng ca riêng và ghi giờ lúc thêm, giờ lúc gỡ trước khi chuyển ca.
2. Trong từng sự kiện, xác định người thực hiện từ `SubjectUserSid` hoặc `SubjectUserName`, thành viên từ `MemberSid` và nhóm từ `TargetSid` hoặc `TargetUserName`. `MemberName` có thể là `-` với nhóm local. Tra SID về tài khoản test khi cần. [Microsoft về event 4732](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4732)
3. Ghi nhận Windows phát sinh sự kiện, Wazuh manager nhận sự kiện và indexer có bản ghi alert. Đối chiếu theo máy nguồn, Event ID, ID bản ghi và thời gian để gắn đúng lần thao tác. Điền kết quả vào ma trận. [Ma trận coverage](../attack-coverage-matrix.md)
4. Xác nhận tài khoản test đã ra khỏi cả hai nhóm local và nhóm AD thường. Lưu log gỡ quyền làm bằng chứng kết thúc ca.

## Bước 3 — Kiểm thử quyền cấp domain bằng log mẫu

1. Từ `AD-AUDIT`, lưu nội dung log đầu vào mà Wazuh đã nhận. Nếu bản ghi Wazuh có `full_log`, lấy giá trị của trường này làm mẫu đầu vào. Giữ cấu trúc sự kiện và thay định danh nhạy cảm bằng giá trị lab nhất quán. Dùng đúng dạng đầu vào đã xác nhận cho mọi mẫu để ruleset trước và sau đọc cùng một cấu trúc.
2. Chuẩn bị log tổng hợp cho `AD-01` với một `4728` có nhóm đích `Domain Admins` và SID nhóm thuộc domain giả lập. Giữ `AD-02` làm ca đối chứng `4728/4729` của nhóm bảo mật global thông thường. Gắn nhãn `synthetic` trong hồ sơ cho từng ca tổng hợp. Dùng danh sách nhóm đích nhất quán giữa log mẫu và rule. [Microsoft về audit nhóm bảo mật](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/audit-security-group-management)
3. Chạy từng mẫu qua **Tools → Ruleset test** trong Wazuh Dashboard hoặc nhập vào `sudo /var/ossec/bin/wazuh-logtest` trên manager. Ghi Event ID đã được decode, các trường actor/member/group, rule ID, mô tả và mức độ. Lưu kết quả dưới dạng **kiểm thử ruleset**. Phép thử từ Dashboard này chạy trong môi trường kiểm thử của ruleset. [Wazuh về kiểm thử rule và decoder](https://documentation.wazuh.com/current/user-manual/ruleset/testing.html)
4. Khi dựng timeline cho `AD-01`, có thể thêm `4624` và `4672` mẫu cho cùng phiên trên cùng máy, rồi một `4729` cho thao tác gỡ. Nối `4624` và `4672` theo cùng máy và Logon ID. Liên hệ lần thêm/gỡ với phiên theo SID tài khoản nhận quyền và thứ tự thời gian. Giữ `CTX-01` là mẫu `4672` độc lập để kiểm tra rule cấp quyền không phát hiện nhầm một phiên đăng nhập có đặc quyền. [Microsoft về event 4672](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4672)
5. Có thể bổ sung `AD-03` với `5136` để nghiên cứu thay đổi đối tượng AD. Nếu cần dữ liệu thật, dùng một thuộc tính lành tính như `description` trên đối tượng test. Bật **Audit Directory Service Changes** trên DC và cấu hình **SACL**, tức thiết lập audit, cho thuộc tính được theo dõi. Ghi DN của đối tượng, tên thuộc tính và tài khoản thực hiện trong bằng chứng. [Microsoft về audit thay đổi AD](https://learn.microsoft.com/windows/security/threat-protection/auditing/audit-directory-service-changes)

## Bước 4 — Ghi và khóa kết quả trước khi chỉnh rule

1. Với mỗi ca đã chạy, lưu giờ UTC, môi trường chạy (`live` hoặc `replay`), bộ log, Event ID kỳ vọng, Event ID quan sát được, kết quả alert, rule ID và đường dẫn bằng chứng đã ẩn dữ liệu nhạy cảm. Ghi rõ ca tùy chọn chưa chạy hoặc phép đo chưa áp dụng.
2. Ghi **telemetry**, **alert chung** và **alert đúng tiêu chí**. Ca dương tính chỉ đạt khi kết quả nhận diện được thay đổi của đúng nhóm đặc quyền và gắn được actor, thành viên, nhóm đích với bằng chứng. Ghi alert chung như một quan sát trong baseline. Cấp quyền đã được phê duyệt vẫn là ca dương tính của rule giám sát thay đổi nhóm. Analyst dùng hồ sơ thay đổi để phân loại sau khi nhận alert.
3. Báo cáo số ca dương tính đạt trên tổng ca dương tính đã chạy và số ca đối chứng bị phát hiện nhầm trên tổng ca đối chứng đã chạy. Tính riêng cho `live` và `replay`. Nếu chưa có ca đủ điều kiện để tính, ghi `N/A`. Giữ cùng bộ mẫu, cách tính và ngưỡng alert khi đo lại sau khi thêm rule. Khi so sánh, dùng tập ca có đủ dữ liệu từ cả hai lượt và ghi số ca bị bỏ trống.
4. Lưu một bản kết quả baseline vào Git trước khi bắt đầu chỉnh rule ở Phase 04. Gắn phiên bản hoặc mã commit của bộ log mẫu vào bản kết quả để có thể chạy lại.

## Nếu thiếu log

| Triệu chứng | Kiểm tra |
|---|---|
| Event Viewer chưa có `4732` hoặc `4728` | Kiểm tra **Audit Security Group Management → Success** có hiệu lực trên đúng máy, nhóm có loại Security và thao tác đã thật sự thay đổi thành viên. |
| Windows có sự kiện nhưng manager chưa thấy | Kiểm tra agent hướng về đúng manager, trạng thái kết nối, cấu hình thu kênh Security và bộ lọc Event ID. Đối chiếu kho archive khi đã bật. |
| Archive có sự kiện nhưng Dashboard không có alert | Xem khung giờ và chỉ mục, rồi kiểm tra rule mặc định và ngưỡng log alert. Nếu đã có trong `alerts.json`, kiểm tra luồng chuyển dữ liệu sang indexer. |
| Ruleset test không decode được mẫu | Đối chiếu log gốc mà manager nhận và dùng một dòng log đầu vào cho mỗi mẫu. Kiểm tra cấu trúc JSON và trường Wazuh decode sau khi ẩn thông tin. |

## Hoàn thành khi

- [ ] Có bằng chứng thu log cho `LOC-01`, `LOC-02` và `AD-AUDIT` từ nguồn Windows đến Wazuh.
- [ ] Có kết quả Ruleset test mặc định cho `AD-01` và `AD-02`.
- [ ] Ma trận có bằng chứng và số đếm ban đầu theo cách chạy. Ca tùy chọn có kết quả hoặc trạng thái chưa chạy.
- [ ] Quyền của tài khoản test trên các nhóm thử nghiệm đã được thu hồi và có log đối chiếu.
- [ ] Phiên bản cấu hình và bộ mẫu đã được ghi để đo lại ở phase tiếp theo. [Phase 04 — Detection as Code](04-detection-as-code.md)
