# SOA-C03 — Amazon Machine Image (AMI)

> Thuộc **Domain 3: Deployment, Provisioning, and Automation (22%)** và **Domain 4: Security and Compliance (16%)**: tạo AMI nhất quán, di chuyển instance, share/copy cross-account, Image Builder, kiểm soát AMI trong production.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **AMI** là template launch EC2 (OS, phần mềm, cấu hình, block device mapping). AMI **gắn với Region**; dùng ở Region khác phải `CopyImage`, AMI ID sẽ khác.
- Tạo AMI (`CreateImage`) cũng **tạo EBS snapshot** cho từng volume. Mặc định EC2 **reboot** instance trước khi chụp để đảm bảo file system integrity; bật **No-Reboot** thì không gián đoạn nhưng có thể không nhất quán.
- **AWS Backup** backup EC2 theo kiểu **crash-consistent**, không reboot. Cần AMI có reboot theo lịch → **EventBridge + Lambda + `CreateImage`**, hoặc **DLM AMI policy** chọn reboot = Yes.
- **Chuyển AZ**: tạo AMI → launch ở AZ đích. **Chuyển Region**: thêm bước `CopyImage`.
- **Share AMI** không đổi owner. AMI mã hóa phải dùng **customer managed key** và share cả key; **AMI mã hóa bằng AWS managed key không share được**.
- **Copy AMI được share** → account đích thành owner, độc lập, re-encrypt được bằng key của mình.
- **EC2 Image Builder**: build → test → phân phối AMI theo lịch; bản thân dịch vụ miễn phí.
- Kiểm soát AMI: IAM theo tag (preventive), AWS Config `approved-amis-by-id/tag` (detective), `[+]` **Allowed AMIs** và **AMI Block Public Access** ở mức account.

---

## 2. Core Concepts

### AMI & nguồn AMI
- **Là gì:** template gồm block device mapping (trỏ tới snapshot), launch permission, metadata (kiến trúc, virtualization, ENA/NVMe). Nguồn: public (AWS/vendor), tự build, Marketplace.
- **Dùng khi nào:** golden AMI để boot nhanh, cấu hình đồng nhất trong ASG, rollback bằng cách đổi Launch Template về AMI cũ.
- **Vì sao quan trọng:** đừng hard-code AMI ID; dùng **SSM public parameter** trỏ tới AMI mới nhất. AMI không tự vá, có bản vá thì phải build lại.

### Tạo AMI & No-Reboot
- **Là gì:** `CreateImage` từ instance đang chạy hoặc đã stop; snapshot đầu tiên full, sau đó incremental. No-Reboot mặc định **tắt**, tức EC2 reboot instance trước khi chụp.
- **Dùng khi nào:** cần nhất quán → để mặc định (reboot) hoặc stop trước. Không được downtime → bật No-Reboot, chấp nhận rủi ro buffer OS chưa ghi xuống đĩa.
- **Vì sao quan trọng:** "instance launch từ AMI bị lỗi filesystem" → thủ phạm thường là No-Reboot trên workload ghi nhiều. Windows cần application-consistent không reboot → dùng VSS.

### AWS Backup, DLM và EventBridge + Lambda
- **Là gì:** **AWS Backup** backup tập trung nhiều dịch vụ, với EC2 thì crash-consistent (không reboot), Windows có tùy chọn VSS. **DLM AMI policy** tạo AMI theo lịch, có lựa chọn reboot trước khi tạo, tự deprecate/deregister AMI cũ. **EventBridge + Lambda** gọi `CreateImage` theo ý mình.
- **Dùng khi nào:** backup tập trung, Vault Lock, báo cáo compliance → AWS Backup. AMI theo lịch + retention, không code → DLM. Logic riêng → EventBridge + Lambda.
- **Vì sao quan trọng:** đề nhấn "guarantee file system integrity" + "scheduled" thì AWS Backup không phải đáp án.

### Di chuyển instance sang AZ / Region khác
- **Là gì:** EC2 và EBS gắn với AZ, không có nút "move". Tạo AMI → launch ở subnet thuộc AZ đích; khác Region thì `CopyImage` trước.
- **Dùng khi nào:** vài instance lẻ. Hàng trăm server, cutover kiểm soát → **AWS Application Migration Service (MGN)**.
- **Vì sao quan trọng:** private IP đổi theo subnet; Elastic IP không chuyển được sang Region khác; dữ liệu instance store không đi theo AMI.

