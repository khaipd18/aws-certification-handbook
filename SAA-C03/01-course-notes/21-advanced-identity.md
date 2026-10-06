# SAA-C03 — Advanced Identity: Organizations, IAM Advanced, Identity Center, Directory Service, Control Tower

> Thuộc **Domain 1: Design Secure Architectures**: quản lý nhiều account, chính sách cấp tổ chức, điều kiện IAM, cross-account, SSO cho workforce, Active Directory và landing zone.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Organizations**: một **management account** + nhiều **member account**, gom theo **OU**; **consolidated billing** (cộng dồn usage để được giảm giá theo bậc, chia sẻ RI/Savings Plans), tạo account bằng API.
- **SCP**: đặt **giới hạn quyền tối đa** cho IAM user/role trong account (không áp cho management account); không tự cấp quyền. **RCP** (resource control policy): giới hạn tối đa phía **tài nguyên** (S3, KMS, STS, SQS, Secrets Manager...).
- IAM conditions hay gặp: `aws:SourceIp`, `aws:RequestedRegion`, `ec2:ResourceTag`, `aws:MultiFactorPresent`, `aws:PrincipalOrgID`, `aws:SecureTransport`.
- Cross-account: **assume role** thì **mất quyền gốc**; **resource-based policy** (S3, SNS, SQS, Lambda...) thì giữ quyền gốc. EventBridge gọi Lambda/SNS/SQS qua resource policy; gọi Kinesis/ECS/Step Functions qua IAM role.
- **Permission boundary**: trần quyền cho user/role; quyền thực = giao của identity policy ∩ boundary ∩ SCP.
- **IAM Identity Center**: một lần đăng nhập cho mọi account trong Organization + ứng dụng SAML; **permission sets**; identity source: built-in, AD, IdP ngoài (Okta, Entra ID).
- **Directory Service**: **AWS Managed Microsoft AD** (AD thật, trust với on-prem), **AD Connector** (proxy về AD on-prem), **Simple AD** (tương thích AD đơn giản, không trust).
- **Control Tower**: dựng **landing zone** đa account theo best practice; ⚠️ "guardrails" → nay gọi là **controls**: preventive (SCP/RCP), detective (Config), proactive (CloudFormation Hooks).

---

## 2. Core Concepts

### AWS Organizations
- **Là gì:** dịch vụ global quản lý nhiều account; management account trả tiền cho tất cả; member account chỉ thuộc một Organization.
- **Dùng khi nào:** tách môi trường (dev/test/prod), theo phòng ban, theo dự án; tag chuẩn cho billing; CloudTrail tổ chức gửi log về S3 trung tâm.
- **Vì sao quan trọng:** **OrganizationAccountAccessRole** cho management account quản trị member account; chia sẻ RI/SP giữa account (tắt được).

### Service Control Policies (SCP)
- **Là gì:** policy gắn ở **OU** hoặc **account**, giới hạn quyền tối đa; không áp cho management account; mặc định có `FullAWSAccess`.
- **Dùng khi nào:** cấm dùng Region không cho phép, cấm tắt CloudTrail, chặn dịch vụ không dùng.
- **Vì sao quan trọng:** SCP **không cấp quyền**; vẫn cần Allow trong IAM. Kế thừa từ root → OU → account: Deny ở cấp cha chặn mọi cấp con.

### `[+]` Resource Control Policies (RCP)
- **Là gì:** policy của Organizations giới hạn quyền tối đa trên **tài nguyên** trong account, kể cả với principal bên ngoài tổ chức.
- **Dùng khi nào:** xây **data perimeter**: chặn mọi principal ngoài Organization truy cập S3/KMS/SQS/Secrets Manager dù resource policy cho phép.
- **Vì sao quan trọng:** SCP giới hạn **principal** của mình; RCP giới hạn **tài nguyên** của mình.

