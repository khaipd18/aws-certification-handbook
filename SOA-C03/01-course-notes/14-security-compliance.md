# SOA-C03 — Security & Compliance: WAF, Shield, GuardDuty, KMS, ACM, Secrets Manager

> Thuộc **Domain 4: Security and Compliance (16%)**: bảo vệ ứng dụng web và DDoS, dịch vụ phát hiện mối đe dọa, mã hóa bằng KMS, chứng chỉ ACM, quản lý secret.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- Chọn theo tầng:
  - **WAF** là Layer 7 (SQL injection, XSS, rate limit), gắn được vào ALB, API Gateway, CloudFront, AppSync, Cognito, App Runner, Verified Access, Amplify.
  - **Shield** chống DDoS Layer 3/4.
  - **WAF không gắn được NLB**.
- **Firewall Manager** áp WAF, Shield Advanced, security group, Network Firewall, DNS Firewall cho cả Organization, kể cả account và tài nguyên mới.
- Dịch vụ phát hiện:
  - **GuardDuty**: phát hiện mối đe dọa từ log.
  - **Inspector**: quét lỗ hổng EC2 / ECR / Lambda.
  - **Macie**: dữ liệu nhạy cảm trong S3.
  - **Security Hub CSPM**: gom finding, cần bật **AWS Config**.
- **KMS rotation**:
  - AWS managed key tự xoay mỗi năm.
  - Customer managed symmetric key xoay tùy chọn, chu kỳ 90–2,560 ngày (mặc định 365), giữ nguyên key ID; xoay on-demand được tối đa 25 lần.
  - ⚠️ Lỗi thời: "imported key chỉ xoay tay" → nay imported symmetric key xoay **on-demand** được (giữ key ID). Asymmetric, HMAC, custom key store: xoay tay bằng key mới + **alias**.
- Không đổi KMS key hay mã hóa EBS volume tại chỗ: phải snapshot → copy với key mới → tạo volume mới.
- **ACM**: certificate public miễn phí, tự gia hạn; **certificate import không tự gia hạn**.
  - ⚠️ "Gia hạn trước 60 ngày" → cert 198 ngày nay gia hạn trước **45 ngày**.
- **Secrets Manager** tự rotate bằng Lambda, bắt buộc mã hóa KMS. **Parameter Store** rẻ hơn, không có rotation sẵn.

---

## 2. Core Concepts

### AWS WAF
- **Là gì:** tường lửa Layer 7; rule theo IP set, header/body/URI, kích thước, geo match, rate-based; action Allow, Block, Count, CAPTCHA, Challenge. Web ACL nay được gọi là **protection pack (web ACL)** trong docs.
- **Dùng khi nào:** chặn SQLi/XSS, chặn IP, giới hạn tần suất request; dùng **AWS Managed Rule Groups** cho OWASP, bad inputs, bot.
- **Vì sao quan trọng:** web ACL theo Region (CloudFront là global); web ACL Regional phải cùng Region với ALB. Thử rule bằng **Count** trước khi Block. `[+]` Tối đa 5,000 WCU mỗi web ACL, vượt 1,500 WCU tính thêm phí.

### WAF với NLB & IP tĩnh
- **Là gì:** NLB là Layer 4 nên không gắn WAF.
- **Dùng khi nào:** cần IP tĩnh **và** WAF → Global Accelerator trỏ tới ALB có WAF.
- **Vì sao quan trọng:** đề hay gài "NLB + WAF" làm đáp án sai.

### AWS Shield
- **Là gì:** **Standard** miễn phí, tự động chống SYN/UDP flood, reflection Layer 3/4. **Advanced** trả phí, bảo vệ EC2, ELB, CloudFront, Global Accelerator, Route 53.
- **Dùng khi nào:** thường xuyên bị tấn công, cần **Shield Response Team (SRT)** 24/7, bảo vệ chi phí khi scale vì DDoS, tự tạo rule WAF giảm thiểu tấn công Layer 7.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "DDoS Response Team (DRT)" trong slide → nay là **Shield Response Team (SRT)**.

