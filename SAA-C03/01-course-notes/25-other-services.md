# SAA-C03 — Other Services: CloudFormation, SES, Pinpoint, Systems Manager, Cost Explorer, Outposts, Batch, AppFlow, Amplify, Instance Scheduler

> Thuộc **Domain 2: Design Resilient Architectures**, **Domain 3: Design High-Performing Architectures** và **Domain 4: Design Cost-Optimized Architectures**: các dịch vụ "nhận diện nhanh" thường xuất hiện làm đáp án hoặc distractor.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **CloudFormation**: IaC khai báo (YAML/JSON), tạo tài nguyên đúng thứ tự, tái tạo môi trường, xóa cả stack; **StackSets** triển khai nhiều account/Region; **service role** để user không cần quyền trực tiếp trên tài nguyên.
- ⚠️ **Infrastructure Composer** (thiết kế template kéo-thả trên console) đóng với khách mới từ 29/10/2026, ngừng hỗ trợ 07/12/2026; trải nghiệm vẫn có trong AWS Toolkit for VS Code.
- **SES**: gửi email hàng loạt/giao dịch, nhận email; DKIM, SPF; thống kê. ⚠️ **Pinpoint**: không nhận khách mới từ 20/05/2025, **ngừng hỗ trợ 30/10/2026** (API SMS/voice/push chuyển sang **AWS End User Messaging**).
- **Systems Manager**: **Session Manager** (shell không SSH/port 22), **Run Command**, **Patch Manager**, **Maintenance Windows**, **Automation** (runbook), Parameter Store.
- **Cost Explorer** (xem/phân tích chi phí, dự báo, khuyến nghị Savings Plans), **Cost Anomaly Detection** (ML phát hiện chi tiêu bất thường, báo qua SNS).
- **Outposts**: rack AWS đặt tại data center của bạn (độ trễ thấp, dữ liệu lưu tại chỗ); bạn chịu trách nhiệm an ninh vật lý.
- **Batch**: chạy batch job container trên EC2/Spot/Fargate, không giới hạn 15 phút như Lambda. **AppFlow**: đồng bộ dữ liệu SaaS (Salesforce, SAP, Zendesk, Slack) ↔ S3/Redshift. **Amplify**: full-stack web/mobile. **Instance Scheduler**: solution bật/tắt EC2/RDS theo lịch.

---

## 2. Core Concepts

### AWS CloudFormation
- **Là gì:** Infrastructure as Code: template mô tả tài nguyên, CloudFormation tự tạo theo đúng thứ tự; tag tự động theo stack; xem chi phí ước tính; tạo sơ đồ từ template.
- **Dùng khi nào:** tái tạo môi trường dev/test/prod, xóa toàn bộ môi trường để tiết kiệm (tắt ban đêm), DR bằng dựng lại hạ tầng.
- **Vì sao quan trọng:** **service role**: user chỉ cần quyền `iam:PassRole` + CloudFormation, stack dùng role riêng để tạo tài nguyên (least privilege); **StackSets** cho nhiều account/Region trong Organization.

### ⚠️ Infrastructure Composer
- **Là gì:** công cụ thiết kế template CloudFormation bằng kéo-thả.
- **Dùng khi nào:** khách hiện có dùng console tới 07/12/2026; khách mới dùng AWS Toolkit for VS Code.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: đóng với khách mới từ 29/10/2026, ngừng hỗ trợ 07/12/2026; stack và template CloudFormation không bị ảnh hưởng.

### Amazon SES
- **Là gì:** gửi email an toàn, quy mô lớn (outbound) và nhận email (inbound); API hoặc SMTP; xác thực **DKIM**, **SPF**; dashboard thống kê deliverability, bounce, complaint.
- **Dùng khi nào:** email giao dịch (xác nhận đơn, đặt lại mật khẩu), marketing, email hàng loạt.
- **Vì sao quan trọng:** từ khóa "send email at scale" → SES (không phải SNS email cho thông báo đơn giản).

### ⚠️ Amazon Pinpoint
- **Là gì:** dịch vụ marketing đa kênh (email, SMS, push, voice, in-app), phân khúc, campaign, journey.
- **Dùng khi nào:** đề cũ: "gửi SMS marketing theo phân khúc, theo dõi phản hồi" → Pinpoint.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: không nhận khách mới từ **20/05/2025**, ngừng hỗ trợ **30/10/2026**; tính năng engagement chuyển sang Amazon Connect (proactive engagement), API kênh SMS/voice/push chuyển sang **AWS End User Messaging**.

