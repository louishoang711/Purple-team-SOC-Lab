# Phase 02 · Thu log Windows và xác nhận trên Wazuh

Thu Windows Security từ `DC01` và `WIN-01`, cùng Sysmon từ `WIN-01`. Dùng các sự kiện vừa tạo để kiểm tra riêng việc ghi log tại nguồn, chuyển log tới Wazuh và tạo cảnh báo.

Mở [Điều kiện lab](../lab-conditions.md) và ghi các phiên bản, IP, múi giờ, chính sách audit và cấu hình lấy log thực tế trong khi làm.

## 1. Cài và đăng ký Wazuh Agent

1. Từ PowerShell trên `DC01` và `WIN-01`, kiểm tra `Test-NetConnection 192.168.149.20 -Port 1514` và `Test-NetConnection 192.168.149.20 -Port 1515`. Dùng IP thực tế của `WAZUH01` nếu đã đổi sơ đồ mạng. [Cổng mặc định Wazuh](https://documentation.wazuh.com/current/getting-started/architecture.html#required-ports)
2. Tải bộ cài Windows Agent từ Wazuh trong lúc chuẩn bị VM hoặc chuyển bộ cài từ `HOST` vào qua ISO. Trên Wazuh Dashboard, dùng chức năng triển khai agent mới để sinh lệnh cài Windows cho địa chỉ manager `192.168.149.20`. Chạy phần cài đặt MSI bằng PowerShell quản trị, dùng đường dẫn tới file MSI đã tải. Đặt tên agent lần lượt là `DC01` và `WIN-01`. [Triển khai Wazuh Agent trên Windows](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html) · [Biến cài đặt agent](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/deployment-variables/deployment-variables-windows.html)
3. Chạy `Start-Service -Name WazuhSvc` trên mỗi VM nếu dịch vụ đang dừng. Kiểm tra hai agent có trạng thái **Active** trên dashboard. Ghi lại tên và ID agent để tìm log ở các bước sau.
4. Mở `C:\Program Files (x86)\ossec-agent\ossec.conf` trên hai VM. Xác nhận có `Security` trong một khối `<localfile>` với `<log_format>eventchannel</log_format>` và kiểm tra XPath hiện có cho phép các event trong bảng ở bước 2. Nếu cần đổi filter, sửa khối hiện có rồi chạy `Restart-Service -Name WazuhSvc`. [Thu thập Windows Event Channel](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html)

## 2. Cấu hình audit policy

1. Trên `DC01`, mở **Group Policy Management**. Tạo và liên kết GPO `SOC-Lab-Audit-DC` tới OU `Domain Controllers`, rồi tạo và liên kết GPO `SOC-Lab-Audit-Clients` tới OU `Lab Workstations` đã tạo ở Phase 1.
2. Trong mỗi GPO, vào **Computer Configuration → Policies → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Audit Policies**. Bật **Success** cho các subcategory sau. [Microsoft Learn về Advanced Audit Policy](https://learn.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/advanced-audit-policy-configuration)

   | Subcategory | Event cần quan sát | Nguồn trong lab |
   |---|---|---|
   | **Account Management → Audit Security Group Management** | `4732/4733` thêm hoặc xóa thành viên nhóm local | `WIN-01` |
   | **Account Management → Audit Security Group Management** | `4728/4729` thêm hoặc xóa thành viên nhóm global AD | `DC01` |
   | **Logon/Logoff → Audit Logon** | `4624` đăng nhập thành công | Máy tạo phiên đăng nhập |
   | **Logon/Logoff → Audit Special Logon** | `4672` đặc quyền nhạy cảm được gán cho phiên đăng nhập mới | Máy tạo phiên đăng nhập đặc quyền |

3. Trên `DC01` và `WIN-01`, chạy hai lệnh sau bằng quyền quản trị. Xác nhận các subcategory đã bật **Success**. Nếu thiếu, dùng `gpresult /scope computer /r` để kiểm tra GPO có được áp dụng cho máy hay chưa. [Kiểm tra Windows event auditing](https://learn.microsoft.com/defender-for-identity/deploy/configure-windows-event-collection)

   ```powershell
   gpupdate /target:computer /force
   auditpol /get /category:*
   ```

4. Dùng `4672` làm ngữ cảnh cho phiên `4624` khi trùng `Logon ID` trên cùng máy. Phase 3 có bài mở rộng `5136` cho thay đổi đối tượng AD nếu cần thu thêm dữ liệu. [Microsoft Learn về `4672`](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4672) · [Audit Directory Service Changes](https://learn.microsoft.com/windows/security/threat-protection/auditing/audit-directory-service-changes)

## 3. Thu Sysmon trên máy trạm

1. Tải Sysmon từ Microsoft và giải nén vào `WIN-01`. Mở PowerShell quản trị trong thư mục đã giải nén và chạy `.\Sysmon64.exe -i` để dùng cấu hình mặc định cho kiểm tra ban đầu. Nếu bạn đã chuẩn bị một cấu hình Sysmon, cài theo cú pháp `.\Sysmon64.exe -i <config.xml>` và ghi phiên bản file XML vào điều kiện lab. [Microsoft Sysinternals Sysmon](https://learn.microsoft.com/sysinternals/downloads/sysmon)
2. Trong `ossec.conf` của agent trên `WIN-01`, thêm đoạn sau bên trong `<ossec_config>` nếu chưa có. Chạy `Restart-Service -Name WazuhSvc`. Khi cần dữ liệu tiến trình trên DC, có thể lặp bước Sysmon trên `DC01` và xác nhận thêm nguồn log đó. [Thu thập Windows Event Channel](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html)

   ```xml
   <localfile>
     <location>Microsoft-Windows-Sysmon/Operational</location>
     <log_format>eventchannel</log_format>
   </localfile>
   ```

## 4. Kiểm tra luồng log từ nguồn đến dashboard

1. Mở `Wazuh Dashboard → Explore → Discover` và chọn `wazuh-alerts-*` để tìm alert. Khi cần xác nhận raw event trên `WAZUH01`, sửa `<logall_json>` thành `yes` trong khối `<global>` của `/var/ossec/etc/ossec.conf`, chạy `sudo systemctl restart wazuh-manager` và xem `/var/ossec/logs/archives/archives.json` trong lúc kiểm tra. File archive bắt đầu ghi sự kiện mới sau khi bật. Sau khi lưu đủ mẫu, trả thiết lập về giá trị trước đó và khởi động lại manager để áp dụng. [Wazuh Event Logging](https://documentation.wazuh.com/current/user-manual/manager/event-logging.html)
2. Đăng nhập mới bằng tài khoản lab chuẩn trên `WIN-01` và bằng tài khoản quản trị lab trên `DC01`. Mở **Event Viewer → Windows Logs → Security** và xem `4624` mới trên từng máy. Ghi thời gian UTC, tên máy và Event Record ID của sự kiện.
3. Trên `WIN-01`, chạy `cmd.exe /c ver` để tạo một tiến trình lành tính. Trong **Event Viewer → Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**, tìm event `1` mới có `Image` là `cmd.exe`. [Sysmon Process Creation](https://learn.microsoft.com/sysinternals/downloads/sysmon#events)
4. Đối chiếu ba sự kiện theo máy phát, kênh, Event ID và giờ phát sinh. Trong archive hoặc alert JSON của Wazuh, kiểm tra `agent.name` và `data.win.system.eventID`. Trong dashboard, chọn một khoảng giờ hẹp quanh lần kiểm tra. Lưu mỗi nguồn một mẫu đã ẩn dữ liệu nhạy cảm và ghi rõ nó được tìm thấy trong archive hay alert. [Các chỉ mục log của Wazuh](https://documentation.wazuh.com/current/getting-started/use-cases/log-analysis.html)

| Dấu hiệu | Kiểm tra tiếp |
|---|---|
| Agent chưa Active | Xem dịch vụ trên VM, IP manager, cổng `1515/TCP` và `1514/TCP`. |
| Thiếu event ngay trong Event Viewer | Kiểm tra audit policy đang có hiệu lực hoặc cấu hình Sysmon. Tạo lại thao tác sau khi chỉnh. |
| Event có tại nguồn, chưa thấy trong archive | Kiểm tra tên kênh, XPath, trạng thái agent và thời điểm bật archive, rồi tạo sự kiện mới. |
| Event có trong archive, chưa thấy trên `wazuh-alerts-*` | Kiểm tra rule, ngưỡng tạo alert và cửa sổ thời gian. Nếu đã có alert trong `/var/ossec/logs/alerts/alerts.json`, kiểm tra Filebeat và indexer. |

## Hoàn thành khi

- [ ] Dashboard ghi nhận hai agent `DC01` và `WIN-01` đã kết nối.
- [ ] Audit policy hiệu lực cho ba subcategory đã được kiểm tra trên cả hai VM Windows.
- [ ] Wazuh nhận được ít nhất một `4624` từ mỗi VM Windows và một Sysmon event `1` từ `WIN-01`.
- [ ] Điều kiện lab có thời gian kiểm tra, phiên bản, cấu hình audit và đường dẫn bằng chứng.

[Quay lại · Phase 1 — Dựng lab](01-rebuild-lab.md) · [Tiếp theo · Phase 3 — Đo baseline](03-privilege-baseline.md)
