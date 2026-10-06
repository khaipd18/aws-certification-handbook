# SAA-C03 — Security & Encryption: KMS, SSM Parameter Store, Secrets Manager, ACM, CloudHSM, WAF, Shield, Firewall Manager, GuardDuty, Inspector, Macie

> Thuộc **Domain 1: Design Secure Architectures**: mã hóa at rest/in transit, quản lý key và secret, chứng chỉ TLS, firewall ứng dụng, chống DDoS và các dịch vụ phát hiện mối đe dọa.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- Mã hóa: **in flight** (TLS), **server-side at rest** (dịch vụ mã hóa sau khi nhận), **client-side** (client mã hóa, server không bao giờ giải mã được).
- **KMS**: key **AWS owned** (miễn phí, ẩn), **AWS managed** (`aws/s3`, `aws/ebs`... tự rotate mỗi năm), **customer managed** (tự tạo, $1/tháng, rotation tự động với **rotation period tùy chỉnh** (mặc định 365 ngày) + **on-demand**). Key KMS **theo Region**; **multi-Region keys** dùng chung key material.
- ⚠️ Lỗi thời: "KMS key có imported key material không rotate được" → nay rotate **on-demand** được (vẫn không có rotation tự động).
- **SSM Parameter Store**: lưu cấu hình/secret (SecureString qua KMS), có version, tier Standard/Advanced (parameter policy như TTL). **Secrets Manager**: secret có **rotation tự động** bằng Lambda, tích hợp RDS/Aurora/Redshift, **replicate multi-Region**.
- **ACM**: cert TLS public miễn phí, tự gia hạn; ⚠️ cert hiện có hiệu lực **198 ngày**, gia hạn **45 ngày** trước hết hạn (slide ghi 60 ngày: chỉ còn đúng với cert 395 ngày cũ). Cert import không tự gia hạn.
- **CloudHSM**: HSM single-tenant do khách giữ key; ⚠️ loại mới **hsm2m.medium** đạt **FIPS 140-3** (slide ghi FIPS 140-2 Level 3 là của loại cũ).
- **WAF** (Layer 7) gắn ALB, API Gateway, CloudFront, AppSync, Cognito, App Runner, Verified Access, Amplify, AgentCore Gateway — **không gắn NLB**. **Shield** Standard (miễn phí) / Advanced (24/7 **SRT**, bảo vệ chi phí). **Firewall Manager** quản rule tập trung cho Organization.
- Phát hiện: **GuardDuty** (mối đe dọa từ log), **Inspector** (lỗ hổng EC2/ECR/Lambda), **Macie** (PII trong S3).

---

## 2. Core Concepts

### AWS KMS
- **Là gì:** dịch vụ quản lý key mã hóa, tích hợp IAM và hầu hết dịch vụ AWS; mọi lần dùng key được ghi CloudTrail; key **symmetric** (AES-256, không bao giờ lấy key ra, dùng qua API) hoặc **asymmetric** (RSA/ECC, public key tải về được).
- **Dùng khi nào:** mã hóa EBS, S3, RDS, secret; envelope encryption cho dữ liệu > 4 KB (GenerateDataKey).
- **Vì sao quan trọng:** **key policy** kiểm soát truy cập (giống bucket policy); không có key policy cho phép thì không ai dùng được, kể cả root trong một số trường hợp.

### KMS rotation & imported key
- **Là gì:** AWS managed key rotate tự động mỗi năm; customer managed key: bật rotation tự động với **rotation period tùy chỉnh** (mặc định 365 ngày) và **on-demand rotation** (tối đa 25 lần/key) cho key symmetric.
- **Dùng khi nào:** compliance yêu cầu chu kỳ rotate ngắn hơn 1 năm → đặt rotation period.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "imported key material không rotate được" → nay **on-demand** được; rotation tự động vẫn không áp cho imported, asymmetric, HMAC, custom key store.

