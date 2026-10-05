# CLF-C02 — AWS Identity & Access Management (IAM)

> Thuộc **Domain 2: Security and Compliance (30%)**: quản lý danh tính, quyền truy cập, MFA, cách truy cập AWS và các công cụ audit quyền.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **IAM** là dịch vụ **global**, quản lý **ai** (user, group, role) được làm **gì** (policy) trên tài nguyên AWS.
- **Root user** có toàn quyền, chỉ dùng cho vài tác vụ đặc biệt; không dùng hằng ngày, không chia sẻ.
- **Group chỉ chứa user**, không chứa group khác. Một user có thể ở nhiều group hoặc không ở group nào.
- **Policy** là tài liệu JSON gán cho user/group/role; luôn áp dụng **least privilege**.
- **MFA** = mật khẩu bạn biết + thiết bị bạn có. Nay AWS **bắt buộc MFA cho root user** của mọi loại account.
- Truy cập AWS theo 3 cách: **Console** (password + MFA), **CLI** và **SDK** (access keys).
- **IAM Role** cấp quyền tạm thời cho **dịch vụ AWS** (vd: EC2 instance role) thay vì nhúng access key.
- Audit: **Credentials Report** (cấp account) và **Access Advisor** (cấp user).

---

## 2. Core Concepts

### Users & Groups
- **Là gì:** user = một người hoặc ứng dụng trong tổ chức; group = tập user để gán quyền chung.
- **Dùng khi nào:** gán policy cho **group** rồi đưa user vào group, thay vì gán từng user.
- **Vì sao quan trọng:** bẫy hay gặp: group **không lồng** được group khác.

### Root user
- **Là gì:** danh tính tạo ra cùng account, có toàn quyền trên mọi dịch vụ và tài nguyên.
- **Dùng khi nào:** chỉ cho các tác vụ chỉ root làm được (đổi email/mật khẩu root, đóng account standalone, đăng ký bán trên RI Marketplace...). Danh sách đầy đủ ở chương 14.
- **Vì sao quan trọng:** best practice: khóa access key của root, bật MFA, dùng user/role khác cho việc hằng ngày.

### IAM Policies
- **Là gì:** tài liệu JSON định nghĩa quyền; gán được cho user, group (và role). Policy gán trực tiếp vào một user gọi là **inline policy**.
- **Dùng khi nào:** user nhận quyền từ tất cả group nó thuộc về cộng với inline policy của riêng nó.
- **Vì sao quan trọng:** **least privilege**: không cấp quyền nhiều hơn mức user cần.

### Cấu trúc policy
- **Là gì:** `Version` (luôn "2012-10-17"), `Id` (tùy chọn), `Statement` (bắt buộc); mỗi statement gồm `Sid`, `Effect` (Allow/Deny), `Principal`, `Action`, `Resource`, `Condition` (tùy chọn).
- **Dùng khi nào:** đọc hiểu policy: Effect + Action + Resource cho biết được làm gì trên đâu.
- **Vì sao quan trọng:** CLF chỉ hỏi ở mức nhận diện các thành phần, không cần viết policy.

### Password policy
- **Là gì:** đặt độ dài tối thiểu, yêu cầu chữ hoa/thường/số/ký tự đặc biệt, cho user tự đổi mật khẩu, hết hạn mật khẩu, cấm dùng lại mật khẩu cũ.
- **Dùng khi nào:** tăng độ an toàn cho đăng nhập Console.
- **Vì sao quan trọng:** là một trong hai biện pháp bảo vệ đăng nhập (cùng với MFA).

### Multi-Factor Authentication (MFA)
- **Là gì:** thêm yếu tố thứ hai ngoài mật khẩu; lộ mật khẩu thì account vẫn chưa bị chiếm.
- **Dùng khi nào:** bật cho root và mọi IAM user. Mỗi root/IAM user đăng ký được **tối đa 8 thiết bị MFA**.
- **Vì sao quan trọng:** AWS nay **bắt buộc MFA cho root user** của mọi loại account (standalone, management, member).

### MFA device types
- ⚠️ Lỗi thời: tên gọi "U2F Security Key" và "Hardware Key Fob (Gemalto)" → nay AWS chia 3 loại: **Passkeys & security keys** (FIDO, vd: YubiKey, passkey đồng bộ), **Virtual authenticator apps** (TOTP: Google Authenticator, Authy), **Hardware TOTP tokens**.
- **Là gì:** passkey/security key chống phishing tốt nhất; một security key dùng được cho nhiều root/IAM user.
- **Dùng khi nào:** virtual app hỗ trợ nhiều token trên một điện thoại; AWS GovCloud (US) vẫn có thiết bị hardware riêng.

