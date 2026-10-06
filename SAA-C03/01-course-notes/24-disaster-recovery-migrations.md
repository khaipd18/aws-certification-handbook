# SAA-C03 — Disaster Recovery & Migrations

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 4: Design Cost-Optimized Architectures**: RPO/RTO, chiến lược DR, Elastic Disaster Recovery, DMS, AWS Backup, công cụ migrate, VMware, chuyển dữ liệu lớn.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **RPO** = lượng dữ liệu chấp nhận mất (tính từ lần backup/replicate cuối); **RTO** = thời gian downtime chấp nhận được.
- 4 chiến lược DR (rẻ → đắt, chậm → nhanh): **Backup & Restore** → **Pilot Light** (chỉ phần lõi như DB chạy sẵn) → **Warm Standby** (bản thu nhỏ chạy đầy đủ) → **Multi-Site / Hot Site** (active-active, chạy đầy đủ).
- ⚠️ Lỗi thời: **CloudEndure Disaster Recovery** → **AWS Elastic Disaster Recovery (DRS)**: replicate block-level liên tục, RPO giây, RTO phút.
- **DMS**: migrate DB, nguồn vẫn chạy; **CDC** replicate liên tục; khác engine (vd: Oracle → Aurora PostgreSQL) cần chuyển schema bằng **AWS SCT** hoặc **DMS Schema Conversion**.
- **AWS Backup**: backup tập trung đa dịch vụ, backup plan, cross-Region/cross-account copy, **Backup Vault Lock** (WORM, kể cả root không xóa được).
- ⚠️ Lỗi thời: **Application Discovery Service** và **Migration Hub** đóng với khách mới từ 07/11/2025 → **AWS Transform**. **Application Migration Service (MGN)** là công cụ lift-and-shift (rehost).
- ❓ **VMware Cloud on AWS**: vẫn xuất hiện trong whitepaper nhưng trạng thái thương mại chưa xác minh; AWS có thêm **Amazon EVS** (VMware Cloud Foundation trên EC2 bare metal trong VPC).
- Chuyển dữ liệu lớn: Internet/VPN (chậm), **Direct Connect** (lắp > 1 tháng), ⚠️ **Snowball** không còn cho khách mới → **DataSync**, **Data Transfer Terminal**; replicate liên tục: DX/VPN + DMS/DataSync.

---

## 2. Core Concepts

### RPO & RTO
- **Là gì:** RPO đo dữ liệu mất (quá khứ); RTO đo thời gian khôi phục (tương lai).
- **Dùng khi nào:** chọn chiến lược DR theo cặp RPO/RTO và ngân sách.
- **Vì sao quan trọng:** đề cho "RPO 1 giờ, RTO 4 giờ, chi phí thấp" → backup & restore hoặc pilot light; "RPO/RTO gần 0" → multi-site.

### Backup & Restore
- **Là gì:** backup định kỳ (EBS snapshot, RDS snapshot, AMI, S3 với lifecycle sang Glacier; on-prem qua Storage Gateway/Snowball cũ) rồi dựng lại khi có sự cố.
- **Dùng khi nào:** RPO/RTO tính bằng giờ, ngân sách thấp nhất.
- **Vì sao quan trọng:** rẻ nhất nhưng chậm nhất; dùng IaC (CloudFormation) để dựng lại nhanh hơn.

### Pilot Light
- **Là gì:** chỉ phần lõi quan trọng chạy sẵn trên cloud (thường là DB replicate liên tục); server ứng dụng tắt, khởi động khi cần.
- **Dùng khi nào:** RPO phút, RTO chục phút, chi phí vừa phải.
- **Vì sao quan trọng:** khác Backup & Restore: dữ liệu luôn mới (replicate) và lõi đã chạy.

### Warm Standby
- **Là gì:** toàn bộ hệ thống chạy ở quy mô nhỏ trên cloud; khi sự cố thì scale lên quy mô production (ASG) và chuyển DNS (Route 53 failover).
- **Dùng khi nào:** RTO phút, có thể phục vụ ngay ở mức thấp.
- **Vì sao quan trọng:** đắt hơn pilot light vì app tier luôn chạy.

### Multi-Site / Hot Site
- **Là gì:** hai môi trường production đầy đủ chạy song song (active-active), có thể on-prem + AWS hoặc AWS đa Region (Aurora Global, DynamoDB Global Tables, Route 53 latency/failover).
- **Dùng khi nào:** RTO/RPO gần như bằng 0, ứng dụng quan trọng nhất.
- **Vì sao quan trọng:** đắt nhất; mẹo DR chung: backup S3 cross-Region, Route 53 health check, RDS cross-Region replica, IaC, chaos testing.

