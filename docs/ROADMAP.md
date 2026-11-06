# Roadmap: Detection-as-Code Purple Team Platform

Phiên bản nâng cao của Purple-Team SOC Lab. Thời lượng dự kiến: **14-16 tuần**, làm tuần tự theo phase, mỗi phase phải có bằng chứng đo được trước khi sang phase sau.

## 1. Ý tưởng khác biệt

Hầu hết portfolio SOC dừng ở "cài Wazuh, chạy vài attack, chụp dashboard". Bản này đi xa hơn theo đúng cách đội detection ở công ty làm việc:

```
Sigma rule (Git) -> CI: lint + test + compile -> Deploy lên Wazuh
        ^                                              |
        |                                              v
Coverage heatmap  <-  Atomic Red Team / Caldera chạy tự động, kiểm tra rule có bắn không
        ^                                              |
        +------------ Incident / gap -> viết rule mới -+
```

Nói một câu trong phỏng vấn: "Tôi quản lý detection như code, mỗi rule đi qua CI, và được kiểm chứng tự động bằng tấn công mô phỏng, có số đo coverage trước và sau."

## 2. Cơ sở research (và độ tin cậy)

Nguồn chủ yếu là portfolio cá nhân, blog và trang vendor, không phải số liệu tuyển dụng chính thức. Hãy coi là xu hướng, không phải sự thật tuyệt đối.