### Cách truy cập AWS: Console, CLI, SDK
- **Là gì:** Console bảo vệ bằng password + MFA; **CLI** (công cụ dòng lệnh, open-source) và **SDK** (thư viện theo ngôn ngữ, nhúng vào app) dùng **access keys**.
- **Dùng khi nào:** CLI để viết script quản trị; SDK để app gọi API AWS (JavaScript, Python, Java, Go, .NET... và SDK cho mobile, IoT).
- **Vì sao quan trọng:** AWS CLI được xây trên SDK cho Python (boto).

### Access Keys
- **Là gì:** cặp **Access Key ID** (~ username) + **Secret Access Key** (~ password), tạo từ Console, user tự quản lý.
- **Dùng khi nào:** truy cập lập trình (CLI/SDK).
- **Vì sao quan trọng:** bí mật như mật khẩu: không chia sẻ, không commit lên Git, xoay vòng định kỳ.

### IAM Roles for Services
- **Là gì:** danh tính có quyền mà dịch vụ AWS đảm nhận để hành động thay bạn; dùng credential tạm thời.
- **Dùng khi nào:** EC2 Instance Role (EC2 đọc S3), Lambda Function Role, role cho CloudFormation.
- **Vì sao quan trọng:** đề hỏi "cách an toàn nhất để EC2 truy cập S3" thì đáp án là **IAM Role**, không phải lưu access key trên instance.

### IAM Security Tools
- **Là gì:** **Credentials Report** (cấp **account**): liệt kê mọi user và trạng thái credential (password, access key, MFA); **Access Advisor** (cấp **user**): quyền được cấp và lần cuối dùng dịch vụ.
- **Dùng khi nào:** Credentials Report để audit toàn account; Access Advisor để cắt bớt quyền không dùng.
- **Vì sao quan trọng:** cặp "account-level vs user-level" là câu hỏi quen thuộc.

### IAM best practices
- **Là gì:** không dùng root; một người = một user; gán quyền qua group; password policy mạnh; bắt buộc MFA; dùng role cho dịch vụ; access key cho CLI/SDK; audit bằng Credentials Report & Access Advisor; không chia sẻ user/key.
- **Dùng khi nào:** gần như mọi câu "best practice for securing AWS account".
- **Vì sao quan trọng:** đáp án đúng thường là cặp **MFA + least privilege**.

### Shared Responsibility cho IAM
- **Là gì:** AWS lo hạ tầng (bảo mật mạng toàn cầu), phân tích cấu hình/lỗ hổng, compliance validation.
- **Dùng khi nào:** khách hàng lo user/group/role/policy, bật MFA, xoay key, dùng công cụ IAM để cấp quyền hợp lý, review quyền.
- **Vì sao quan trọng:** "ai quản lý IAM user và policy" → **khách hàng**.

### `[+]` Temporary credentials & IAM Identity Center
- **Là gì:** AWS khuyến nghị người dùng (con người) dùng **credential tạm thời** qua **IAM Identity Center** thay vì IAM user với access key dài hạn.
- **Dùng khi nào:** công ty có nhiều account hoặc đã có identity provider (AD, Okta...). Chi tiết ở chương 17.
- **Vì sao quan trọng:** đề mới có thể chọn Identity Center là cách quản lý truy cập "best practice" cho nhiều account.