### Share AMI cross-account
- **Là gì:** cấp launch permission (`ModifyImageAttribute`) cho account, Organization hoặc OU; owner vẫn là account nguồn, vẫn trả tiền snapshot.
- **Dùng khi nào:** account trung tâm phát hành golden AMI cho các account workload.
- **Vì sao quan trọng:** snapshot mã hóa phải dùng **customer managed key** và cho account đích quyền dùng key (`kms:DescribeKey`, `kms:CreateGrant`, `kms:Decrypt`, `kms:GenerateDataKey*`, `kms:ReEncrypt*`). Không share được AMI mã hóa bằng `aws/ebs`. AMI mã hóa không public được.

### Copy AMI
- **Là gì:** account đích `CopyImage` từ AMI được share → tạo AMI mới (ID mới) do account đích sở hữu.
- **Dùng khi nào:** cần độc lập với account nguồn, cần re-encrypt bằng key riêng, cần đưa sang Region khác.
- **Vì sao quan trọng:** nguồn xóa AMI thì bản copy vẫn chạy. AMI có billing product code (Marketplace, một số Windows/RHEL trả phí) thường không copy được → subscribe ở account đích.

### EC2 Image Builder
- **Là gì:** pipeline gồm image recipe (base image + build component) → builder instance → AMI mới → test instance chạy test component → distribution sang nhiều Region/account.
- **Dùng khi nào:** vá OS định kỳ, hardening theo CIS, chỉ phát hành AMI qua test; chạy theo lịch hoặc khi có package mới.
- **Vì sao quan trọng:** dịch vụ miễn phí, chỉ trả EC2/EBS/snapshot/data transfer. Builder instance điều khiển qua SSM, nên private subnet không có endpoint SSM thì pipeline treo. Có **image lifecycle policy** dọn AMI cũ.

### Governance AMI trong production
- **Là gì:** IAM cho `ec2:RunInstances` chỉ khi AMI có tag duyệt (`ec2:ResourceTag/<key>`); AWS Config `approved-amis-by-id` / `approved-amis-by-tag` phát hiện instance vi phạm, remediation qua SSM Automation.
- **Dùng khi nào:** chặn trước → IAM/SCP; tìm instance đang chạy sai chuẩn → Config.
- **Vì sao quan trọng:** policy theo tag bị lách nếu người dùng tự gắn tag được → khóa luôn `ec2:CreateTags`/`ec2:DeleteTags` trên AMI.

### Vòng đời AMI & chi phí snapshot
- **Là gì:** **Deprecate** (ẩn khỏi tìm kiếm, vẫn launch được bằng ID), **Disable** (chặn launch, bật lại được), **Deregister** (xóa đăng ký).
- **Dùng khi nào:** buộc chuyển sang bản mới → deprecate; chặn tạm, hoàn tác được → disable. `[+]` Public AMI mặc định deprecate sau 2 năm kể từ ngày tạo.
- **Vì sao quan trọng:** `DeregisterImage` **không xóa snapshot** → snapshot mồ côi làm phình hóa đơn. Recycle Bin retention 1–365 ngày cho AMI và snapshot giúp khôi phục xóa nhầm.

