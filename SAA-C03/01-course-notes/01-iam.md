# SAA-C03 — IAM & AWS CLI

> Thuộc **Domain 1: Design Secure Architectures (30%)**: user, group, role, policy, MFA, access key và công cụ audit quyền.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **IAM** là dịch vụ **global**: quản lý **ai** (user, group, role) được làm **gì** (policy) trên tài nguyên nào.
- **Root user** chỉ dùng cho vài tác vụ đặc biệt; không dùng hằng ngày, không tạo access key cho root.
- **Group chỉ chứa user**, không lồng group. Một user có thể ở nhiều group.
- **Policy** là JSON (Effect / Action / Resource / Condition); luôn theo **least privilege**; **explicit Deny luôn thắng**.
- **IAM Role** cấp credential **tạm thời** cho dịch vụ AWS (EC2 instance role, Lambda role) hoặc người dùng khác assume.
- Truy cập: **Console** (password + MFA), **CLI / SDK** (access key), **CloudShell** (CLI chạy sẵn trong console).
- Audit: **Credentials Report** (cấp account) và **Access Advisor / last accessed** (cấp user/role).

---

## 2. Core Concepts

### Users & Groups
- **Là gì:** user = một người hoặc một ứng dụng; group = tập user dùng chung policy.
- **Dùng khi nào:** gán policy cho group rồi đưa user vào, thay vì gán lẻ từng user.
- **Vì sao quan trọng:** group **không lồng nhau** và **không phải principal** trong resource-based policy.

### IAM Policies
- **Là gì:** tài liệu JSON gồm `Version`, `Id` (tùy chọn) và `Statement` (`Sid`, `Effect`, `Principal`, `Action`, `Resource`, `Condition`).
- **Dùng khi nào:** user nhận quyền từ mọi group nó thuộc về cộng với inline policy riêng.
- **Vì sao quan trọng:** đề SAA hay cho một policy rồi hỏi "user làm được gì" → đọc Effect + Action + Resource + Condition.

### Managed vs inline policy
- **Là gì:** **AWS managed** (AWS soạn, tự cập nhật), **customer managed** (tự soạn, dùng lại nhiều nơi, có version), **inline** (gắn chặt vào một user/group/role).
- **Dùng khi nào:** mặc định dùng managed; inline chỉ khi cần quan hệ 1-1 nghiêm ngặt.
- **Vì sao quan trọng:** "tái sử dụng, quản lý tập trung, có version" → customer managed policy.

### Password policy & MFA
- **Là gì:** password policy đặt độ dài, ký tự bắt buộc, hết hạn, cấm dùng lại; MFA = mật khẩu + thiết bị.
- **Dùng khi nào:** bật MFA cho root và mọi user có quyền cao; mỗi user đăng ký tối đa 8 thiết bị MFA.
- **Vì sao quan trọng:** AWS nay **bắt buộc MFA cho root user**.

### MFA device types
- ⚠️ Lỗi thời: "U2F Security Key" và "Hardware Key Fob (Gemalto)" → nay chia 3 loại: **Passkeys & security keys** (FIDO), **Virtual authenticator apps** (TOTP), **Hardware TOTP tokens**.
- **Là gì:** passkey/security key chống phishing tốt nhất; virtual app chứa được nhiều token trên một điện thoại.
- **Dùng khi nào:** "phishing-resistant MFA" → passkey / FIDO security key.

### Access Keys, CLI, SDK, CloudShell
- **Là gì:** Access Key ID + Secret Access Key dùng cho CLI và SDK; **CloudShell** là terminal trên trình duyệt, đã có sẵn CLI và credential của user đang đăng nhập.
- **Dùng khi nào:** CLI để script quản trị; SDK để app gọi API; CloudShell khi không muốn cài gì trên máy.
- **Vì sao quan trọng:** access key bí mật như mật khẩu: không commit lên Git, xoay vòng định kỳ, ưu tiên credential tạm thời.

### IAM Roles for Services
- **Là gì:** danh tính có quyền mà dịch vụ AWS **assume** để hành động thay bạn, nhận credential tạm thời qua STS.
- **Dùng khi nào:** EC2 instance role (qua instance profile), Lambda execution role, role cho CloudFormation.
- **Vì sao quan trọng:** "cách an toàn nhất để EC2 đọc S3" → **IAM Role**, không lưu access key trên instance.

### IAM Security Tools
- **Là gì:** **Credentials Report** liệt kê mọi user và trạng thái password/key/MFA; **Access Advisor** cho biết quyền được cấp và lần cuối dùng từng dịch vụ.
- **Dùng khi nào:** Report để audit toàn account; Advisor để cắt quyền thừa.
- **Vì sao quan trọng:** cặp "account-level vs user-level" xuất hiện rất thường xuyên.