### Copy dữ liệu mã hóa giữa Region/account
- **Là gì:** snapshot EBS mã hóa sang Region khác: copy và **mã hóa lại bằng key KMS ở Region đích**; sang account khác: chia sẻ snapshot + cho account kia quyền dùng key qua **key policy**.
- **Dùng khi nào:** DR cross-Region, chia sẻ AMI mã hóa giữa account.
- **Vì sao quan trọng:** key KMS không ra khỏi Region (trừ multi-Region keys).

### KMS multi-Region keys
- **Là gì:** key primary + replica ở Region khác có cùng key ID và key material; mã hóa ở Region này giải mã ở Region kia không cần gọi chéo Region.
- **Dùng khi nào:** client-side encryption toàn cầu, DynamoDB Global Tables/Aurora Global với mã hóa phía client, chữ ký số toàn cầu.
- **Vì sao quan trọng:** không phải key global; mỗi bản quản lý độc lập (policy riêng). Không khuyến nghị dùng tràn lan.

### S3 replication với mã hóa
- **Là gì:** object SSE-S3 và không mã hóa replicate mặc định; SSE-C tùy chọn (nay bị chặn mặc định trên bucket mới); **SSE-KMS** phải bật tùy chọn và chỉ định key đích, cập nhật key policy, IAM role của replication cần kms:Decrypt/Encrypt.
- **Dùng khi nào:** CRR cho bucket dùng SSE-KMS.
- **Vì sao quan trọng:** có thể tăng quota KMS vì mỗi object phải giải mã + mã hóa lại; có thể dùng multi-Region key nhưng S3 vẫn coi là key riêng.

### SSM Parameter Store
- **Là gì:** kho cấu hình và secret serverless, có version, mã hóa SecureString bằng KMS, phân cấp đường dẫn (`/app/prod/db-url`), kiểm soát bằng IAM, tích hợp EventBridge, CloudFormation.
- **Dùng khi nào:** cấu hình ứng dụng, tham chiếu secret (đọc được cả secret của Secrets Manager qua đường dẫn đặc biệt), lấy AMI mới nhất qua public parameter.
- **Vì sao quan trọng:** tier **Advanced** cho parameter tới 8 KB và **parameter policies** (TTL/hết hạn, thông báo); không có rotation tự động như Secrets Manager.

### AWS Secrets Manager
- **Là gì:** lưu secret, **bắt buộc rotation sau N ngày** bằng Lambda; tích hợp gốc RDS (MySQL, PostgreSQL, Oracle, SQL Server, MariaDB), Aurora, Redshift, DocumentDB; mã hóa KMS.
- **Dùng khi nào:** mật khẩu DB cần tự xoay vòng; **multi-Region secrets** replicate sang Region khác, promote khi DR.
- **Vì sao quan trọng:** "rotate DB credentials automatically" → Secrets Manager. ❓ Slide ghi "Secrets Manager xoay vòng IAM access key": làm được bằng Lambda rotation tùy chỉnh, nhưng không phải tích hợp sẵn; cần kiểm chứng thêm trước khi chọn làm đáp án.

### AWS Certificate Manager (ACM)
- **Là gì:** cấp và quản lý cert TLS public (miễn phí) và private; tích hợp ELB, CloudFront, API Gateway (**không** cài trực tiếp lên EC2 được); xác thực domain bằng **DNS** (khuyến nghị, tự gia hạn) hoặc **email**.
- **Dùng khi nào:** HTTPS cho ALB/CloudFront; cert import từ CA ngoài (không tự gia hạn, theo dõi bằng EventBridge **ACM Certificate Approaching Expiration** từ 45 ngày trước, hoặc Config rule `acm-certificate-expiration-check`).
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "ACM tự gia hạn 60 ngày trước" → cert hiệu lực 198 ngày gia hạn trước **45 ngày** (cert 395 ngày cũ mới là 60 ngày). API Gateway edge-optimized cần cert ở **us-east-1**.

### AWS CloudHSM
- **Là gì:** phần cứng HSM single-tenant do AWS cung cấp nhưng **khách tự quản key** (AWS không truy cập được); hỗ trợ key symmetric và asymmetric; cluster nhiều AZ.
- **Dùng khi nào:** yêu cầu FIPS mức cao, key do khách độc quyền, SSL offload, Oracle TDE; làm **custom key store** cho KMS.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "CloudHSM = FIPS 140-2 Level 3" → loại **hsm2m.medium** đạt **FIPS 140-3**; hsm1.medium cũ là FIPS 140-2 Level 3.