### `[+]` Đánh giá quyền: explicit Deny thắng
- **Là gì:** mặc định mọi thứ bị từ chối (implicit deny); một `Allow` cấp quyền; một `Deny` tường minh **luôn thắng** mọi `Allow`.
- **Dùng khi nào:** khi user nhận nhiều policy mâu thuẫn từ nhiều group.
- **Vì sao quan trọng:** giải thích vì sao SCP hoặc bucket policy có Deny chặn được cả admin (chương 6, 16).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| IAM | Dịch vụ global quản lý danh tính và quyền | "manage users and permissions" |
| Root user | Danh tính toàn quyền tạo cùng account | "account owner", "full access" |
| IAM user | Một người/ứng dụng có credential dài hạn | "one physical person = one user" |
| IAM group | Tập user dùng chung policy; không lồng nhau | "assign permissions to many users" |
| IAM policy | Tài liệu JSON định nghĩa quyền | "JSON document", "permissions" |
| Inline policy | Policy gắn trực tiếp vào một user/role | "attached directly to one user" |
| Least privilege | Chỉ cấp quyền tối thiểu cần thiết | "only the permissions required" |
| Effect / Action / Resource | Allow/Deny; API được phép; tài nguyên áp dụng | "policy elements" |
| Password policy | Quy tắc độ mạnh, hết hạn, cấm dùng lại mật khẩu | "enforce strong passwords" |
| MFA | Yếu tố thứ hai ngoài mật khẩu | "additional layer of security at sign-in" |
| Passkey / security key | MFA dựa trên FIDO, chống phishing | "YubiKey", "phishing-resistant" |
| Virtual authenticator app | App TOTP trên điện thoại | "Google Authenticator", "Authy" |
| Hardware TOTP token | Thiết bị phần cứng sinh mã 6 số | "key fob" |
| Access Key ID / Secret Access Key | Credential cho CLI/SDK | "programmatic access" |
| AWS CLI | Công cụ dòng lệnh gọi API AWS | "command line", "scripts" |
| AWS SDK | Thư viện theo ngôn ngữ, nhúng vào app | "from application code" |
| IAM Role | Danh tính có quyền tạm thời cho dịch vụ/người dùng đảm nhận | "EC2 needs to access S3" |
| Credentials Report | Báo cáo cấp account về credential của mọi user | "list all users and credential status" |
| Access Advisor | Quyền đã cấp và lần cuối dùng, cấp user | "services last accessed" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| IAM User | IAM Role | User có credential **dài hạn** cho một người; Role cấp credential **tạm thời** cho dịch vụ/danh tính đảm nhận | Dịch vụ AWS cần quyền → Role |
| IAM Group | IAM Role | Group gom **user**; Role được **assume** bởi dịch vụ/user | Group không đăng nhập được |
| Credentials Report | Access Advisor | Report: **account-level**, trạng thái credential mọi user; Advisor: **user-level**, dịch vụ đã dùng | "All users" → Report; "last accessed" → Advisor |
| Password policy | MFA | Password policy làm mật khẩu mạnh; MFA thêm yếu tố thứ hai | Lộ mật khẩu vẫn an toàn → MFA |
| Console | CLI / SDK | Console: password + MFA; CLI/SDK: access keys | "Programmatic access" → access keys |
| AWS CLI | AWS SDK | CLI gõ lệnh/script; SDK là thư viện trong code app | "Embedded in application" → SDK |
| Root user | Admin IAM user | Root làm được vài tác vụ riêng; admin user thì không | Đóng account standalone → Root |
| IAM (global) | Hầu hết dịch vụ (regional) | IAM không chọn Region | User tạo một lần dùng mọi Region |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "EC2 application needs to read S3 securely" | IAM Role (instance role) | Không lưu access key trên EC2 |
| "grant same permissions to 50 developers" | IAM Group + policy | |
| "protect account if password is stolen" | MFA | |
| "enforce minimum password length and rotation" | IAM password policy | |
| "list all users and status of their MFA/keys" | IAM Credentials Report | Account-level |
| "find unused permissions of a user" | IAM Access Advisor | User-level |
| "only give the permissions needed" | Least privilege | |
| "access AWS from a script / terminal" | AWS CLI + access keys | |
| "access AWS from Python application code" | AWS SDK | |
| "who manages IAM users and policies" | Customer | Shared Responsibility |
| "phishing-resistant MFA" | Passkey / FIDO security key | |
| "best practice for root user" | Bật MFA, không tạo/khóa access key, không dùng hằng ngày | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một ứng dụng chạy trên EC2 cần ghi file vào S3. Cách an toàn nhất để cấp quyền là gì?
- A. Lưu access key của root trên instance
- B. Tạo IAM user và lưu access key trong file cấu hình
- C. Gắn IAM Role có quyền S3 vào EC2 instance
- D. Mở bucket S3 public

<details><summary>Đáp án</summary>

**C.** Role cấp credential tạm thời, tự xoay vòng, không phải lưu key.
- A sai: không bao giờ dùng key của root.
- B sai: key dài hạn lưu trên máy dễ bị lộ.
- D sai: public bucket là rủi ro rò rỉ dữ liệu.
</details>

