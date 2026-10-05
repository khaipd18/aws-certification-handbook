# CLF-C02 — Security & Compliance

> Chương nặng nhất của **Domain 2: Security and Compliance (30%)**. Trọng tâm: Shared Responsibility theo từng dịch vụ, chống DDoS, mã hóa, và "dịch vụ bảo mật nào làm gì".
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Shared Responsibility:** AWS = security **OF** the cloud (hạ tầng, dịch vụ managed); khách hàng = security **IN** the cloud (guest OS của EC2, firewall, IAM, mã hóa dữ liệu). **Shared controls**: patch management, configuration management, awareness & training.
- **DDoS:** **Shield Standard** (miễn phí, tự bật, Layer 3/4) và **Shield Advanced** (trả phí, 24/7 Shield Response Team, bảo vệ chi phí khi bị tấn công); **WAF** lọc request Layer 7 (SQL injection, XSS, rate-based).
- **Mã hóa:** **KMS** (AWS quản lý phần mềm và key) vs **CloudHSM** (AWS cấp phần cứng HSM, **bạn tự quản key**); **ACM** cấp chứng chỉ TLS; **Secrets Manager** lưu secret có **tự xoay vòng**.
- **Phát hiện & đánh giá:** **GuardDuty** (threat detection bằng ML từ CloudTrail, VPC Flow Logs, DNS logs), **Inspector** (lỗ hổng phần mềm EC2/ECR/Lambda), **Macie** (PII trong S3), **Config** (lịch sử cấu hình & compliance), **Detective** (tìm root cause), **Security Hub** (gom finding tập trung).
- **Compliance:** **Artifact** = cổng tải báo cáo compliance (SOC, ISO, PCI) và ký thỏa thuận (BAA).
- **Root user** có một số quyền riêng (đổi email/mật khẩu root, đóng account standalone, đăng ký bán RI...). **IAM Access Analyzer** tìm tài nguyên chia sẻ ra ngoài vùng tin cậy.
- **Pentest** trên một số dịch vụ không cần xin phép; **cấm** DoS/DDoS và flooding.

---

## 2. Core Concepts

### Shared Responsibility Model
- **Là gì:** AWS bảo vệ hạ tầng (phần cứng, phần mềm, cơ sở vật chất, mạng) chạy mọi dịch vụ, kể cả dịch vụ managed (S3, DynamoDB, RDS).
- **Dùng khi nào:** khách hàng: với EC2 phải quản **guest OS** (vá bảo mật), firewall & network config, IAM, **mã hóa dữ liệu ứng dụng**.
- **Vì sao quan trọng:** **Shared controls** (cả hai cùng làm, mỗi bên ở tầng của mình): **Patch Management, Configuration Management, Awareness & Training**.

### Ví dụ: RDS
- **Là gì:** AWS: quản EC2 bên dưới, **tắt SSH**, **vá DB và OS tự động**, audit instance và disk.
- **Dùng khi nào:** khách hàng: kiểm tra port/IP/security group inbound của DB, tạo user và quyền trong DB, quyết định DB có public hay không, cấu hình chỉ cho phép kết nối **SSL**, bật **mã hóa DB**.
- **Vì sao quan trọng:** "ai vá OS của RDS" → **AWS** (khác EC2).

### Ví dụ: S3
- **Là gì:** AWS: đảm bảo storage không giới hạn, cung cấp khả năng mã hóa, **tách biệt dữ liệu giữa các khách hàng**, nhân viên AWS không truy cập được dữ liệu của bạn.
- **Dùng khi nào:** khách hàng: cấu hình bucket, bucket policy/public setting, IAM user & role, **bật mã hóa**.
- **Vì sao quan trọng:** cấu hình sai bucket là lỗi của khách hàng, không phải AWS.

### DDoS & bảo vệ trên AWS
- **Là gì:** DDoS (Distributed Denial-of-Service) = nhiều bot đồng loạt làm server không phản hồi được.
- **Dùng khi nào:** kiến trúc tham chiếu: **Shield Standard** (mọi khách hàng), **Shield Advanced** (24/7), **WAF** (lọc request), **CloudFront + Route 53** (bảo vệ availability ở edge), **Auto Scaling** (sẵn sàng scale).
- **Vì sao quan trọng:** đề hỏi "những dịch vụ nào giúp giảm tác động DDoS" thường chọn Shield, WAF, CloudFront, Route 53.