### AWS Firewall Manager
- **Là gì:** áp security policy (WAF, Shield Advanced, security group, Network Firewall, Route 53 Resolver DNS Firewall) cho mọi account trong Organization.
- **Dùng khi nào:** chuẩn hóa WAF cho hàng chục account, tự bảo vệ tài nguyên và account mới.
- **Vì sao quan trọng:** policy theo Region; cần Organizations và Config.

### Amazon GuardDuty
- **Là gì:** phát hiện mối đe dọa bằng ML và threat intel, bật một click, không agent; nguồn chính: CloudTrail management event, VPC Flow Logs, DNS logs; tính năng thêm: S3, EKS, Runtime Monitoring, RDS login, Malware Protection, Lambda.
- **Dùng khi nào:** phát hiện đào coin, instance liên lạc với IP độc hại, credential bị dùng bất thường.
- **Vì sao quan trọng:** finding → **EventBridge** → Lambda/SNS để tự động phản ứng.

### Inspector, Macie, Detective
- **Là gì:** **Inspector** quét CVE cho EC2, image ECR, Lambda và network reachability (EC2), có risk score; **Macie** tìm PII trong S3; **Detective** điều tra finding bằng đồ thị từ VPC Flow Logs, CloudTrail, GuardDuty.
- **Dùng khi nào:** tìm lỗ hổng phần mềm → Inspector; tìm dữ liệu nhạy cảm → Macie; tìm nguyên nhân gốc sau finding → Detective.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: Inspector Classic (cần agent, chạy assessment) **tắt hẳn 20/05/2026**; Inspector hiện tại quét liên tục.

### Security Hub CSPM, Audit Manager, Trusted Advisor
- **Là gì:** **Security Hub CSPM** gom finding (GuardDuty, Inspector, Macie, Config, Access Analyzer, Firewall Manager…) và chạy check theo chuẩn (CIS, AWS Foundational); **Audit Manager** thu bằng chứng cho kiểm toán (PCI DSS, HIPAA, SOC 2, GDPR); **Trusted Advisor** check cost, performance, security, fault tolerance, service limits, operational excellence.
- **Dùng khi nào:** một màn hình tuân thủ bảo mật đa account → Security Hub CSPM (qua delegated admin); chuẩn bị audit bên ngoài → Audit Manager.
- **Vì sao quan trọng:** Security Hub cần **Config bật trước**. ⚠️ Đổi tên: Security Hub (CSPM) trong docs hiện hành. Toàn bộ check Trusted Advisor cần gói Support trả phí; gói hiện tại: kiểm tra Support console.

### AWS KMS: loại key & chi phí
- **Là gì:** symmetric AES-256 (dịch vụ AWS dùng loại này); asymmetric RSA/ECC (public key tải về được, dùng ngoài AWS). AWS owned và AWS managed (`aws/<service>`) miễn phí; customer managed và imported tính phí theo tháng + phí API.
- **Dùng khi nào:** cần kiểm soát key policy, rotation, cross-account → customer managed key.
- **Vì sao quan trọng:** ⚠️ Đổi tên: "Customer Master Key (CMK)" → **KMS key**. `[+]` Envelope encryption: KMS chỉ mã hóa trực tiếp dữ liệu nhỏ (tới 4 KB); dữ liệu lớn dùng `GenerateDataKey`; tải lớn có thể bị throttle API.

### KMS rotation & deletion
- **Là gì:** AWS managed key xoay hằng năm; customer managed symmetric xoay tùy chọn (90–2,560 ngày, mặc định 365), giữ key ID; on-demand rotation tối đa 25 lần mỗi key. Xóa key phải chờ 7–30 ngày (mặc định 30).
- **Dùng khi nào:** imported symmetric key → import key material mới rồi on-demand rotate; asymmetric/HMAC/custom key store → tạo key mới, chuyển **alias** sang key mới.
- **Vì sao quan trọng:** key đang chờ xóa không dùng để mã hóa/giải mã được; hủy được trong thời gian chờ. Phát hiện ai còn dùng key: CloudTrail → metric filter → alarm.

