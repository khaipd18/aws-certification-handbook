# SOA-C03 — AWS Systems Manager (Management of EC2 at Scale)

> Thuộc **Domain 3: Deployment, Provisioning, and Automation (22%)** và **Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization (22%)**: quản lý fleet EC2 và on-prem bằng SSM Agent, Run Command, State Manager, Automation, Patch Manager, Session Manager, Parameter Store.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Systems Manager (SSM)** quản lý EC2, server on-prem, VM ở quy mô lớn, cả Windows lẫn Linux.
- Một node thành **managed node** khi có đủ ba điều kiện: **SSM Agent đang chạy**, **quyền IAM** (`AmazonSSMManagedInstanceCore` hoặc **DHMC**), **đường ra HTTPS 443** tới endpoint SSM. Không cần inbound port nào.
- **SSM Document** (JSON/YAML) định nghĩa parameters và actions. **Run Command** chạy một lần; **State Manager** giữ trạng thái theo lịch; **Automation** điều phối nhiều bước, gọi được API AWS.
- **Patch Manager**: patch baseline (vá gì) + patch group (máy nào) qua `AWS-RunPatchBaseline`, chạy trong **Maintenance Window** với rate control. ⚠️ Patch policy (Quick Setup) là cách mới.
- **Session Manager**: shell không cần SSH key, không mở port 22, không bastion; log ra S3 / CloudWatch Logs; CloudTrail ghi `StartSession`.
- **Parameter Store**: Standard 10,000 tham số, 4 KB, miễn phí; Advanced 100,000 tham số, 8 KB, có parameter policies, tính phí.
- **Tag** và **Resource Groups** là cách chọn target hàng loạt và phân bổ chi phí.
- ⚠️ Lỗi thời: Change Manager và Incident Manager **đóng với khách mới từ 07/11/2025**. Tier advanced-instances cho node lai bị bỏ từ 30/06/2026.

---

## 2. Core Concepts

### SSM Agent, IAM và kết nối mạng
- **Là gì:** agent chạy trên node, chủ động kết nối ra endpoint SSM để nhận lệnh. Instance role cần `AmazonSSMManagedInstanceCore`. Node on-prem đăng ký qua Hybrid Activation có ID `mi-`.
- **Dùng khi nào:** private subnet không có NAT → tạo interface endpoint `ssm`, `ssmmessages`, `ec2messages` (thêm `s3`, `logs`, `kms` nếu cần), bật DNS hostnames/support, SG của endpoint cho phép 443 từ instance.
- **Vì sao quan trọng:** node không hiện trong Fleet Manager → kiểm tra theo thứ tự IAM role → mạng/endpoint → agent (đã cài, đang chạy, phiên bản). `AmazonSSMManagedInstanceCore` không tự cấp quyền ghi vào bucket S3 hay log group của bạn.

### Default Host Management Configuration (DHMC)
- **Là gì:** cài đặt account + Region tự biến mọi EC2 thành managed node mà không cần gắn instance profile, dùng role `AWSSystemsManagerDefaultEC2InstanceManagementRole`.
- **Dùng khi nào:** muốn không bỏ sót instance nào, không quản lý role cho từng máy.
- **Vì sao quan trọng:** **bắt buộc IMDSv2**, không hỗ trợ IMDSv1; SSM Agent từ 3.2.582.0; bật riêng từng Region. DHMC chỉ lo quản lý SSM, ứng dụng vẫn cần role riêng để gọi API khác.

### Tags & Resource Groups
- **Là gì:** tag là cặp key–value; Resource Groups nhóm tài nguyên theo tag hoặc CloudFormation stack, phạm vi Region.
- **Dùng khi nào:** chọn target cho Run Command, Patch Manager, Automation; điều kiện trong IAM; cost allocation.
- **Vì sao quan trọng:** cost allocation tag phải **kích hoạt trong Billing** và không hồi tố. Bắt buộc tag chuẩn toàn Organization → Tag Policies + Config rule `required-tags`.