### AWS WAF
- **Là gì:** firewall Layer 7 chống khai thác web; **Web ACL** với rule: IP set, header/body/URI, **SQL injection**, **XSS**, giới hạn kích thước, **geo match**, **rate-based rule** (chống flood); managed rule groups.
- **Dùng khi nào:** gắn ALB, API Gateway, CloudFront, AppSync, Cognito user pool, App Runner, Verified Access, Amplify, AgentCore Gateway.
- **Vì sao quan trọng:** **không gắn NLB**; cần IP cố định + WAF → **Global Accelerator** (IP tĩnh) trước **ALB** có WAF. Web ACL là regional, trừ CloudFront (global).

### AWS Shield
- **Là gì:** **Shield Standard** miễn phí cho mọi khách, chống SYN/UDP flood, reflection (Layer 3/4); **Shield Advanced** (~$3,000/tháng/Organization) bảo vệ EC2, ELB, CloudFront, Global Accelerator, Route 53, có **SRT** 24/7, bảo vệ chi phí khi bị DDoS, tự tạo rule WAF giảm thiểu Layer 7.
- **Dùng khi nào:** ứng dụng quan trọng cần đội phản ứng DDoS và hoàn phí scale do tấn công.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "DDoS Response Team (DRT)" → nay **Shield Response Team (SRT)**.

### AWS Firewall Manager
- **Là gì:** quản lý tập trung rule bảo mật cho mọi account trong Organization: WAF, Shield Advanced, Security Group, Network Firewall, Route 53 Resolver DNS Firewall.
- **Dùng khi nào:** áp chính sách bảo mật theo Region cho tài nguyên hiện có và tài nguyên tạo mới.
- **Vì sao quan trọng:** WAF = rule cho từng Web ACL; Firewall Manager = áp WAF rule hàng loạt cho nhiều account.