### Key policy, cross-account & Multi-Region key
- **Là gì:** key policy bắt buộc phải cho phép thì IAM mới có tác dụng; cross-account phải ghi account kia vào key policy. Multi-Region key có cùng key ID và key material ở nhiều Region (primary + replica, quản lý riêng).
- **Dùng khi nào:** share snapshot/AMI mã hóa → customer managed key + key policy; mã hóa phía client toàn cầu, DynamoDB Global Tables → Multi-Region key.
- **Vì sao quan trọng:** copy snapshot sang Region khác phải re-encrypt bằng key ở Region đích (trừ khi dùng Multi-Region key).

### AWS Certificate Manager (ACM)
- **Là gì:** cấp và quản lý certificate TLS public (miễn phí) và private; gắn vào ELB, CloudFront, API Gateway; validate bằng **DNS** (CNAME, tự gia hạn) hoặc email.
- **Dùng khi nào:** HTTPS cho ALB, CloudFront (certificate ở **us-east-1**), API Gateway edge-optimized (us-east-1) hoặc Regional (cùng Region).
- **Vì sao quan trọng:** certificate import không tự gia hạn → event **ACM Certificate Approaching Expiration** (mặc định 45 ngày với private/imported, 30 ngày với public, chỉnh được) và Config rule `acm-certificate-expiration-check`. `[+]` Nay có **exportable public certificate** để dùng ngoài dịch vụ tích hợp.

### AWS Secrets Manager
- **Là gì:** lưu secret, bắt buộc mã hóa KMS, rotate tự động qua Lambda (có sẵn cho RDS, Aurora, Redshift, DocumentDB), multi-Region secret promote được.
- **Dùng khi nào:** mật khẩu DB cần rotate định kỳ; ứng dụng đa Region.
- **Vì sao quan trọng:** xóa secret có recovery window mặc định 30 ngày. Theo dõi `RotationFailed`, `RotationAbandoned` trong CloudTrail. Rotation lỗi thường do Lambda trong VPC không tới được DB hoặc endpoint Secrets Manager (thiếu NAT/endpoint, SG), hoặc thiếu `kms:Decrypt`.

