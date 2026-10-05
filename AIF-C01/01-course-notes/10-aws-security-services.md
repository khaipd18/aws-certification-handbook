# AIF-C01 — AWS Services: Security & more

> Các dịch vụ nền tảng (IAM, S3, EC2, Lambda, VPC) và dịch vụ security/governance. Đề AIF-C01 chỉ hỏi ở mức **định nghĩa và mục đích**: dịch vụ này dùng để làm gì, và kết hợp thế nào với Bedrock/SageMaker.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời. Mốc kiến thức: 10/2026.

---

## 1. TL;DR

- **IAM**: Users (người), Groups (chỉ chứa user), **Policies** (JSON định nghĩa quyền), **Roles** (cấp quyền cho **dịch vụ AWS**, vd Bedrock đọc S3). Luôn theo **least privilege**.
- **S3**: lưu object trong bucket (bucket thuộc **Region**); durability **11 số 9** ở mọi storage class; chọn **storage class** theo tần suất truy cập; là kho data chính cho training và RAG.
- **EC2** = IaaS (VM, EBS, ELB, ASG); **Lambda** = serverless, event-driven, trả theo request và thời gian chạy.
- **VPC Endpoint / PrivateLink**: truy cập Bedrock, S3, SageMaker **riêng tư, không qua Internet**. Đây là dạng câu hỏi VPC phổ biến nhất của AIF-C01.
- Security/governance: **Macie** (PII trong S3), **Config** (cấu hình và tuân thủ rule), **Inspector** (lỗ hổng EC2/ECR/Lambda), **CloudTrail** (log API call), **Artifact** (báo cáo compliance), **Trusted Advisor** (gợi ý best practice).
- ⚠️ **AWS Audit Manager** không nhận khách hàng mới từ 30/04/2026 → nay dùng **AWS Config Conformance Packs**.
- **Bedrock + security:** IAM (ai được gọi model), Guardrails (nội dung), CloudTrail (ai gọi API gì), Config (thay đổi cấu hình), PrivateLink (gọi private), **KMS** (Bedrock cần IAM role có quyền S3 + **kms:Decrypt** để train custom model từ bucket được mã hóa).

---

## 2. Core Concepts

### IAM: Users, Groups, Policies
- **Là gì:** **IAM** = Identity and Access Management, dịch vụ **global**; **root account** tạo mặc định, không nên dùng hay chia sẻ; **Users** = người trong tổ chức; **Groups** chỉ chứa users (không chứa group khác); một user có thể thuộc nhiều group hoặc không thuộc group nào.
- **Dùng khi nào:** gắn **policy** (JSON) cho user/group để cấp quyền; user kế thừa quyền của mọi group mình thuộc, ngoài ra có thể có **inline policy** riêng.
- **Vì sao quan trọng:** policy gồm Version, Statement (Sid, **Effect** Allow/Deny, **Principal**, **Action**, **Resource**, Condition). Nguyên tắc **least privilege**: chỉ cấp đúng quyền cần.

### IAM Roles for Services
- **Là gì:** cấp quyền cho **dịch vụ AWS** để làm việc thay bạn (EC2 instance role, Lambda function role, role cho CloudFormation, Bedrock, SageMaker).
- **Dùng khi nào:** vd SageMaker notebook cần đọc data S3, Bedrock cần đọc bucket training data mã hóa bằng KMS.
- **Vì sao quan trọng:** `[+]` role cấp **credential tạm thời**, an toàn hơn nhúng access key dài hạn vào code. Đề hỏi "dịch vụ X cần truy cập dịch vụ Y" → **IAM role**.

