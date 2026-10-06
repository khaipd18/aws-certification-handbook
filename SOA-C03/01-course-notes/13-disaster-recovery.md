# SOA-C03 — Disaster Recovery: DataSync, AWS Backup & DR Strategies

> Thuộc **Domain 2: Reliability and Business Continuity (22%)**: chuyển dữ liệu, backup tập trung, chống xóa backup, RPO/RTO và bốn chiến lược DR.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **DataSync** chuyển dữ liệu số lượng lớn:
  - Từ on-prem hoặc cloud khác lên AWS: **cần agent**.
  - Giữa các dịch vụ AWS: **không cần agent**.
  - Chạy on-demand hoặc theo lịch giờ / ngày / tuần.
- Đích của DataSync: S3 (mọi storage class, kể cả Glacier), EFS, FSx. Giữ nguyên quyền và metadata file, kiểm tra toàn vẹn sau khi chuyển.
- **AWS Backup** gom backup cho EC2/EBS, S3, RDS/Aurora, DynamoDB, EFS, FSx, Storage Gateway…; copy được cross-Region và cross-account.
- **Backup plan** gồm tần suất, backup window, chuyển cold storage, retention; chọn tài nguyên theo **tag** hoặc ID.
- **Vault Lock** có hai mode:
  - **Governance**: người có đủ quyền IAM vẫn quản lý được.
  - **Compliance**: sau grace time thì bất biến, không ai xóa được, kể cả root hay AWS.
- `[+]` **RPO** = lượng dữ liệu chấp nhận mất; **RTO** = thời gian chấp nhận ngừng.
- `[+]` Bốn chiến lược DR, từ rẻ/chậm tới đắt/nhanh: **Backup & Restore → Pilot Light → Warm Standby → Multi-Site**.
- ⚠️ Lỗi thời: CloudEndure Disaster Recovery → nay là **AWS Elastic Disaster Recovery (DRS)**.

---

## 2. Core Concepts

### AWS DataSync
- **Là gì:** dịch vụ chuyển dữ liệu qua NFS, SMB, HDFS, object storage (S3 API); mã hóa TLS khi truyền, giữ quyền POSIX/SMB và metadata.
- **Dùng khi nào:** migrate lần đầu, đồng bộ định kỳ on-prem → AWS, copy giữa EFS và FSx, S3 và EFS.
- **Vì sao quan trọng:** có thể giới hạn băng thông để không làm nghẽn đường truyền site. Agent cần ra **TCP 443** tới AWS; nguồn NFS dùng 2049, SMB 445, HDFS NameNode mặc định 8020.

### DataSync vs các lựa chọn khác
- **Là gì:** DataSync chuyển theo lô/lịch qua mạng; Storage Gateway cho truy cập lai liên tục; S3 CRR sao chép liên tục S3 → S3.
- **Dùng khi nào:** "migrate 200 TB một lần, có kiểm tra" → DataSync; "replicate bucket liên tục sang Region khác" → CRR.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "băng thông kém, petabyte → Snowball Edge" → Snowball Edge **không nhận khách mới từ 07/11/2025**; docs gợi ý DataSync (online) hoặc giải pháp offline của Partner.

### AWS Backup
- **Là gì:** backup tập trung có vault, plan, báo cáo; hỗ trợ cross-Region và cross-account copy (cần lưu ý chia sẻ KMS key).
- **Dùng khi nào:** một chính sách backup chung cho EBS, RDS, DynamoDB, EFS, FSx; cần báo cáo compliance (Backup Audit Manager).
- **Vì sao quan trọng:** PITR chỉ có với dịch vụ hỗ trợ. Với EC2, Backup tạo backup crash-consistent; Windows có tùy chọn VSS. Danh sách dịch vụ hỗ trợ thay đổi theo thời gian, xem docs.

### Backup plan & lifecycle
- **Là gì:** tần suất (12 giờ, ngày, tuần, tháng, cron), backup window, chuyển cold storage, retention; gán tài nguyên theo tag.
- **Dùng khi nào:** "mọi tài nguyên có tag `Backup=Gold` backup hằng ngày giữ 35 ngày, hằng tháng giữ 7 năm" → một plan với nhiều rule.
- **Vì sao quan trọng:** gán theo tag giúp tài nguyên mới tự được backup mà không phải sửa plan.

