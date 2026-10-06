# SOA-C03 — AWS CloudFormation

> Thuộc **Domain 3: Deployment, Provisioning, and Automation (22%)**: viết template, kiểm soát update/delete, bootstrap EC2, nested stack, StackSets, drift và xử lý stack lỗi.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- CloudFormation là IaC **khai báo**: mô tả tài nguyên, CloudFormation tự sắp thứ tự, tạo, cập nhật, xóa. Xóa stack là xóa mọi tài nguyên nó tạo.
- Template: **Resources là mục bắt buộc duy nhất**; ngoài ra có Parameters, Mappings, Conditions, Outputs, Metadata, Transform.
- Hàm hay gặp: `Ref`, `Fn::GetAtt`, `Fn::FindInMap`, `Fn::ImportValue`, `Fn::Sub`, `Fn::Base64` (UserData), hàm điều kiện.
- Bốn lớp bảo vệ khác nhau: **DeletionPolicy**, **UpdateReplacePolicy**, **Stack Policy**, **Termination Protection**.
- **Service role**: CloudFormation thay người dùng thao tác tài nguyên; người dùng cần `iam:PassRole`. Thiếu capability (`CAPABILITY_IAM`, `CAPABILITY_NAMED_IAM`, `CAPABILITY_AUTO_EXPAND`) → `InsufficientCapabilitiesException`.
- Bootstrap EC2: `cfn-init` áp metadata, `cfn-signal` báo kết quả, `cfn-hup` áp lại khi metadata đổi, **CreationPolicy** để stack chờ tín hiệu.
- **Nested stack** để tái dùng thành phần (update từ root); **cross-stack** (Export + `Fn::ImportValue`) cho stack khác vòng đời; **StackSets** cho nhiều account và Region.
- `[+]` **Change Set** xem trước thay đổi; **drift detection** tìm sửa tay ngoài CloudFormation.

---

## 2. Core Concepts

### Template & cách triển khai
- **Là gì:** file YAML hoặc JSON; triển khai qua console, CLI (`CreateStack`), pipeline; soạn bằng editor hoặc Infrastructure Composer.
- **Dùng khi nào:** tách stack theo lớp (network / app) thay vì một template khổng lồ; dựng và xóa môi trường dev theo giờ.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "template phải upload lên S3" → template nhỏ truyền inline được (tối đa 51,200 byte); lớn hơn thì để ở S3 (tối đa 1 MB). Không sửa template đã deploy tại chỗ, phải gửi version mới.

### Parameters, pseudo parameters, Mappings
- **Là gì:** Parameters là input lúc deploy (`AllowedValues`, `AllowedPattern`, `NoEcho`); kiểu AWS-specific kiểm tra tài nguyên có thật; kiểu SSM lấy giá trị từ Parameter Store. Pseudo parameter: `AWS::AccountId`, `AWS::Region`, `AWS::StackName`, `AWS::NoValue`. Mappings là bảng tra cố định (`Fn::FindInMap`).
- **Dùng khi nào:** giá trị người deploy nhập → Parameters; giá trị biết trước theo Region/môi trường → Mappings; AMI mới nhất → SSM public parameter.
- **Vì sao quan trọng:** giới hạn 200 parameters, 200 mappings, 200 outputs mỗi template. Dùng pseudo parameter thay vì hard-code account/Region để template dùng lại được.

### Outputs, cross-stack, Conditions
- **Là gì:** Output có `Export` thì stack khác đọc bằng `Fn::ImportValue`; Conditions dùng `Fn::Equals`, `Fn::If`, `Fn::And`, `Fn::Or`, `Fn::Not`.
- **Dùng khi nào:** stack network export VPC ID cho nhiều stack app; một template cho cả dev và prod.
- **Vì sao quan trọng:** **không xóa được stack có export đang được import**; tên export là duy nhất trong account + Region.