### SSM Documents
- **Là gì:** kịch bản JSON/YAML; loại Command (Run Command, State Manager), Automation (runbook), Session, Package (Distributor).
- **Dùng khi nào:** tên bắt đầu bằng `AWS-` thường là document AWS dựng sẵn: `AWS-RunShellScript`, `AWS-RunPatchBaseline`, `AWS-UpdateSSMAgent`, `AWS-ConfigureAWSPackage`, `AWS-RestartEC2Instance`.
- **Vì sao quan trọng:** document có version, đặt default version để association dùng bản đã kiểm chứng; share được cross-account.

### Run Command
- **Là gì:** chạy một document trên nhiều node **một lần**, chọn target theo ID, tag hoặc Resource Group.
- **Dùng khi nào:** restart service, cài gói, thu log, cài CloudWatch Agent hàng loạt; có thể kích hoạt từ EventBridge.
- **Vì sao quan trọng:** **rate control** (concurrency + error threshold) giới hạn thiệt hại khi script sai. Output trong console bị cắt → gửi S3 / CloudWatch Logs; thông báo trạng thái qua SNS; CloudTrail ghi ai chạy gì.

### State Manager
- **Là gì:** **association** = document + target + lịch; mỗi lần chạy báo Compliant / Non-compliant.
- **Dùng khi nào:** bootstrap phần mềm bắt buộc, đảm bảo cấu hình luôn đúng (antivirus cài sẵn, port 22 đóng), thu Inventory định kỳ.
- **Vì sao quan trọng:** node mới khớp target (ví dụ do ASG tạo) tự được áp association, điều Run Command không làm.

### Automation & runbook
- **Là gì:** runbook là document loại Automation, gọi API AWS (EC2, EBS, AMI, ASG, RDS), có approval step, chạy cross-account / cross-Region.
- **Dùng khi nào:** kích hoạt thủ công, qua **EventBridge**, theo **Maintenance Window**, hoặc **Config remediation**. Ví dụ golden AMI: launch từ source AMI → `AWS-RunPatchBaseline` → stop → create image → terminate → Launch Template mới → cập nhật ASG → Instance Refresh.
- **Vì sao quan trọng:** gọi API thay bạn nên cần **Automation assume role**, không dựa vào instance profile.

### Parameter Store
- **Là gì:** kho key–value `String`, `StringList`, `SecureString` (KMS), có version, phân cấp đường dẫn (tối đa 15 cấp), đọc theo nhánh bằng `GetParametersByPath`.
- **Dùng khi nào:** cấu hình, endpoint, secret không cần rotate. `/aws/service/...` là public parameter của AWS (AMI mới nhất); `/aws/reference/secretsmanager/<id>` đọc secret của Secrets Manager.
- **Vì sao quan trọng:** **Advanced** cần khi giá trị > 4 KB, cần > 10,000 tham số, cần parameter policies (`Expiration`, `ExpirationNotification`, `NoChangeNotification` qua EventBridge) hoặc share cross-account. Rotate tự động → Secrets Manager.

### Patch Manager
- **Là gì:** **patch baseline** (AWS định sẵn không sửa được; custom có approved/rejected, auto-approve sau N ngày) + **patch group** (tag `Patch Group` hoặc `PatchGroup`); thực thi bằng `AWS-RunPatchBaseline`; chế độ **Scan** hoặc **Scan and install**.
- **Dùng khi nào:** vá OS và ứng dụng cho EC2/on-prem, báo cáo compliance, xuất S3; một node thuộc một patch group, một patch group gắn một baseline mỗi OS.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: patch group không dùng trong **patch policy** (Quick Setup) và console không hỗ trợ patch group cho account-Region chưa dùng trước 22/12/2022. Private subnet cần đường tới kho patch của OS.

### Maintenance Windows
- **Là gì:** khung giờ được phép tác động: schedule, duration (kèm cutoff), registered targets, registered tasks (Run Command, Automation, Lambda, Step Functions).
- **Dùng khi nào:** "chỉ được vá 2–4 giờ sáng Chủ nhật", kết hợp rate control và priority của task.
- **Vì sao quan trọng:** khác **Change Calendar** (khai báo khoảng CLOSED cấm thay đổi, Automation kiểm tra trước khi chạy).