- **Detection-as-code** là mẫu lặp lại trong các repo tốt: lint, validate, compile Sigma sang nhiều backend (Splunk SPL, Sentinel KQL, Elastic), rồi test hành vi thật bằng Atomic Red Team. Một tác giả tách rõ: CI kiểm tra cấu trúc, còn lab kiểm tra rule có thật sự bắt được kỹ thuật. Xem [detection-as-code](https://github.com/ijaz-aj/detection-as-code) và [bài write-up](https://zionboggan-preview.pages.dev/detection-as-code.html).
- **Wazuh có hướng chính thức** cho việc này: [Wazuh ruleset as code](https://wazuh.com/blog/wazuh-ruleset-as-code-rac/) (đồng bộ rule đã review lên manager sau khi merge, có thể dùng self-hosted runner khi Wazuh ở mạng nội bộ). Đây là nguồn đáng tin nhất trong danh sách.
- **Vòng kiểm chứng**: chạy atomic test, kiểm tra SIEM, tune rule, chạy lại; xuất heatmap ATT&CK Navigator. Atomic Red Team có độ phủ rộng nhất, Caldera mạnh ở emulation tự động. Xem [so sánh của Red Canary](https://redcanary.com/blog/comparing-red-team-platforms/) (bài cũ, con số cần kiểm tra lại trên repo hiện tại).
- **Thị trường (entry-level, quốc tế)**: SIEM xuất hiện trong khoảng 78% tin, Security+ khoảng 70% theo một phân tích; EDR, networking và OS cơ bản được kỳ vọng; KQL chủ yếu ở mid-level (là điểm cộng); cloud fundamentals và AI/automation literacy đang tăng. Xem [Tier 1 SOC Analyst](https://insightglobal.com/jobs/find_a_job/north-carolina/morrisville/tier-1-soc-analyst/job-535368/) và [Abnormal: SOC Analyst 2026](https://abnormal.ai/blog/soc-analyst). Con số lấy từ blog vendor, chỉ mang tính tham khảo.
- **Thị trường Việt Nam**: search không tìm được tin tuyển dụng Viettel/FPT/VNPT/CMC nào, nên **chưa có dữ liệu VN**. Việc cần làm: tự đọc 15-20 tin trên TopCV, ITviec, VietnamWorks, rồi ghi lại công cụ lặp lại nhiều nhất (xem Phase 0).
- **LLM triage**: có repo và luận văn dùng LLM làm co-pilot cho Tier 1 trên Wazuh; kết luận chung là cần human-in-the-loop. Tôi **không tìm thấy** nguồn nói về cách kiểm soát hallucination, nên phần này phải tự thiết kế và tự đo. Xem [Wazuh + Shuffle + TheHive lab](https://github.com/guideops/SOC-Automation-Lab) và [luận văn đánh giá LLM agent](https://www.utupub.fi/handle/11111/19158).
- **Cloud**: Sentinel có trial 31 ngày (thông tin từ blog 2024, cần kiểm tra lại trang giá của Microsoft), đặt daily cap thấp để tránh tốn tiền. Xem [Sentinel lab notes](https://blog.51sec.org/2024/10/sentinel-lab-notes.html).

## 3. Các phase

### Phase 0 - Chuẩn bị (tuần 0, 2-3 ngày)
- Đọc 15-20 tin tuyển SOC trên TopCV/ITviec/VietnamWorks, lập bảng tần suất công cụ và chứng chỉ. Điều chỉnh roadmap theo kết quả.
- Chốt phần cứng (RAM, CPU) và hypervisor. Gợi ý tối thiểu 16 GB RAM, nên 32 GB nếu chạy cả Wazuh, TheHive, AD.
- Tạo repo GitHub, đặt remote, push scaffold hiện có.
- **Xong khi:** có bảng khảo sát tin tuyển dụng và repo public.

### Phase 1 - Hạ tầng và log pipeline (tuần 1-3)
- Dựng AD (DC + 2 client), Linux web server, pfSense/OPNsense + Suricata, Kali.
- Sysmon (cấu hình SwiftOnSecurity hoặc tương đương), Auditd, Wazuh agent, đẩy log về Wazuh.
- Vẽ sơ đồ kiến trúc, ghi lại các lỗi gặp phải và cách sửa (phần này thể hiện bạn tự làm).
- **Xong khi:** thấy log Sysmon, Windows Security, Auditd, Suricata trong Wazuh; có sơ đồ và hướng dẫn dựng lại.

### Phase 2 - Baseline attack simulation (tuần 4-5)
- Chạy kill chain thủ công + Atomic Red Team: PowerShell (T1059.001), brute force (T1110), LSASS dump (T1003.001), Kerberoasting (T1558.003), lateral movement (T1021), persistence (T1053.005), exfil (T1041).
- Ghi nhận **trước khi viết rule nào**: kỹ thuật nào Wazuh mặc định bắt được, kỹ thuật nào không. Đây là baseline cho mọi con số sau này.
- **Xong khi:** `docs/attack-coverage-matrix.md` có cột "before" điền đầy đủ.

### Phase 3 - Detection-as-Code (tuần 6-9) - trọng tâm của project
- Viết 15-25 rule Sigma, mỗi rule có metadata ATT&CK, mức độ, false-positive notes.
- GitHub Actions: `sigma check` -> test schema -> compile sang Wazuh/Splunk SPL/Sentinel KQL -> fail build nếu lỗi.
- Triển khai rule lên Wazuh theo cách Wazuh RaC (self-hosted runner nếu Wazuh ở mạng lab).
- **Vòng kiểm chứng tự động:** script chạy từng atomic test, truy vấn Wazuh API xem alert có xuất hiện không, ghi pass/fail.
- Sinh ATT&CK Navigator layer (heatmap) từ kết quả; so sánh before/after.
- **Xong khi:** PR thêm rule tự chạy CI; có heatmap before/after; có bảng detection rate, số false positive trước và sau tuning.

### Phase 4 - SOAR và incident response (tuần 10-11)
- Wazuh -> Shuffle -> TheHive: tự tạo case, enrich IOC (VirusTotal, AbuseIPDB).
- Playbook: brute force, phishing, malware/ransomware.
- 3-5 incident report đầy đủ theo [mẫu](incident-report-template.md), mỗi báo cáo từ một kịch bản tấn công thật trong lab.
- **Xong khi:** một alert đi hết chuỗi tự động, có case trong TheHive; có ít nhất 3 báo cáo.

### Phase 5 - Điểm nhấn nâng cao (tuần 12-14), chọn 1-2 trong 3
- **A. Cloud SOC (khuyến nghị nếu nhắm công ty dùng Microsoft):** Sentinel + Entra ID logs, port 5-8 rule sang KQL, đặt daily cap. Chứng minh rule Sigma biên dịch chạy được trên cả Wazuh lẫn Sentinel.
- **B. DevSecOps cho chính project:** CI có Gitleaks, Semgrep, Trivy, Checkov; pipeline fail khi có lỗ hổng cao. Nếu muốn mục tiêu là ứng dụng có thật, gắn log API gateway vào Wazuh và viết rule cho tấn công API (đã bàn ở trên). Đóng vòng: incident SOC dẫn đến thêm một gate vào CI.
- **C. LLM triage có kiểm soát:** LLM chỉ gợi ý và tóm tắt, quyết định cuối cùng do người; **đo độ chính xác** trên bộ alert đã gán nhãn (bao nhiêu gợi ý đúng, sai, bịa). Nếu không đo thì không ghi vào CV. Đây là phần ít nguồn tham khảo nhất nên rủi ro cao nhất.

### Phase 6 - Đóng gói (tuần 15-16)
- README viết lại với kết quả thật, ảnh dashboard, heatmap, demo GIF/video 3-5 phút.
- 1-2 bài write-up (Medium hoặc GitHub Pages) kể lại: một gap bạn tìm ra, cách sửa, kết quả.
- Cập nhật CV và LinkedIn; chuẩn bị 5 câu hỏi phỏng vấn về project (xem mục 6).

## 4. Chỉ số cần đo (điền số thật, không bịa)

| Chỉ số | Cách đo |
|---|---|
| Số kỹ thuật ATT&CK mô phỏng | Đếm trong coverage matrix |
| Số rule Sigma tự viết | Đếm trong repo |
| Detection rate trước/sau | Số atomic test được bắt / tổng số |
| Số false positive trước/sau tuning | Đếm alert sai trong cùng khoảng thời gian chạy baseline |
| Thời gian từ alert đến case | Đo trong lab (đây là MTTD/MTTR của lab, ghi rõ là lab) |
| Số incident report | Đếm file |

## 5. Mẫu bullet CV (chỉ dùng khi đã có số thật)

> **Detection-as-Code Purple Team Platform** | GitHub link
> - Xây SOC lab (AD, pfSense/Suricata, Wazuh) và quản lý X rule Sigma như code: CI lint/test/compile, tự triển khai lên Wazuh.
> - Kiểm chứng tự động bằng Atomic Red Team trên Y kỹ thuật ATT&CK; nâng detection rate từ A% lên B%, giảm Z false positive; xuất ATT&CK Navigator heatmap.
> - Tự động hóa triage với Shuffle + TheHive; viết N báo cáo incident theo quy trình NIST.

## 6. Câu hỏi phỏng vấn nên chuẩn bị

1. Rule nào bạn viết bị false positive nhiều nhất, và bạn tune như thế nào?
2. Kỹ thuật nào Wazuh không bắt được mặc định, vì sao?
3. Tại sao dùng Sigma thay vì viết rule Wazuh trực tiếp?
4. Atomic test pass nghĩa là gì, và nó **không** chứng minh điều gì? (Lab khác môi trường thật, attacker thật sẽ né.)
5. Nếu alert này xuất hiện trong môi trường thật, bạn điều tra thế nào?

## 7. Rủi ro và lưu ý trung thực

- **Đây là lab, không phải kinh nghiệm production.** Ghi rõ trên CV và README. Detection rate trong lab đo trên tấn công bạn tự chọn, không đại diện cho attacker thật.
- **Đừng làm Phase 5 khi Phase 3 chưa xong.** Một project có phần lõi vững tốt hơn nhiều project dở dang.
- **Không dùng project của người khác** (kể cả repo API đã xem ở trên nếu không phải của bạn) làm sản phẩm của mình. Có thể dùng làm nguồn cảm hứng hoặc fork có ghi nguồn.
- Chỉ chạy tấn công mô phỏng trong mạng lab cô lập. Các atomic test thay đổi trạng thái hệ thống và có thể kích hoạt cảnh báo.
- Không commit mật khẩu, key, IP thật, file ảnh VM vào repo.
- Con số từ blog vendor (78%, 70%, giá Sentinel) cần kiểm tra lại ở nguồn gốc trước khi dùng trong tài liệu chính thức.