### GuardDuty, Inspector, Macie
- **Là gì:** **GuardDuty** phân tích bằng ML/threat intel trên CloudTrail, VPC Flow Logs, DNS logs và các nguồn tùy chọn (S3 data events, EKS, RDS login, Lambda network, EBS malware) để phát hiện mối đe dọa (cả **tấn công đào tiền mã hóa**); **Inspector** quét lỗ hổng phần mềm & mạng cho EC2 (qua SSM agent), image ECR, Lambda; **Macie** dùng ML tìm **PII/dữ liệu nhạy cảm** trong S3.
- **Dùng khi nào:** GuardDuty finding → EventBridge → Lambda/SNS để phản ứng tự động.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: tên **Security Hub** bản cũ → nay **Security Hub CSPM** (tổng hợp finding và kiểm tra tư thế bảo mật).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Customer managed key | Key KMS tự tạo, tự kiểm soát policy/rotation | "control key rotation", "audit" |
| AWS managed key | Key KMS AWS tạo cho từng dịch vụ | "aws/s3", "default" |
| Key policy | Resource policy của KMS key | "cross-account key access" |
| Envelope encryption | Mã hóa bằng data key | "data larger than 4 KB" |
| Multi-Region key | Cùng key material nhiều Region | "encrypt in one Region, decrypt in another" |
| On-demand rotation | Rotate key ngay khi cần | "rotate imported key material" |
| Parameter Store | Cấu hình/secret có version | "store configuration securely" |
| Secrets Manager | Secret với rotation tự động | "rotate database password every 30 days" |
| ACM | Cert TLS public miễn phí | "HTTPS on ALB/CloudFront" |
| CloudHSM | HSM single-tenant, khách giữ key | "FIPS", "full control of keys" |
| Web ACL | Bộ rule WAF | "block SQL injection" |
| Rate-based rule | Giới hạn request theo IP | "HTTP flood" |
| Shield Advanced / SRT | Chống DDoS nâng cao + đội hỗ trợ | "24/7 DDoS response team" |
| Firewall Manager | Quản rule tập trung đa account | "enforce WAF across organization" |
| GuardDuty | Phát hiện mối đe dọa từ log | "crypto mining", "malicious IP" |
| Inspector | Quét lỗ hổng | "CVE scanning on EC2/ECR/Lambda" |
| Macie | Tìm PII trong S3 | "discover sensitive data in S3" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| KMS | CloudHSM | KMS multi-tenant, AWS quản HSM; CloudHSM single-tenant, khách quản key | "Full control, FIPS 140-3" → CloudHSM |
| AWS managed key | Customer managed key | AWS managed không chỉnh policy/rotation; customer managed chỉnh được | Audit + kiểm soát → customer managed |
| Parameter Store | Secrets Manager | Parameter Store rẻ, không tự rotation; Secrets Manager rotation tự động | "Auto rotation" → Secrets Manager |
| ACM public cert | Imported cert | ACM tự gia hạn; imported phải tự gia hạn | Theo dõi hết hạn → EventBridge/Config |
| WAF | Shield | WAF chặn Layer 7 theo rule; Shield chống DDoS Layer 3/4 (+7 với Advanced) | SQLi/XSS → WAF |
| WAF | Security Group/NACL | WAF đọc nội dung HTTP; SG/NACL chỉ IP/port | |
| WAF | Firewall Manager | WAF rule; Firewall Manager áp rule cho nhiều account | "Across all accounts" → Firewall Manager |
| GuardDuty | Inspector | GuardDuty phát hiện hành vi đáng ngờ; Inspector tìm lỗ hổng phần mềm | CVE → Inspector |
| Macie | GuardDuty | Macie tìm dữ liệu nhạy cảm; GuardDuty tìm mối đe dọa | PII → Macie |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "audit every use of encryption key" | KMS customer managed key + CloudTrail | |
| "rotate key every 90 days automatically" | KMS rotation period tùy chỉnh | |
| "copy encrypted snapshot to another Region" | Copy + re-encrypt bằng key Region đích | |
| "same key in multiple Regions for client-side encryption" | KMS multi-Region keys | |
| "auto-rotate RDS password" | Secrets Manager | |
| "store config values, cheap, versioned" | SSM Parameter Store | |
| "free TLS cert for ALB" | ACM | |
| "alert before imported cert expires" | EventBridge ACM expiration event / Config rule | |
| "dedicated HSM, FIPS 140-3" | CloudHSM (hsm2m.medium) | |
| "block SQL injection and XSS" | AWS WAF | |
| "limit requests per IP" | WAF rate-based rule | |
| "fixed IP + WAF" | Global Accelerator + ALB + WAF | NLB không có WAF |
| "24/7 DDoS experts, cost protection" | Shield Advanced (SRT) | |
| "enforce WAF rules across all accounts" | Firewall Manager | |
| "detect crypto mining on EC2" | GuardDuty | |
| "scan ECR images for CVEs" | Inspector | |
| "find credit card numbers in S3" | Macie | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Compliance yêu cầu key mã hóa S3 rotate mỗi 180 ngày và mọi lần dùng key phải được audit. Chọn gì?
- A. SSE-S3
- B. SSE-KMS với customer managed key, rotation period 180 ngày
- C. AWS managed key `aws/s3`
- D. SSE-C

<details><summary>Đáp án</summary>

**B.**
- A sai: không audit theo key, không chỉnh rotation.
- C sai: rotation cố định 1 năm, không chỉnh được.
- D sai: khách tự quản key, AWS không audit việc dùng key.
</details>

**Câu 2.** Mật khẩu RDS phải tự đổi mỗi 30 ngày, ứng dụng luôn lấy mật khẩu mới. Chọn gì?
- A. SSM Parameter Store SecureString
- B. AWS Secrets Manager với rotation
- C. KMS
- D. Lưu trong user data

<details><summary>Đáp án</summary>

**B.**
- A sai: không có rotation tự động.
- C sai: quản lý key, không phải secret.
- D sai: không an toàn.
</details>