### AWS Elastic Disaster Recovery (DRS)
- **Là gì:** replicate liên tục block-level server vật lý, ảo, cloud (on-prem hoặc Region khác) vào staging area chi phí thấp trên AWS; khi sự cố thì launch instance phục hồi trong vài phút; hỗ trợ **drill** và **failback**.
- **Dùng khi nào:** DR cho ứng dụng on-prem (Oracle, SQL Server, SAP, file server) sang AWS.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: tên **CloudEndure Disaster Recovery** → nay là **AWS Elastic Disaster Recovery**; AWS mô tả DRS theo mô hình pilot light.

### AWS DMS
- **Là gì:** migrate database nhanh, an toàn, nguồn vẫn hoạt động; chạy trên **replication instance** (EC2) hoặc DMS Serverless; **CDC** để replicate liên tục; **Multi-AZ** có standby đồng bộ.
- **Dùng khi nào:** homogeneous (Oracle → Oracle) hoặc heterogeneous (SQL Server → Aurora); nguồn/đích: RDS, Aurora, S3, Redshift, DynamoDB, OpenSearch, Kinesis, DocumentDB, Kafka...
- **Vì sao quan trọng:** khác engine thì cần **AWS SCT** (công cụ cài đặt) hoặc **DMS Schema Conversion** (trong console) để chuyển schema; cùng engine thì không cần.

### Migrate sang RDS/Aurora
- **Là gì:** RDS MySQL → Aurora MySQL: **snapshot restore** (có downtime) hoặc tạo **Aurora read replica** rồi promote (ít downtime hơn); MySQL ngoài RDS: Percona XtraBackup → S3 → Aurora, hoặc mysqldump; liên tục: DMS.
- **Dùng khi nào:** tương tự cho PostgreSQL (snapshot, replica, import từ S3 bằng extension `aws_s3`).
- **Vì sao quan trọng:** "ít downtime nhất từ RDS MySQL sang Aurora" → Aurora read replica rồi promote.

### On-premises strategy với AWS
- **Là gì:** tải **Amazon Linux** image dạng VM chạy on-prem; **VM Import/Export** đưa VM lên EC2 và ngược lại; **MGN** rehost server lên AWS; **DMS** cho DB; **Storage Gateway/DataSync** cho dữ liệu.
- **Dùng khi nào:** lift-and-shift nhanh nhất → **Application Migration Service (MGN)**: replicate liên tục, cutover ít downtime.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: **Application Discovery Service** (thu thập cấu hình, hiệu năng, phụ thuộc server on-prem) và **Migration Hub** (theo dõi tiến độ) đóng với khách mới từ 07/11/2025 → **AWS Transform**. ❓ Server Migration Service (SMS) trong đề cũ: không tìm thấy trang docs còn hoạt động; công cụ rehost hiện hành là MGN.

### AWS Backup
- **Là gì:** quản lý backup tập trung cho EC2/EBS, S3, RDS/Aurora, DynamoDB, DocumentDB, Neptune, EFS, FSx, Storage Gateway...; **backup plan** (tần suất, retention, chuyển cold storage); backup on-demand hoặc theo lịch; dựa trên tag.
- **Dùng khi nào:** chính sách backup thống nhất cho nhiều dịch vụ và nhiều account (Organizations backup policy); **cross-Region**, **cross-account** copy; PITR cho dịch vụ hỗ trợ.
- **Vì sao quan trọng:** **Backup Vault Lock** (WORM): không ai, kể cả root, xóa hay rút ngắn retention được → chống ransomware.

### VMware workloads
- **Là gì:** ❓ **VMware Cloud on AWS** (SDDC VMware chạy trên phần cứng AWS) vẫn có trong whitepaper; **Amazon EVS** chạy VMware Cloud Foundation trên EC2 bare metal **trong VPC của bạn**.
- **Dùng khi nào:** mở rộng data center VMware lên cloud, giữ công cụ vSphere/vSAN/NSX, DR cho VMware.
- **Vì sao quan trọng:** đề cũ chọn VMware Cloud on AWS; thiết kế mới cần kiểm tra trạng thái dịch vụ, cân nhắc EVS hoặc MGN để rehost sang EC2.

### Chuyển dữ liệu lớn
- **Là gì:** ví dụ 200 TB qua đường 100 Mbps: Internet/VPN mất khoảng 185 ngày; Direct Connect 1 Gbps: lắp > 1 tháng + khoảng 18 ngày truyền; Snowball: khoảng 1 tuần (đề cũ).
- **Dùng khi nào:** replicate liên tục sau lần chuyển đầu: VPN/DX + DMS (DB) hoặc DataSync (file).
- **Vì sao quan trọng:** ⚠️ Lỗi thời: Snowball không còn cho khách mới → **DataSync** (online), **AWS Data Transfer Terminal** (mang ổ tới cơ sở AWS), hoặc partner.