### Systems Manager — Session Manager & Run Command
- **Là gì:** **Session Manager**: shell an toàn vào EC2/on-prem qua SSM Agent, không cần SSH key, bastion hay port 22, ghi log phiên vào S3/CloudWatch; **Run Command**: chạy document/script trên cả fleet, không cần SSH, tích hợp IAM, CloudTrail, EventBridge.
- **Dùng khi nào:** quản trị server không mở inbound; chạy lệnh hàng loạt.
- **Vì sao quan trọng:** đáp án bảo mật hơn bastion host.

### Systems Manager — Patch Manager, Maintenance Windows, Automation
- **Là gì:** **Patch Manager** vá OS/ứng dụng theo lịch hoặc on-demand, báo cáo compliance; **Maintenance Windows** lịch thời gian thực hiện tác vụ; **Automation** runbook tự động (restart instance, tạo AMI, snapshot EBS), kích hoạt từ console/SDK/EventBridge/**Config remediation**.
- **Dùng khi nào:** vá fleet EC2 và on-prem theo lịch; tự sửa tài nguyên không tuân thủ.
- **Vì sao quan trọng:** "patch hàng trăm server đồng loạt theo lịch" → Patch Manager + Maintenance Windows.

### Cost Explorer & Cost Anomaly Detection
- **Là gì:** **Cost Explorer** trực quan hóa, phân tích chi phí theo tháng/giờ/tài nguyên, chọn Savings Plans tối ưu, **dự báo** chi phí; **Cost Anomaly Detection** dùng ML theo dõi liên tục, phát hiện chi tiêu bất thường, báo nguyên nhân gốc qua SNS (cá nhân/tổng hợp).
- **Dùng khi nào:** báo cáo chi phí, lập kế hoạch cam kết; phát hiện tăng chi bất thường mà không đặt ngưỡng.
- **Vì sao quan trọng:** khác **Budgets** (cảnh báo theo ngưỡng do bạn đặt).

### AWS Outposts
- **Là gì:** rack server do AWS lắp, quản lý tại on-prem của bạn, chạy cùng API/công cụ AWS (EC2, EBS, S3 on Outposts, EKS, ECS, RDS, EMR).
- **Dùng khi nào:** cần độ trễ cực thấp tới hệ thống on-prem, xử lý dữ liệu tại chỗ, **data residency**, migrate dần.
- **Vì sao quan trọng:** shared responsibility đổi: khách lo an ninh vật lý rack; là phương án edge compute AWS gợi ý thay Snowball Edge.

### AWS Batch
- **Là gì:** chạy batch job (Docker image) quy mô lớn, tự cấp EC2/Spot hoặc Fargate đúng số lượng; lập lịch job, hàng đợi.
- **Dùng khi nào:** xử lý ảnh, mô phỏng, tính toán dài; job chạy hơn 15 phút.
- **Vì sao quan trọng:** **Batch vs Lambda**: Lambda giới hạn thời gian, runtime, `/tmp`, serverless; Batch không giới hạn thời gian, runtime bất kỳ (Docker), dùng EBS/instance store.

### Amazon AppFlow
- **Là gì:** tích hợp dữ liệu managed giữa **SaaS** (Salesforce, SAP, Zendesk, Slack, ServiceNow) và AWS (S3, Redshift) hoặc SaaS khác; chạy theo lịch, sự kiện hoặc on-demand; lọc, kiểm tra dữ liệu.
- **Dùng khi nào:** đồng bộ dữ liệu SaaS vào data lake mà không viết code tích hợp.
- **Vì sao quan trọng:** truyền mã hóa qua Internet công cộng hoặc riêng qua **PrivateLink**.

### AWS Amplify
- **Là gì:** bộ công cụ xây và host ứng dụng web/mobile full-stack (auth, storage, API REST/GraphQL, CI/CD, hosting) trên các dịch vụ AWS.
- **Dùng khi nào:** "Beanstalk cho mobile/web frontend": developer muốn dựng nhanh frontend + backend serverless.
- **Vì sao quan trọng:** WAF nay gắn được trực tiếp cho ứng dụng Amplify Hosting.

### `[+]` Instance Scheduler on AWS
- **Là gì:** **AWS Solution** (triển khai bằng CloudFormation), tự động start/stop EC2, ASG, RDS theo lịch; hỗ trợ cross-account, cross-Region.
- **Dùng khi nào:** tắt môi trường dev/test ngoài giờ làm việc để cắt chi phí.
- **Vì sao quan trọng:** là solution chứ không phải dịch vụ; đề hay hỏi "giảm chi phí cho môi trường non-production".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| CloudFormation | IaC khai báo | "recreate infrastructure", "template" |
| StackSets | Stack nhiều account/Region | "deploy to all accounts in org" |
| Service role | Role riêng cho stack | "users without direct permissions" |
| SES | Gửi/nhận email quy mô lớn | "transactional email", "bulk email" |
| End User Messaging | API SMS/voice/push (thay Pinpoint) | "send SMS at scale" |
| Session Manager | Shell không cần SSH | "no port 22, audit sessions" |
| Run Command | Chạy lệnh trên fleet | "execute script on many instances" |
| Patch Manager | Vá OS/app | "automate patching" |
| SSM Automation | Runbook tự động | "remediate", "restart instances" |
| Cost Explorer | Phân tích, dự báo chi phí | "forecast spending", "SP recommendations" |
| Cost Anomaly Detection | ML phát hiện chi bất thường | "unexpected spend spike" |
| Outposts | Hạ tầng AWS tại on-prem | "low latency to on-prem", "data residency" |
| AWS Batch | Batch job container | "long-running batch jobs" |
| AppFlow | Đồng bộ dữ liệu SaaS | "Salesforce to S3" |
| Amplify | Full-stack web/mobile | "frontend developers build quickly" |
| Instance Scheduler | Bật/tắt theo lịch | "stop dev instances at night" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CloudFormation | Elastic Beanstalk | CloudFormation mọi tài nguyên; Beanstalk nền tảng web app | "Infrastructure as code" → CloudFormation |
| SES | SNS | SES email phong phú, quy mô lớn; SNS pub/sub, email thông báo đơn giản | Email marketing → SES |
| Pinpoint | End User Messaging | Pinpoint engagement (ngừng 30/10/2026); End User Messaging cho API kênh | |
| Session Manager | Bastion host | Session Manager không mở port, có audit; bastion cần SSH + public subnet | Bảo mật → Session Manager |
| Run Command | Automation | Run Command chạy lệnh; Automation chạy runbook nhiều bước | |
| Cost Explorer | Budgets | Explorer phân tích/dự báo; Budgets cảnh báo theo ngưỡng | "Alert when > $X" → Budgets |
| Cost Anomaly Detection | Budgets | Anomaly tự học mẫu; Budgets ngưỡng cố định | "Unusual" → Anomaly Detection |
| Batch | Lambda | Batch dài, Docker, EC2/Fargate; Lambda ≤ 15 phút | > 15 phút → Batch |
| Outposts | Local Zones | Outposts tại data center của bạn; Local Zones do AWS đặt gần thành phố | "In our data center" → Outposts |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "recreate same environment in another Region" | CloudFormation (StackSets) | |
| "users deploy stacks without resource permissions" | CloudFormation service role | |
| "send order confirmation emails at scale" | SES | |
| "send SMS/push at scale (new customer)" | AWS End User Messaging | Pinpoint đóng |
| "secure shell without opening ports" | SSM Session Manager | |
| "patch fleet on schedule" | Patch Manager + Maintenance Windows | |
| "detect unusual cost spikes automatically" | Cost Anomaly Detection | |
| "forecast next months' spending" | Cost Explorer | |
| "run AWS services in our own data center" | AWS Outposts | |
| "batch jobs hours long using Docker" | AWS Batch | |
| "sync Salesforce data to S3 without code" | Amazon AppFlow | |
| "full-stack web/mobile app quickly" | AWS Amplify | |
| "stop dev EC2/RDS outside business hours" | Instance Scheduler on AWS | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Team cần dựng lại y hệt môi trường production ở Region khác cho DR và xóa môi trường test mỗi tối. Chọn gì?
- A. Elastic Beanstalk
- B. AWS CloudFormation
- C. AWS Batch
- D. Systems Manager Run Command

<details><summary>Đáp án</summary>

**B.**
- A sai: cho web app, không phải mọi tài nguyên.
- C sai: batch job.
- D sai: chạy lệnh trên instance.
</details>

**Câu 2.** Security cấm mở port 22 nhưng admin vẫn cần shell vào EC2 private và phải lưu log phiên. Chọn gì?
- A. Bastion host
- B. SSM Session Manager
- C. EC2 Instance Connect qua public IP
- D. Site-to-Site VPN

<details><summary>Đáp án</summary>

**B.**
- A sai: cần port 22.
- C sai: cần SSH và public IP.
- D sai: không phải công cụ truy cập shell có audit.
</details>

**Câu 3.** (Chọn 2) Công ty muốn giảm chi phí môi trường dev bằng cách tắt EC2 và RDS ngoài giờ, đồng thời được cảnh báo khi chi tiêu tăng bất thường. Chọn gì?
- A. Instance Scheduler on AWS
- B. Cost Anomaly Detection
- C. AWS Outposts
- D. Amazon AppFlow
- E. AWS Batch

<details><summary>Đáp án</summary>

**A, B.**
- C sai: hạ tầng on-prem.
- D sai: tích hợp SaaS.
- E sai: batch job.
</details>

**Câu 4.** Job xử lý video mỗi cái mất 2 giờ, đóng gói Docker, muốn dùng Spot để tiết kiệm. Chọn gì?
- A. Lambda
- B. AWS Batch với compute environment Spot
- C. Step Functions Express
- D. API Gateway

<details><summary>Đáp án</summary>

**B.**
- A sai: tối đa 15 phút.
- C sai: Express tối đa 5 phút.
- D sai: API.
</details>

**Câu 5.** Ứng dụng nhà máy cần độ trễ cực thấp tới hệ thống on-prem và dữ liệu không được rời khỏi cơ sở, nhưng muốn dùng API EC2/EBS. Chọn gì?
- A. Local Zones
- B. AWS Outposts
- C. Wavelength
- D. Region gần nhất

<details><summary>Đáp án</summary>

**B.**
- A sai: đặt ở cơ sở AWS gần thành phố, dữ liệu rời khỏi nhà máy.
- C sai: dành cho mạng 5G.
- D sai: dữ liệu rời cơ sở, độ trễ cao hơn.
</details>

**Câu 6.** Marketing cần đồng bộ dữ liệu khách hàng từ Salesforce vào S3 hằng ngày, không viết code. Chọn gì?
- A. Amazon AppFlow
- B. AWS DataSync
- C. AWS DMS
- D. Kinesis Data Streams

<details><summary>Đáp án</summary>

**A.**
- B sai: cho NFS/SMB/HDFS/object storage, không phải SaaS.
- C sai: cho database.
- D sai: cần tự viết producer/consumer.
</details>

**Câu 7.** Khách hàng mới (10/2026) cần gửi SMS OTP hàng loạt. Chọn gì?
- A. Amazon Pinpoint
- B. AWS End User Messaging
- C. Amazon SES
- D. Amazon MQ

<details><summary>Đáp án</summary>

**B.**
- A sai: không nhận khách mới từ 20/05/2025, ngừng 30/10/2026.
- C sai: email.
- D sai: message broker.
</details>

**Câu 8.** Developer không có quyền tạo EC2/RDS trực tiếp nhưng cần deploy stack CloudFormation đã được duyệt. Cấu hình gì?
- A. Cấp AdministratorAccess cho developer
- B. Dùng CloudFormation service role; developer chỉ cần quyền CloudFormation + `iam:PassRole`
- C. Dùng root user
- D. Dùng Cost Explorer

<details><summary>Đáp án</summary>

**B.** Least privilege: stack dùng role riêng để tạo tài nguyên.
- A, C sai: vi phạm least privilege.
- D sai: không liên quan.
</details>

---

### Nguồn đã đối chiếu
- AWS CloudFormation service role: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-iam-servicerole.html
- CloudFormation StackSets: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html
- Infrastructure Composer end of support (đóng khách mới 29/10/2026, ngừng 07/12/2026): https://docs.aws.amazon.com/infrastructure-composer/latest/dg/infrastructure-composer-end-of-support.html
- Amazon Pinpoint end of support (đóng khách mới 20/05/2025, ngừng 30/10/2026): https://docs.aws.amazon.com/pinpoint/latest/userguide/migrate.html
- Amazon SES: https://docs.aws.amazon.com/ses/latest/dg/Welcome.html
- Systems Manager Session Manager: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html
- AWS Cost Anomaly Detection: https://docs.aws.amazon.com/cost-management/latest/userguide/manage-ad.html
- AWS Outposts: https://docs.aws.amazon.com/outposts/latest/userguide/what-is-outposts.html
- AWS Batch: https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html
- Amazon AppFlow: https://docs.aws.amazon.com/appflow/latest/userguide/what-is-appflow.html
- Instance Scheduler on AWS: https://docs.aws.amazon.com/solutions/latest/instance-scheduler-on-aws/solution-overview.html