### IAM best practices & Shared Responsibility
- **Là gì:** không dùng root; một người = một user; gán quyền qua group; password policy mạnh; MFA; role cho dịch vụ; audit định kỳ.
- **Dùng khi nào:** khách hàng lo user/group/role/policy, MFA, xoay key, review quyền; AWS lo hạ tầng IAM.
- **Vì sao quan trọng:** "ai quản lý IAM policy" → **khách hàng**.

### `[+]` Policy evaluation logic
- **Là gì:** mặc định **implicit deny**; một Allow cấp quyền; một **explicit Deny** ở bất kỳ policy nào (identity, resource, SCP, RCP, permissions boundary) đều thắng.
- **Dùng khi nào:** user nhận policy mâu thuẫn, hoặc cross-account (cần Allow ở **cả hai phía**).
- **Vì sao quan trọng:** giải thích vì sao admin vẫn bị SCP hay bucket policy chặn (chương 21).

### `[+]` Temporary credentials & IAM Identity Center
- **Là gì:** AWS khuyến nghị con người đăng nhập bằng **credential tạm thời** qua **IAM Identity Center** hoặc federation, thay vì IAM user có access key dài hạn.
- **Dùng khi nào:** nhiều account trong Organizations hoặc đã có IdP (Microsoft Entra ID, Okta, AD).
- **Vì sao quan trọng:** đề mới hay chọn Identity Center cho "workforce access to multiple accounts" (chương 21).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| IAM | Dịch vụ global quản lý danh tính và quyền | "manage users and permissions" |
| Root user | Danh tính toàn quyền tạo cùng account | "account owner", "full access" |
| IAM group | Tập user dùng chung policy; không lồng nhau | "assign permissions to many users" |
| Customer managed policy | Policy tự soạn, dùng lại, có version | "reusable", "central management" |
| Inline policy | Policy gắn chặt vào một principal | "strict one-to-one" |
| Least privilege | Chỉ cấp quyền tối thiểu cần thiết | "only the permissions required" |
| Explicit Deny | Deny tường minh, thắng mọi Allow | "even administrators cannot" |
| IAM Role | Danh tính được assume, credential tạm thời | "EC2 needs to access S3" |
| Instance profile | Vỏ chứa role để gắn vào EC2 | "attach role to instance" |
| STS | Dịch vụ cấp credential tạm thời | "temporary credentials", "AssumeRole" |
| Access keys | Credential dài hạn cho CLI/SDK | "programmatic access" |
| CloudShell | Terminal trên trình duyệt có sẵn CLI | "no local installation" |
| Credentials Report | Báo cáo credential mọi user | "list all users and MFA status" |
| Access Advisor | Dịch vụ đã dùng và lần cuối truy cập | "remove unused permissions" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| IAM User | IAM Role | User: credential **dài hạn**; Role: credential **tạm thời**, được assume | Dịch vụ AWS cần quyền → Role |
| IAM Group | IAM Role | Group gom user, không đăng nhập được; Role được assume | Group ≠ principal |
| Credentials Report | Access Advisor | Report: account-level, trạng thái credential; Advisor: user-level, dịch vụ đã dùng | "All users" vs "last accessed" |
| Managed policy | Inline policy | Managed dùng lại, có version; inline gắn chặt 1 principal | Mặc định managed |
| AWS CLI | AWS SDK | CLI gõ lệnh/script; SDK là thư viện trong code | "From application code" → SDK |
| CloudShell | Cài CLI trên laptop | CloudShell không cần cài, dùng credential console | "Browser-based shell" |
| Implicit deny | Explicit deny | Implicit: không có Allow; Explicit: có Deny, thắng Allow | Deny tường minh không thể ghi đè |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "EC2 application needs to read S3 securely" | IAM Role (instance profile) | Không lưu access key |
| "grant same permissions to 50 developers" | IAM Group + policy | |
| "protect account if password is stolen" | MFA | |
| "phishing-resistant MFA" | Passkey / FIDO security key | |
| "list all users and status of their MFA/keys" | Credentials Report | |
| "find unused permissions" | Access Advisor / last accessed | |
| "reusable policy with version history" | Customer managed policy | |
| "run CLI without installing anything" | AWS CloudShell | |
| "workforce users across many accounts" | IAM Identity Center | Chương 21 |
| "who manages IAM users and policies" | Customer | Shared Responsibility |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng trên EC2 cần ghi object vào một S3 bucket. Cách an toàn nhất?
- A. Lưu access key của root trong file cấu hình
- B. Tạo IAM user, lưu access key trong biến môi trường của instance
- C. Gắn IAM Role có quyền `s3:PutObject` vào instance
- D. Mở bucket public write