### IAM conditions
- **Là gì:** điều kiện trong policy: `aws:SourceIp` (IP client), `aws:RequestedRegion` (Region gọi API), `ec2:ResourceTag/...` và `aws:PrincipalTag/...` (ABAC), `aws:MultiFactorPresent`, `aws:PrincipalOrgID`.
- **Dùng khi nào:** chỉ cho phép thao tác từ IP văn phòng; chỉ cho phép một số Region; bắt MFA trước khi stop instance.
- **Vì sao quan trọng:** S3: `s3:ListBucket` áp lên ARN **bucket**, `s3:GetObject/PutObject/DeleteObject` áp lên ARN **object** (`bucket/*`).

### `aws:PrincipalOrgID` & resource policies
- **Là gì:** điều kiện trong resource policy cho phép mọi principal thuộc một Organization.
- **Dùng khi nào:** bucket dùng chung cho mọi account trong tổ chức mà không liệt kê từng account.
- **Vì sao quan trọng:** đáp án gọn cho "chỉ account trong Organization được truy cập".

### IAM Role vs resource-based policy (cross-account)
- **Là gì:** assume role: tạm bỏ quyền gốc, chỉ có quyền của role; resource-based policy: principal giữ quyền gốc và được thêm quyền trên tài nguyên.
- **Dùng khi nào:** user ở account A cần đọc DynamoDB ở A và ghi S3 ở B cùng lúc → bucket policy ở B (không phải assume role).
- **Vì sao quan trọng:** dịch vụ hỗ trợ resource policy: S3, SNS, SQS, Lambda, KMS, Secrets Manager, ECR...

### Permission boundaries
- **Là gì:** managed policy đặt **trần quyền** cho user/role (không dùng cho group).
- **Dùng khi nào:** cho developer tự tạo role nhưng không vượt quyền được giao; tránh leo thang đặc quyền.
- **Vì sao quan trọng:** quyền thực = identity policy ∩ boundary ∩ SCP (và session policy nếu có).

### IAM policy evaluation
- **Là gì:** thứ tự: explicit Deny → SCP/RCP → resource-based policy → permission boundary → session policy → identity policy.
- **Dùng khi nào:** đọc đề "user có quyền X không?" khi có nhiều lớp policy.
- **Vì sao quan trọng:** thiếu Allow ở lớp giới hạn nào đó → bị từ chối.

### AWS IAM Identity Center
- **Là gì:** SSO cho mọi account trong Organization, ứng dụng SaaS (Salesforce, Box, M365), ứng dụng SAML 2.0, EC2 Windows; identity source: built-in store, AD (qua Directory Service), IdP ngoài.
- **Dùng khi nào:** **permission set** (tập policy) gán cho user/group trên account; **ABAC** theo thuộc tính user.
- **Vì sao quan trọng:** "workforce đăng nhập một lần vào nhiều account" → Identity Center (kế nhiệm AWS SSO). ❓ Slide ghi "password policy đồng nhất cho nhiều account → Identity Center": chưa xác minh được trong docs; IAM password policy vẫn đặt theo từng account, còn mật khẩu của Identity Center do identity source (built-in store hoặc IdP/AD) quyết định.

### AWS Directory Service
- **Là gì:** **AWS Managed Microsoft AD** (AD thật trên AWS, MFA, trust hai chiều với AD on-prem); **AD Connector** (proxy chuyển request về AD on-prem, không lưu user); **Simple AD** (tương thích AD cơ bản, Samba, không nối on-prem).
- **Dùng khi nào:** cần AD độc lập trên AWS có trust → Managed AD; chỉ muốn dùng AD on-prem → AD Connector; nhỏ, không cần on-prem → Simple AD.
- **Vì sao quan trọng:** Identity Center nối on-prem AD qua Managed AD (trust) hoặc AD Connector.