### Rollback & trạng thái lỗi
- **Là gì:** create lỗi mặc định rollback toàn bộ; tùy chọn `OnFailure`: `ROLLBACK`, `DO_NOTHING`, `DELETE`. Update lỗi tự rollback về trạng thái tốt gần nhất.
- **Dùng khi nào:** cần vào instance lỗi đọc log → `DO_NOTHING` / tắt rollback.
- **Vì sao quan trọng:** `UPDATE_ROLLBACK_FAILED` → sửa tay rồi `ContinueUpdateRollback`. `[+]` `ROLLBACK_COMPLETE` (create đầu tiên lỗi) không update được, phải xóa và tạo lại. Nguyên nhân gốc nằm ở sự kiện `CREATE_FAILED` **đầu tiên**.

### Service role & capabilities
- **Là gì:** service role là IAM role CloudFormation assume để thao tác tài nguyên. Capability là xác nhận rõ ràng, không phải quyền.
- **Dùng khi nào:** least privilege: người dùng chỉ có quyền CloudFormation + `iam:PassRole`, không có quyền EC2/S3 trực tiếp.
- **Vì sao quan trọng:** template tạo IAM có tên → `CAPABILITY_NAMED_IAM`; macro hoặc nested stack → `CAPABILITY_AUTO_EXPAND`.

### DeletionPolicy & UpdateReplacePolicy
- **Là gì:** DeletionPolicy áp khi xóa stack **hoặc** bỏ resource khỏi template; UpdateReplacePolicy áp khi update **thay thế** resource. Giá trị: `Delete` (mặc định), `Retain`, `Snapshot`.
- **Dùng khi nào:** giữ dữ liệu DB khi xóa stack → `Snapshot` hoặc `Retain`; đổi thuộc tính buộc thay DB → `UpdateReplacePolicy: Snapshot`.
- **Vì sao quan trọng:** `Snapshot` chỉ cho resource có snapshot (EBS volume, RDS instance/cluster, ElastiCache, Redshift, Neptune, DocumentDB); snapshot nằm ngoài stack.

### Stack Policy & Termination Protection
- **Là gì:** Stack Policy (JSON) chặn **update** lên resource cụ thể; khi đã đặt thì mọi resource bị bảo vệ trừ khi có `Allow`. Termination Protection chặn **xóa stack**.
- **Dùng khi nào:** không cho update đụng DB production → Stack Policy; chống xóa nhầm cả stack → Termination Protection.
- **Vì sao quan trọng:** cả hai không phải IAM; IAM kiểm soát ai được gọi API. Stack Policy có thể override tạm thời cho một lần update.

### Custom resources & dynamic references
- **Là gì:** custom resource gọi Lambda hoặc SNS qua `ServiceToken` (cùng Region) khi create/update/delete; dynamic reference đọc `ssm`, `ssm-secure`, `secretsmanager` lúc deploy.
- **Dùng khi nào:** dọn S3 bucket trước khi xóa, tài nguyên ngoài AWS; giữ mật khẩu ngoài template.
- **Vì sao quan trọng:** RDS/Aurora có `ManageMasterUserPassword`: RDS tự tạo và rotate secret trong Secrets Manager, gọn hơn dynamic reference.

### Bootstrap EC2: UserData & helper scripts
- **Là gì:** UserData phải qua `Fn::Base64`, log ở `/var/log/cloud-init-output.log`. `AWS::CloudFormation::Init` do `cfn-init` áp theo thứ tự packages → groups → users → sources → files → commands → services, log ở `/var/log/cfn-init.log`.
- **Dùng khi nào:** `cfn-signal` + **CreationPolicy** (Count, Timeout) để stack chờ app sẵn sàng; `cfn-hup` áp lại cấu hình khi metadata đổi, không thay instance.
- **Vì sao quan trọng:** wait condition timeout thường do AMI thiếu helper scripts (cài sẵn trên Amazon Linux AMI), `cfn-signal` không chạy, hoặc instance không có đường ra 443 tới endpoint CloudFormation.

