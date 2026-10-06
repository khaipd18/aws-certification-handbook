# SOA-C03 — Account Management: AWS Health, Organizations & Billing

> Thuộc **Domain 4: Security and Compliance (16%)** và **Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization (22%)**: sự kiện AWS Health, quản trị nhiều account, SCP, Control Tower, Service Catalog, công cụ chi phí.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **AWS Health Dashboard**:
  - **Service history**: tình trạng chung của mọi dịch vụ, có RSS.
  - **Your account**: sự kiện ảnh hưởng tài nguyên của bạn, có hướng dẫn khắc phục và lịch bảo trì.
- Sự kiện Health tự động hóa qua **EventBridge**, ví dụ xóa access key bị lộ, restart instance sắp bị retire.
- **Organizations**: consolidated billing, giảm giá theo tổng khối lượng, chia sẻ RI / Savings Plans. Một account chỉ thuộc **một** organization.
- **SCP** là rào chắn, không cấp quyền:
  - **Không áp cho management account**.
  - Phải có allow trên **toàn đường đi** từ root tới account; Deny ở bất kỳ tầng nào đều thắng.
- **Control Tower** dựng landing zone nhiều account. ⚠️ "Guardrails" nay gọi là **controls**.
- **Service Catalog**: portfolio gồm các product (template CloudFormation) để người dùng tự triển khai đúng chuẩn.
- Công cụ chi phí:
  - **Cost Explorer**: phân tích 13 tháng, dự báo 18 tháng.
  - **Budgets**: cảnh báo và hành động.
  - **CUR**: dữ liệu chi tiết nhất.
  - **Cost Allocation Tags**: gán chi phí cho team/dự án.
- **Metric billing chỉ có ở us-east-1**. **Compute Optimizer** cần ít nhất 30 giờ metric mới đưa khuyến nghị cho EC2.

---

## 2. Core Concepts

### AWS Health Dashboard
- **Là gì:** Service history (công khai, mọi Region/dịch vụ) và Your account (cá nhân hóa theo tài nguyên, xem gộp được cho cả Organization).
- **Dùng khi nào:** xác định sự cố là của AWS hay của mình; lên kế hoạch cho instance retirement, bảo trì theo lịch.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Service Health Dashboard" và "Personal Health Dashboard" → nay gộp chung tên **AWS Health Dashboard**.

### Health events & Health API
- **Là gì:** EventBridge nhận account-specific event và public event của Health → Lambda, SNS, SQS, Kinesis.
- **Dùng khi nào:** tự động hóa: event `AWS_RISK_CREDENTIALS_EXPOSED` → Lambda xóa key; event retirement → restart instance.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: Health API (gọi trực tiếp, xem cấp Organization) cần gói **Business Support+, Enterprise hoặc Unified Operations** (hoặc gói cũ Business / Enterprise On-Ramp / Enterprise); gói Developer ngừng từ 01/01/2027. Gói hiện tại của bạn: kiểm tra trong Support console.

### AWS Organizations
- **Là gì:** dịch vụ global; management account + member account nhóm trong OU; tạo account bằng API.
- **Dùng khi nào:** consolidated billing, tách blast radius theo account, gom CloudTrail và log về account trung tâm, cross-account role cho quản trị.
- **Vì sao quan trọng:** RI / Savings Plans chia sẻ mặc định; payer tắt được theo account; chỉ chia sẻ khi **cả hai account** đều bật.

### Service Control Policies (SCP)
- **Là gì:** giới hạn tối đa quyền của user/role trong account thành viên; quyền thực tế = SCP cho phép **và** IAM cho phép.
- **Dùng khi nào:** chặn Region không được duyệt, chặn `RunInstances` thiếu tag, chặn tắt CloudTrail/GuardDuty.
- **Vì sao quan trọng:** **không áp cho management account** và service-linked role. Mặc định gắn `FullAWSAccess`; chiến lược blocklist giữ nó và thêm Deny, allowlist thay bằng allow hẹp.

### Điều kiện theo Organization & Tag Policies
- **Là gì:** `aws:PrincipalOrgID` trong resource-based policy giới hạn truy cập cho principal thuộc Organization; Tag Policies chuẩn hóa key/value tag.
- **Dùng khi nào:** bucket chỉ cho account trong tổ chức truy cập, không cần liệt kê account ID; chuẩn hóa tag cho cost allocation.
- **Vì sao quan trọng:** Tag Policies **không tác động tài nguyên chưa gắn tag**; theo dõi vi phạm qua báo cáo tuân thủ và EventBridge.