### AWS Shield
- **Là gì:** **Shield Standard**: miễn phí, tự bật cho mọi khách hàng, chống SYN/UDP flood, reflection attack và tấn công **Layer 3/4**.
- **Dùng khi nào:** **Shield Advanced**: trả phí (slide: $3.000/tháng/organization), chống tấn công tinh vi hơn trên EC2, ELB, CloudFront, Global Accelerator, Route 53; **bảo vệ khỏi phí tăng đột biến** do DDoS.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: tên "DDoS Response Team (DRT/DRP)" → nay gọi là **Shield Response Team (SRT)**; muốn dùng SRT cần thêm gói AWS Support cấp business trở lên.

### AWS WAF (Web Application Firewall)
- **Là gì:** bảo vệ web app khỏi exploit phổ biến ở **Layer 7** (HTTP); deploy trên **ALB, API Gateway, CloudFront**.
- **Dùng khi nào:** định nghĩa **Web ACL**: rule theo IP, HTTP header, body, URI; chống **SQL injection** và **XSS**; giới hạn kích thước, **geo-match** (chặn quốc gia).
- **Vì sao quan trọng:** **rate-based rule** đếm số request để chống DDoS ở Layer 7.

### AWS Network Firewall
- **Là gì:** firewall bảo vệ **toàn bộ VPC** từ Layer 3 tới Layer 7, kiểm tra mọi hướng: VPC ↔ VPC, ra/vào Internet, Direct Connect, Site-to-Site VPN.
- **Dùng khi nào:** cần kiểm tra traffic ở cấp mạng cho cả VPC.
- **Vì sao quan trọng:** exam guide CLF-C02 hiện xếp Network Firewall vào **out-of-scope**, vẫn nên biết để loại đáp án.

### AWS Firewall Manager
- **Là gì:** quản lý **rule bảo mật cho mọi account trong AWS Organization** từ một chỗ.
- **Dùng khi nào:** security policy chung gồm: Security Group (EC2, ALB...), WAF rule, Shield Advanced, Network Firewall.
- **Vì sao quan trọng:** rule tự áp dụng cho **tài nguyên mới và account mới** trong Organization → tốt cho compliance.

### Penetration Testing trên AWS
- **Là gì:** khách hàng được tự kiểm thử bảo mật **không cần xin phép trước** trên một số dịch vụ: EC2, NAT Gateway, ELB, RDS, CloudFront, Aurora, API Gateway, Lambda & Lambda@Edge, Lightsail, Elastic Beanstalk.
- **Dùng khi nào:** **bị cấm**: DNS zone walking qua Route 53 hosted zone, **DoS/DDoS** (kể cả giả lập), port/protocol/request flooding.
- **Vì sao quan trọng:** danh sách dịch vụ có thể thay đổi, đề không hỏi chi tiết; chỉ cần nhớ "DoS/DDoS bị cấm".

### Data at rest vs in transit
- **Là gì:** **at rest** = dữ liệu lưu trên thiết bị (ổ cứng, RDS, Glacier Deep Archive); **in transit** = dữ liệu di chuyển qua mạng (on-prem → AWS, EC2 → DynamoDB).
- **Dùng khi nào:** mã hóa ở cả hai trạng thái bằng encryption key.
- **Vì sao quan trọng:** in transit dùng TLS (chứng chỉ ACM); at rest dùng KMS/CloudHSM.

### AWS KMS (Key Management Service)
- **Là gì:** "Encryption trên AWS" → hầu như là **KMS**; AWS quản lý encryption key cho bạn.
- **Dùng khi nào:** **mã hóa tùy chọn (opt-in)**: EBS, S3 (**SSE-S3 bật mặc định**, SSE-KMS tùy chọn), Redshift, RDS, EFS; **tự động bật**: CloudTrail Logs, S3 Glacier, Storage Gateway.
- **Vì sao quan trọng:** từ khóa "managed encryption keys", "integrated with AWS services" → KMS.

