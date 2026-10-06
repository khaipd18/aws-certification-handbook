# SOA-C03 — Identity: IAM Advanced, Federation & STS

> Thuộc **Domain 4: Security and Compliance (16%)**: permission boundary, công cụ audit IAM, Access Analyzer, federation, STS và truy cập cross-account.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Permission boundary** đặt **trần** quyền, không cấp quyền: quyền thực tế = identity policy ∩ boundary. Chỉ gắn cho **user và role, không gắn group**.
- Hai công cụ audit:
  - **Credentials Report**: cấp account, file CSV, tạo lại tối đa 4 giờ một lần.
  - **Access Advisor** (last accessed): cấp user/role, service nào dùng lần cuối khi nào.
- **IAM Access Analyzer** tìm tài nguyên được chia sẻ **ra ngoài zone of trust** (account hoặc Organization). Có cả finding unused access và kiểm tra policy.
- **Federation** không cần tạo IAM user:
  - **SAML 2.0** cho doanh nghiệp (AD/ADFS).
  - **Custom identity broker** chỉ khi IdP không hỗ trợ SAML.
  - **Cognito Identity Pools** cho ứng dụng mobile/web.
- **STS** cấp credential tạm: `AssumeRole`, `AssumeRoleWithSAML`, `AssumeRoleWithWebIdentity`, `GetSessionToken` (MFA).
- ⚠️ Lỗi thời: "STS token tối đa 1 giờ" →
  - Role session tới **12 giờ** (theo max session duration của role).
  - **Role chaining** tối đa 1 giờ.
  - `GetSessionToken` của IAM user tới 36 giờ.
- Cross-account: role ở account đích có **trust policy** cho phép principal nguồn **và** principal nguồn có quyền `sts:AssumeRole`.
- **Policy Simulator** thử identity policy, resource policy, permission boundary, SCP trước khi áp.

---

## 2. Core Concepts

### Permission boundaries
- **Là gì:** managed policy làm giới hạn tối đa cho một user hoặc role; bản thân boundary không cấp quyền.
- **Dùng khi nào:** cho người không phải admin tạo user/role mà không vượt boundary; dev tự quản quyền mà không tự nâng lên admin.
- **Vì sao quan trọng:** **không gắn cho group**. Giới hạn một principal → boundary; giới hạn cả account/OU → SCP.

### Credentials Report & Access Advisor
- **Là gì:** Credentials Report là CSV mọi user với trạng thái password, access key, MFA, lần dùng cuối; Access Advisor cho biết service nào principal được cấp và lần cuối truy cập.
- **Dùng khi nào:** tìm key cũ, user không bật MFA → Credentials Report; cắt bớt quyền không dùng → Access Advisor.
- **Vì sao quan trọng:** Credentials Report tạo lại tối đa **mỗi 4 giờ**, gọi lại sớm hơn sẽ nhận bản cũ.

### IAM Access Analyzer
- **Là gì:** phân tích resource-based policy để tìm truy cập từ ngoài zone of trust; hỗ trợ S3 (cả directory bucket), IAM role, KMS key, Lambda, SQS, Secrets Manager, SNS, EBS snapshot, RDS snapshot, ECR, EFS, DynamoDB table/stream.
- **Dùng khi nào:** phát hiện bucket/snapshot bị share cho account lạ hoặc public.
- **Vì sao quan trọng:** chỉ phát hiện, cần kết hợp Block Public Access, SCP/RCP để ngăn. `[+]` Có thêm **unused access findings**, **policy validation / custom policy checks** và tạo policy từ hoạt động CloudTrail.

### Federation & IAM Identity Center
- **Là gì:** người dùng ngoài AWS được IdP bên thứ ba xác thực rồi assume role tạm; SAML 2.0 gọi `AssumeRoleWithSAML`.
- **Dùng khi nào:** doanh nghiệp có AD/ADFS hoặc IdP SAML/OIDC; vòng đời người dùng quản lý ngoài AWS.
- **Vì sao quan trọng:** ⚠️ Đổi tên: AWS SSO → **IAM Identity Center**, cách khuyến nghị cho truy cập nhiều account bằng permission set. Federation SAML dựng tay từng account là cách cũ.