### Nested stacks, cross-stack, DependsOn
- **Là gì:** nested stack (`AWS::CloudFormation::Stack`) đóng gói thành phần tái dùng; `DependsOn` ép thứ tự tạo, tự có khi dùng `Ref`/`GetAtt`.
- **Dùng khi nào:** chuẩn hóa ALB/SG dùng chung → nested; giá trị dùng chung giữa stack khác vòng đời → cross-stack.
- **Vì sao quan trọng:** luôn update nested stack qua **root stack**. Một thao tác nested stack tối đa 2,500 resources.

### StackSets
- **Là gì:** một template triển khai thành stack instance trên nhiều account và Region từ administrator account.
- **Dùng khi nào:** **self-managed** (tự tạo `AWSCloudFormationStackSetAdministrationRole` và `AWSCloudFormationStackSetExecutionRole`); **service-managed** (Organizations tạo role, cần all features, **tự deploy cho account mới**).
- **Vì sao quan trọng:** stack instance lỗi thường do thiếu quyền/trust ở account đích, trùng tên toàn cục (S3 bucket), hoặc chạm quota. Delegated admin cần bật trusted access trước.

### Drift detection & update ASG
- **Là gì:** drift so cấu hình thực tế với template, tìm thay đổi làm tay ngoài CloudFormation. `UpdatePolicy` cho ASG: `AutoScalingRollingUpdate` (thay theo lô trong ASG cũ), `AutoScalingReplacingUpdate` (tạo ASG mới, lỗi thì giữ ASG cũ).
- **Dùng khi nào:** "ai đó sửa SG bằng console" → drift. Rollout an toàn với đường lùi sạch → replacing update.
- **Vì sao quan trọng:** thay đổi qua CloudFormation trực tiếp lên stack (không qua StackSet) không tính là drift của StackSet. Replacing update cần đủ capacity cho hai ASG.

