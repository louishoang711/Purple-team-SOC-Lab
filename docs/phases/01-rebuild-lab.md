# Phase 01 · Dựng lại lab

Dựng một Windows host chạy ba VM trên cùng mạng host-only. Cuối phase, máy trạm vào được domain và host mở được Wazuh Dashboard.

## 1. Chuẩn bị máy và bộ cài

1. Cài VMware Workstation trên Windows host. Kiểm tra Task Manager → Performance → CPU để xác nhận ảo hóa đã bật.
2. Chuẩn bị ISO Ubuntu Server được Wazuh hỗ trợ, Windows Server có **Desktop Experience** để dùng Server Manager, và Windows client bản **Pro** để join domain. Tải bộ cài từ nhà phát hành và ghi lại phiên bản thực tế. [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html) · [Cài Windows Server](https://learn.microsoft.com/windows-server/get-started/install-windows-server) · [Yêu cầu Windows 11](https://learn.microsoft.com/windows/whats-new/windows-11-requirements)
3. Nếu host vẫn có 16 GB RAM, cài và cập nhật từng VM trước khi bật cả ba. Có thể thử cấp `WAZUH01` **5 GB**, `DC01` **3 GB**, `WIN-01` **4 GB** để chừa khoảng 4 GB cho host trước phần bộ nhớ VMware dùng thêm. Đây là cấu hình thử cho lab, thấp hơn các mức khuyến nghị 8 GiB của Wazuh all-in-one và 4 GB của Windows Server Desktop Experience. Nếu cần tài nguyên lúc cài Wazuh, tắt hai VM Windows và cấp 8 GiB cho Ubuntu trong lúc cài. [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html) · [Yêu cầu Windows Server](https://learn.microsoft.com/windows-server/get-started/hardware-requirements) · [Yêu cầu Windows 11](https://learn.microsoft.com/windows/whats-new/windows-11-requirements)

## 2. Tạo mạng host-only

1. Trong VMware, mở **Edit → Virtual Network Editor**. Chọn hoặc tạo một VMnet kiểu **Host-only**. Bật **Connect a host virtual adapter to this network** để host truy cập dashboard. Tắt DHCP trên VMnet này nếu dùng sơ đồ IP tĩnh bên dưới. [Virtual Network Editor](https://knowledge.broadcom.com/external/article?legacyId=1018697)
2. Chọn subnet ít xung đột. Bảng sau dùng `192.168.149.0/24` làm ví dụ. Nếu subnet này đã thuộc LAN, VPN hoặc VMnet NAT, chọn subnet khác và thay IP trong toàn bộ hướng dẫn. Dùng `ipconfig` trên Windows host để xem IP của adapter VMnet.
3. Gắn card mạng lab của cả ba VM vào đúng VMnet host-only. Khi cần Internet để cài đặt hoặc cập nhật, có thể gắn tạm một card NAT riêng vào VM đang chuẩn bị. Gỡ hoặc ngắt card NAT trước khi đo baseline. Với hai VM Windows, ngắt card NAT trước khi cấu hình DNS và join domain. Để trống default gateway trên card host-only.

| Máy | Vai trò | IP mẫu trên VMnet | DNS trên card lab |
|---|---|---|---|
| `HOST` | Windows vật lý, VMware và trình duyệt | Kiểm tra adapter VMnet. Ví dụ `192.168.149.1` | Giữ nguyên cấu hình của host |
| `WAZUH01` | Ubuntu Server với Wazuh all-in-one | `192.168.149.20/24` | Có thể để trống trên card lab |
| `DC01` | Windows Server chạy AD DS và DNS | `192.168.149.10/24` | `192.168.149.10` |
| `WIN-01` | Windows Pro, thành viên domain | `192.168.149.30/24` | `192.168.149.10` |

## 3. Cài Wazuh trên Ubuntu

1. Đặt hostname `WAZUH01`. Dùng `ip -br addr` trong console Ubuntu để xác định card nối VMnet host-only. Trong cấu hình Netplan hiện có, đặt mục dưới đây vào `ethernets`, đổi `ens33` thành tên card thực tế và giữ mục NAT tạm trên card riêng nếu đang tải gói. Dùng `sudo netplan try` từ console VM rồi xác nhận IP bằng `ip -br addr`. [Canonical về cấu hình mạng Ubuntu Server](https://ubuntu.com/server/docs/explanation/networking/configuring-networks/)

   ```yaml
   ens33:
     dhcp4: false
     addresses:
       - 192.168.149.20/24
   ```
2. Theo mục **Installing Wazuh** của Quickstart để cài manager, indexer và dashboard trên cùng VM. Ghi phiên bản đã cài và lưu thông tin đăng nhập do trình cài tạo trong nơi quản lý mật khẩu. [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html)
3. Trên `HOST`, dùng `Test-NetConnection 192.168.149.20 -Port 443` rồi mở `https://192.168.149.20` bằng trình duyệt. Nếu chứng chỉ tự ký, đối chiếu địa chỉ và chứng chỉ từ console VM trước khi đăng nhập. [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html)
4. Khi hoàn tất cài đặt, ngắt NAT và tải lại dashboard để xác nhận luồng host-only. Giữ dashboard trên mạng lab. Các cổng cho agent ở Phase 2 là `1514/TCP` để gửi sự kiện và `1515/TCP` để đăng ký theo cấu hình mặc định. [Cổng mặc định Wazuh](https://documentation.wazuh.com/current/getting-started/architecture.html#required-ports)

## 4. Dựng Active Directory

1. Trong `DC01`, đặt tên máy rồi chạy `ncpa.cpl`. Mở **Properties → Internet Protocol Version 4 (TCP/IPv4)** của card lab. Điền IP và DNS theo bảng, mask `255.255.255.0` và để trống gateway. Khởi động lại nếu đổi tên máy.
2. Mở **Server Manager → Add Roles and Features**, thêm **Active Directory Domain Services** và management tools. Chọn **Promote this server to a domain controller → Add a new forest**. Dùng `soclab.local`, giữ DNS Server được chọn và lưu mật khẩu DSRM riêng. Hoàn thành kiểm tra điều kiện rồi cài và khởi động lại. [Microsoft Learn về cài AD DS và DNS](https://learn.microsoft.com/windows-server/networking/core-network-guide/core-network-guide#install-ad-ds-and-dns-for-a-new-forest)
3. Trong **Active Directory Users and Computers**, tạo OU `Lab Workstations` cho máy trạm và một tài khoản lab chuẩn, ví dụ `lab-user`, để kiểm tra đăng nhập. Dùng riêng tài khoản quản trị lab khi cấu hình.
4. Xác nhận bản ghi dịch vụ của domain từ PowerShell trên `DC01`, rồi chạy lại từ `WIN-01` sau khi đã đặt DNS máy trạm.

   ```powershell
   Resolve-DnsName -Name '_ldap._tcp.dc._msdcs.soclab.local' -Type SRV -Server '192.168.149.10'
   ```

## 5. Join máy trạm vào domain

1. Đặt tên VM Windows Pro thành `WIN-01`, đặt IP `192.168.149.30/24` và DNS `192.168.149.10`. Kiểm tra lại bản ghi SRV ở bước trước.
2. Mở **Settings → Accounts → Access work or school → Connect → Join this device to a local Active Directory domain**. Nhập `soclab.local`, xác thực bằng tài khoản được phép join và khởi động lại. [Microsoft Learn về join domain](https://learn.microsoft.com/windows-server/identity/ad-ds/manage/join-computer-to-domain)
3. Trên `DC01`, di chuyển đối tượng máy `WIN-01` vào OU `Lab Workstations` để gắn GPO ở Phase 2. Đăng nhập `WIN-01` bằng tài khoản lab chuẩn và chạy `whoami`. Từ PowerShell quản trị trên máy trạm, kiểm tra `Test-ComputerSecureChannel` trả về `True`.
4. Đối chiếu ngày giờ của hai VM Windows và Ubuntu theo UTC. Nếu join thất bại, kiểm tra DNS trên card host-only, bản ghi SRV, giờ của hai máy và tài khoản được phép join trước khi sửa cấu hình khác.
5. Bật đồng thời ba VM. Kiểm tra đăng nhập domain trên `WIN-01` và tải dashboard trên host. Trên Ubuntu, chạy `systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard` và xác nhận cả ba dịch vụ trả `active`. Nếu Windows host liên tục dùng đĩa để phân trang và lab chậm, đóng ứng dụng không cần thiết rồi thử lại. Kiểm tra từng cặp VM để tìm nguyên nhân nếu cấu hình còn thiếu tài nguyên, sau đó kiểm tra lại cả ba cùng lúc.

## Hoàn thành khi

- [ ] Ghi lại phiên bản OS, Wazuh, VMnet, IP và tài nguyên của từng VM. [Điều kiện lab](../lab-conditions.md)
- [ ] `HOST` mở được dashboard qua VMnet host-only sau khi ngắt NAT.
- [ ] `DC01` phân giải bản ghi dịch vụ AD và `WIN-01` join được domain.
- [ ] `WIN-01` đăng nhập được bằng tài khoản lab chuẩn và có trong OU `Lab Workstations`.
- [ ] Khi bật cả ba VM, dịch vụ Wazuh ở trạng thái `active` và host vẫn chạy ổn định.
- [ ] Trên mỗi VM, chỉ còn card lab được kết nối khi đo. Kiểm tra không có default route ra mạng ngoài bằng `ip route show default` và `ip -6 route show default` trên Ubuntu hoặc `Get-NetRoute -DestinationPrefix '0.0.0.0/0','::/0'` trên Windows.

[Roadmap](../ROADMAP.md) · [Tiếp theo · Phase 2 — Thu log](02-telemetry.md)