### Amazon S3: Buckets & Objects
- **Là gì:** lưu **object** (file) trong **bucket**; bucket được tạo ở một **Region** dù S3 trông như dịch vụ global. Tên bucket: **unique toàn cầu** (shared global namespace), hoặc dùng **account regional namespace** (tên gắn account ID + Region, chỉ account đó tạo được).
- **Dùng khi nào:** use cases: backup, DR, archive, hybrid storage, hosting app/media, **data lake & analytics**, static website; kho data cho ML training và nguồn cho Bedrock Knowledge Bases.
- **Vì sao quan trọng:** object có **key** = đường dẫn đầy đủ (prefix + tên; không có "thư mục" thật); dung lượng tối đa khoảng **50 TB** (docs ghi 48.8 TiB); upload trên 5 GB phải dùng **multipart upload**; có metadata, tags, version ID.

### S3 Storage Classes
- **Là gì:** **Standard** (truy cập thường xuyên, 99.99% availability) · **Standard-IA** (ít truy cập nhưng cần lấy nhanh, 99.9%) · **One Zone-IA** (1 AZ, 99.5%, mất data nếu AZ bị phá hủy) · **Glacier Instant Retrieval** (mili giây, tối thiểu 90 ngày) · **Glacier Flexible Retrieval** (1 phút đến 12 giờ, tối thiểu 90 ngày) · **Glacier Deep Archive** (12–48 giờ, tối thiểu 180 ngày, rẻ nhất).
- **Dùng khi nào:** **Intelligent-Tiering** tự chuyển object giữa các tier theo mức truy cập (30 ngày → IA, 90 ngày → Archive Instant), có phí monitoring nhỏ, **không phí retrieval**; chuyển class thủ công hoặc bằng **S3 Lifecycle**.
- **Vì sao quan trọng:** **Durability 11 số 9 cho mọi class**; **availability khác nhau theo class**. Giá cụ thể thay đổi theo thời gian và Region, nên học **thứ tự rẻ/đắt** và đặc điểm, đừng học số.

### Amazon EC2
- **Là gì:** IaaS: thuê VM (**EC2**), ổ đĩa mạng (**EBS**, EFS) hoặc Instance Store, cân bằng tải (**ELB**), scale bằng **Auto Scaling Group**; chọn OS, CPU, RAM, storage, network, **security group** (firewall).
- **Dùng khi nào:** **EC2 User Data** = script bootstrap chạy **một lần** khi instance khởi động lần đầu (cài update, cài phần mềm, tải file), chạy với quyền root.
- **Vì sao quan trọng:** nền tảng để hiểu cloud; với AI là nơi chạy GPU/Trainium/Inferentia instance khi tự host model.

### AWS Lambda
- **Là gì:** **serverless, Function as a Service**: không quản server, chạy on-demand, **scale tự động**, **event-driven**; tối đa 10 GB RAM (tăng RAM thì tăng cả CPU và network). `[+]` Thời gian chạy tối đa **15 phút**.
- **Dùng khi nào:** vd ảnh mới upload lên S3 → trigger Lambda tạo thumbnail; cron job serverless chạy mỗi giờ qua ⚠️ CloudWatch Events → nay là **Amazon EventBridge**; làm **action group / reward function** cho GenAI (Bedrock RFT).
- **Vì sao quan trọng:** giá: trả theo **số request + thời gian chạy (GB-giây)**; free tier 1 triệu request và 400.000 GB-giây mỗi tháng; hỗ trợ Node.js, Python, Java, .NET, Ruby, custom runtime, container image.

### Amazon Macie
- **Là gì:** dịch vụ fully managed về data security và privacy, dùng **ML + pattern matching** để **phát hiện data nhạy cảm (PII)** trong **S3**.
- **Dùng khi nào:** quét bucket chứa training data trước khi fine-tune; cảnh báo qua **EventBridge**.
- **Vì sao quan trọng:** đề có "find PII in S3" → Macie.

### AWS Config
- **Là gì:** ghi lại **cấu hình tài nguyên và thay đổi theo thời gian**, đánh giá **compliance** theo rule; lưu data vào S3 (phân tích bằng Athena), cảnh báo qua SNS.
- **Dùng khi nào:** trả lời các câu như: security group có mở SSH ra toàn Internet không? bucket có public không? cấu hình ALB đã đổi thế nào theo thời gian?
- **Vì sao quan trọng:** dịch vụ **theo Region**, có thể gom (aggregate) nhiều Region và account; xem được CloudTrail API calls liên quan đến một tài nguyên. **Conformance Packs** = bộ Config Rules đóng gói theo framework (HIPAA, NIST, PCI DSS...).