### Các loại KMS key
- **Là gì:** **Customer managed key**: bạn tạo, quản, bật/tắt; có thể xoay vòng, có thể bring-your-own-key; **AWS managed key**: AWS tạo và dùng thay bạn (`aws/s3`, `aws/ebs`, `aws/redshift`); **AWS owned key**: AWS sở hữu, dùng cho nhiều account, bạn không xem được.
- **Dùng khi nào:** **CloudHSM key (custom key store)**: key sinh trong thiết bị CloudHSM của bạn, thao tác mật mã chạy trong cluster CloudHSM.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: thuật ngữ "CMK" và "xoay vòng mỗi năm" → nay gọi là **KMS key**; automatic rotation mặc định mỗi năm nhưng **chỉnh được rotation period**, và có **on-demand rotation**.

### AWS CloudHSM
- **Là gì:** AWS cấp **phần cứng mã hóa chuyên dụng** (HSM); **bạn tự quản key hoàn toàn**, AWS không quản key.
- **Dùng khi nào:** yêu cầu compliance phải kiểm soát key trên HSM riêng, chống can thiệp vật lý (tamper resistant).
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "FIPS 140-2 Level 3" → loại HSM mới **hsm2m.medium** đạt **FIPS 140-3 Level 3** (loại cũ hsm1 là FIPS 140-2 Level 3).

### AWS Certificate Manager (ACM)
- **Là gì:** dễ dàng provision, quản lý, deploy **chứng chỉ SSL/TLS** để mã hóa in-flight (HTTPS).
- **Dùng khi nào:** hỗ trợ chứng chỉ public và private; **miễn phí** chứng chỉ public; **tự gia hạn**.
- **Vì sao quan trọng:** tích hợp **ELB, CloudFront, API Gateway**.

### AWS Secrets Manager
- **Là gì:** dịch vụ lưu **secret**, có khả năng **bắt buộc xoay vòng** secret mỗi X ngày, tự sinh secret mới khi xoay (dùng Lambda).
- **Dùng khi nào:** tích hợp **RDS** (MySQL, PostgreSQL, Aurora); secret được mã hóa bằng KMS.
- **Vì sao quan trọng:** "rotate database credentials automatically" → Secrets Manager (không phải Parameter Store).

### AWS Artifact
- **Là gì:** **cổng** (không hẳn là dịch vụ) cho truy cập on-demand vào tài liệu compliance và thỏa thuận của AWS.
- **Dùng khi nào:** **Artifact Reports**: tải báo cáo của kiểm toán viên bên thứ ba (ISO, PCI, SOC); **Artifact Agreements**: xem, chấp nhận, theo dõi thỏa thuận như **BAA (HIPAA)** cho một account hoặc cả Organization.
- **Vì sao quan trọng:** hỗ trợ audit nội bộ hoặc compliance. "Download SOC 2 report" → Artifact.

### Amazon GuardDuty
- **Là gì:** **phát hiện mối đe dọa thông minh** bằng ML, phát hiện bất thường và dữ liệu bên thứ ba; bật một click (dùng thử 30 ngày), không cần cài phần mềm.
- **Dùng khi nào:** nguồn: **CloudTrail** (management events, S3 data events), **VPC Flow Logs**, **DNS logs**; tính năng tùy chọn: EKS audit logs & runtime, RDS/Aurora login, EBS, Lambda network, S3.
- **Vì sao quan trọng:** gửi finding qua **EventBridge** (→ Lambda/SNS); có finding riêng cho **tấn công đào tiền mã hóa**.

### Amazon Inspector
- **Là gì:** **đánh giá bảo mật tự động** cho **EC2** (lỗ hổng OS, network reachability), **container image trong ECR** (khi push), **Lambda** (lỗ hổng trong code và dependency).
- **Dùng khi nào:** quét liên tục, mỗi lỗ hổng có **risk score** để ưu tiên; báo cáo vào Security Hub, gửi EventBridge.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "EC2 quét qua SSM agent" → nay mặc định là **hybrid**: dùng SSM agent hoặc **agentless** (quét qua EBS snapshot). Inspector **chỉ** cho EC2, ECR, Lambda.