### Session Manager
- **Là gì:** shell qua console, CLI, SDK; xác thực bằng IAM; hỗ trợ port forwarding, KMS mã hóa phiên, Run As.
- **Dùng khi nào:** không mở inbound port, bỏ bastion, cần log lệnh đã gõ; tunnel tới RDS/RDP trong private subnet bằng port forwarding.
- **Vì sao quan trọng:** giới hạn ai vào máy nào bằng IAM theo tag instance. `[+]` Idle timeout mặc định 20 phút (1–60); max session duration mặc định không giới hạn (1–1,440 phút).

### Fleet Manager, Inventory, Distributor, OpsCenter
- **Là gì:** **Fleet Manager** quản lý node từ xa, RDP trong trình duyệt; **Inventory** thu metadata phần mềm/OS qua association, gom đa account bằng **Resource Data Sync**; **Distributor** đóng gói phần mềm (zip mỗi OS + manifest trên S3); **OpsCenter** gom OpsItem từ CloudWatch, Config, EventBridge, Security Hub.
- **Dùng khi nào:** desktop Windows trong private subnet → Fleet Manager; "liệt kê phần mềm đã cài" → Inventory (Athena/Quick Sight); duy trì version phần mềm nội bộ → Distributor + State Manager.
- **Vì sao quan trọng:** Inventory mô tả hiện trạng; Compliance đánh giá đúng/sai; Inspector tìm CVE. ⚠️ Lỗi thời: Incident Manager (on-call, escalation) đóng với khách mới từ 07/11/2025.

### `[+]` Thay đổi giá & dịch vụ 2025–2026
- **Là gì:** tier advanced-instances cho node lai bị bỏ từ **30/06/2026** (hết giới hạn 1,000 node); từ **30/09/2026** Session Manager và Run Command trên node lai tính pay-as-you-go.
- **Dùng khi nào:** đề cũ hỏi "vượt 1,000 server on-prem phải lên advanced tier" → kiến thức cũ.
- **Vì sao quan trọng:** Change Manager đóng với khách mới từ 07/11/2025 (AWS gợi ý giải pháp Partner); SSM CloudWatch Dashboard ngừng sau 30/04/2026; dashboard Explorer rời console SSM từ 31/12/2026 (OpsData vẫn giữ). Chi phí thực tế: kiểm tra trang pricing và hóa đơn của account.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Managed node | Node đăng ký với SSM | "not showing in Fleet Manager" |
| `AmazonSSMManagedInstanceCore` | Managed policy cho instance role | "instance cannot be managed" |
| DHMC | Quản lý mọi EC2 không cần instance profile | "without attaching roles to each instance" |
| Run Command | Chạy document một lần trên nhiều node | "run script on 200 instances now" |
| Rate control | Concurrency + error threshold | "limit blast radius" |
| State Manager association | Duy trì trạng thái theo lịch | "ensure software always installed" |
| Automation runbook | Workflow gọi API AWS | "create AMI and update ASG" |
| Patch baseline | Quy tắc duyệt patch | "auto-approve after 7 days" |
| `Patch Group` tag | Gắn node vào baseline | "dev/test/prod patching" |
| Maintenance Window | Khung giờ được phép tác động | "patch only during off-hours" |
| Change Calendar | Khoảng CLOSED cấm thay đổi | "freeze changes during peak season" |
| Session Manager | Shell qua IAM, không port | "no port 22, no bastion, session logs" |
| SecureString | Tham số mã hóa KMS | "store DB password in Parameter Store" |
| Parameter policies | TTL / thông báo cho advanced parameter | "force password update" |
| Resource Data Sync | Gom inventory đa account về S3 | "centralize inventory across accounts" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Run Command | State Manager | Một lần; duy trì theo lịch, tự áp cho node mới | "Always", "new instances" → State Manager |
| Run Command | Automation | Bên trong OS; điều phối cả tài nguyên AWS | AMI/ASG/snapshot → Automation |
| Maintenance Window | Change Calendar | Mở cửa sổ để làm; đóng cửa cấm làm | "Freeze" → Change Calendar |
| Parameter Store | Secrets Manager | Standard miễn phí, không tự rotate; Secrets Manager có rotation, replication | "Rotate every 30 days" → Secrets Manager |
| Standard parameter | Advanced parameter | 10k / 4 KB / miễn phí; 100k / 8 KB / policies / có phí | > 4 KB hoặc TTL → Advanced |
| Session Manager | EIC Endpoint | Cần agent + role, không key, có log; không agent nhưng là SSH, cần key | "No keys, log commands" → Session Manager |
| Session Manager | Fleet Manager RDP | Shell/CLI; desktop Windows | Windows GUI → Fleet Manager |
| Patch Manager | Amazon Inspector | Cài patch, báo compliance; tìm CVE | "Find vulnerabilities" → Inspector |
| Inventory | Compliance | Có gì trên máy; đạt chuẩn hay không | "% compliant" → Compliance |
| Instance profile | DHMC | Gắn từng instance; bật một lần mỗi Region, cần IMDSv2 | "Every instance, no role per instance" → DHMC |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "instance not visible in Systems Manager" | IAM role → endpoint/mạng → agent | |
| "private subnet, no internet, use SSM" | Interface endpoint `ssm`, `ssmmessages`, `ec2messages` | |
| "shell without SSH keys or open ports" | Session Manager | |
| "record every command engineers run" | Session Manager logging → S3 / CloudWatch Logs | |
| "connect to RDS in private subnet from laptop, no bastion" | Session Manager port forwarding | |
| "run script once across fleet" | Run Command + rate control | |
| "keep agent installed on new ASG instances" | State Manager association | |
| "patch only Sunday 2–4 AM" | Maintenance Window + `AWS-RunPatchBaseline` | |
| "assess missing patches without changing prod" | Patch Manager Scan | |
| "patch across the organization" | Patch policy (Quick Setup) | |
| "remediate non-compliant resource automatically" | Config rule + Automation runbook | |
| "parameter larger than 4 KB / needs expiration" | Advanced parameter | |
| "latest Amazon Linux AMI ID in template" | Public parameter `/aws/service/...` | |
| "inventory across all accounts in S3" | Resource Data Sync | |