### Amazon Inspector
- **Là gì:** **đánh giá bảo mật tự động** cho **EC2** (lỗ hổng OS, network reachability), **container image trên ECR** (khi push), **Lambda function** (lỗ hổng trong code và dependency khi deploy).
- **Dùng khi nào:** quét liên tục dựa trên database **CVE**, mỗi lỗ hổng có **risk score** để ưu tiên; gửi kết quả tới Security Hub và EventBridge.
- **Vì sao quan trọng:** chỉ áp dụng cho **EC2, ECR images, Lambda**. EC2 quét qua SSM agent hoặc agentless (dùng EBS snapshot).

### AWS CloudTrail
- **Là gì:** **lịch sử API call** trong account (từ Console, SDK, CLI, dịch vụ AWS), phục vụ governance, compliance, audit; **bật mặc định**.
- **Dùng khi nào:** đẩy log vào CloudWatch Logs hoặc S3; trail áp dụng cho mọi Region (mặc định) hoặc một Region. Tài nguyên bị xóa → **xem CloudTrail đầu tiên**.
- **Vì sao quan trọng:** `[+]` Event history giữ management events **90 ngày miễn phí**; cần lâu hơn thì tạo trail (lưu S3) hoặc CloudTrail Lake. Với Bedrock: biết **ai** đã gọi API nào (vd ListCustomModels).

### AWS Artifact
- **Là gì:** cổng (không hẳn là dịch vụ) truy cập **tài liệu compliance và thỏa thuận** của AWS: **Artifact Reports** (báo cáo ISO, PCI, SOC từ auditor bên thứ ba), **Artifact Agreements** (xem, chấp nhận, theo dõi thỏa thuận như **BAA** cho HIPAA).
- **Dùng khi nào:** cung cấp bằng chứng cho audit nội bộ hoặc compliance. **Third-Party Reports**: báo cáo compliance của ISV (qua AWS Marketplace Vendor Insights).
- **Vì sao quan trọng:** đề có "download AWS SOC / ISO / PCI report" → Artifact.

### AWS Audit Manager
- ⚠️ Lỗi thời: **AWS Audit Manager** chuyển sang maintenance mode, không cho thiết lập ở account mới từ **30/04/2026** → nay dùng **AWS Config Conformance Packs** (template sẵn cho HIPAA, NIST, PCI DSS...).
- **Là gì:** đánh giá rủi ro và compliance của workload, **liên tục thu thập bằng chứng** theo framework (CIS, GDPR, HIPAA, PCI DSS, SOC 2), tạo báo cáo sẵn cho audit.
- **Vì sao quan trọng:** đề cũ có "continuously collect audit evidence" → Audit Manager; với khách mới, đáp án tương đương là Config Conformance Packs.

### AWS Trusted Advisor
- **Là gì:** đánh giá account ở mức cao, không cần cài gì; gợi ý theo **6 nhóm**: Cost optimization, Performance, Security, Fault tolerance, Service limits, Operational excellence.
- **Dùng khi nào:** cần danh sách việc nên làm theo best practice (vd: bucket public, tài nguyên không dùng, sắp chạm quota).
- **Vì sao quan trọng:** **toàn bộ checks** và API chỉ có với support plan cao. ⚠️ Lỗi thời: "Business & Enterprise Support plan" → Business Support (và Developer, Enterprise On-Ramp) ngừng từ 01/01/2027; nay full checks đi kèm **Business Support+, Enterprise Support, Unified Operations**.