### `[+]` Resource Control Policies (RCP) & các loại policy khác
- **Là gì:** RCP đặt mức tối đa cho những gì **resource-based policy** cho phép, áp cho tài nguyên trong account thành viên; Organizations còn có tag policy, backup policy, AI services opt-out policy.
- **Dùng khi nào:** chặn tài nguyên (S3, KMS, STS…) bị truy cập bởi principal ngoài tổ chức, kể cả khi bucket policy viết lỏng.
- **Vì sao quan trọng:** SCP giới hạn principal; RCP giới hạn tài nguyên. RCP cũng không áp cho management account.

### AWS Control Tower
- **Là gì:** dựng **landing zone** (log archive, audit account, CloudTrail và Config tập trung) trên Organizations; **Account Factory** tạo account theo khuôn mẫu.
- **Dùng khi nào:** cần môi trường nhiều account tuân thủ best practice nhanh, có dashboard tuân thủ; enroll được cả account có sẵn.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "guardrails" → **controls**: preventive (SCP/RCP), detective (Config rule), proactive (CloudFormation Hooks).

### AWS Service Catalog
- **Là gì:** admin xuất bản **product** (template CloudFormation) trong **portfolio**; IAM quyết định ai thấy portfolio nào; TagOptions gắn tag cho sản phẩm được tạo.
- **Dùng khi nào:** người dùng ít kinh nghiệm tự tạo stack đúng chuẩn, đúng tag.
- **Vì sao quan trọng:** **launch constraint** gắn IAM role để stack chạy bằng quyền của role, không phải của người dùng. Share portfolio dạng reference (tự đồng bộ) hoặc copy.

### Billing alarm & AWS Budgets
- **Là gì:** metric `EstimatedCharges` chỉ có ở **us-east-1**, phản ánh tổng chi phí. Budgets có loại Cost, Usage, Reservation, Savings Plans; dữ liệu cập nhật tới 3 lần mỗi ngày.
- **Dùng khi nào:** cảnh báo theo dịch vụ, tag, account, theo chi phí thực tế hoặc dự báo → Budgets. **Budgets Actions** áp IAM policy/SCP hoặc stop EC2/RDS khi vượt ngưỡng.
- **Vì sao quan trọng:** đề nói "ngăn chi tiêu thêm" → Budgets Actions; "chỉ báo" → alert. Có độ trễ giữa lúc dùng và lúc tính tiền, nên có thể vượt ngưỡng trước khi nhận cảnh báo.

### Cost Explorer, CUR, Cost Allocation Tags, Anomaly Detection
- **Là gì:** Cost Explorer xem 13 tháng gần nhất, dự báo 18 tháng, gợi ý Savings Plans/RI; CUR là dữ liệu chi tiết theo giờ/ngày vào S3 (Athena, Redshift, Quick Sight); Cost Anomaly Detection phát hiện chi tiêu bất thường bằng ML.
- **Dùng khi nào:** phân tích xu hướng → Cost Explorer; phân tích sâu bằng SQL → CUR; chi tiêu biến động tự nhiên → Anomaly Detection.
- **Vì sao quan trọng:** cost allocation tag (`aws:` do AWS sinh, `user:` do người dùng) phải **kích hoạt** trong Billing, không hồi tố.