### `[+]` Network Firewall & DNS Firewall
- **Là gì:** **AWS Network Firewall** kiểm tra traffic stateful ở mức VPC (IPS, lọc domain); **Route 53 Resolver DNS Firewall** chặn truy vấn DNS ra ngoài tới domain độc hại.
- **Dùng khi nào:** lọc egress theo domain, chống exfiltration qua DNS.
- **Vì sao quan trọng:** cả hai quản lý tập trung bằng Firewall Manager; bổ sung cho SG (stateful, instance) và NACL (stateless, subnet).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| WAF rate-based rule | Giới hạn request theo IP | "HTTP flood from few IPs" |
| WAF Count action | Thử rule không chặn | "test rule before blocking" |
| Shield Advanced | DDoS trả phí + SRT | "24/7 DDoS experts", "cost protection" |
| Firewall Manager | Policy bảo mật toàn Organization | "apply WAF to all accounts" |
| GuardDuty | Phát hiện mối đe dọa | "crypto mining", "malicious IP" |
| Inspector | Quét CVE | "software vulnerabilities on EC2/ECR" |
| Macie | Tìm PII trong S3 | "sensitive data in buckets" |
| Security Hub CSPM | Gom finding + chuẩn bảo mật | "single pane of glass" |
| KMS key policy | Ai được dùng key | "cross-account cannot decrypt" |
| Multi-Region key | Cùng key ở nhiều Region | "decrypt in another Region" |
| ACM DNS validation | Xác thực domain bằng CNAME | "automatic renewal" |
| Imported certificate | Cert ngoài ACM | "certificate expired unnoticed" |
| Secrets Manager rotation | Lambda đổi secret định kỳ | "rotate DB password every 30 days" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| WAF | Shield | Layer 7; Layer 3/4 DDoS | SQLi/XSS → WAF |
| Shield Standard | Shield Advanced | Miễn phí tự động; trả phí, SRT, bảo vệ chi phí | "DDoS experts" → Advanced |
| WAF | Firewall Manager | Một tài nguyên/account; toàn Organization | Many accounts → FM |
| GuardDuty | Inspector | Mối đe dọa từ log; lỗ hổng phần mềm | CVE → Inspector |
| GuardDuty | Detective | Phát hiện; điều tra nguyên nhân | "Investigate" → Detective |
| Security Hub | Audit Manager | Tư thế bảo mật; bằng chứng kiểm toán | "Prepare for audit" → Audit Manager |
| AWS managed key | Customer managed key | Miễn phí, xoay mỗi năm, không share; tính phí, chỉnh rotation, share được | Cross-account → customer managed |
| Automatic / on-demand rotation | Manual rotation | Giữ key ID; key mới + alias | Asymmetric/HMAC → manual |
| ACM public cert | Imported cert | Tự gia hạn; phải tự gia hạn | Cert hết hạn bất ngờ → imported |
| Secrets Manager | Parameter Store | Rotation sẵn, bắt buộc KMS; rẻ, không rotation sẵn | Rotate → Secrets Manager |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "block SQL injection on ALB" | WAF | |
| "need static IP and WAF" | Global Accelerator → ALB + WAF | NLB không có WAF |
| "rate limit per IP" | WAF rate-based rule | |
| "DDoS cost protection, response team" | Shield Advanced | |
| "enforce WAF across all accounts" | Firewall Manager | |
| "EC2 communicating with known bad IPs" | GuardDuty | |
| "scan ECR images for CVEs" | Inspector | |
| "find credit card numbers in S3" | Macie | |
| "Security Hub cannot enable standards" | Bật AWS Config trước | |
| "share encrypted snapshot cross-account" | Customer managed key + key policy | |
| "rotate asymmetric KMS key" | Key mới + đổi alias | Imported symmetric: on-demand |
| "change KMS key of EBS volume" | Snapshot → copy với key mới → volume mới | |
| "CloudFront custom domain certificate" | ACM ở us-east-1 | |
| "imported certificate about to expire" | ACM expiration event / Config rule | |
| "DB password rotation fails in VPC" | NAT hoặc endpoint Secrets Manager, SG | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng sau NLB bị tấn công SQL injection; khách hàng cần IP tĩnh. Kiến trúc nào?
- A. Gắn WAF vào NLB
- B. Global Accelerator → ALB có WAF
- C. Shield Standard
- D. Security group chặn SQL

<details><summary>Đáp án</summary>

**B.** Global Accelerator cấp IP tĩnh, WAF gắn vào ALB.
- A sai: WAF không hỗ trợ NLB.
- C sai: Layer 3/4, không chặn SQLi.
- D sai: SG không đọc nội dung HTTP.
</details>

**Câu 2.** Công ty có 60 account, cần đảm bảo mọi ALB hiện có và tạo sau này đều có cùng bộ rule WAF. Chọn gì?
- A. Tạo web ACL thủ công ở mỗi account
- B. Firewall Manager WAF policy
- C. Shield Standard
- D. GuardDuty

<details><summary>Đáp án</summary>

**B.**
- A sai: không tự áp cho tài nguyên mới.
- C sai: không quản lý WAF.
- D sai: phát hiện mối đe dọa.
</details>

**Câu 3.** (Chọn 2) Security Hub báo lỗi khi bật chuẩn CIS ở vài account. Hai việc cần kiểm tra?
- A. AWS Config đã bật và đang ghi tài nguyên ở account/Region đó
- B. Account đã được thêm vào delegated administrator của Security Hub
- C. Bật Shield Advanced
- D. Bật Macie
- E. Tạo web ACL

<details><summary>Đáp án</summary>

**A, B.**
- C, D, E sai: không phải điều kiện của Security Hub.
</details>