### Backup Vault Lock
- **Là gì:** WORM cho recovery point trong vault: chặn xóa và chặn rút ngắn retention.
- **Dùng khi nào:** chống ransomware, chống người trong nội bộ xóa backup, yêu cầu quy định (SEC 17a-4, FINRA).
- **Vì sao quan trọng:** **Governance mode** người có quyền IAM vẫn sửa/xóa lock được; **Compliance mode** có grace time (tối thiểu 3 ngày) rồi bất biến, không ai gỡ được. Không tính phí thêm, nhưng backup bị khóa vẫn tính phí lưu.

### `[+]` RPO & RTO
- **Là gì:** RPO đo ngược từ lúc sự cố (mất bao nhiêu dữ liệu); RTO đo xuôi (bao lâu mới chạy lại).
- **Dùng khi nào:** RPO thấp → replication liên tục; RTO thấp → giữ sẵn hạ tầng chạy.
- **Vì sao quan trọng:** đề thường siết một trong hai chỉ số để chỉ ra đáp án.

### `[+]` Bốn chiến lược DR
- **Là gì:** **Backup & Restore** (rẻ nhất, khôi phục sau sự cố); **Pilot Light** (dữ liệu replicate, chỉ phần lõi như DB chạy); **Warm Standby** (bản thu nhỏ nhưng đầy đủ chức năng, scale lên khi failover); **Multi-Site** (active-active đủ công suất).
- **Dùng khi nào:** chọn theo RPO/RTO và ngân sách đề cho.
- **Vì sao quan trọng:** đề cho "chạy sẵn bản thu nhỏ đầy đủ" → Warm Standby; "chỉ DB replicate, compute dựng khi cần" → Pilot Light.

### `[+]` AWS Elastic Disaster Recovery (DRS)
- **Là gì:** replicate liên tục mức block của server on-prem/cloud vào staging area chi phí thấp; chỉ dựng instance đầy đủ khi drill hoặc recovery, có failback.
- **Dùng khi nào:** lift-and-shift server cần mất dữ liệu gần như bằng 0 mà không trả tiền standby đầy đủ.
- **Vì sao quan trọng:** docs mô tả recovery trong vài phút với trạng thái mới nhất hoặc một thời điểm trước đó. ❓ Con số RPO/RTO cụ thể của slide chưa đối chiếu.

### `[+]` Thành phần DR theo lớp dữ liệu
- **Là gì:** S3 → CRR (thêm RTC để có SLA 15 phút); EBS/AMI → copy snapshot/AMI sang Region khác; RDS → read replica cross-Region hoặc copy snapshot; Aurora → Global Database; DynamoDB → Global Tables.
- **Dùng khi nào:** RPO tính bằng giây → replication; RPO tính bằng giờ → backup.
- **Vì sao quan trọng:** AWS Backup lo phần backup; thiết kế RPO thấp cần replication.

