# CLF-C02 — Advanced Identity

> Thuộc **Domain 2: Security and Compliance** (identity & access management). Trọng tâm: chọn đúng dịch vụ danh tính cho nhân viên, cho người dùng app, cho Microsoft AD.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **IAM** = danh tính và quyền **trong một AWS account**, cho người bạn tin tưởng (nhân viên công ty).
- **STS (Security Token Service)** = cấp **credential tạm thời, quyền giới hạn**, có thời hạn; dùng cho identity federation, cross-account role, EC2 role.
- **Cognito** = danh tính cho **người dùng web/mobile app** (có thể hàng triệu), hỗ trợ đăng nhập bằng Facebook, Google... thay vì tạo IAM user.
- **Directory Service** tích hợp **Microsoft Active Directory**: **AWS Managed Microsoft AD** (AD thật trên AWS, trust với on-prem), **AD Connector** (proxy về AD on-prem), **Simple AD** (AD-compatible độc lập).
- **IAM Identity Center** (kế nhiệm AWS Single Sign-On) = **một lần đăng nhập** cho mọi account trong Organizations, ứng dụng SaaS, app SAML 2.0, EC2 Windows.
- **Organizations** quản lý nhiều account (chương 16).

---

## 2. Core Concepts

### AWS STS (Security Token Service)
- **Là gì:** tạo **credential tạm thời, quyền giới hạn** để truy cập tài nguyên AWS; bạn cấu hình thời hạn hết hạn.
- **Dùng khi nào:** **identity federation** (danh tính quản lý ở hệ thống ngoài, nhận token STS để vào AWS); **IAM role cross-account hoặc cùng account**; **IAM role cho EC2**.
- **Vì sao quan trọng:** người dùng/dịch vụ **assume role** → STS trả credential tạm → truy cập tài nguyên. Credential tạm an toàn hơn access key dài hạn.

### Amazon Cognito
- **Là gì:** danh tính cho người dùng **web và mobile app** (có thể hàng triệu); tạo user trong Cognito thay vì IAM user.
- **Dùng khi nào:** app cần đăng ký/đăng nhập, hoặc đăng nhập bằng **social identity provider** (Facebook, Google, Twitter...).
- **Vì sao quan trọng:** "millions of app users", "social login" → Cognito, **không** phải IAM.

### Microsoft Active Directory (AD)
- **Là gì:** có trên Windows Server với AD Domain Services; database các object: user account, computer, printer, file share, security group.
- **Dùng khi nào:** quản lý bảo mật tập trung: tạo account, cấp quyền qua Domain Controller.
- **Vì sao quan trọng:** nhiều doanh nghiệp đã có AD on-prem, cần tích hợp với AWS.

### AWS Directory Service
- **Là gì:** **AWS Managed Microsoft AD**: tạo AD của bạn trên AWS, quản lý user tại đó, hỗ trợ MFA, tạo quan hệ **trust** với AD on-prem.
- **Dùng khi nào:** **AD Connector**: **proxy** chuyển yêu cầu xác thực về AD on-prem, hỗ trợ MFA, **user vẫn quản lý ở on-prem**; **Simple AD**: directory tương thích AD, managed trên AWS, **không kết nối được** với AD on-prem.
- **Vì sao quan trọng:** đề cho tình huống và hỏi loại nào: muốn giữ user on-prem → AD Connector; cần AD thật trên cloud có trust → Managed Microsoft AD; nhỏ, độc lập, rẻ → Simple AD.

### AWS IAM Identity Center
- **Là gì:** kế nhiệm **AWS Single Sign-On**: **một lần đăng nhập** cho mọi **AWS account trong Organizations**, ứng dụng cloud doanh nghiệp (Salesforce, Box, Microsoft 365...), app hỗ trợ **SAML 2.0**, **EC2 Windows**.
- **Dùng khi nào:** identity provider: **identity store có sẵn** trong Identity Center, hoặc bên thứ ba: **Active Directory, OneLogin, Okta**...
- **Vì sao quan trọng:** AWS hiện khuyến nghị con người truy cập AWS qua Identity Center với **credential tạm thời** thay vì IAM user.