---

## 5. Câu hỏi tình huống

**Câu 1.** EC2 mới trong private subnet (không NAT, không IGW) không hiện trong Fleet Manager. Instance có role `AmazonSSMManagedInstanceCore`, AMI có SSM Agent. Thiếu gì?
- A. Mở inbound port 22
- B. Interface endpoint cho `ssm`, `ssmmessages`, `ec2messages`
- C. Gắn Elastic IP
- D. Bật Detailed Monitoring

<details><summary>Đáp án</summary>

**B.** Agent cần đường ra HTTPS tới endpoint SSM.
- A sai: SSM không cần inbound port.
- C sai: không có IGW nên EIP không có tác dụng.
- D sai: không liên quan đăng ký node.
</details>

**Câu 2.** Công ty muốn mọi EC2 trong Region đều được SSM quản lý mà không phải gắn instance profile từng cái. Sau khi bật, vài instance vẫn không đăng ký. Nguyên nhân khả dĩ nhất?
- A. Instance chỉ chạy IMDSv1
- B. Instance không có public IP
- C. Instance thuộc ASG
- D. Instance chưa có tag `Patch Group`

<details><summary>Đáp án</summary>

**A.** DHMC bắt buộc IMDSv2.
- B sai: chỉ cần đường tới endpoint, không cần public IP.
- C sai: ASG không ảnh hưởng DHMC.
- D sai: tag chỉ dùng cho patching.
</details>

**Câu 3.** Phải đảm bảo agent bảo mật luôn được cài trên mọi instance, kể cả instance ASG tạo về sau. Chọn gì?
- A. Run Command chạy một lần
- B. State Manager association nhắm theo tag
- C. Session Manager
- D. Inventory

<details><summary>Đáp án</summary>

**B.** Association chạy theo lịch và tự áp cho node mới khớp target.
- A sai: không áp cho instance sinh sau.
- C sai: công cụ truy cập.
- D sai: chỉ thu thập thông tin.
</details>

**Câu 4.** (Chọn 2) Đội bảo mật yêu cầu bỏ bastion, không mở port 22, và lưu lại mọi lệnh kỹ sư gõ trên server. Chọn hai thành phần.
- A. Session Manager
- B. Bật session logging tới S3 hoặc CloudWatch Logs
- C. EC2 Instance Connect Endpoint
- D. Security Group mở 22 từ IP văn phòng
- E. VPC Flow Logs