<details><summary>Đáp án</summary>

**C.** Role cấp credential tạm thời, tự xoay vòng.
- A sai: không bao giờ dùng key của root.
- B sai: key dài hạn trên máy dễ lộ.
- D sai: public write là lỗ hổng nghiêm trọng.
</details>

**Câu 2.** (Chọn 2) Best practice nào của IAM?
- A. Dùng root cho tác vụ quản trị hằng ngày
- B. Bật MFA cho root và user có quyền cao
- C. Một IAM user dùng chung cho cả team
- D. Gán quyền qua group theo least privilege
- E. Nhúng access key vào mã nguồn

<details><summary>Đáp án</summary>

**B, D.**
- A sai: root chỉ cho tác vụ đặc biệt.
- C sai: một người = một user.
- E sai: access key phải giữ bí mật.
</details>

**Câu 3.** User thuộc group A (Allow `s3:*`) và group B (Deny `s3:DeleteObject`). User xóa object được không?
- A. Được, vì Allow `s3:*` bao gồm DeleteObject
- B. Không, vì explicit Deny thắng mọi Allow
- C. Được, vì group nào gán trước thì thắng
- D. Tùy Region

<details><summary>Đáp án</summary>

**B.** Explicit Deny luôn thắng.
- A sai: Allow không ghi đè được Deny.
- C sai: không có khái niệm thứ tự gán.
- D sai: IAM là global.
</details>

**Câu 4.** Audit cần danh sách mọi IAM user và trạng thái password, access key, MFA. Dùng gì?
- A. IAM Access Advisor
- B. IAM Credentials Report
- C. AWS CloudTrail
- D. Amazon Inspector

<details><summary>Đáp án</summary>

**B.**
- A sai: Advisor ở cấp user, cho biết dịch vụ đã dùng.
- C sai: CloudTrail ghi API call.
- D sai: Inspector quét lỗ hổng.
</details>

**Câu 5.** Team bảo mật muốn một policy dùng chung cho 20 role, sửa một chỗ áp dụng mọi nơi và rollback được. Chọn gì?
- A. Inline policy cho từng role
- B. Customer managed policy
- C. AWS managed policy AdministratorAccess
- D. Bucket policy

<details><summary>Đáp án</summary>

**B.** Dùng lại được và có tối đa 5 version để rollback.
- A sai: inline phải sửa từng role.
- C sai: quá rộng, vi phạm least privilege.
- D sai: chỉ áp cho S3 bucket.
</details>

**Câu 6.** Admin cần chạy vài lệnh CLI nhanh nhưng máy công ty cấm cài phần mềm. Giải pháp?
- A. AWS CloudShell
- B. Cài AWS CLI vào máy cá nhân rồi copy access key
- C. Dùng root user trên Console
- D. AWS Systems Manager Session Manager

<details><summary>Đáp án</summary>

**A.** CloudShell chạy trên trình duyệt, có sẵn CLI và credential của phiên console.
- B sai: lộ key, vi phạm chính sách.
- C sai: không dùng root.
- D sai: Session Manager để vào shell của EC2/managed node, không phải để gọi CLI quản trị account.
</details>

**Câu 7.** (Chọn 2) Theo Shared Responsibility, khách hàng chịu trách nhiệm gì với IAM?
- A. Bảo mật hạ tầng vật lý chạy IAM
- B. Tạo và quản lý user, group, role, policy
- C. Compliance validation của dịch vụ IAM
- D. Bật MFA và xoay vòng access key
- E. Vá phần mềm của IAM

<details><summary>Đáp án</summary>

**B, D.**
- A, C, E sai: thuộc "security of the cloud" của AWS.
</details>

**Câu 8.** Developer cần thu hồi quyền thừa của một role đã chạy 6 tháng. Bước đầu tiên hợp lý nhất?
- A. Xem Access Advisor / last accessed của role
- B. Xóa role và tạo lại
- C. Bật AWS Shield Advanced
- D. Đổi password policy

<details><summary>Đáp án</summary>

**A.** Biết dịch vụ nào không được dùng để cắt bớt.
- B sai: gây gián đoạn, không dựa trên dữ liệu.
- C sai: Shield chống DDoS.
- D sai: không liên quan quyền của role.
</details>

---

### Nguồn đã đối chiếu
- AWS MFA in IAM (3 loại MFA, tối đa 8 thiết bị, MFA bắt buộc cho root): https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html
- IAM policy evaluation logic: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html
- Managed policies and inline policies: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html
- IAM roles for EC2 (instance profile): https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html
- AWS CloudShell: https://docs.aws.amazon.com/cloudshell/latest/userguide/welcome.html
- IAM Identity Center: https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html
