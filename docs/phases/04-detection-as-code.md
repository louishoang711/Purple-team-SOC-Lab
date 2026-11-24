# Phase 04 · Detection as Code

## Mục tiêu

Đưa rule Wazuh XML vào Git, kiểm thử với cùng bộ log của baseline, triển khai thủ công lên `WAZUH01` và chứng minh có thể quay lại bản trước.

## Đầu vào

Hoàn thành Phase 03 và xác nhận phiên bản Wazuh, bộ rule mặc định, ngưỡng alert, dữ liệu và kỳ vọng của từng ca. [Phase 03](03-privilege-baseline.md) · [Ma trận coverage](../attack-coverage-matrix.md)

## 1. Chọn mục tiêu của rule

1. Mở kết quả baseline cho `LOC-01` và `AD-01`. Nếu rule sẵn có đã nhận diện đúng nhóm đặc quyền, giữ kết quả đó là một lần phát hiện. Mã nguồn Wazuh có rule mặc định cho thay đổi nhóm Administrators (60154) và Domain Admins (60159). Xác nhận kết quả thực tế trên phiên bản của lab. [Ruleset Windows Security của Wazuh](https://github.com/wazuh/wazuh/blob/master/ruleset/rules/0580-win-security_rules.xml)
2. Bắt đầu với `LOC-01` trên máy trạm. Chọn cải tiến đo được, ví dụ tách cấp quyền và thu hồi quyền trong tiêu đề alert, đồng thời giữ thông tin người thực hiện, thành viên và nhóm. Dùng `LOC-02` làm đối chứng.
3. Mở rộng đến `AD-01` khi rule đầu đã qua kiểm thử. Dùng `AD-02` làm đối chứng offline và `AD-AUDIT` làm đối chứng cho đường thu log từ DC. `CTX-01` kiểm tra 4672 đứng riêng chỉ cung cấp ngữ cảnh. Để `AD-03` ở phần mở rộng nếu chưa thu được 5136 đúng phạm vi.

## 2. Viết và kiểm thử rule XML

1. Tạo `03-detection-engineering/wazuh-rules/local-privilege.xml`. Dùng ID còn trống trong dải custom **100000–120000** và giữ rule gốc của Wazuh trong thư mục hệ thống. Từ kết quả baseline, chọn rule cha qua `if_sid` và xác minh tên trường đã được decoder đọc. [Wazuh Custom rules](https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html) · [Cú pháp rule](https://documentation.wazuh.com/current/user-manual/ruleset/rules/ruleset-xml-syntax/rules.html)
2. Với `LOC-01`, xét `win.system.eventID` 4732 để tìm thao tác thêm thành viên và 4733 cho thao tác xóa. Xác định nhóm qua `win.eventdata.targetSid` bằng SID đầy đủ `S-1-5-32-544` và giới hạn ở máy trạm theo tên máy đã xác minh trên log. Với `AD-01`, dùng 4728/4729 và SID Domain Admins của domain đang được kiểm thử. Giữ SID trong fixture tổng hợp nhất quán với SID mà cấu hình kiểm thử sử dụng. [Microsoft Security Identifiers](https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-identifiers)
3. Đặt mô tả nói rõ thao tác, nhóm và máy. Giữ các trường actor (`subjectUserName` hoặc `subjectUserSid`), thành viên (`memberSid`, `memberName` nếu có) và nhóm (`targetSid` hoặc `targetUserName`) trong dữ liệu điều tra. Xác minh tên trường trên dữ liệu đã decode. Đặt ngưỡng rule sao cho alert mong muốn được ghi ra theo ngưỡng hiện tại của manager. [Ngưỡng alert của Wazuh](https://documentation.wazuh.com/current/user-manual/manager/alert-management.html)

   Mẫu cho phần thêm thành viên của `LOC-01`, khi baseline khớp rule 60154 và tên máy trong log là `WIN-01.soclab.local`. Thay ID hoặc mẫu tên máy theo kết quả ở bước 1 và 2.

   ```xml
   <group name="privilege_monitoring,">
     <rule id="100100" level="12">
       <if_sid>60154</if_sid>
       <field name="win.system.eventID">^4732$</field>
       <field name="win.system.computer" type="pcre2">(?i)^win-01[.]soclab[.]local$</field>
       <description>Local Administrators member added on $(win.system.computer)</description>
     </rule>
   </group>
   ```

   Nếu theo dõi cả thao tác thu hồi, thêm rule có ID riêng cho 4733 và mô tả tương ứng trong cùng nhóm. Giữ rule mặc định cho các máy và loại sự kiện nằm ngoài phạm vi.

4. Lưu log mẫu đã ẩn dữ liệu và kỳ vọng dưới `03-detection-engineering/tests/`. Dùng một dòng log thô cho mỗi ca `wazuh-logtest`. Nếu lấy từ Wazuh, trích phần log thô trong `full_log` khi có trường này. Chạy trên một Wazuh manager thử nghiệm với cùng phiên bản hoặc chạy trước lúc restart manager ở bước triển khai. [Wazuh Logtest](https://documentation.wazuh.com/current/user-manual/ruleset/testing.html)

   ```bash
   sudo /var/ossec/bin/wazuh-logtest < LOC-01-add.log
   ```

5. Đọc decoder và rule ID ở kết quả. Ca dương tính phải khớp rule dự kiến đúng thao tác và nhóm. Ca đối chứng phải chạy hết quá trình decode và giữ kết quả dự kiến của rule gốc hoặc không sinh alert mục tiêu. Lỗi decode, lỗi nạp rule hay lỗi tiến trình đều làm ca kiểm thử thất bại.

## 3. Đưa kiểm thử vào CI

1. Tạo workflow ở `.github/workflows/` với trigger `pull_request`. Dùng runner Ubuntu do GitHub cấp và khớp với phiên bản OS được Wazuh hỗ trợ. Trong job, checkout repo, kiểm tra cấu trúc XML, ID trùng và các mẫu log còn thiếu.
2. Trên runner, thêm nguồn gói chính thức của Wazuh và cài đúng phiên bản gói `wazuh-manager` đã ghi ở Phase 02. Nạp rule vào thư mục custom rule trên runner, chạy `sudo /var/ossec/bin/wazuh-analysisd -t`, rồi chạy `sudo systemctl start wazuh-manager` và xác nhận dịch vụ `active`. Bài test này dùng manager và log mẫu cục bộ. [Cài đặt Wazuh Manager](https://documentation.wazuh.com/current/installation-guide/wazuh-server/step-by-step.html#installing-the-wazuh-manager) · [Danh sách gói](https://documentation.wazuh.com/current/installation-guide/packages-list.html)
3. Kiểm tra rule ID, level và decoder bằng tùy chọn `-U <rule-id:alert-level:decoder-name>`, với đúng một dòng log trong mỗi lần gọi. Với ca đối chứng, đối chiếu kết quả decoder và rule dự kiến. Dừng workflow khi công cụ lỗi hoặc kết quả khác kỳ vọng. [Tham chiếu tùy chọn wazuh-logtest](https://documentation.wazuh.com/current/user-manual/reference/tools/wazuh-logtest.html)
4. Kiểm tra trong một pull request rằng một kỳ vọng cố ý sai làm CI đỏ, rồi khôi phục kỳ vọng đúng để CI xanh. Lưu liên kết đến lần chạy xanh và commit đã kiểm thử. Workflow chỉ dùng rule và mẫu log đã ẩn dữ liệu trong repo.

## 4. Triển khai và thử rollback

1. Chọn commit mà CI đã kiểm tra. Trên `WAZUH01`, lưu bản XML đang dùng cùng quyền sở hữu và quyền đọc vào thư mục backup ngoài `/var/ossec/etc/rules/`. Nếu đây là lần đầu triển khai, ghi lại rằng chưa có file trước đó.
2. Đưa file từ commit vào `/var/ossec/etc/rules/local-privilege.xml`, giữ quyền truy cập tương đương các custom rule của manager. Chạy kiểm tra cấu hình và các ca logtest trước khi restart dịch vụ. Nếu một trong các kiểm tra lỗi, khôi phục file trước và kiểm lại cấu hình. [Wazuh về xác minh cấu hình](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/verifying-configuration.html)

   ```bash
   sudo /var/ossec/bin/wazuh-analysisd -t
   sudo /var/ossec/bin/wazuh-logtest < LOC-01-add.log
   ```

3. Khi các kiểm tra đạt, restart và kiểm tra manager. Mở Dashboard, xác nhận hai agent hoạt động và ghi mã commit cùng mốc UTC của lần triển khai.

   ```bash
   sudo systemctl restart wazuh-manager
   sudo systemctl is-active wazuh-manager
   ```

4. Chạy lại `LOC-01`, `LOC-02` và `AD-AUDIT` theo Phase 03. Đối chiếu event gốc và alert trong đúng khoảng thời gian của lần chạy mới để xác nhận cả đường từ agent đến Dashboard. Dùng `wazuh-logtest` với dữ liệu `AD-01` và `AD-02`, rồi ghi rõ đây là kết quả kiểm thử bộ rule.
5. Nếu service không lên hoặc alert khác kỳ vọng, xem `sudo journalctl -u wazuh-manager -n 50 --no-pager`. Khôi phục bản XML và quyền truy cập từ backup. Với lần triển khai đầu tiên, chuyển file custom ra khỏi thư mục rule để quay về bộ rule cũ. Chạy lại kiểm tra cấu hình, restart, kiểm tra trạng thái và thu một event kiểm tra mới. Ghi lại kết quả của thao tác rollback.
6. Trong lab, diễn tập bước 5 kể cả khi triển khai thành công. Xác nhận phiên bản cũ hoạt động, sau đó triển khai lại commit đã kiểm thử và xác nhận một alert mới trước khi sang Phase 05.

## 5. So sánh trước và sau

Chạy cùng bộ log mẫu trên bộ rule mặc định và bộ rule có bổ sung. Dùng cùng cấu hình và tiêu chí phát hiện đã chốt. Ghi kết quả live của những ca được chạy lại ở dòng dữ liệu live tương ứng. [Ma trận coverage](../attack-coverage-matrix.md)

- **Tỷ lệ phát hiện** bằng số ca dương tính có phát hiện đúng chia cho số ca dương tính đã kiểm thử. Ghi cả tử số và mẫu số.
- **Tỷ lệ báo nhầm** bằng số ca đối chứng phát alert mục tiêu chia cho số ca đối chứng đã kiểm thử. Thay đổi nhóm đặc quyền được phê duyệt vẫn là một phát hiện đúng vì rule giám sát hành vi thay đổi. Trạng thái phê duyệt được đối chiếu khi điều tra.
- Nếu tỷ lệ phát hiện giữ nguyên, ghi rõ phần giá trị đã đo được, chẳng hạn phân biệt cấp với thu hồi quyền hoặc có kiểm thử hồi quy.

## Xong khi

- [ ] Ít nhất một rule theo dõi nhóm đặc quyền nằm trong Git, có ca dương tính và đối chứng.
- [ ] Pull request có kết quả CI với đúng phiên bản Wazuh.
- [ ] `LOC-01` xác minh được từ event trên Windows tới alert trong Dashboard sau khi deploy.
- [ ] `AD-01` và `AD-02` có kết quả từ bộ thử rule, cùng bằng chứng thu log DC của `AD-AUDIT`.
- [ ] Có kết quả so sánh trước/sau và đã thử khôi phục rule cùng trạng thái dịch vụ.

Tiếp theo [Phase 05 · Điều tra và đóng gói](05-investigate-and-present.md) · [Roadmap](../ROADMAP.md)