### AWS Config
- **Là gì:** ghi lại **cấu hình tài nguyên và thay đổi theo thời gian**, đánh giá **compliance** theo rule.
- **Dùng khi nào:** trả lời: SG có cho SSH không giới hạn không? Bucket có public không? Cấu hình ALB đổi thế nào theo thời gian? Gửi cảnh báo SNS khi có thay đổi; lưu dữ liệu vào S3 (phân tích bằng Athena).
- **Vì sao quan trọng:** dịch vụ **theo Region**, gom được nhiều Region và account; xem được CloudTrail API call liên quan tới tài nguyên.

### Amazon Macie
- **Là gì:** dịch vụ bảo mật và quyền riêng tư dữ liệu, dùng **ML và pattern matching** để tìm và bảo vệ dữ liệu nhạy cảm.
- **Dùng khi nào:** phát hiện và cảnh báo **PII** trong **S3 bucket**; gửi kết quả qua EventBridge.
- **Vì sao quan trọng:** "PII in S3" → Macie.

### AWS Security Hub
- **Là gì:** công cụ bảo mật **trung tâm** cho nhiều account: dashboard tổng hợp, kiểm tra bảo mật tự động, gom alert từ Config, GuardDuty, Inspector, Macie, IAM Access Analyzer, Systems Manager, Firewall Manager, Health, đối tác.
- **Dùng khi nào:** cần một chỗ xem tình trạng bảo mật và compliance của cả Organization.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: Security Hub chỉ là "aggregator cần bật Config trước" → từ **12/2025**, dịch vụ cũ đổi tên thành **Security Hub CSPM** (kiểm tra control, cần AWS Config), còn **AWS Security Hub** mới là giải pháp hợp nhất: tương quan finding (GuardDuty, Inspector, Macie, CSPM) thành **exposure finding**, attack path graph, unused access analysis.

### Amazon Detective
- **Là gì:** **phân tích, điều tra, tìm root cause** của vấn đề bảo mật hoặc hoạt động đáng ngờ bằng ML và graph.
- **Dùng khi nào:** sau khi GuardDuty/Macie/Security Hub đưa ra finding, cần điều tra sâu.
- **Vì sao quan trọng:** tự thu thập event từ VPC Flow Logs, CloudTrail, GuardDuty để tạo góc nhìn thống nhất. "Root cause investigation" → Detective.

### AWS Abuse
- **Là gì:** kênh báo cáo tài nguyên AWS bị dùng vào mục đích lạm dụng hoặc phạm pháp.
- **Dùng khi nào:** spam từ IP của AWS, port scanning, DoS/DDoS, cố gắng xâm nhập, host nội dung vi phạm bản quyền, phát tán malware.
- **Vì sao quan trọng:** liên hệ qua **AWS abuse form** hoặc `abuse@amazonaws.com`.

### Root user privileges
- **Là gì:** root = chủ account, toàn quyền; khóa access key của root, không dùng cho việc hằng ngày kể cả quản trị.
- **Dùng khi nào:** chỉ root làm được: đổi **email, mật khẩu, access key của root** (account standalone); **đóng account** standalone; **khôi phục quyền IAM** khi admin tự khóa mình; xem một số hóa đơn thuế; **đăng ký bán trên RI Marketplace**; **bật MFA Delete cho S3**; sửa/xóa bucket policy (hoặc SQS policy) đang **deny mọi principal**; **đăng ký GovCloud**; bật IAM access vào Billing console.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: slide liệt kê "đổi tên account" và "**đổi/hủy AWS Support plan**" là tác vụ chỉ root → danh sách hiện tại của IAM **không còn** hai mục này; với AWS Organizations, nhiều tác vụ (đóng member account, đổi email root) làm được từ management account.