### VPC, Subnets, Internet Gateway, NAT Gateway
- **Là gì:** **VPC** = mạng riêng để deploy tài nguyên (theo **Region**); **Subnet** chia mạng trong VPC (theo **AZ**): **public** (truy cập được từ Internet) và **private** (không truy cập được từ Internet).
- **Dùng khi nào:** **Internet Gateway** cho VPC ra Internet (public subnet có route tới IGW); **NAT Gateway** (AWS managed) cho instance trong private subnet **đi ra** Internet mà vẫn giữ private.
- **Vì sao quan trọng:** câu hỏi VPC trong AIF-C01 thường xoay quanh **deploy model riêng tư** và **truy cập dịch vụ AWS không qua Internet**.

### VPC Endpoints & AWS PrivateLink
- **Là gì:** mặc định dịch vụ AWS được truy cập qua Internet công cộng; **VPC endpoint** cho phép truy cập **riêng tư**, traffic nằm trong mạng AWS; thường được cung cấp bởi **PrivateLink**.
- **Dùng khi nào:** app trong private subnet gọi **Bedrock** qua Bedrock VPC endpoint; SageMaker notebook/training job đọc **S3** qua **S3 Gateway Endpoint** (hoặc S3 Interface Endpoint); có thể gắn **endpoint policy** để giới hạn thêm.
- **Vì sao quan trọng:** `[+]` 2 loại: **Gateway endpoint** (chỉ cho **S3 và DynamoDB**, miễn phí, qua route table, không dùng PrivateLink) và **Interface endpoint** (PrivateLink, dùng cho hầu hết dịch vụ, gồm Bedrock).