### `[+]` AWS Transform
- **Là gì:** dịch vụ dùng agentic AI hỗ trợ discovery, lập kế hoạch và hiện đại hóa migration (VMware, mainframe, .NET), thay vai trò Migration Hub/ADS.
- **Dùng khi nào:** khách mới cần đánh giá hạ tầng on-prem trước migrate.
- **Vì sao quan trọng:** đề cũ vẫn có thể hỏi ADS (agentless qua VMware vCenter, agent-based cho chi tiết process/network).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| RPO | Dữ liệu tối đa chấp nhận mất | "lose at most 15 minutes of data" |
| RTO | Downtime tối đa chấp nhận | "recover within 1 hour" |
| Backup & Restore | Rẻ nhất, chậm nhất | "hours of RTO, lowest cost" |
| Pilot Light | Lõi DB chạy sẵn | "core components always running" |
| Warm Standby | Bản thu nhỏ chạy đủ | "scaled-down version running" |
| Multi-Site | Active-active | "near-zero RTO" |
| Elastic Disaster Recovery | Replicate block-level, recover phút | "DR for on-prem servers" |
| DMS + CDC | Migrate DB, replicate liên tục | "minimal downtime DB migration" |
| AWS SCT / DMS Schema Conversion | Chuyển schema khác engine | "Oracle to PostgreSQL" |
| AWS Backup | Backup tập trung | "centralized backup policy" |
| Backup Vault Lock | WORM cho backup | "ransomware protection" |
| MGN | Lift-and-shift server | "rehost with minimal downtime" |
| VM Import/Export | Đưa VM lên EC2 | "import VMware image" |
| ADS / Migration Hub | Discovery & theo dõi (đóng khách mới) | "plan migration" |
| AWS Transform | Discovery/modernization bằng AI | "new customers plan migration" |
| Amazon EVS | VMware Cloud Foundation trong VPC | "run VMware on EC2 bare metal" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Pilot Light | Warm Standby | Pilot: chỉ DB/lõi chạy; Warm: cả app chạy thu nhỏ | "Scaled-down full environment" → Warm |
| Warm Standby | Multi-Site | Warm phải scale lên; Multi-site chạy full sẵn | RTO ≈ 0 → Multi-site |
| DRS | MGN | DRS cho DR (replicate chờ sự cố); MGN cho migrate một lần | "Disaster recovery" → DRS |
| DMS | SCT | DMS chuyển dữ liệu; SCT chuyển schema | Khác engine → cả hai |
| DMS | DataSync | DMS cho database; DataSync cho file/object | |
| AWS Backup | EBS snapshot / DLM | Backup đa dịch vụ, đa account; DLM chỉ EBS/AMI | Nhiều dịch vụ → AWS Backup |
| Backup Vault Lock | S3 Object Lock | Vault Lock cho recovery point trong AWS Backup; Object Lock cho object S3 | |
| MGN | VM Import/Export | MGN replicate liên tục, cutover ít downtime; VM Import upload image | Ít downtime → MGN |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "lowest cost DR, RTO hours" | Backup & Restore | |
| "DB replicated, app servers off until needed" | Pilot Light | |
| "scaled-down copy always running" | Warm Standby | |
| "active-active in two Regions" | Multi-Site | |
| "DR for on-prem servers, RPO seconds, RTO minutes" | Elastic Disaster Recovery | |
| "migrate Oracle to Aurora PostgreSQL" | SCT/DMS Schema Conversion + DMS | |
| "keep source DB running during migration" | DMS với CDC | |
| "RDS MySQL to Aurora with minimal downtime" | Aurora read replica rồi promote | |
| "centralized backups across accounts with WORM" | AWS Backup + Vault Lock | |
| "lift-and-shift servers to EC2" | Application Migration Service (MGN) | |
| "discover on-prem servers (new customer)" | AWS Transform | ADS đóng |
| "VMware environment in own VPC" | Amazon EVS | |
| "ongoing file replication after initial transfer" | DataSync qua VPN/DX | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Công ty chấp nhận RTO 12 giờ và RPO 24 giờ, muốn chi phí DR thấp nhất. Chọn gì?
- A. Multi-Site
- B. Warm Standby
- C. Backup & Restore
- D. Pilot Light

<details><summary>Đáp án</summary>

**C.**
- A, B, D sai: đắt hơn mức cần.
</details>