**Câu 2.** (Chọn 2) Đâu là best practice của IAM?
- A. Dùng root user cho tác vụ quản trị hằng ngày
- B. Bật MFA cho root và các IAM user
- C. Chia sẻ một IAM user cho cả team để tiện quản lý
- D. Gán quyền qua group theo nguyên tắc least privilege
- E. Nhúng access key vào mã nguồn trên GitHub

<details><summary>Đáp án</summary>

**B, D.**
- A sai: root chỉ dùng cho tác vụ đặc biệt.
- C sai: một người = một user.
- E sai: access key phải được giữ bí mật.
</details>

**Câu 3.** Bộ phận audit cần một báo cáo liệt kê **tất cả** IAM user trong account và trạng thái password, access key, MFA của họ. Dùng công cụ nào?
- A. IAM Access Advisor
- B. IAM Credentials Report
- C. AWS CloudTrail
- D. Amazon Inspector

<details><summary>Đáp án</summary>

**B.** Credentials Report là báo cáo cấp account về credential.
- A sai: Access Advisor ở cấp user, cho biết dịch vụ đã dùng.
- C sai: CloudTrail ghi lịch sử API call.
- D sai: Inspector quét lỗ hổng phần mềm.
</details>

**Câu 4.** Admin muốn biết một developer đã lâu không dùng những dịch vụ nào để thu hồi quyền thừa. Công cụ phù hợp nhất?
- A. IAM Access Advisor
- B. IAM Credentials Report
- C. AWS Trusted Advisor
- D. IAM password policy

<details><summary>Đáp án</summary>

**A.** Access Advisor cho thấy quyền được cấp và lần cuối truy cập từng dịch vụ.
- B sai: chỉ có trạng thái credential, không có lịch sử dùng dịch vụ.
- C sai: Trusted Advisor đưa khuyến nghị best practice ở mức account.
- D sai: chỉ quy định độ mạnh mật khẩu.
</details>

**Câu 5.** Phát biểu nào về IAM group là **đúng**?
- A. Group có thể chứa group khác
- B. Một user chỉ được thuộc một group
- C. Group chỉ chứa user, và một user có thể thuộc nhiều group
- D. Group có thể được EC2 assume như role

<details><summary>Đáp án</summary>

**C.**
- A sai: group không lồng nhau.
- B sai: một user có thể ở nhiều group hoặc không ở group nào.
- D sai: chỉ role mới được assume.
</details>

**Câu 6.** Developer muốn viết script tự động tạo tài nguyên từ terminal của laptop. Họ cần gì?
- A. Mật khẩu Console và MFA
- B. AWS CLI với access key của IAM user của họ
- C. Root user credentials
- D. IAM Credentials Report

<details><summary>Đáp án</summary>

**B.** CLI dùng access key cho truy cập lập trình.
- A sai: dùng cho Console.
- C sai: không dùng root cho việc hằng ngày.
- D sai: là báo cáo audit.
</details>

**Câu 7.** (Chọn 2) Theo Shared Responsibility Model, khách hàng chịu trách nhiệm gì với IAM?
- A. Bảo mật hạ tầng mạng toàn cầu của IAM
- B. Quản lý user, group, role, policy
- C. Compliance validation cho dịch vụ IAM
- D. Bật MFA và xoay vòng access key
- E. Vá lỗi phần mềm chạy dịch vụ IAM

<details><summary>Đáp án</summary>

**B, D.**
- A, C, E sai: thuộc phần AWS (security of the cloud).
</details>

**Câu 8.** Công ty muốn loại MFA chống phishing tốt nhất cho tài khoản admin. Lựa chọn nào phù hợp?
- A. SMS one-time code
- B. Passkey hoặc FIDO security key
- C. Password dài hơn 20 ký tự
- D. Gửi mã qua email

<details><summary>Đáp án</summary>

**B.** Passkey/security key dựa trên FIDO, dùng mật mã khóa công khai, chống phishing.
- A, D sai: không phải loại MFA mà IAM hỗ trợ cho đăng nhập.
- C sai: password dài vẫn chỉ là một yếu tố.
</details>

---

### Nguồn đã đối chiếu
- AWS MFA in IAM (3 loại MFA, tối đa 8 thiết bị, MFA bắt buộc cho root): https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html
- Root user best practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html
- Tasks that require root user credentials: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html
- IAM policy evaluation logic: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html
- IAM Identity Center: https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html