### `[+]` Allowed AMIs & AMI Block Public Access
- **Là gì:** **Allowed AMIs** là cài đặt account + Region chỉ cho phép AMI từ nhà cung cấp tin cậy (`amazon`, `aws-marketplace`, account ID…), có trạng thái enabled / disabled / audit-mode. **AMI Block Public Access** chặn public AMI.
- **Dùng khi nào:** chặn AMI không tin cậy cho cả account, không phụ thuộc IAM viết đúng; chống vô tình public AMI nội bộ.
- **Vì sao quan trọng:** Block Public Access bật mặc định cho account mới và account chưa có public AMI. Thử Allowed AMIs bằng audit-mode trước khi bật; dùng SCP để không ai tắt.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| `CreateImage` | Tạo AMI + snapshot từ instance | "create image from instance" |
| No-Reboot | Tạo AMI không reboot | "no downtime", "inconsistent file system" |
| `CopyImage` | Copy AMI sang Region/account | "another Region", "own the AMI" |
| Launch permission | Quyền dùng AMI được share | "share with account / OU" |
| Customer managed key | Key KMS share được cross-account | "encrypted AMI shared, launch fails" |
| EC2 Image Builder | Pipeline build/test/distribute AMI | "patch AMIs weekly, test before release" |
| DLM AMI policy | Tạo/dọn AMI theo lịch, chọn reboot | "automate AMI creation and retention" |
| `approved-amis-by-id` | Config rule kiểm tra AMI | "detect non-compliant instances" |
| Allowed AMIs | Chặn AMI theo nhà cung cấp, cấp account | "only trusted AMI providers" |
| AMI Block Public Access | Chặn public AMI | "prevent AMIs from being made public" |
| Deprecate / Disable | Ẩn / chặn launch AMI | "discourage old AMI", "reversible" |
| Recycle Bin | Khôi phục AMI/snapshot bị xóa | "accidentally deregistered" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| AMI | EBS snapshot | Snapshot là dữ liệu một volume; AMI là template launch gồm snapshot + metadata | Launch giống hệt → AMI |
| Share AMI | Copy AMI | Share: owner vẫn là nguồn; Copy: đích sở hữu, đổi key, sang Region | "Độc lập với nguồn" → Copy |
| No-Reboot bật | No-Reboot tắt | Bật: không downtime, có thể không nhất quán; tắt: có reboot, nhất quán | Integrity → để mặc định |
| AWS Backup | EventBridge + Lambda / DLM | Backup crash-consistent, không reboot; Lambda/DLM có thể reboot | AMI có reboot theo lịch → không phải Backup |
| DLM | EC2 Image Builder | DLM chụp tài nguyên có sẵn; Image Builder build AMI mới từ recipe, có test | "Patch, install, test, distribute" → Image Builder |
| Deprecate | Disable | Deprecate vẫn launch bằng ID; Disable chặn launch | "Reversible block" → Disable |
| IAM theo tag | Allowed AMIs | IAM theo principal, lách được qua tag; Allowed AMIs chặn ở account theo nguồn AMI | "Regardless of IAM" → Allowed AMIs |
| IAM / SCP | AWS Config | Chặn trước khi launch; phát hiện sau khi chạy | "Find running instances" → Config |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "ensure file system integrity when creating AMI" | Stop instance / để reboot mặc định | |
| "create AMI without downtime" | No-Reboot | Rủi ro không nhất quán |
| "Windows application-consistent without reboot" | VSS | |
| "scheduled AMI with reboot" | EventBridge + Lambda `CreateImage` hoặc DLM reboot = Yes | |
| "move instance to another AZ" | AMI → launch ở AZ đích | |
| "move instance to another Region" | AMI → `CopyImage` → launch | |
| "shared encrypted AMI cannot be launched" | Đổi sang customer managed key + share key | |
| "recipient must keep AMI if source deletes it" | Copy AMI | |
| "copy Marketplace AMI fails" | Billing product code → subscribe ở account đích | |
| "snapshot costs rising after deregistering AMIs" | Xóa snapshot mồ côi, DLM / lifecycle policy | |
| "only launch approved AMIs" | IAM `ec2:ResourceTag` + khóa `CreateTags` | |
| "block untrusted AMIs account-wide" | Allowed AMIs + SCP | |
| "patch, test and distribute AMIs to 4 Regions" | EC2 Image Builder | |

---

## 5. Câu hỏi tình huống

**Câu 1.** AMI tạo hằng đêm từ instance ghi dữ liệu liên tục, có bật No-Reboot. Instance launch từ AMI thỉnh thoảng không boot. Nguyên nhân gốc?
- A. AMI tạo ở AZ khác instance đích
- B. No-Reboot khiến buffer OS chưa ghi xuống đĩa, filesystem không nhất quán
- C. AMI chưa được share
- D. Snapshot incremental thiếu dữ liệu

<details><summary>Đáp án</summary>

**B.**
- A sai: AMI cấp Region, dùng được mọi AZ.
- C sai: cùng account, không cần share.
- D sai: snapshot incremental vẫn khôi phục được volume đầy đủ.
</details>

**Câu 2.** (Chọn 2) Dùng AWS Backup cho EC2 nhưng AMI không đảm bảo integrity. Cần AMI theo lịch **có reboot**. Chọn hai thành phần.
- A. Backup plan của AWS Backup với tùy chọn reboot
- B. EventBridge rule theo schedule
- C. Lambda gọi `CreateImage` không bật no-reboot
- D. AWS Config remediation
- E. Scheduled action của Auto Scaling

<details><summary>Đáp án</summary>

**B, C.**
- A sai: AWS Backup với EC2 là crash-consistent, không reboot.
- D sai: Config dùng để đánh giá tuân thủ.
- E sai: scheduled action đổi capacity ASG, không tạo AMI.
</details>

**Câu 3.** Account A share AMI mã hóa bằng **AWS managed key** cho Account B; B không launch được. Cách sửa?
- A. Cấp cho B quyền `kms:Decrypt` trên AWS managed key
- B. A copy AMI, mã hóa lại bằng customer managed key, cho B dùng key rồi share AMI mới
- C. B tự tạo grant trên key của A
- D. Đặt AMI thành public