### So sánh nhanh các dịch vụ danh tính
- **Là gì:** IAM = người tin tưởng trong một account; Organizations = nhiều account; STS = credential tạm; Cognito = người dùng app.
- **Dùng khi nào:** Directory Service = tích hợp Microsoft AD; IAM Identity Center = SSO cho nhiều account và ứng dụng.
- **Vì sao quan trọng:** đây chính là summary của slide, đề hỏi rất sát.

### `[+]` Cognito User Pools vs Identity Pools
- **Là gì:** **User pool** = thư mục người dùng (đăng ký, đăng nhập, MFA); **Identity pool** = đổi danh tính đã xác thực thành **credential AWS tạm thời** để app gọi trực tiếp S3, DynamoDB.
- **Dùng khi nào:** mobile app cần upload ảnh trực tiếp vào S3 với quyền giới hạn.
- **Vì sao quan trọng:** CLF hiếm khi hỏi sâu; chỉ cần biết Cognito lo cả đăng nhập lẫn cấp quyền AWS cho người dùng app.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| AWS STS | Cấp credential tạm thời, quyền giới hạn | "temporary credentials", "assume role" |
| Identity federation | Dùng danh tính từ hệ thống ngoài để vào AWS | "users managed outside AWS" |
| Cross-account role | Role cho phép account khác assume | "access resources in another account" |
| Amazon Cognito | Danh tính cho người dùng web/mobile app | "app sign-up/sign-in", "social login", "millions of users" |
| Microsoft Active Directory | Thư mục user/computer của Windows | "Windows domain", "AD" |
| AWS Managed Microsoft AD | AD managed trên AWS, trust với on-prem | "run AD in AWS", "trust relationship" |
| AD Connector | Proxy xác thực về AD on-prem | "keep users in on-prem AD" |
| Simple AD | Directory AD-compatible độc lập | "standalone, cannot join on-prem AD" |
| IAM Identity Center | SSO cho nhiều account + app SaaS/SAML | "single sign-on", "one login for all accounts" |
| SAML 2.0 | Chuẩn liên kết danh tính | "SAML-enabled applications" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| IAM | Cognito | Nhân viên/dịch vụ trong account vs người dùng cuối của app | "Customers of the app" → Cognito |
| IAM | IAM Identity Center | Quản lý trong một account vs SSO cho nhiều account + app | "One login for all accounts" → Identity Center |
| STS | IAM user access key | Tạm thời, tự hết hạn vs dài hạn | "Short-term credentials" → STS |
| Managed Microsoft AD | AD Connector | AD đầy đủ trên AWS (user quản lý trên AWS) vs chỉ proxy, user ở on-prem | "No user data in cloud" → AD Connector |
| AD Connector | Simple AD | Kết nối AD on-prem vs độc lập, không join on-prem | |
| IAM Identity Center | Cognito | Workforce (nhân viên) vs customer (người dùng app) | |
| Organizations | IAM Identity Center | Quản lý account và billing vs quản lý đăng nhập vào các account | Thường dùng cùng nhau |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "temporary, limited-privilege credentials" | AWS STS | |
| "mobile app users sign in with Google/Facebook" | Amazon Cognito | |
| "millions of web app users" | Amazon Cognito | |
| "employees sign in once to access all AWS accounts" | IAM Identity Center | |
| "SSO to Salesforce, Microsoft 365 and AWS accounts" | IAM Identity Center | |
| "run Microsoft AD in AWS with trust to on-prem AD" | AWS Managed Microsoft AD | |
| "authenticate AWS apps against existing on-prem AD, keep users on-prem" | AD Connector | |
| "simple standalone AD-compatible directory" | Simple AD | |
| "EC2 needs temporary credentials" | IAM Role (STS bên dưới) | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng mobile cần cho hàng triệu khách hàng đăng ký và đăng nhập bằng tài khoản Google. Chọn gì?
- A. IAM users
- B. Amazon Cognito
- C. AWS IAM Identity Center
- D. AWS Directory Service

<details><summary>Đáp án</summary>

**B.** Cognito quản lý danh tính người dùng app, hỗ trợ social login.
- A sai: IAM dành cho người tin tưởng trong tổ chức, không cho hàng triệu khách hàng.
- C sai: SSO cho nhân viên.
- D sai: tích hợp Microsoft AD.
</details>