### `[+]` Chuyển traffic khi failover
- **Là gì:** Route 53 health check + failover routing tự chuyển sang Region phụ; Global Accelerator chuyển ở mức IP.
- **Dùng khi nào:** pilot light, warm standby cần cutover tự động.
- **Vì sao quan trọng:** DNS TTL làm chậm cutover thực tế → hạ TTL trước, hoặc dùng Global Accelerator.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| DataSync agent | VM ở nguồn on-prem | "NFS/SMB on-premises to S3/EFS" |
| DataSync task | Lần chuyển dữ liệu theo lịch | "scheduled transfer, verify integrity" |
| AWS Backup plan | Tần suất + retention + lifecycle | "centralized backup policy" |
| Backup vault | Nơi chứa recovery point | "store backups separately" |
| Vault Lock Governance | Người có quyền vẫn quản lý được | "restrict to privileged users" |
| Vault Lock Compliance | Bất biến sau grace time | "WORM", "even root cannot delete" |
| RPO | Lượng dữ liệu chấp nhận mất | "lose at most 15 minutes of data" |
| RTO | Thời gian chấp nhận ngừng | "back online within 1 hour" |
| Pilot Light | Lõi dữ liệu chạy sẵn | "minimal core running" |
| Warm Standby | Bản thu nhỏ đầy đủ chạy sẵn | "scaled-down functional copy" |
| Multi-Site | Active-active đủ công suất | "near-zero RTO" |
| AWS DRS | Replication block liên tục | "continuous replication of servers" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| DataSync | Storage Gateway | Chuyển theo lô; truy cập lai liên tục | "Migrate once" → DataSync |
| DataSync | S3 CRR | Lịch, nhiều loại storage; liên tục, chỉ S3 | "Continuously replicate bucket" → CRR |
| DataSync | AWS Backup | Di chuyển dữ liệu; recovery point có retention | Retention, compliance → Backup |
| AWS Backup | DLM | Đa dịch vụ, vault, Vault Lock; chỉ EBS snapshot/AMI | Nhiều dịch vụ → Backup |
| Vault Lock Governance | Vault Lock Compliance | Admin có quyền vẫn gỡ; bất biến | "No one can delete" → Compliance |
| Backup Vault Lock | S3 Object Lock | Recovery point của Backup; object version S3 | Đề nói AWS Backup → Vault Lock |
| Backup | Replication | Theo thời điểm; liên tục | RPO giây → replication |
| Pilot Light | Warm Standby | Chỉ lõi dữ liệu; bản thu nhỏ đầy đủ | "Fully functional, scaled down" → Warm |
| Warm Standby | Multi-Site | Scale lên khi failover; đủ công suất cả hai | Near-zero RTO → Multi-Site |
| AWS DRS | AWS Backup | Replication block liên tục; backup theo lịch | Gần 0 mất dữ liệu cho server → DRS |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "move NFS data on-prem to EFS weekly" | DataSync (agent) | |
| "copy data from EFS to FSx" | DataSync, không cần agent | |
| "limit transfer bandwidth during business hours" | DataSync bandwidth limit | |
| "one backup policy for EBS, RDS, DynamoDB, EFS" | AWS Backup plan | |
| "backups of new resources automatically" | Gán theo tag | |
| "backups must be immutable, even root cannot delete" | Vault Lock Compliance | |
| "copy backups to another account for ransomware isolation" | Cross-account backup copy | |
| "lowest cost DR, hours of RTO OK" | Backup & Restore | |
| "only database replicated, compute launched on failover" | Pilot Light | |
| "scaled-down full environment always running" | Warm Standby | |
| "active-active in two Regions" | Multi-Site | |
| "near-zero data loss for on-prem servers" | AWS Elastic Disaster Recovery | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Cần chuyển 150 TB từ NAS NFS on-prem lên Amazon EFS, có kiểm tra toàn vẹn, sau đó đồng bộ thay đổi hằng đêm. Chọn gì?
- A. Storage Gateway S3 File Gateway
- B. DataSync với agent on-prem, task chạy theo lịch hằng ngày
- C. S3 Cross-Region Replication
- D. AWS Backup

<details><summary>Đáp án</summary>

**B.**
- A sai: ghi vào S3, không phải EFS; dùng cho truy cập lai.
- C sai: chỉ S3 → S3.
- D sai: tạo recovery point, không di chuyển dữ liệu on-prem lên EFS.
</details>

**Câu 2.** Công ty muốn một chính sách backup chung cho EBS, RDS, DynamoDB, EFS; tài nguyên mới tự được backup. Chọn gì?
- A. DLM
- B. AWS Backup plan gán tài nguyên theo tag
- C. Lambda + EventBridge cho từng dịch vụ
- D. DataSync

<details><summary>Đáp án</summary>

**B.**
- A sai: chỉ EBS snapshot/AMI.
- C sai: tự viết, khó duy trì.
- D sai: không phải công cụ backup.
</details>