<details><summary>Đáp án</summary>

**B.** Docs ghi rõ không share được AMI mã hóa bằng AWS managed key mặc định.
- A, C sai: không sửa được key policy của AWS managed key.
- D sai: AMI mã hóa không public được.
</details>

**Câu 4.** Account B copy AMI do A share. Phát biểu nào đúng?
- A. A vẫn là owner, thu hồi lúc nào cũng được
- B. B là owner AMI đích; A xóa AMI gốc thì B không bị ảnh hưởng
- C. AMI copy giữ nguyên AMI ID
- D. B không đổi được KMS key khi copy

<details><summary>Đáp án</summary>

**B.**
- A sai: đó là đặc điểm của share.
- C sai: copy sinh AMI ID mới.
- D sai: copy cho phép re-encrypt bằng key của B.
</details>

**Câu 5.** Account production chỉ được launch từ AMI của account shared-services hoặc của Amazon, và yêu cầu này **không phụ thuộc IAM policy viết đúng hay sai**. Chọn gì?
- A. IAM policy với `ec2:ResourceTag`
- B. Config rule `approved-amis-by-tag`
- C. Allowed AMIs chỉ tin `amazon` và account shared-services, SCP khóa cài đặt
- D. AMI Block Public Access

<details><summary>Đáp án</summary>

**C.**
- A sai: phụ thuộc IAM, có thể lách qua tag.
- B sai: chỉ phát hiện, không chặn.
- D sai: chặn public AMI, không giới hạn nguồn AMI được dùng.
</details>

**Câu 6.** (Chọn 2) Cần pipeline vá OS hằng tuần, cài agent nội bộ, chạy test rồi phân phối AMI sang 4 Region. Hai phát biểu nào đúng?
- A. Image Builder chạy build component và test component trước khi phát hành
- B. Image Builder tính phí theo số AMI tạo ra
- C. Image Builder có distribution settings cho nhiều Region và account
- D. Phải dùng AWS Backup để phân phối AMI đa Region
- E. AWS Config dùng để build AMI

<details><summary>Đáp án</summary>

**A, C.**
- B sai: dịch vụ miễn phí, chỉ trả tài nguyên bên dưới.
- D sai: Backup copy recovery point, không phải pipeline phát hành AMI.
- E sai: Config đánh giá tuân thủ.
</details>

**Câu 7.** Instance ở `us-east-1a` cần chuyển sang `us-east-1b`. Trình tự đúng?
- A. Stop, đổi subnet trong Instance Settings, start
- B. Tạo AMI, launch từ AMI vào subnet ở 1b, cập nhật EIP/target group/DNS, terminate instance cũ
- C. Detach EBS rồi attach sang instance ở 1b
- D. `CopyImage` sang `us-east-1b`

<details><summary>Đáp án</summary>

**B.**
- A sai: không đổi subnet/AZ của instance có sẵn.
- C sai: EBS chỉ attach trong cùng AZ.
- D sai: `CopyImage` dùng giữa Region/account, không theo AZ.
</details>

**Câu 8.** Sau vài tháng build AMI hằng tuần và deregister AMI cũ, chi phí snapshot vẫn tăng. Nguyên nhân và cách xử lý bền vững?
- A. Snapshot incremental luôn lưu bản full
- B. Deregister không xóa snapshot → xóa snapshot mồ côi, dùng DLM hoặc image lifecycle policy để dọn tự động
- C. AMI nằm ở nhiều AZ nên nhân đôi chi phí
- D. Bật AMI Block Public Access để giảm chi phí

<details><summary>Đáp án</summary>

**B.**
- A sai: snapshot sau là incremental.
- C sai: AMI cấp Region, không nhân theo AZ.
- D sai: không liên quan chi phí.
</details>

---

### Nguồn đã đối chiếu
- Share AMI (không share được AMI mã hóa bằng AWS managed key): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sharingamis-explicit.html
- AWS Backup với EBS/EC2 (crash-consistent mặc định): https://docs.aws.amazon.com/aws-backup/latest/devguide/multi-volume-crash-consistent.html
- AWS Backup Windows VSS: https://docs.aws.amazon.com/aws-backup/latest/devguide/windows-backups.html
- DLM AMI policy (tùy chọn reboot): https://docs.aws.amazon.com/ebs/latest/userguide/ami-policy.html
- Deprecate AMI (public AMI 2 năm): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ami-deprecate.html
- Allowed AMIs: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-allowed-amis.html
- AMI Block Public Access: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/block-public-access-to-amis.html
- Recycle Bin: https://docs.aws.amazon.com/ebs/latest/userguide/recycle-bin-concepts.html