### AWS Services for Bedrock (tổng hợp)
- **Là gì:** **IAM** (xác thực, phân quyền tới mức tài nguyên, role cho data scientist), **Guardrails** (chặn topic, lọc nội dung, tuân thủ chính sách an toàn), **CloudTrail** (phân tích API call tới Bedrock), **Config** (thay đổi cấu hình), **PrivateLink** (giữ mọi API call trong VPC).
- **Dùng khi nào:** train custom model từ bucket mã hóa **SSE-KMS** → Bedrock cần **IAM role** có quyền đọc S3 **và** quyền **decrypt** trên KMS key.
- **Vì sao quan trọng:** SageMaker trong VPC: notebook, training job, endpoint đặt trong **private subnet** + security group + S3 VPC endpoint + IAM role + endpoint policy.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| IAM | Quản lý danh tính và quyền truy cập, dịch vụ global | "who can access what" |
| Root account | Account gốc, không dùng cho việc hằng ngày | "do not use root" |
| IAM User | Danh tính cho một người | "employee needs console access" |
| IAM Group | Tập user (không chứa group khác) | "assign same permissions to a team" |
| IAM Policy | JSON định nghĩa Allow/Deny cho Action trên Resource | "define permissions" |
| Least privilege | Chỉ cấp quyền tối thiểu cần thiết | "minimum permissions" |
| IAM Role | Quyền cho dịch vụ AWS (credential tạm thời) | "allow Bedrock / SageMaker / EC2 to access S3" |
| Amazon S3 | Object storage, bucket theo Region | "store training data", "data lake" |
| S3 object key | Đường dẫn đầy đủ của object (prefix + tên) | "folder structure in S3" |
| Multipart upload | Bắt buộc khi upload object trên 5 GB | "upload very large file" |
| Durability (11 9's) | Khả năng không mất data, như nhau ở mọi class | "data loss probability" |
| Availability | Mức sẵn sàng truy cập, khác nhau theo class | "uptime of storage class" |
| S3 Standard | Truy cập thường xuyên | "frequently accessed" |
| S3 Standard-IA / One Zone-IA | Ít truy cập; One Zone chỉ 1 AZ | "infrequent", "recreatable data" |
| S3 Glacier (Instant / Flexible / Deep Archive) | Lưu trữ dài hạn, rẻ, có phí retrieval | "archive", "compliance retention" |
| S3 Intelligent-Tiering | Tự chuyển tier theo mức truy cập | "unknown / changing access patterns" |
| S3 Lifecycle | Rule tự chuyển class hoặc xóa object | "move to Glacier after 90 days" |
| Amazon EC2 | Virtual machine (IaaS) | "full control of server" |
| EC2 User Data | Script chạy một lần khi khởi động lần đầu | "bootstrap instance" |
| Security group | Firewall cho instance/endpoint | "allow inbound port" |
| AWS Lambda | Serverless function, event-driven, tối đa 15 phút | "run code without servers", "triggered by S3 event" |
| Amazon EventBridge | Event bus / lịch chạy (trước là CloudWatch Events) | "schedule every hour", "react to events" |
| Amazon Macie | Phát hiện PII trong S3 | "sensitive data in S3" |
| AWS Config | Ghi cấu hình và đánh giá tuân thủ rule | "unrestricted SSH?", "configuration history" |
| Config Conformance Pack | Bộ Config Rules theo framework compliance | "deploy HIPAA/PCI controls across accounts" |
| Amazon Inspector | Quét lỗ hổng EC2, ECR, Lambda | "CVE", "software vulnerabilities" |
| AWS CloudTrail | Log mọi API call trong account | "who deleted...", "audit API activity" |
| AWS Artifact | Báo cáo compliance (Reports) và thỏa thuận (Agreements) | "SOC 2 report", "sign BAA" |
| AWS Audit Manager | Thu thập bằng chứng audit (không nhận khách mới) | "continuous audit evidence" (đề cũ) |
| AWS Trusted Advisor | Gợi ý best practice theo 6 nhóm | "cost / security / limits recommendations" |
| VPC | Mạng riêng ảo theo Region | "isolated network" |
| Public / Private subnet | Subnet truy cập / không truy cập được từ Internet | "no internet access" |
| Internet Gateway | Cổng ra/vào Internet cho VPC | "public subnet internet access" |
| NAT Gateway | Cho private subnet đi ra Internet | "private instances download updates" |
| VPC Endpoint | Truy cập dịch vụ AWS riêng tư, không qua Internet | "access Bedrock / S3 privately" |
| AWS PrivateLink | Công nghệ phía sau interface endpoint | "keep traffic within AWS network" |
| `[+]` Gateway endpoint | Endpoint miễn phí cho S3 và DynamoDB | "S3 access from private subnet" |
| Endpoint policy | Policy giới hạn quyền đi qua endpoint | "restrict which buckets via endpoint" |
| AWS KMS / SSE-KMS | Quản lý khóa / mã hóa S3 bằng khóa KMS | "encrypted bucket", "kms:Decrypt" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| IAM User | IAM Role | User: danh tính lâu dài cho **người**; Role: quyền **tạm thời** cho **dịch vụ** (hoặc người assume) | "Bedrock needs to read S3" → Role |
| IAM Group | IAM Role | Group gom **user**; Role gắn cho **dịch vụ** | Group không chứa group |
| Durability | Availability | Durability: **không mất** data (11 9's, như nhau mọi class); Availability: **truy cập được** (khác nhau theo class) | One Zone-IA vẫn 11 9's nhưng chỉ 1 AZ |
| Standard-IA | One Zone-IA | Standard-IA: ≥ 3 AZ; One Zone-IA: 1 AZ, rẻ hơn, mất data nếu AZ hỏng | Data tái tạo được → One Zone-IA |
| Glacier Instant | Glacier Flexible / Deep Archive | Instant: lấy trong **mili giây**; Flexible: phút đến giờ; Deep Archive: 12–48 giờ, rẻ nhất | "Archive but need immediate access quarterly" → Glacier Instant |
| Intelligent-Tiering | Lifecycle rule | Intelligent-Tiering **tự động** theo mức truy cập thực tế; Lifecycle theo **luật bạn đặt** (sau N ngày) | Access pattern không biết trước → Intelligent-Tiering |
| EC2 | Lambda | EC2: server chạy liên tục, tự scale (ASG); Lambda: function ngắn (≤ 15 phút), chạy khi có event, scale tự động | "Short, event-driven, no servers" → Lambda |
| AWS Config | AWS CloudTrail | Config: **trạng thái và thay đổi cấu hình** + compliance rule; CloudTrail: **ai gọi API gì, khi nào** | "Who changed it?" → CloudTrail; "Is it compliant?" → Config |
| Amazon Inspector | Amazon Macie | Inspector: **lỗ hổng phần mềm** (EC2, ECR, Lambda); Macie: **data nhạy cảm** trong S3 | CVE → Inspector; PII → Macie |
| Amazon Inspector | Trusted Advisor | Inspector: quét lỗ hổng **sâu** ở workload; Trusted Advisor: gợi ý best practice **mức account** | |
| AWS Artifact | AWS Audit Manager | Artifact: **báo cáo của AWS** (AWS đã đạt chuẩn gì); Audit Manager: **bằng chứng về workload của bạn** | "AWS's ISO certificate" → Artifact |
| Audit Manager | Config Conformance Packs | Audit Manager không nhận khách mới; Conformance Packs là hướng thay thế để triển khai và theo dõi control theo framework | Khách mới → Conformance Packs |
| Internet Gateway | NAT Gateway | IGW: public subnet ra/vào Internet; NAT: private subnet **chỉ đi ra** | "Private instance needs updates" → NAT |
| NAT Gateway | VPC Endpoint | NAT vẫn đi **qua Internet**; VPC Endpoint giữ traffic **trong mạng AWS** | "Without going over the internet" → VPC Endpoint |
| `[+]` Gateway endpoint | Interface endpoint (PrivateLink) | Gateway: chỉ S3 và DynamoDB, miễn phí, qua route table; Interface: PrivateLink, hầu hết dịch vụ (Bedrock, SageMaker API...) | Bedrock → Interface endpoint |
| Guardrails | IAM | Guardrails kiểm soát **nội dung** prompt/response; IAM kiểm soát **ai được gọi** API | |
| KMS key policy / quyền decrypt | Quyền S3 | Đọc bucket SSE-KMS cần **cả** quyền S3 **và** quyền decrypt KMS | Bedrock lỗi đọc data đã mã hóa → thiếu kms:Decrypt |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "grant an AWS service permission to access another service" | IAM Role | |
| "give minimum required permissions" | Least privilege (IAM policy) | |
| "store large training datasets durably" | Amazon S3 | |
| "rarely accessed archive, cheapest, retrieval within 48h OK" | S3 Glacier Deep Archive | |
| "archive accessed quarterly, needs millisecond access" | S3 Glacier Instant Retrieval | |
| "unpredictable access pattern, optimize cost automatically" | S3 Intelligent-Tiering | |
| "move objects to cheaper class after N days" | S3 Lifecycle | |
| "run code on S3 upload, no servers" | AWS Lambda (trigger từ S3) | |
| "run task every hour serverlessly" | EventBridge schedule + Lambda | |
| "discover PII in S3 buckets" | Amazon Macie | |
| "track configuration changes, check compliance rules" | AWS Config | |
| "deploy compliance controls (HIPAA, PCI) as rules across accounts" | Config Conformance Packs | |
| "scan EC2 / container images / Lambda for vulnerabilities" | Amazon Inspector | |
| "who made this API call", "resource deleted, investigate" | AWS CloudTrail | |
| "download AWS compliance reports", "accept BAA" | AWS Artifact | |
| "continuously collect audit evidence" | ⚠️ Audit Manager (đề cũ) → Config Conformance Packs | |
| "best-practice checks: cost, security, service limits" | AWS Trusted Advisor | |
| "private instances need outbound internet" | NAT Gateway | |
| "access Bedrock from VPC without internet" | Interface VPC Endpoint (PrivateLink) | |
| "SageMaker in private subnet reads S3 privately" | S3 Gateway Endpoint | |
| "Bedrock custom model training fails on encrypted bucket" | IAM role cần quyền S3 + KMS decrypt | |
| "log which users invoked Bedrock APIs" | CloudTrail | |
| "restrict Bedrock topics / filter harmful content" | Bedrock Guardrails | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Amazon Bedrock cần đọc dataset trong một S3 bucket được mã hóa bằng **SSE-KMS** để fine-tune custom model, nhưng job bị lỗi Access Denied. Nguyên nhân có khả năng nhất?
- A. Bucket chưa bật versioning
- B. IAM role của Bedrock thiếu quyền **decrypt** trên KMS key (hoặc thiếu quyền đọc S3)
- C. Bedrock không hỗ trợ data trong S3
- D. Cần bật Trusted Advisor

<details><summary>Đáp án</summary>

**B.** Bedrock cần IAM role có cả quyền đọc S3 lẫn quyền decrypt trên KMS key dùng để mã hóa bucket.
- A sai: versioning không liên quan đến quyền truy cập.
- C sai: Bedrock dùng S3 làm nguồn data training.
- D sai: Trusted Advisor chỉ đưa gợi ý, không cấp quyền.
</details>

**Câu 2.** (Chọn 2) Một ứng dụng chạy trong **private subnet** cần gọi Amazon Bedrock và đọc file từ S3 mà **không đi qua Internet công cộng**. Cần gì?
- A. Interface VPC endpoint cho Bedrock (AWS PrivateLink)
- B. S3 Gateway Endpoint
- C. NAT Gateway
- D. Internet Gateway gắn vào private subnet
- E. Public IP cho instance

<details><summary>Đáp án</summary>

**A, B.** VPC endpoint giữ traffic trong mạng AWS: interface endpoint cho Bedrock, gateway endpoint cho S3.
- C sai: NAT vẫn đi ra Internet.
- D, E sai: biến tài nguyên thành public, đi ngược yêu cầu.
</details>

**Câu 3.** Team bảo mật muốn biết **user nào** đã xóa một custom model trên Bedrock tuần trước. Dùng dịch vụ nào?
- A. AWS Config
- B. AWS CloudTrail
- C. Amazon Inspector
- D. Amazon Macie

<details><summary>Đáp án</summary>

**B.** CloudTrail ghi lại mọi API call (ai, lúc nào, từ đâu); event history giữ 90 ngày.
- A sai: Config cho biết trạng thái cấu hình, không trả lời "ai gọi".
- C sai: Inspector quét lỗ hổng.
- D sai: Macie tìm PII.
</details>

**Câu 4.** Trước khi dùng các file log CSKH trong S3 để fine-tune model, công ty muốn tự động phát hiện file nào chứa **số thẻ tín dụng và số điện thoại**. Dùng dịch vụ nào?
- A. Amazon Macie
- B. AWS Artifact
- C. AWS Trusted Advisor
- D. Amazon Inspector

<details><summary>Đáp án</summary>

**A.** Macie dùng ML và pattern matching để phát hiện PII trong S3.
- B sai: Artifact cung cấp báo cáo compliance.
- C sai: Trusted Advisor đưa gợi ý best practice.
- D sai: Inspector quét lỗ hổng phần mềm.
</details>

**Câu 5.** Một công ty lưu data training cũ cần giữ 7 năm theo quy định, **gần như không bao giờ truy cập**, chấp nhận chờ tới 48 giờ khi cần lấy lại, và muốn **chi phí thấp nhất**. Chọn storage class nào?
- A. S3 Standard
- B. S3 Standard-IA
- C. S3 Glacier Instant Retrieval
- D. S3 Glacier Deep Archive

<details><summary>Đáp án</summary>

**D.** Deep Archive là class rẻ nhất cho lưu trữ dài hạn, retrieval 12–48 giờ.
- A, B sai: đắt hơn và dành cho data truy cập thường xuyên hơn.
- C sai: cho truy cập mili giây, đắt hơn Deep Archive.
</details>

**Câu 6.** (Chọn 2) Phát biểu nào **đúng** về Amazon S3?
- A. Durability 11 số 9 áp dụng cho mọi storage class
- B. Availability giống nhau ở mọi storage class
- C. Bucket được tạo trong một Region cụ thể
- D. S3 One Zone-IA lưu data trên ít nhất 3 AZ
- E. Object lớn hơn 5 GB có thể upload bằng một PUT đơn

<details><summary>Đáp án</summary>

**A, C.**
- B sai: availability khác nhau theo class (vd Standard 99.99%, One Zone-IA 99.5%).
- D sai: One Zone-IA chỉ dùng 1 AZ.
- E sai: object trên 5 GB phải dùng multipart upload.
</details>

**Câu 7.** Một kiểm toán viên yêu cầu báo cáo **PCI DSS và SOC 2 của AWS**, đồng thời công ty y tế cần ký **Business Associate Addendum (BAA)** để dùng AWS cho dữ liệu HIPAA. Dùng gì?
- A. AWS Config
- B. AWS Artifact
- C. AWS CloudTrail
- D. Amazon Macie

<details><summary>Đáp án</summary>

**B.** Artifact Reports cung cấp báo cáo compliance; Artifact Agreements để xem và chấp nhận BAA.
- A, C, D sai: không cung cấp báo cáo hay thỏa thuận compliance của AWS.
</details>

**Câu 8.** Team DevOps muốn **tự động kiểm tra** xem có security group nào mở SSH (port 22) ra toàn Internet và xem lịch sử thay đổi cấu hình của tài nguyên. Dùng dịch vụ nào?
- A. AWS Config
- B. Amazon Inspector
- C. AWS Artifact
- D. AWS Lambda

<details><summary>Đáp án</summary>

**A.** Config ghi cấu hình theo thời gian và đánh giá compliance theo rule (vd: không cho mở SSH ra 0.0.0.0/0).
- B sai: Inspector có network reachability cho EC2 nhưng không quản lý lịch sử cấu hình hay compliance rule.
- C sai: Artifact cung cấp báo cáo compliance của AWS.
- D sai: Lambda chỉ chạy code, không phải dịch vụ compliance.
</details>

---

### Nguồn đã đối chiếu
- AIF-C01 exam guide, Domain 5 (IAM, encryption, Macie, PrivateLink, Config, Inspector, Artifact, CloudTrail, Trusted Advisor): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain5.html
- S3 bucket naming (account regional namespace): https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html
- S3 multipart upload limits (max object 48.8 TiB): https://docs.aws.amazon.com/AmazonS3/latest/userguide/qfacts.html
- S3 storage classes: https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html
- Lambda quotas (timeout 15 phút): https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
- Lambda pricing (free tier 1M requests, 400.000 GB-s; $0.20/1M requests): https://aws.amazon.com/lambda/pricing/
- Amazon Inspector EC2 scanning (agent-based / agentless): https://docs.aws.amazon.com/inspector/latest/user/scanning-ec2.html
- CloudTrail (event history 90 ngày): https://docs.aws.amazon.com/awscloudtrail/latest/userguide/how-cloudtrail-works.html
- AWS Artifact third-party reports (Vendor Insights): https://docs.aws.amazon.com/marketplace/latest/buyerguide/buyer-vendor-insights.html
- AWS Audit Manager availability change → Config Conformance Packs: https://docs.aws.amazon.com/audit-manager/latest/userguide/audit-manager-availability-change.html
- Trusted Advisor categories: https://docs.aws.amazon.com/awssupport/latest/user/get-started-with-aws-trusted-advisor.html
- Trusted Advisor check reference (thay đổi support plans 01/01/2027): https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor-check-reference.html
- Gateway VPC endpoints (S3, DynamoDB): https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html
- Amazon Macie (sensitive data discovery): https://docs.aws.amazon.com/whitepapers/latest/navigating-gdpr-compliance/discovering-and-protecting-data-at-scale-with-amazon-macie.html