**Câu 4.** Account B không giải mã được snapshot EBS mà account A đã share. Snapshot mã hóa bằng customer managed key. Thiếu gì?
- A. Key policy của key ở A chưa cho account B dùng
- B. B phải bật EBS encryption by default
- C. Snapshot phải public
- D. Phải dùng AWS managed key

<details><summary>Đáp án</summary>

**A.**
- B sai: không liên quan quyền key nguồn.
- C sai: snapshot mã hóa không public được.
- D sai: AWS managed key không share được.
</details>

**Câu 5.** Ứng dụng ký dữ liệu bằng **asymmetric** KMS key, yêu cầu xoay key mỗi năm mà ứng dụng không phải đổi cấu hình. Cách làm?
- A. Bật automatic rotation
- B. Tạo key asymmetric mới, chuyển alias mà ứng dụng dùng sang key mới
- C. Xóa key cũ ngay
- D. Dùng on-demand rotation

<details><summary>Đáp án</summary>

**B.** Ứng dụng gọi qua alias nên không cần đổi; giữ key cũ để verify chữ ký cũ.
- A, D sai: asymmetric key không hỗ trợ automatic hay on-demand rotation.
- C sai: không verify/giải mã được dữ liệu cũ.
</details>

**Câu 6.** Certificate cho ALB được import từ CA bên ngoài và đã hết hạn mà không ai biết. Cách phòng ngừa?
- A. Bật tự gia hạn ACM cho certificate import
- B. EventBridge rule cho event ACM Certificate Approaching Expiration (và/hoặc Config rule `acm-certificate-expiration-check`) → SNS
- C. Chuyển certificate sang us-east-1
- D. Dùng Shield Advanced

<details><summary>Đáp án</summary>

**B.**
- A sai: ACM không gia hạn certificate import.
- C sai: Region không ảnh hưởng gia hạn.
- D sai: không liên quan certificate.
</details>

**Câu 7.** (Chọn 2) Rotation mật khẩu RDS trong Secrets Manager thất bại; Lambda rotation chạy trong private subnet không có NAT. Hai biện pháp?
- A. Tạo interface VPC endpoint cho Secrets Manager
- B. Cho SG của DB nhận kết nối từ SG của Lambda trên port DB
- C. Chuyển sang Parameter Store
- D. Tắt mã hóa KMS
- E. Tăng memory Lambda

<details><summary>Đáp án</summary>

**A, B.**
- C sai: không có rotation sẵn.
- D sai: Secrets Manager bắt buộc KMS.
- E sai: không giải quyết kết nối.
</details>

**Câu 8.** GuardDuty báo instance đang đào crypto. Cần tự động cô lập instance trong vài giây. Chọn gì?
- A. EventBridge rule bắt finding GuardDuty → Lambda đổi SG của instance sang SG cô lập
- B. Đợi Trusted Advisor
- C. Config rule
- D. Macie

<details><summary>Đáp án</summary>

**A.**
- B sai: không phản ứng theo finding.
- C sai: đánh giá cấu hình, không nhận finding GuardDuty.
- D sai: chỉ cho S3.
</details>

---

### Nguồn đã đối chiếu
- WAF protected resources: https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html
- WAF quotas (5,000 WCU, trên 1,500 tính phí): https://docs.aws.amazon.com/waf/latest/developerguide/limits.html
- KMS automatic rotation (90–2,560 ngày): https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-enable.html
- KMS on-demand rotation: https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-on-demand.html
- KMS key deletion (7–30 ngày): https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html
- ACM managed renewal (45 ngày với cert 198 ngày): https://docs.aws.amazon.com/acm/latest/userguide/dns-renewal-validation.html
- ACM events: https://docs.aws.amazon.com/acm/latest/userguide/supported-events.html
- ACM exportable public certificates: https://docs.aws.amazon.com/acm/latest/userguide/acm-exportable-certificates.html
- Secrets Manager DeleteSecret (recovery window 30 ngày): https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_DeleteSecret.html
- Security Hub CSPM: https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
- Services shut down (Inspector Classic 20/05/2026): https://docs.aws.amazon.com/general/latest/gr/full_shutdown_services.html