### `[+]` Change Sets, resource import, IaC generator, Hooks
- **Là gì:** Change Set (`CreateChangeSet` / `ExecuteChangeSet`) xem trước add/modify/**Replacement**; resource import đưa tài nguyên có sẵn vào stack không tạo lại; **IaC generator** quét tài nguyên trong account để sinh template; **Hooks** kiểm tra trước khi provision và chặn thay đổi không tuân thủ.
- **Dùng khi nào:** "xem tác động trước khi đụng prod" → Change Set; "đưa hạ tầng làm tay vào IaC không downtime" → import / IaC generator; chặn trước → Hooks (Config chỉ phát hiện sau).
- **Vì sao quan trọng:** quota: 500 resources mỗi stack, 100 Hooks mỗi account mỗi Region. Số stack thực tế của account: kiểm tra Service Quotas.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| `Ref` | Giá trị parameter / physical ID | "physical ID of resource" |
| `Fn::GetAtt` | Thuộc tính của resource | "need the ARN / DNS name" |
| `Fn::ImportValue` | Đọc export của stack khác | "share VPC ID between stacks" |
| `NoEcho` | Ẩn giá trị parameter | "must not be displayed" |
| DeletionPolicy | Hành vi khi xóa stack/resource | "keep data when stack deleted" |
| UpdateReplacePolicy | Hành vi khi update thay resource | "replacement must keep data" |
| Stack Policy | Chặn update resource | "protect production DB from updates" |
| Termination Protection | Chặn xóa stack | "prevent accidental stack deletion" |
| Service role | Role CloudFormation assume | "deploy without resource permissions" |
| CreationPolicy | Chờ tín hiệu `cfn-signal` | "COMPLETE before app ready" |
| `cfn-hup` | Áp lại metadata khi đổi | "update config without replacing" |
| StackSets | Nhiều account + Region | "every account in the organization" |
| Drift detection | So thực tế với template | "manual changes outside CloudFormation" |
| Change Set | Xem trước thay đổi | "preview before executing" |
| `ContinueUpdateRollback` | Tiếp tục rollback bị kẹt | "UPDATE_ROLLBACK_FAILED" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Parameters | Mappings | Input lúc deploy; bảng tra cố định | Biết trước theo Region → Mappings |
| `Ref` | `Fn::GetAtt` | ID / giá trị; thuộc tính cụ thể | ARN, DNS → GetAtt |
| Cross-stack | Nested stack | Chia sẻ giá trị giữa stack khác vòng đời; tái dùng thành phần | VPC ID dùng chung → cross-stack |
| DeletionPolicy | UpdateReplacePolicy | Xóa stack/bỏ resource; update thay resource | "Forces replacement" → UpdateReplacePolicy |
| `Retain` | `Snapshot` | Giữ resource sống; chụp rồi xóa | Resource phải chạy tiếp → Retain |
| Stack Policy | Termination Protection | Chặn update; chặn xóa stack | Update vs delete |
| `cfn-init` | `cfn-hup` | Áp cấu hình lần đầu; theo dõi và áp lại | "Without replacing instance" → cfn-hup |
| CreationPolicy | DependsOn | Chờ tín hiệu app; chỉ thứ tự tạo | "App ready" → CreationPolicy |
| Custom resource | Macro / Transform | Tạo thứ CFN không hỗ trợ; viết lại template | Empty S3 bucket → custom resource |
| Self-managed StackSets | Service-managed | Tự tạo role; Organizations, auto-deploy | "New accounts automatically" → service-managed |
| Drift detection | AWS Config | So với template; so với rule | "Manual changes to stack" → drift |
| `ROLLBACK_COMPLETE` | `UPDATE_ROLLBACK_COMPLETE` | Không update được; update được | ROLLBACK_COMPLETE → xóa, tạo lại |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "Region-specific AMI IDs in one template" | Mappings + `Fn::FindInMap` | Hoặc SSM public parameter |
| "cannot delete network stack" | Stack khác còn import export | |
| "same template to all accounts and Regions" | StackSets | |
| "user deploys stacks without EC2/S3 permissions" | Service role + `iam:PassRole` | |
| "`InsufficientCapabilitiesException`, named IAM role" | `CAPABILITY_NAMED_IAM` | |
| "`DELETE_FAILED` on S3 bucket" | Bucket không rỗng → custom resource dọn bucket hoặc Retain | |
| "`UPDATE_ROLLBACK_FAILED`" | Sửa tay → `ContinueUpdateRollback` | |
| "need to inspect failed instance" | `OnFailure=DO_NOTHING` / tắt rollback | |
| "wait condition did not receive signals" | Helper script, `cfn-signal`, egress 443 | |
| "RDS password not in template, auto-rotated" | `ManageMasterUserPassword` | |
| "see whether update replaces resources" | Change Set | |
| "bring manually created resources under IaC" | Resource import / IaC generator | |
| "block non-compliant resources before creation" | CloudFormation Hooks | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Stack network export VPC ID cho 12 stack ứng dụng. Kỹ sư không xóa được stack network. Vì sao?
- A. Thiếu Termination Protection
- B. Các stack khác còn import giá trị export của nó
- C. Stack Policy chặn xóa
- D. Thiếu `CAPABILITY_IAM`

<details><summary>Đáp án</summary>

**B.** Không xóa được stack có export đang được import.
- A sai: Termination Protection bật mới chặn; ở đây không nhắc.
- C sai: Stack Policy chặn update, không chặn xóa.
- D sai: capability liên quan tạo IAM.
</details>

**Câu 2.** Đổi `AvailabilityZone` của RDS trong template buộc thay DB. Phải giữ lại dữ liệu DB cũ. Cấu hình gì?
- A. `DeletionPolicy: Retain`
- B. `UpdateReplacePolicy: Snapshot`
- C. Stack Policy
- D. Termination Protection

<details><summary>Đáp án</summary>

**B.** UpdateReplacePolicy áp khi update thay thế resource.
- A sai: DeletionPolicy áp khi xóa stack hoặc bỏ resource.
- C sai: chặn update chứ không giữ dữ liệu khi đã cho update.
- D sai: chỉ chặn xóa stack.
</details>

**Câu 3.** Stack báo `CREATE_COMPLETE` trong khi ứng dụng trên EC2 vẫn đang cài. Cách đúng?
- A. Thêm `DependsOn`
- B. Dùng `cfn-signal` kèm `CreationPolicy` có Count và Timeout
- C. Tăng timeout của stack
- D. Dùng `cfn-hup`

<details><summary>Đáp án</summary>

**B.**
- A sai: chỉ ép thứ tự tạo resource.
- C sai: stack vẫn không chờ app.
- D sai: dùng để áp lại thay đổi metadata.
</details>

**Câu 4.** (Chọn 2) Dev cần deploy stack nhưng không được có quyền trực tiếp lên EC2/S3. Cần gì?
- A. Service role có quyền EC2/S3 cho CloudFormation
- B. Quyền `iam:PassRole` cho dev trên role đó
- C. Gắn `AdministratorAccess` cho dev
- D. Stack Policy cho phép mọi update
- E. `CAPABILITY_AUTO_EXPAND`

<details><summary>Đáp án</summary>

**A, B.**
- C sai: trái least privilege.
- D sai: Stack Policy không cấp quyền IAM.
- E sai: dành cho macro/nested stack.
</details>

**Câu 5.** Update stack lỗi, rollback cũng lỗi; stack ở `UPDATE_ROLLBACK_FAILED`. Làm gì?
- A. Xóa stack và tạo lại ngay
- B. Sửa tài nguyên gây lỗi bằng tay rồi gọi `ContinueUpdateRollback`
- C. Tạo Change Set mới
- D. Bật drift detection

<details><summary>Đáp án</summary>

**B.**
- A sai: không cần mất cả stack.
- C sai: stack chưa ở trạng thái update được.
- D sai: chỉ phát hiện lệch.
</details>

**Câu 6.** Cần bật một baseline (Config rule, IAM role) cho mọi account trong Organization, kể cả account tạo sau này. Chọn gì?
- A. Nested stacks
- B. StackSets service-managed với automatic deployment
- C. Cross-stack export
- D. StackSets self-managed

<details><summary>Đáp án</summary>

**B.**
- A, C sai: trong một account/Region.
- D sai: không tự deploy cho account mới.
</details>

**Câu 7.** Ai đó sửa inbound rule của SG (do CloudFormation tạo) bằng console. Cách phát hiện nhanh?
- A. Drift detection trên stack
- B. Change Set
- C. Stack Policy
- D. CloudFormation Hooks

<details><summary>Đáp án</summary>

**A.** Drift so cấu hình thực tế với template.
- B sai: xem trước thay đổi template.
- C sai: chỉ chặn update qua CloudFormation, không chặn console.
- D sai: chạy trước khi CloudFormation provision.
</details>

**Câu 8.** Trước khi update stack production, cần biết thay đổi có làm **thay thế** DB không. Chọn gì?
- A. Drift detection
- B. Change Set
- C. `DeletionPolicy: Snapshot`
- D. Termination Protection

<details><summary>Đáp án</summary>

**B.** Change Set cho thấy `Replacement: True/False` trước khi thực thi.
- A sai: so thực tế với template hiện tại.
- C sai: hành vi khi xóa, không xem trước.
- D sai: chỉ chặn xóa stack.
</details>

---

### Nguồn đã đối chiếu
- CloudFormation quotas (500 resources, 200 parameters/outputs/mappings, 51,200 byte / 1 MB, nested 2,500, Hooks 100): https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html
- Helper scripts (cài sẵn trên Amazon Linux AMI): https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/cfn-helper-scripts-reference.html
- IaC generator: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/generate-IaC.html
- DeletionPolicy: https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-deletionpolicy.html
- UpdateReplacePolicy: https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-updatereplacepolicy.html
- StackSets permission models: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-concepts.html
- Drift detection: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html
- Continue rolling back an update: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-continueupdaterollback.html