**Câu 2.** DB phải replicate liên tục sang AWS; app server chỉ bật khi có sự cố; RTO khoảng 30 phút. Chiến lược nào?
- A. Backup & Restore
- B. Pilot Light
- C. Multi-Site
- D. Không cần DR

<details><summary>Đáp án</summary>

**B.**
- A sai: dữ liệu không replicate liên tục.
- C sai: app chạy full, đắt.
- D sai: không đáp ứng yêu cầu.
</details>

**Câu 3.** (Chọn 2) Migrate Oracle on-prem sang Aurora PostgreSQL, nguồn vẫn phục vụ trong lúc chuyển. Chọn gì?
- A. AWS SCT hoặc DMS Schema Conversion
- B. AWS DMS với CDC
- C. AWS DataSync
- D. AWS Backup
- E. VM Import/Export

<details><summary>Đáp án</summary>

**A, B.**
- C sai: cho file.
- D sai: backup, không migrate khác engine.
- E sai: chuyển VM, không chuyển DB khác engine.
</details>

**Câu 4.** Cần DR cho 200 server on-prem với RPO vài giây, RTO vài phút, chi phí staging thấp. Chọn gì?
- A. AWS Elastic Disaster Recovery
- B. AWS Backup hằng ngày
- C. Multi-Site full
- D. S3 CRR

<details><summary>Đáp án</summary>

**A.**
- B sai: RPO 24 giờ.
- C sai: rất đắt.
- D sai: cho object S3.
</details>

**Câu 5.** Công ty phải bảo vệ backup khỏi bị xóa bởi ransomware hoặc admin bị chiếm quyền, áp dụng cho EBS, RDS, DynamoDB. Chọn gì?
- A. AWS Backup với Backup Vault Lock
- B. EBS snapshot thủ công
- C. S3 versioning
- D. MFA Delete

<details><summary>Đáp án</summary>

**A.**
- B sai: xóa được.
- C, D sai: chỉ áp cho S3.
</details>

**Câu 6.** Muốn chuyển RDS MySQL sang Aurora MySQL với downtime nhỏ nhất. Cách nào?
- A. Snapshot RDS rồi restore sang Aurora
- B. Tạo Aurora read replica từ RDS MySQL, đợi lag về 0 rồi promote
- C. mysqldump
- D. Dùng DataSync

<details><summary>Đáp án</summary>

**B.**
- A sai: phải dừng ghi trong lúc snapshot/restore.
- C sai: chậm, downtime lớn.
- D sai: không cho DB.
</details>

**Câu 7.** Khách hàng mới (2026) muốn đánh giá 500 server on-prem trước khi lift-and-shift. Chọn gì?
- A. Application Discovery Service + Migration Hub
- B. AWS Transform để discovery/lập kế hoạch, sau đó MGN để rehost
- C. Server Migration Service
- D. Snowball

<details><summary>Đáp án</summary>

**B.**
- A sai: đóng với khách mới từ 07/11/2025.
- C sai: SMS không còn trong docs hiện hành (❓ trạng thái chính thức), công cụ rehost hiện hành là MGN.
- D sai: không còn cho khách mới và không phải công cụ đánh giá.
</details>

**Câu 8.** Ứng dụng quan trọng cần RTO gần 0, chạy đồng thời ở 2 Region. Chọn gì?
- A. Warm Standby
- B. Multi-Site active-active (Aurora Global/DynamoDB Global Tables + Route 53)
- C. Pilot Light
- D. Backup & Restore

<details><summary>Đáp án</summary>

**B.**
- A, C, D sai: đều cần thời gian scale hoặc dựng lại.
</details>

---

### Nguồn đã đối chiếu
- Disaster recovery options in the cloud (4 chiến lược, DRS theo pilot light): https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html
- AWS Elastic Disaster Recovery: https://docs.aws.amazon.com/drs/latest/userguide/what-is-drs.html
- AWS DMS Schema Conversion: https://docs.aws.amazon.com/dms/latest/userguide/CHAP_SchemaConversion.html
- AWS Backup Vault Lock: https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html
- Application Discovery Service availability change (07/11/2025, AWS Transform): https://docs.aws.amazon.com/application-discovery/latest/userguide/application-discovery-service-availability-change.html
- Application Migration Service: https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html
- AWS Transform (agent cho migration, gồm VMware): https://docs.aws.amazon.com/transform/latest/userguide/transform-app-vmware.html
- Amazon EVS: https://docs.aws.amazon.com/evs/latest/userguide/what-is-evs.html
- VMware Cloud on AWS (whitepaper tổng quan): https://docs.aws.amazon.com/whitepapers/latest/aws-overview/compute-services.html
- Snowball Edge availability change: https://docs.aws.amazon.com/snowball/latest/developer-guide/snowball-edge-availability-change.html