<details><summary>Đáp án</summary>

**A, B.**
- C sai: vẫn là SSH, không có sẵn log lệnh.
- D sai: trái yêu cầu không mở port 22.
- E sai: Flow Logs ghi metadata gói tin, không ghi lệnh.
</details>

**Câu 5.** Ứng dụng cần lưu một chứng chỉ 6 KB trong Parameter Store và muốn nhận cảnh báo nếu giá trị không được đổi trong 90 ngày. Cấu hình nào?
- A. Standard parameter loại `SecureString`
- B. Advanced parameter với policy `NoChangeNotification`
- C. Standard parameter với policy `Expiration`
- D. Lưu trong tag của instance

<details><summary>Đáp án</summary>

**B.** Advanced cho giá trị tới 8 KB và hỗ trợ parameter policies.
- A sai: standard tối đa 4 KB.
- C sai: standard không có parameter policies.
- D sai: tag không dùng để lưu secret.
</details>

**Câu 6.** Cần vá hằng tuần chỉ trong khung 2–4 giờ sáng Chủ nhật, mỗi lần tối đa 10% fleet. Chọn gì?
- A. Maintenance Window chạy `AWS-RunPatchBaseline` với rate control
- B. Change Calendar
- C. Inspector
- D. Run Command chạy tay mỗi Chủ nhật

<details><summary>Đáp án</summary>

**A.**
- B sai: Change Calendar dùng để cấm thay đổi, không chạy task.
- C sai: Inspector phát hiện lỗ hổng, không cài patch.
- D sai: thủ công, dễ sót.
</details>

**Câu 7.** Cần quy trình: vá AMI gốc, tạo AMI mới, cập nhật Launch Template và refresh ASG, có bước phê duyệt. Chọn gì?
- A. Run Command
- B. Automation runbook với approval step
- C. State Manager
- D. Distributor

<details><summary>Đáp án</summary>

**B.** Automation gọi API EC2/ASG và hỗ trợ approval.
- A sai: chỉ chạy lệnh trong OS.
- C sai: duy trì cấu hình node.
- D sai: phân phối gói phần mềm.
</details>

**Câu 8.** Cần trả lời "có bao nhiêu server trên 20 account còn chạy OpenSSL bản cũ", truy vấn bằng SQL. Chọn gì?
- A. Patch Manager compliance
- B. Inventory + Resource Data Sync về S3, truy vấn bằng Athena
- C. CloudTrail
- D. OpsCenter

<details><summary>Đáp án</summary>

**B.**
- A sai: báo trạng thái patch theo baseline, không liệt kê version phần mềm tùy ý.
- C sai: ghi API call.
- D sai: quản lý OpsItem.
</details>

---

### Nguồn đã đối chiếu
- Setup VPC endpoints cho SSM: https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-create-vpc.html
- Default Host Management Configuration (IMDSv2, agent 3.2.582.0): https://docs.aws.amazon.com/systems-manager/latest/userguide/fleet-manager-default-host-management-configuration.html
- Patch groups (tag `Patch Group`/`PatchGroup`, patch policy): https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager-patch-groups.html
- Session Manager idle timeout: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-preferences-timeout.html
- Session Manager max duration: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-preferences-max-timeout.html
- Parameter Store tiers: https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-advanced-parameters.html
- Parameter hierarchies (15 cấp): https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-hierarchies.html
- Instance tiers cho node lai (bỏ advanced tier 2026): https://docs.aws.amazon.com/systems-manager/latest/userguide/fleet-manager-configure-instance-tiers.html
- Change Manager availability change: https://docs.aws.amazon.com/systems-manager/latest/userguide/change-manager-availability-change.html
- Incident Manager availability change: https://docs.aws.amazon.com/incident-manager/latest/userguide/incident-manager-availability-change.html
- SSM CloudWatch Dashboard (ngừng sau 30/04/2026): https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-cloudwatch-dashboards.html
- Changes to Explorer (31/12/2026): https://docs.aws.amazon.com/systems-manager/latest/userguide/changes-to-explorer.html