### IAM Access Analyzer
- **Là gì:** tìm tài nguyên được **chia sẻ ra ngoài** vùng tin cậy: S3 bucket, IAM role, KMS key, Lambda function & layer, SQS queue, Secrets Manager secret.
- **Dùng khi nào:** định nghĩa **Zone of Trust** = AWS Account hoặc AWS Organization; truy cập từ ngoài zone → **finding**.
- **Vì sao quan trọng:** `[+]` nay còn có **unused access analysis**: tìm role, user, access key, quyền không dùng để thực thi least privilege.

### `[+]` AWS Security Incident Response
- **Là gì:** dịch vụ giúp chuẩn bị, phản ứng và nhận hướng dẫn khôi phục sau sự cố bảo mật (chiếm account, rò rỉ dữ liệu, ransomware).
- **Dùng khi nào:** đi kèm gói **Enterprise Support** (chương 16).
- **Vì sao quan trọng:** đề mới có thể hỏi "dịch vụ hỗ trợ phản ứng sự cố bảo mật".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Shared controls | Patch, configuration management, training | "responsibility of both AWS and customer" |
| DDoS | Tấn công từ chối dịch vụ phân tán | "flood of traffic", "website unavailable" |
| Shield Standard | Chống DDoS L3/L4 miễn phí, tự bật | "no additional cost DDoS protection" |
| Shield Advanced | DDoS cao cấp, SRT 24/7, bảo vệ chi phí | "24/7 DDoS response team", "cost protection" |
| AWS WAF | Firewall Layer 7, Web ACL | "SQL injection", "XSS", "block countries", "rate-based" |
| AWS Network Firewall | Firewall cấp VPC, L3–L7 | "protect entire VPC" |
| AWS Firewall Manager | Quản lý rule bảo mật cho cả Organization | "apply WAF rules across all accounts" |
| Penetration testing | Kiểm thử bảo mật; DoS bị cấm | "allowed security testing" |
| Data at rest / in transit | Dữ liệu lưu / đang truyền | "stored" / "moving over network" |
| AWS KMS | Dịch vụ quản lý key mã hóa managed | "encryption keys managed by AWS" |
| Customer managed key | KMS key bạn tạo và quản | "control key rotation and policy" |
| AWS managed key | KMS key AWS tạo cho dịch vụ (`aws/s3`) | "default service key" |
| AWS CloudHSM | HSM chuyên dụng, bạn tự quản key | "dedicated hardware", "FIPS 140 Level 3" |
| ACM | Chứng chỉ SSL/TLS, public miễn phí, tự gia hạn | "HTTPS certificate" |
| Secrets Manager | Lưu secret, tự xoay vòng | "rotate RDS credentials" |
| AWS Artifact | Báo cáo compliance và thỏa thuận | "SOC report", "BAA", "PCI" |
| Amazon GuardDuty | Threat detection bằng ML | "malicious activity", "crypto mining" |
| Amazon Inspector | Quét lỗ hổng EC2, ECR, Lambda | "CVE", "software vulnerabilities" |
| AWS Config | Lịch sử cấu hình và compliance | "configuration changes over time" |
| Amazon Macie | Tìm PII trong S3 | "sensitive data in S3" |
| AWS Security Hub | Tổng hợp tình trạng bảo mật nhiều account | "central security dashboard" |
| Amazon Detective | Điều tra root cause | "investigate root cause" |
| AWS Abuse | Báo cáo tài nguyên AWS bị lạm dụng | "spam from AWS IP", "report abuse" |
| IAM Access Analyzer | Tài nguyên chia sẻ ra ngoài zone of trust | "shared externally" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Shield | WAF | DDoS L3/L4 (Standard) vs lọc request L7 theo rule | "SQL injection" → WAF |
| Shield Standard | Shield Advanced | Miễn phí, tự động vs trả phí, SRT 24/7, bảo vệ chi phí | "24/7 DDoS experts" → Advanced |
| WAF | Network Firewall | Web app (ALB, API GW, CloudFront) vs cả VPC | "Protect entire VPC" → Network Firewall |
| WAF | Firewall Manager | Một bộ rule vs quản lý rule trên cả Organization | "Across all accounts" → Firewall Manager |
| KMS | CloudHSM | AWS quản key vs bạn tự quản key trên HSM riêng | "Full control of keys, dedicated hardware" → CloudHSM |
| Secrets Manager | Parameter Store | Có tự xoay vòng secret (RDS) vs lưu config/secret đơn giản | "Automatic rotation" → Secrets Manager |
| ACM | KMS | Chứng chỉ TLS (in transit) vs key mã hóa (at rest) | "HTTPS" → ACM |
| GuardDuty | Inspector | Phát hiện **mối đe dọa đang xảy ra** vs **lỗ hổng phần mềm** | "Unusual API calls" → GuardDuty; "CVE" → Inspector |
| GuardDuty | Macie | Threat từ log vs dữ liệu nhạy cảm trong S3 | "PII" → Macie |
| Config | CloudTrail | Cấu hình tài nguyên theo thời gian vs ai gọi API | "Is SG compliant" → Config |
| Security Hub | Detective | Gom finding & kiểm tra tự động vs điều tra root cause | "Investigate" → Detective |
| Artifact | Config | Báo cáo compliance **của AWS** vs compliance **tài nguyên của bạn** | "Download ISO report" → Artifact |
| IAM Access Analyzer | Trusted Advisor | Tài nguyên chia sẻ ra ngoài vs khuyến nghị best practice | |
| AWS managed key | Customer managed key | AWS quản xoay vòng vs bạn kiểm soát policy, xoay vòng | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "free DDoS protection for all customers" | Shield Standard | |
| "24/7 DDoS response team, cost protection" | Shield Advanced | |
| "block SQL injection / XSS / specific countries" | AWS WAF | |
| "centrally manage firewall rules across Organization" | Firewall Manager | |
| "manage encryption keys, integrated with S3/EBS" | AWS KMS | |
| "dedicated HSM, customer manages keys, FIPS 140 Level 3" | AWS CloudHSM | |
| "free public TLS certificates with auto renewal" | ACM | |
| "rotate database passwords automatically" | Secrets Manager | |
| "download SOC/PCI/ISO reports, accept BAA" | AWS Artifact | |
| "detect compromised instance, crypto mining, unusual API" | GuardDuty | |
| "scan EC2/ECR/Lambda for vulnerabilities (CVE)" | Amazon Inspector | |
| "track configuration changes and compliance rules" | AWS Config | |
| "discover PII in S3" | Amazon Macie | |
| "aggregate security findings from multiple accounts" | AWS Security Hub | |
| "find root cause of security finding" | Amazon Detective | |
| "report AWS resource used for spam/attack" | AWS Abuse | |
| "which resources are shared outside my account/org" | IAM Access Analyzer | |
| "close standalone AWS account" | Root user | |