### AWS Compute Optimizer
- **Là gì:** ML trên metric CloudWatch để gợi ý right-size cho EC2, ASG, EBS, Lambda, ECS trên Fargate, RDS/Aurora, license thương mại.
- **Dùng khi nào:** tìm instance quá lớn/quá nhỏ.
- **Vì sao quan trọng:** EC2/ASG cần **ít nhất 30 giờ metric trong 14 ngày** (93 ngày nếu bật enhanced infrastructure metrics). Không có metric memory thì khuyến nghị kém chính xác → cài CloudWatch Agent.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| AWS Health Dashboard | Sự kiện AWS ảnh hưởng tài nguyên | "scheduled maintenance", "retirement" |
| Health EventBridge rule | Tự động hóa theo sự kiện Health | "exposed access key auto-remediation" |
| Consolidated billing | Một hóa đơn cho nhiều account | "volume discounts across accounts" |
| SCP | Giới hạn quyền tối đa | "deny regions", "guardrail" |
| RCP | Giới hạn quyền trên tài nguyên | "restrict resource-based policies org-wide" |
| `aws:PrincipalOrgID` | Điều kiện theo Organization | "only accounts in our org" |
| Tag Policy | Chuẩn hóa tag | "standardize tag values" |
| Control Tower | Landing zone + controls | "set up governed multi-account" |
| Service Catalog | Portfolio product đã duyệt | "self-service compliant provisioning" |
| Launch constraint | Role dùng khi launch product | "users lack permissions to create resources" |
| AWS Budgets Actions | Hành động khi vượt ngân sách | "stop spending automatically" |
| Cost and Usage Report | Dữ liệu chi phí chi tiết nhất | "most granular cost data" |
| Compute Optimizer | Gợi ý right-sizing | "rightsize EC2 based on utilization" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Service history | Your account | Tình trạng chung; ảnh hưởng tài nguyên của bạn | "Affects my instances" → Your account |
| SCP | IAM policy | Giới hạn tối đa, không cấp quyền; cấp quyền | SCP không bao giờ grant |
| SCP | RCP | Giới hạn principal; giới hạn tài nguyên | Chặn người ngoài vào bucket → RCP |
| Management account | Member account | SCP không áp; SCP áp | Management luôn full quyền |
| Control Tower | Organizations | Landing zone + controls; nền tảng account/OU/SCP | "Set up quickly with best practices" → Control Tower |
| Service Catalog | CloudFormation | Self-service product đã duyệt; công cụ IaC | Người dùng không chuyên → Service Catalog |
| Billing alarm | Budgets | Tổng chi phí, us-east-1; theo dịch vụ/tag/dự báo, có action | Chi tiết hoặc hành động → Budgets |
| Budgets | Cost Anomaly Detection | Ngưỡng cố định; bất thường theo ML | "Unexpected spike" → Anomaly Detection |
| Cost Explorer | CUR | Trực quan, dự báo; dữ liệu thô chi tiết nhất | SQL/Athena → CUR |
| Compute Optimizer | Trusted Advisor | Right-size bằng ML theo metric; check đa lĩnh vực | Right-size → Compute Optimizer |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "notified when AWS schedules maintenance on our instances" | Health + EventBridge | |
| "auto-remediate exposed IAM access key" | Health event → EventBridge → Lambda | |
| "deny all Regions except two for every account" | SCP trên root/OU | Không áp management account |
| "SCP attached but management account still can" | Đúng: SCP không áp management | |
| "bucket accessible only from our organization" | `aws:PrincipalOrgID` | |
| "prevent any principal outside org from accessing resources" | RCP | |
| "multi-account landing zone quickly" | Control Tower | |
| "users launch approved stacks without IAM rights" | Service Catalog + launch constraint | |
| "billing alarm metric not found" | Chuyển sang us-east-1 | |
| "stop dev instances when budget exceeded" | Budgets Actions | |
| "most detailed hourly billing data in S3" | Cost and Usage Report | |
| "costs by team not visible" | Kích hoạt cost allocation tag | Không hồi tố |
| "no rightsizing recommendation for new instance" | Chưa đủ 30 giờ metric | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Muốn tự động khởi động lại instance khi AWS báo instance sắp bị retire. Chọn gì?
- A. CloudTrail + Lambda
- B. EventBridge rule cho sự kiện AWS Health → Lambda / SSM Automation
- C. Config rule
- D. Trusted Advisor

<details><summary>Đáp án</summary>

**B.**
- A sai: CloudTrail ghi API call, không phát sự kiện retirement.
- C sai: Config theo dõi cấu hình.
- D sai: không phát sự kiện bảo trì theo lịch.
</details>

**Câu 2.** Gắn SCP chặn `ec2:RunInstances` ngoài `eu-west-1` lên root, nhưng admin ở management account vẫn tạo instance ở `us-east-1`. Vì sao?
- A. SCP cần thời gian lan truyền 24 giờ
- B. SCP không áp cho management account
- C. Phải gắn SCP vào từng user
- D. SCP chỉ áp cho root user

<details><summary>Đáp án</summary>

**B.**
- A sai: không có độ trễ 24 giờ như vậy.
- C sai: SCP gắn vào root, OU hoặc account, không gắn user.
- D sai: SCP áp cho mọi principal trong account thành viên.
</details>