### Custom identity broker
- **Là gì:** ứng dụng tự viết xác thực người dùng rồi gọi STS thay họ.
- **Dùng khi nào:** **chỉ khi IdP không tương thích SAML 2.0**.
- **Vì sao quan trọng:** broker phải tự quyết định role/policy cho từng người, rủi ro thuộc về bạn.

### Cognito Identity Pools
- **Là gì:** cấp credential AWS tạm cho ứng dụng client sau khi đăng nhập qua IdP (social, OIDC, SAML, user pool) hoặc cho khách ẩn danh.
- **Dùng khi nào:** app mobile ghi thẳng vào S3/DynamoDB với quyền hẹp.
- **Vì sao quan trọng:** `AssumeRoleWithWebIdentity` trực tiếp làm được nhưng AWS khuyên dùng Cognito.

### AWS STS
- **Là gì:** `AssumeRole` (cùng hoặc khác account), `AssumeRoleWithSAML`, `AssumeRoleWithWebIdentity`, `GetSessionToken` (thường để dùng MFA).
- **Dùng khi nào:** mọi truy cập tạm thời thay cho access key dài hạn.
- **Vì sao quan trọng:** role session từ 15 phút tới max session duration của role (1–12 giờ, mặc định 1 giờ); role chaining tối đa 1 giờ. `GetSessionToken`: IAM user 15 phút–36 giờ (mặc định 12 giờ), root tối đa 1 giờ.

### Cross-account access
- **Là gì:** account đích tạo role có quyền cần thiết và trust account nguồn; account nguồn cấp `sts:AssumeRole` cho user/role; user gọi `AssumeRole` và dùng credential trả về.
- **Dùng khi nào:** đội vận hành quản lý nhiều account; tool CI/CD deploy sang account khác.
- **Vì sao quan trọng:** thiếu một trong hai phía (trust policy hoặc identity policy) là AccessDenied. Bên thứ ba → yêu cầu `sts:ExternalId`.

### IAM Policy Simulator
- **Là gì:** thử nghiệm identity policy, resource policy, permission boundary, SCP mà không cần thực thi thật.
- **Dùng khi nào:** debug AccessDenied, kiểm tra policy mới trước khi áp.
- **Vì sao quan trọng:** không đánh giá đường mạng, KMS key policy hay Block Public Access.

### `[+]` Logic đánh giá policy
- **Là gì:** mặc định deny ngầm; **Deny tường minh ở đâu cũng thắng**; còn lại phải được cho phép bởi SCP/RCP, permission boundary, session policy và identity/resource policy.
- **Dùng khi nào:** admin vẫn bị AccessDenied → kiểm tra từng lớp.
- **Vì sao quan trọng:** đây là khung để giải mọi câu "vì sao bị từ chối".

### `[+]` External ID & confused deputy
- **Là gì:** điều kiện `sts:ExternalId` trong trust policy cho bên thứ ba (SaaS, MSP).
- **Dùng khi nào:** cấp quyền cross-account cho vendor phục vụ nhiều khách hàng.
- **Vì sao quan trọng:** chống bên thứ ba bị lợi dụng truy cập nhầm account. Với service principal trong resource policy, dùng `aws:SourceAccount` / `aws:SourceArn`.