### AWS Control Tower
- **Là gì:** dựng và quản trị môi trường đa account an toàn, tuân thủ; chạy trên Organizations; dashboard tuân thủ; Account Factory tạo account theo chuẩn.
- **Dùng khi nào:** công ty muốn landing zone chuẩn trong vài click thay vì tự ghép Organizations + SCP + Config + CloudTrail.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "guardrails" → **controls**; **preventive** (SCP/RCP, ngăn chặn), **detective** (AWS Config, phát hiện), **proactive** (CloudFormation Hooks, kiểm tra trước khi tạo tài nguyên).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Management account | Account trả tiền, quản Organization | "payer account" |
| OU | Nhóm account | "apply policy to group of accounts" |
| Consolidated billing | Gộp hóa đơn, giảm giá theo bậc | "volume discount across accounts" |
| SCP | Trần quyền cho principal trong account | "prevent any user from ..." |
| RCP | Trần quyền trên tài nguyên trong account | "data perimeter", "block external principals" |
| aws:RequestedRegion | Điều kiện theo Region | "restrict to approved Regions" |
| aws:PrincipalOrgID | Điều kiện theo Organization | "only accounts in our org" |
| ABAC | Phân quyền theo tag | "permissions based on tags" |
| Permission boundary | Trần quyền cho user/role | "delegate role creation safely" |
| IAM Identity Center | SSO đa account | "single sign-on to all accounts" |
| Permission set | Bộ quyền gán qua Identity Center | "assign access per account" |
| Managed Microsoft AD | AD trên AWS, trust on-prem | "AD with trust relationship" |
| AD Connector | Proxy về AD on-prem | "use existing on-prem AD" |
| Simple AD | AD đơn giản, không trust | "small, low-cost directory" |
| Control Tower | Landing zone đa account | "set up multi-account quickly" |
| Controls | Preventive/detective/proactive | "guardrails" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| SCP | IAM policy | SCP giới hạn, không cấp; IAM policy cấp quyền | Cần cả hai |
| SCP | RCP | SCP giới hạn principal; RCP giới hạn tài nguyên | "External principals" → RCP |
| SCP | Permission boundary | SCP cấp account/OU; boundary cấp user/role | Một user cụ thể → boundary |
| Assume role | Resource-based policy | Role bỏ quyền gốc; resource policy giữ quyền gốc | Cần cả hai bên cùng lúc → resource policy |
| Identity Center | IAM user | Identity Center: SSO, credential tạm; IAM user: credential dài hạn từng account | Nhiều account → Identity Center |
| Managed Microsoft AD | AD Connector | Managed AD có user trên AWS + trust; AD Connector chỉ proxy | "Users remain on-prem" → AD Connector |
| AD Connector | Simple AD | AD Connector nối on-prem; Simple AD độc lập | |
| Control Tower | Organizations | Control Tower tự dựng landing zone + controls trên Organizations | "Automated setup best practices" → Control Tower |
| Preventive control | Detective control | Preventive chặn (SCP); detective phát hiện (Config) | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "prevent member accounts from using unapproved Regions" | SCP + aws:RequestedRegion | |
| "prevent disabling CloudTrail in all accounts" | SCP | |
| "S3 buckets must deny principals outside the org" | RCP (hoặc bucket policy + aws:PrincipalOrgID) | |
| "share bucket with all accounts in org" | Bucket policy với aws:PrincipalOrgID | |
| "allow only from corporate IP range" | Condition aws:SourceIp | |
| "developers create roles but can't escalate" | Permission boundary | |
| "single sign-on to all AWS accounts" | IAM Identity Center | |
| "use on-prem AD credentials, no sync" | AD Connector | |
| "AD on AWS with trust to on-prem" | AWS Managed Microsoft AD | |
| "quickly set up governed multi-account environment" | Control Tower | |
| "detect non-compliant accounts" | Control Tower detective controls (Config) | |
| "volume discounts across accounts" | Organizations consolidated billing | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Công ty muốn đảm bảo không account nào trong Organization tạo tài nguyên ngoài `eu-west-1`, kể cả admin. Chọn gì?
- A. IAM policy trong mỗi account
- B. SCP gắn ở root/OU với Deny khi `aws:RequestedRegion` khác `eu-west-1`
- C. AWS Config rule
- D. Permission boundary cho từng user

<details><summary>Đáp án</summary>

**B.** SCP áp cho mọi principal của member account, kể cả admin.
- A sai: admin sửa được policy.
- C sai: chỉ phát hiện.
- D sai: phải gắn từng user, dễ sót.
</details>