**Câu 3.** (Chọn 2) Ứng dụng cần IP cố định để đối tác whitelist và phải được bảo vệ bởi WAF. Chọn gì?
- A. AWS Global Accelerator
- B. ALB gắn Web ACL của WAF
- C. NLB gắn WAF
- D. Elastic IP gắn ALB
- E. CloudFront với IP cố định

<details><summary>Đáp án</summary>

**A, B.** GA cấp IP tĩnh, trỏ tới ALB có WAF.
- C sai: WAF không hỗ trợ NLB.
- D sai: không gắn EIP vào ALB.
- E sai: IP CloudFront thay đổi.
</details>

**Câu 4.** Web API bị một số IP gửi hàng nghìn request/giây. Cách chặn nhanh nhất?
- A. WAF rate-based rule
- B. Security Group deny
- C. NACL allow
- D. Macie

<details><summary>Đáp án</summary>

**A.** Tự chặn IP vượt ngưỡng.
- B sai: SG không có rule deny.
- C sai: phải biết IP trước, không theo tốc độ.
- D sai: tìm PII.
</details>

**Câu 5.** Snapshot EBS mã hóa bằng KMS cần chuyển sang Region khác để DR. Cách làm?
- A. Copy snapshot và mã hóa lại bằng key KMS ở Region đích
- B. Dùng nguyên key cũ vì KMS key là global
- C. Giải mã trước rồi copy
- D. Không thể copy snapshot mã hóa

<details><summary>Đáp án</summary>

**A.**
- B sai: key KMS theo Region (trừ multi-Region key).
- C sai: không cần và không an toàn.
- D sai: copy được.
</details>

**Câu 6.** Cần phát hiện EC2 đang liên lạc với IP độc hại đã biết và đào tiền mã hóa. Chọn gì?
- A. Amazon Inspector
- B. Amazon GuardDuty
- C. Amazon Macie
- D. AWS Config

<details><summary>Đáp án</summary>

**B.**
- A sai: quét lỗ hổng phần mềm.
- C sai: dữ liệu nhạy cảm S3.
- D sai: cấu hình.
</details>

**Câu 7.** Cert TLS mua từ CA ngoài, import vào ACM để dùng cho ALB. Làm sao tránh hết hạn bất ngờ?
- A. ACM tự gia hạn cert import
- B. Dùng EventBridge rule cho sự kiện ACM Certificate Approaching Expiration (hoặc Config rule) → SNS
- C. Dùng Shield
- D. Dùng Macie

<details><summary>Đáp án</summary>

**B.** Sự kiện phát hằng ngày từ 45 ngày trước khi cert import hết hạn.
- A sai: cert import không tự gia hạn.
- C, D sai: không liên quan cert.
</details>

**Câu 8.** Ngân hàng yêu cầu HSM dành riêng, AWS không thể truy cập key, chuẩn FIPS mới nhất. Chọn gì?
- A. KMS AWS managed key
- B. AWS CloudHSM (hsm2m.medium, FIPS 140-3)
- C. Secrets Manager
- D. SSE-S3

<details><summary>Đáp án</summary>

**B.**
- A sai: multi-tenant, AWS quản.
- C sai: lưu secret.
- D sai: key do S3 quản.
</details>

---

### Nguồn đã đối chiếu
- KMS automatic key rotation (rotation period, mặc định 365 ngày): https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-enable.html
- KMS on-demand rotation (gồm key imported material): https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-on-demand.html
- KMS multi-Region keys: https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html
- Secrets Manager rotation: https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html
- ACM renewal for DNS-validated domains (198 ngày → 45 ngày; 395 ngày → 60 ngày): https://docs.aws.amazon.com/acm/latest/userguide/dns-renewal-validation.html
- ACM EventBridge events (Approaching Expiration): https://docs.aws.amazon.com/acm/latest/userguide/supported-events.html
- CloudHSM hardware types (hsm2m.medium FIPS 140-3): https://docs.aws.amazon.com/cloudhsm/latest/userguide/hsm-types.html
- Resources AWS WAF can protect: https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html
- AWS Shield Response Team: https://docs.aws.amazon.com/waf/latest/developerguide/ddos-srt-support.html
- Security Hub CSPM: https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub-v2.html