### `[+]` ABAC
- **Là gì:** cấp quyền bằng cách so tag của principal với tag tài nguyên (`aws:PrincipalTag`, `aws:ResourceTag`, `aws:RequestTag`, `aws:TagKeys`).
- **Dùng khi nào:** nhiều team/dự án, không muốn viết policy riêng cho từng nhóm; IdP truyền session tag (`sts:TagSession`).
- **Vì sao quan trọng:** ABAC chỉ an toàn khi kiểm soát được ai gắn/sửa tag.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Permission boundary | Trần quyền của user/role | "delegate admin without escalation" |
| Credentials Report | CSV trạng thái credential mọi user | "users without MFA", "old access keys" |
| Access Advisor | Service dùng lần cuối | "remove unused permissions" |
| IAM Access Analyzer | Phát hiện chia sẻ ra ngoài | "resources shared outside org" |
| Zone of trust | Account hoặc Organization | "external access finding" |
| SAML 2.0 federation | Đăng nhập bằng AD/ADFS | "use corporate credentials" |
| IAM Identity Center | Truy cập nhiều account tập trung | "workforce SSO across accounts" |
| Cognito Identity Pool | Credential tạm cho app | "mobile app uploads to S3" |
| `AssumeRole` | Lấy credential của role | "cross-account access" |
| Role chaining | Assume role từ phiên role | "session limited to 1 hour" |
| `GetSessionToken` | Credential tạm cho user, kèm MFA | "require MFA for CLI" |
| `sts:ExternalId` | Chống confused deputy | "third-party vendor access" |
| Policy Simulator | Thử policy | "test policy before applying" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Permission boundary | SCP | Một user/role; cả account/OU | Một principal → boundary |
| Permission boundary | Identity policy | Trần, không cấp; cấp quyền | Boundary không grant |
| Credentials Report | Access Advisor | Credential cả account; quyền thực dùng của một principal | "Unused services" → Advisor |
| Access Analyzer | Access Advisor | Chia sẻ ra ngoài; dịch vụ dùng lần cuối | "External access" → Analyzer |
| SAML federation | Custom broker | IdP hỗ trợ SAML; IdP không hỗ trợ | Không SAML → broker |
| Cognito Identity Pool | Web identity federation trực tiếp | Khuyến nghị; được nhưng không khuyến nghị | App mobile → Cognito |
| `AssumeRole` | `GetSessionToken` | Đổi sang quyền role; giữ quyền user, thêm MFA | MFA cho CLI → GetSessionToken |
| Trust policy | Identity policy | Ai được assume role; caller được gọi `sts:AssumeRole` | Cross-account cần cả hai |
| Max session duration | Role chaining | Tới 12 giờ; tối đa 1 giờ | Chuỗi role → 1 giờ |
| IAM Identity Center | IAM user | SSO tập trung, credential tạm; credential dài hạn | Nhiều account → Identity Center |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "let developers create roles but not exceed permissions" | Permission boundary | |
| "list users without MFA and old keys" | Credentials Report | 4 giờ/lần |
| "remove permissions never used" | Access Advisor / unused access findings | |
| "bucket shared with unknown account" | IAM Access Analyzer | |
| "employees sign in with Active Directory" | SAML 2.0 / IAM Identity Center | |
| "IdP not SAML-compatible" | Custom identity broker | |
| "mobile users upload to their S3 prefix" | Cognito Identity Pools | |
| "role session needs 8 hours" | Tăng max session duration của role | Tối đa 12 giờ |
| "chained role session expires after 1 hour" | Giới hạn role chaining | |
| "third party needs access to our account" | Role + `sts:ExternalId` | |
| "AccessDenied despite admin policy" | Kiểm tra SCP, boundary, session policy, Deny tường minh | |
| "test whether policy allows action" | Policy Simulator | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Team lead được phép tạo IAM role cho ứng dụng nhưng không được tạo role có quyền vượt quá bộ quyền chuẩn. Chọn gì?
- A. Gắn group policy cho team lead
- B. Permission boundary bắt buộc gắn cho role mới (điều kiện `iam:PermissionsBoundary`)
- C. Cấp `AdministratorAccess`
- D. Bật Access Analyzer

<details><summary>Đáp án</summary>

**B.**
- A sai: không giới hạn quyền của role được tạo ra.
- C sai: cho phép leo thang quyền.
- D sai: chỉ phát hiện chia sẻ ra ngoài.
</details>

**Câu 2.** Kiểm toán yêu cầu danh sách mọi IAM user, trạng thái MFA và tuổi access key. Chọn gì?
- A. Access Advisor
- B. Credentials Report
- C. CloudTrail
- D. Policy Simulator