**Câu 2.** User ở account A cần đọc DynamoDB ở A và ghi vào S3 ở account B trong cùng một tác vụ. Cách đơn giản nhất?
- A. Assume role ở account B
- B. Bucket policy ở B cho phép principal của A + IAM policy ở A
- C. Tạo IAM user ở B
- D. SCP

<details><summary>Đáp án</summary>

**B.** Resource-based policy giữ quyền gốc nên làm cả hai việc cùng lúc.
- A sai: assume role thì mất quyền DynamoDB ở A.
- C sai: credential dài hạn, thêm quản lý.
- D sai: SCP không cấp quyền.
</details>

**Câu 3.** (Chọn 2) Công ty có AD on-prem, muốn nhân viên đăng nhập một lần vào nhiều account AWS bằng tài khoản AD, không đồng bộ user lên cloud. Chọn gì?
- A. IAM Identity Center
- B. AD Connector
- C. Simple AD
- D. Tạo IAM user cho mỗi nhân viên
- E. Cognito user pool

<details><summary>Đáp án</summary>

**A, B.** Identity Center dùng AD on-prem làm identity source qua AD Connector.
- C sai: không nối on-prem.
- D sai: không phải SSO, khó quản lý.
- E sai: dành cho user ứng dụng.
</details>

**Câu 4.** Team lead muốn developer tự tạo IAM role cho Lambda nhưng không được tạo role mạnh hơn quyền của họ. Chọn gì?
- A. Permission boundary
- B. SCP
- C. Inline policy
- D. IAM group

<details><summary>Đáp án</summary>

**A.** Bắt role mới phải gắn boundary, ngăn leo thang quyền.
- B sai: cấp account, không nhắm user cụ thể.
- C, D sai: không đặt trần quyền.
</details>

**Câu 5.** Bucket S3 cần cho phép mọi account trong Organization đọc, account mới tạo cũng tự có quyền. Chọn gì?
- A. Liệt kê từng account ID trong bucket policy
- B. Bucket policy với điều kiện `aws:PrincipalOrgID`
- C. ACL public read
- D. SCP

<details><summary>Đáp án</summary>

**B.**
- A sai: phải sửa mỗi khi thêm account.
- C sai: public cho cả Internet.
- D sai: không cấp quyền.
</details>

**Câu 6.** Công ty mới muốn dựng môi trường đa account theo best practice (log tập trung, account bảo mật, kiểm tra tuân thủ) nhanh nhất. Chọn gì?
- A. Tự tạo Organizations, SCP, CloudTrail, Config
- B. AWS Control Tower
- C. IAM Identity Center
- D. Trusted Advisor

<details><summary>Đáp án</summary>

**B.**
- A sai: làm được nhưng tốn công.
- C sai: chỉ SSO.
- D sai: khuyến nghị, không dựng môi trường.
</details>

**Câu 7.** Chỉ cho phép user dừng EC2 nếu đăng nhập có MFA. Dùng điều kiện nào?
- A. `aws:SourceIp`
- B. `aws:MultiFactorPresent`
- C. `aws:RequestedRegion`
- D. `aws:PrincipalOrgID`

<details><summary>Đáp án</summary>

**B.**
- A sai: theo IP.
- C sai: theo Region.
- D sai: theo Organization.
</details>

**Câu 8.** Ứng dụng Windows trên EC2 cần AD đầy đủ trên AWS, có trust hai chiều với AD on-prem. Chọn gì?
- A. Simple AD
- B. AD Connector
- C. AWS Managed Microsoft AD
- D. Cognito

<details><summary>Đáp án</summary>

**C.**
- A sai: không trust on-prem.
- B sai: chỉ proxy, không có AD trên AWS.
- D sai: không phải AD.
</details>

---

### Nguồn đã đối chiếu
- Service control policies: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html
- Resource control policies: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html
- IAM policy evaluation logic: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html
- Permissions boundaries: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html
- AWS global condition context keys: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html
- IAM Identity Center: https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html
- AWS Directory Service: https://docs.aws.amazon.com/directoryservice/latest/admin-guide/what_is.html
- Control Tower controls (preventive, detective, proactive): https://docs.aws.amazon.com/controltower/latest/controlreference/controls.html