---

## 5. Câu hỏi tình huống

**Câu 1.** (Chọn 2) Với Amazon RDS, trách nhiệm nào thuộc về **khách hàng**?
- A. Vá hệ điều hành của DB instance
- B. Cấu hình security group inbound cho DB
- C. Tạo user và phân quyền bên trong database
- D. Bảo trì phần cứng của host
- E. Tắt quyền SSH vào instance bên dưới

<details><summary>Đáp án</summary>

**B, C.**
- A, D, E sai: AWS vá OS/DB, bảo trì phần cứng và tắt SSH.
</details>

**Câu 2.** Website bị tấn công SQL injection. Dịch vụ nào chặn được ở tầng ứng dụng?
- A. AWS Shield Standard
- B. AWS WAF
- C. Amazon GuardDuty
- D. AWS Config

<details><summary>Đáp án</summary>

**B.** WAF lọc request Layer 7, có rule chống SQL injection.
- A sai: chống DDoS Layer 3/4.
- C sai: phát hiện threat, không chặn request web.
- D sai: theo dõi cấu hình.
</details>

**Câu 3.** Công ty phải tự quản lý toàn bộ key mã hóa trên phần cứng chuyên dụng, AWS không được truy cập key. Chọn gì?
- A. AWS KMS với AWS managed key
- B. AWS CloudHSM
- C. AWS Certificate Manager
- D. AWS Secrets Manager

<details><summary>Đáp án</summary>