<details><summary>Đáp án</summary>

**B.**
- A sai: theo từng principal, về quyền đã dùng.
- C sai: ghi API call.
- D sai: thử policy.
</details>

**Câu 3.** Phải phát hiện bucket S3, KMS key, snapshot bị chia sẻ cho account ngoài Organization. Chọn gì?
- A. IAM Access Analyzer với zone of trust là Organization
- B. Trusted Advisor
- C. Credentials Report
- D. Macie

<details><summary>Đáp án</summary>

**A.**
- B sai: không phân tích resource policy theo zone of trust.
- C sai: về credential của user.
- D sai: tìm dữ liệu nhạy cảm trong S3.
</details>

**Câu 4.** Kỹ sư assume role A rồi từ đó assume role B; phiên role B luôn hết hạn sau 1 giờ dù role B đặt max session 8 giờ. Vì sao?
- A. Max session duration chưa lưu
- B. Role chaining bị giới hạn 1 giờ
- C. STS chỉ cho 1 giờ
- D. Phải dùng `GetSessionToken`

<details><summary>Đáp án</summary>

**B.**
- A sai: không liên quan.
- C sai: role session tới 12 giờ nếu không chaining.
- D sai: không đổi được giới hạn role chaining.
</details>

**Câu 5.** (Chọn 2) User ở account A cần quản trị tài nguyên ở account B qua role. Hai điều kiện nào cần?
- A. Trust policy của role ở B cho phép principal ở A
- B. Identity policy ở A cho phép `sts:AssumeRole` trên ARN role ở B
- C. Tạo IAM user cho người đó ở B
- D. Bật permission boundary ở B
- E. Bật SCP `FullAWSAccess` ở A

<details><summary>Đáp án</summary>

**A, B.**
- C sai: không cần user dài hạn.
- D sai: không phải điều kiện.
- E sai: SCP mặc định đã cho phép; SCP không cấp quyền.
</details>

**Câu 6.** Nhà cung cấp SaaS cần truy cập account của công ty. Làm sao tránh confused deputy?
- A. Cấp access key của một IAM user
- B. Role cross-account với điều kiện `sts:ExternalId` duy nhất
- C. Public bucket
- D. Dùng root account

<details><summary>Đáp án</summary>

**B.**
- A sai: credential dài hạn, khó thu hồi.
- C, D sai: không an toàn.
</details>

**Câu 7.** Công ty dùng IdP nội bộ không hỗ trợ SAML 2.0, muốn nhân viên dùng console AWS mà không tạo IAM user. Chọn gì?
- A. Cognito User Pool
- B. Custom identity broker gọi STS
- C. IAM user cho từng người
- D. Access Analyzer

<details><summary>Đáp án</summary>

**B.**
- A sai: dành cho người dùng ứng dụng, không phải đăng nhập console doanh nghiệp.
- C sai: trái yêu cầu.
- D sai: không liên quan đăng nhập.
</details>

**Câu 8.** Ứng dụng mobile cần cho người dùng đăng nhập bằng Google rồi upload ảnh vào S3, mỗi người một prefix. Chọn gì?
- A. Nhúng access key vào app
- B. Cognito Identity Pools cấp credential tạm, policy giới hạn theo identity
- C. IAM user cho mỗi người dùng
- D. `GetSessionToken`

<details><summary>Đáp án</summary>

**B.**
- A sai: lộ credential.
- C sai: không scale, trái best practice.
- D sai: cần IAM user gọi.
</details>

---

### Nguồn đã đối chiếu
- STS AssumeRole (15 phút–12 giờ, role chaining 1 giờ): https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html
- STS GetSessionToken (tới 36 giờ, root 1 giờ): https://docs.aws.amazon.com/STS/latest/APIReference/API_GetSessionToken.html
- Credential report (4 giờ): https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_getting-report.html
- Access Analyzer supported resources: https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html
- Permissions boundaries: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html
- Policy evaluation logic: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html
- Confused deputy: https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html