**Câu 3.** (Chọn 2) Bucket S3 phải chỉ cho principal thuộc Organization truy cập; công ty muốn đặt mức trần này cho mọi account, không phụ thuộc bucket policy từng nơi. Chọn gì?
- A. Điều kiện `aws:PrincipalOrgID` trong bucket policy
- B. Resource Control Policy ở cấp Organization
- C. Tag Policy
- D. Budgets Actions
- E. Service Catalog

<details><summary>Đáp án</summary>

**A, B.** A giới hạn cho bucket cụ thể; B đặt trần cho mọi tài nguyên được hỗ trợ trong account thành viên.
- C sai: chuẩn hóa tag.
- D sai: quản lý chi phí.
- E sai: cung cấp product.
</details>

**Câu 4.** Không tìm thấy metric `EstimatedCharges` khi tạo alarm ở `ap-southeast-1`. Vì sao?
- A. Chưa bật Cost Explorer
- B. Metric billing chỉ có ở us-east-1 (và phải bật nhận billing alert)
- C. Phải dùng CUR
- D. Account không trong Organization

<details><summary>Đáp án</summary>

**B.**
- A sai: không liên quan.
- C sai: CUR không tạo metric CloudWatch.
- D sai: account đơn lẻ vẫn có billing metric.
</details>

**Câu 5.** Account sandbox phải tự động dừng EC2 và RDS khi chi phí tháng vượt $500. Chọn gì?
- A. Billing alarm + SNS
- B. AWS Budgets với Budgets Actions
- C. Cost Explorer forecast
- D. Cost Anomaly Detection

<details><summary>Đáp án</summary>

**B.**
- A sai: chỉ thông báo.
- C sai: chỉ dự báo.
- D sai: phát hiện bất thường, không dừng tài nguyên theo ngưỡng.
</details>

**Câu 6.** Nhân viên phòng ban cần tự tạo môi trường VPC + RDS chuẩn công ty nhưng không có quyền IAM tạo các tài nguyên đó. Chọn gì?
- A. Cấp `AdministratorAccess` tạm thời
- B. Service Catalog product với launch constraint
- C. Control Tower Account Factory
- D. Tag Policy

<details><summary>Đáp án</summary>

**B.** Stack tạo bằng quyền của role trong launch constraint.
- A sai: trái least privilege.
- C sai: tạo account, không phải stack trong account.
- D sai: chuẩn hóa tag.
</details>

**Câu 7.** Báo cáo chi phí không chia được theo tag `Team` dù tài nguyên đã gắn tag từ 3 tháng trước. Lý do khả dĩ nhất?
- A. Tag phải viết hoa toàn bộ
- B. Tag chưa được kích hoạt làm cost allocation tag; kích hoạt rồi cũng không hồi tố
- C. Phải bật CloudTrail
- D. Tag chỉ áp cho EC2

<details><summary>Đáp án</summary>

**B.**
- A sai: không có yêu cầu này.
- C sai: không liên quan.
- D sai: cost allocation tag áp cho nhiều loại tài nguyên.
</details>

**Câu 8.** (Chọn 2) Dựng môi trường nhiều account mới theo best practice nhanh nhất, có log tập trung và rule phát hiện vi phạm. Hai phát biểu nào đúng?
- A. Control Tower tạo landing zone với log archive và audit account
- B. Control Tower dùng controls loại preventive, detective, proactive
- C. Control Tower thay thế hoàn toàn Organizations
- D. SCP áp cho management account khi dùng Control Tower
- E. Control Tower chỉ dùng được cho organization mới

<details><summary>Đáp án</summary>

**A, B.**
- C sai: Control Tower chạy trên Organizations.
- D sai: SCP vẫn không áp management account.
- E sai: enroll được account/organization có sẵn.
</details>

---

### Nguồn đã đối chiếu
- AWS Health concepts (đổi tên dashboard): https://docs.aws.amazon.com/health/latest/ug/aws-health-concepts-and-terms.html
- AWS Health API (gói Support yêu cầu): https://docs.aws.amazon.com/health/latest/ug/health-api.html
- AWS Support plans: https://docs.aws.amazon.com/awssupport/latest/user/aws-support-plans.html
- Service control policies: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html
- Resource control policies: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html
- Cost Explorer (13 tháng, dự báo 18 tháng): https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html
- AWS Budgets: https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html
- Compute Optimizer requirements (30 giờ): https://docs.aws.amazon.com/compute-optimizer/latest/ug/requirements.html