**Câu 3.** (Chọn 2) Yêu cầu: backup không bị xóa hay rút ngắn retention, kể cả bởi root, và có bản cách ly ở account khác để chống ransomware. Chọn gì?
- A. Backup Vault Lock ở Compliance mode
- B. Copy backup cross-account sang vault của account khác
- C. Vault Lock ở Governance mode
- D. S3 Versioning
- E. DLM

<details><summary>Đáp án</summary>

**A, B.**
- C sai: người có quyền IAM vẫn gỡ được.
- D sai: không áp cho recovery point của Backup.
- E sai: không có vault lock.
</details>

**Câu 4.** Ứng dụng chấp nhận mất tối đa 24 giờ dữ liệu và ngừng tối đa 12 giờ; ưu tiên chi phí thấp nhất. Chọn chiến lược nào?
- A. Multi-Site
- B. Warm Standby
- C. Backup & Restore
- D. Pilot Light

<details><summary>Đáp án</summary>

**C.** RPO/RTO tính bằng giờ, rẻ nhất.
- A, B, D sai: tốn kém hơn mức cần thiết.
</details>

**Câu 5.** Ở Region DR, chỉ DB được replicate liên tục; web và app server chỉ dựng khi có sự cố. Đây là chiến lược nào?
- A. Backup & Restore
- B. Pilot Light
- C. Warm Standby
- D. Multi-Site

<details><summary>Đáp án</summary>

**B.**
- A sai: dữ liệu không replicate liên tục.
- C sai: Warm Standby có cả stack thu nhỏ đang chạy.
- D sai: Multi-Site chạy đủ công suất.
</details>

**Câu 6.** Cần DR cho 200 server on-prem với mất dữ liệu gần như bằng 0, khôi phục trong vài phút, không muốn trả tiền chạy server dự phòng đầy đủ. Chọn gì?
- A. AWS Backup hằng ngày
- B. AWS Elastic Disaster Recovery
- C. DataSync hằng giờ
- D. Multi-Site active-active

<details><summary>Đáp án</summary>

**B.** Replication block liên tục vào staging rẻ, dựng instance khi cần.
- A sai: RPO tới 24 giờ.
- C sai: theo lịch, chỉ dữ liệu file.
- D sai: tốn chi phí chạy đầy đủ.
</details>

**Câu 7.** Quản trị viên ở Governance mode muốn gỡ Vault Lock. Phát biểu nào đúng?
- A. Không ai gỡ được
- B. User có quyền IAM phù hợp gỡ được
- C. Chỉ AWS Support gỡ được
- D. Phải chờ 72 giờ

<details><summary>Đáp án</summary>

**B.**
- A sai: đó là Compliance mode sau grace time.
- C sai: không có cơ chế này.
- D sai: grace time là khái niệm của Compliance mode.
</details>

**Câu 8.** (Chọn 2) Kiến trúc pilot light ở Region phụ cần tự chuyển traffic khi Region chính sập. Hai thành phần nào phù hợp?
- A. Route 53 health check
- B. Route 53 failover routing policy
- C. Route 53 weighted 50/50
- D. CloudFront Geo Restriction
- E. DataSync

<details><summary>Đáp án</summary>

**A, B.**
- C sai: chia traffic cả khi Region chính khỏe.
- D sai: lọc quốc gia.
- E sai: chuyển dữ liệu, không chuyển traffic.
</details>

---

### Nguồn đã đối chiếu
- DataSync network requirements (443, NFS, SMB, HDFS 8020): https://docs.aws.amazon.com/datasync/latest/userguide/datasync-network.html
- AWS Backup Vault Lock (Governance / Compliance, grace time): https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html
- AWS Backup EBS/EC2 (crash-consistent): https://docs.aws.amazon.com/aws-backup/latest/devguide/multi-volume-crash-consistent.html
- AWS Elastic Disaster Recovery: https://docs.aws.amazon.com/drs/latest/userguide/what-is-drs.html
- Aurora Global Database: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html
- Snowball Edge availability change: https://docs.aws.amazon.com/snowball/latest/developer-guide/snowball-edge-availability-change.html