**Câu 2.** Nhân viên muốn đăng nhập **một lần** để truy cập 30 AWS account trong Organization và cả Microsoft 365. Chọn gì?
- A. AWS IAM Identity Center
- B. Amazon Cognito
- C. AWS STS trực tiếp
- D. Simple AD

<details><summary>Đáp án</summary>

**A.** Identity Center là SSO cho nhiều account và ứng dụng SaaS/SAML.
- B sai: cho người dùng app.
- C sai: STS là cơ chế cấp credential bên dưới, không phải giải pháp SSO.
- D sai: directory độc lập.
</details>

**Câu 3.** Công ty muốn ứng dụng trên AWS xác thực bằng AD on-premises hiện có, **không** lưu dữ liệu user trên cloud. Chọn gì?
- A. AWS Managed Microsoft AD
- B. AD Connector
- C. Simple AD
- D. Amazon Cognito

<details><summary>Đáp án</summary>

**B.** AD Connector là proxy chuyển yêu cầu về AD on-prem; user vẫn quản lý ở on-prem.
- A sai: tạo AD trên AWS, quản lý user trên cloud.
- C sai: không kết nối được AD on-prem.
- D sai: cho người dùng app.
</details>

**Câu 4.** Một ứng dụng cần credential **tạm thời** để truy cập tài nguyên ở một AWS account khác. Dịch vụ nào cấp credential này?
- A. AWS STS
- B. AWS KMS
- C. AWS Secrets Manager
- D. Amazon Cognito user pool

<details><summary>Đáp án</summary>

**A.** Assume cross-account role → STS cấp credential tạm.
- B sai: quản lý key mã hóa.
- C sai: lưu secret dài hạn.
- D sai: thư mục người dùng app.
</details>

**Câu 5.** (Chọn 2) IAM Identity Center hỗ trợ những identity provider nào?
- A. Identity store có sẵn trong IAM Identity Center
- B. Active Directory hoặc IdP bên thứ ba như Okta
- C. Amazon S3 bucket
- D. AWS Trusted Advisor
- E. Amazon CloudFront

<details><summary>Đáp án</summary>

**A, B.**
- C, D, E sai: không phải identity provider.
</details>

**Câu 6.** Công ty cần chạy Microsoft AD đầy đủ trên AWS, hỗ trợ MFA và thiết lập quan hệ trust với AD on-prem. Chọn gì?
- A. AWS Managed Microsoft AD
- B. AD Connector
- C. Simple AD
- D. IAM groups

<details><summary>Đáp án</summary>

**A.** Managed Microsoft AD là AD thật trên AWS, có trust với on-prem.
- B sai: chỉ là proxy.
- C sai: không join được on-prem.
- D sai: không phải Microsoft AD.
</details>

**Câu 7.** Phát biểu nào **đúng** về IAM so với Cognito?
- A. IAM dành cho người dùng cuối của ứng dụng công khai
- B. IAM dành cho người dùng tin tưởng trong tổ chức; Cognito cho người dùng web/mobile app
- C. Cognito thay thế hoàn toàn IAM
- D. Cả hai đều chỉ dùng cho EC2

<details><summary>Đáp án</summary>

**B.**
- A sai: ngược lại.
- C sai: hai dịch vụ cho hai đối tượng khác nhau.
- D sai: không liên quan riêng EC2.
</details>

**Câu 8.** (Chọn 2) STS thường được dùng trong trường hợp nào?
- A. Identity federation cho user quản lý ở hệ thống ngoài
- B. Cấp credential tạm cho EC2 thông qua IAM role
- C. Lưu trữ mật khẩu database dài hạn
- D. Mã hóa dữ liệu S3
- E. Gửi email cho khách hàng

<details><summary>Đáp án</summary>

**A, B.**
- C sai: Secrets Manager.
- D sai: KMS.
- E sai: SES.
</details>

---

### Nguồn đã đối chiếu
- AWS STS (temporary security credentials): https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html
- Amazon Cognito: https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html
- AWS Directory Service: https://docs.aws.amazon.com/directoryservice/latest/admin-guide/what_is.html
- IAM Identity Center: https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html
- IAM MFA (khuyến nghị dùng Identity Center và credential tạm cho con người): https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html