**B.** CloudHSM cấp HSM chuyên dụng, khách hàng tự quản key.
- A sai: AWS quản key.
- C sai: chứng chỉ TLS.
- D sai: lưu secret.
</details>

**Câu 4.** Kiểm toán viên cần báo cáo SOC 2 về hạ tầng AWS. Lấy ở đâu?
- A. AWS Config
- B. AWS Artifact
- C. AWS Security Hub
- D. AWS Trusted Advisor

<details><summary>Đáp án</summary>

**B.** Artifact cung cấp báo cáo compliance của AWS.
- A sai: compliance tài nguyên của bạn.
- C sai: tổng hợp finding bảo mật.
- D sai: khuyến nghị best practice.
</details>

**Câu 5.** Team muốn phát hiện EC2 bị chiếm quyền đang đào tiền mã hóa, dựa trên CloudTrail, VPC Flow Logs và DNS logs, không cài agent. Chọn gì?
- A. Amazon Inspector
- B. Amazon GuardDuty
- C. Amazon Macie
- D. AWS Artifact

<details><summary>Đáp án</summary>

**B.** GuardDuty phân tích log bằng ML, có finding riêng cho crypto mining.
- A sai: quét lỗ hổng phần mềm.
- C sai: PII trong S3.
- D sai: báo cáo compliance.
</details>

**Câu 6.** (Chọn 2) Amazon Inspector đánh giá bảo mật cho tài nguyên nào?
- A. EC2 instances
- B. Container image trong Amazon ECR
- C. S3 bucket chứa PII
- D. IAM password policy
- E. Route 53 hosted zone

<details><summary>Đáp án</summary>

**A, B.** (Và Lambda function.)
- C sai: đó là Macie.
- D, E sai: Inspector không đánh giá các tài nguyên này.
</details>

**Câu 7.** Công ty muốn biết cấu hình security group đã thay đổi thế nào trong 6 tháng qua và có SG nào mở SSH cho 0.0.0.0/0 không. Chọn gì?
- A. AWS CloudTrail
- B. AWS Config
- C. Amazon Detective
- D. AWS Shield

<details><summary>Đáp án</summary>

**B.** Config ghi lịch sử cấu hình và đánh giá compliance theo rule.
- A sai: cho biết ai gọi API, không đánh giá compliance cấu hình.
- C sai: điều tra root cause.
- D sai: DDoS.
</details>

**Câu 8.** Theo AWS, hoạt động nào **bị cấm** khi tự pentest trên AWS?
- A. Quét lỗ hổng EC2 của chính mình
- B. Kiểm thử API Gateway của chính mình
- C. Giả lập tấn công DDoS
- D. Kiểm thử bảo mật RDS của chính mình

<details><summary>Đáp án</summary>

**C.** DoS/DDoS, kể cả giả lập, và các loại flooding đều bị cấm.
- A, B, D sai: được phép trên tài nguyên của bạn mà không cần xin phép trước.
</details>

---

### Nguồn đã đối chiếu
- Shield Response Team (SRT) support: https://docs.aws.amazon.com/waf/latest/developerguide/ddos-srt-support.html
- Rotate AWS KMS keys (rotation period tùy chỉnh, on-demand rotation): https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html
- CloudHSM HSM types (hsm2m.medium FIPS 140-3): https://docs.aws.amazon.com/cloudhsm/latest/userguide/hsm-types.html
- Amazon Inspector scan types (hybrid, agentless): https://docs.aws.amazon.com/inspector/latest/user/scanning-resources.html
- Introduction to AWS Security Hub (bản hợp nhất mới): https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub-v2.html
- Security Hub document history (GA 02/12/2025, Security Hub CSPM): https://docs.aws.amazon.com/securityhub/latest/userguide/doc-history.html
- Tasks that require root user credentials: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html
- CLF-C02 out-of-scope services (AWS Network Firewall): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
- AWS Support plans (Enterprise kèm Security Incident Response): https://docs.aws.amazon.com/awssupport/latest/user/aws-support-plans.html
- Penetration testing policy: https://aws.amazon.com/security/penetration-testing/
