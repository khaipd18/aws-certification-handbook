# SOA-C03 — EC2 Storage & Data Management: EBS, Instance Store, EFS

> Thuộc **Domain 2: Reliability and Business Continuity (22%)** và **Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization (22%)**: chọn và tối ưu volume, snapshot tự động, khôi phục nhanh, xử lý EFS không mount được.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **EBS** là ổ mạng gắn với **một AZ**, trả theo dung lượng cấp phát. Chuyển AZ/Region: **snapshot → restore**.
- Volume type: gp2/gp3 (SSD đa dụng), io1/io2 Block Express (SSD IOPS cao), st1/sc1 (HDD). **Chỉ SSD làm boot volume**.
- ⚠️ Lỗi thời: "gp3 tối đa 16,000 IOPS / 1,000 MiB/s / 16 TiB" → nay **80,000 IOPS / 2,000 MiB/s / 64 TiB**. io2 Block Express: 256,000 IOPS, 64 TiB.
- **Snapshot** incremental, lưu trên hạ tầng S3. Block đầu tiên sau restore đọc chậm, trừ khi bật **Fast Snapshot Restore** (tính phí theo phút).
- **DLM** tự tạo/giữ/xóa snapshot và AMI theo **tag**. **Snapshot Archive** rẻ hơn tới 75%, restore 24–72 giờ. **Recycle Bin** chống xóa nhầm.
- **EFS**: NFSv4.1 (TCP **2049**), nhiều AZ, chỉ Linux/POSIX, có lifecycle sang IA và Archive.
- Chọn nhanh:
  - Một instance, block storage → EBS.
  - Nhiều instance, nhiều AZ → EFS.
  - Tạm thời, IOPS cao nhất → instance store.

---

## 2. Core Concepts

### EBS volume
- **Là gì:** block device qua mạng, gắn một AZ, detach/attach nhanh trong cùng AZ.
- **Dùng khi nào:** root volume, DB trên EC2, dữ liệu bền gắn một instance.
- **Vì sao quan trọng:** dữ liệu sống sót khi terminate, nhưng **root volume mặc định bị xóa** (`DeleteOnTermination`).

### gp2 vs gp3
- **Là gì:** gp2: 3 IOPS/GiB, burst tới 3,000, tối đa 16,000. gp3: baseline 3,000 IOPS và 125 MiB/s, chỉnh IOPS/throughput độc lập.
- **Dùng khi nào:** cần thêm IOPS mà không tăng dung lượng → gp3; chuyển gp2 → gp3 tại chỗ bằng Elastic Volumes.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: giới hạn gp3 16k IOPS / 1,000 MiB/s → nay 80k / 2,000 MiB/s, tới 64 TiB.

### io1 / io2 Block Express & Multi-Attach
- **Là gì:** SSD provisioned IOPS, độ trễ dưới 1 ms; io2 BE tới 256,000 IOPS, 64 TiB; io1 tối đa 64,000 IOPS trên Nitro.
- **Dùng khi nào:** cần hơn 80,000 IOPS hoặc DB nhạy độ trễ. Multi-Attach gắn io1/io2 vào nhiều instance cùng AZ.
- **Vì sao quan trọng:** Multi-Attach cần file system **cluster-aware**; XFS/ext4 thường sẽ hỏng dữ liệu. Workload mới nên dùng io2.

### st1 & sc1
- **Là gì:** st1 tối ưu throughput (tối đa 500 MiB/s), sc1 rẻ nhất cho dữ liệu lạnh (250 MiB/s); từ 125 GiB.
- **Dùng khi nào:** big data, log, đọc tuần tự → st1; ít truy cập → sc1.
- **Vì sao quan trọng:** không làm boot volume. `BurstBalance` áp cho gp2, st1, sc1.

### Elastic Volumes
- **Là gì:** đổi size, type, IOPS, throughput khi volume đang chạy, không cần detach.
- **Dùng khi nào:** chuyển gp2 → gp3, nới ổ đầy, tăng IOPS.
- **Vì sao quan trọng:** chỉ **tăng** size, không giảm; sau đó phải mở rộng file system trong OS. ⚠️ Lỗi thời: "chờ 6 giờ mới sửa lại" → nay sửa tối đa **4 lần trong 24 giờ trượt**, mỗi lần phải chờ lần trước hoàn tất.

### EBS Snapshots & Fast Snapshot Restore
- **Là gì:** bản sao point-in-time, incremental, copy được sang AZ/Region; volume từ snapshot được nạp dần từ S3.
- **Dùng khi nào:** khôi phục cần hiệu năng ngay → **FSR** (bật theo snapshot + AZ, tính phí theo phút) hoặc đọc trước toàn bộ block.
- **Vì sao quan trọng:** snapshot dùng I/O của volume, tránh giờ cao điểm. Mã hóa volume có sẵn: snapshot → copy bật mã hóa → tạo volume mới.

### Data Lifecycle Manager (DLM)
- **Là gì:** policy tạo, giữ, xóa snapshot và AMI EBS-backed theo lịch, chọn tài nguyên bằng tag, copy cross-account/Region.
- **Dùng khi nào:** backup EBS định kỳ không cần code.
- **Vì sao quan trọng:** DLM **không quản lý** snapshot/AMI tạo ngoài DLM, cũng không quản lý AMI instance store-backed.

### Snapshot Archive & Recycle Bin
- **Là gì:** Archive rẻ hơn tới 75%, dành cho snapshot giữ ít nhất 90 ngày, restore 24–72 giờ. Recycle Bin giữ tài nguyên bị xóa theo rule: snapshot và AMI 1–365 ngày, volume 1–7 ngày.
- **Dùng khi nào:** lưu trữ compliance nhiều năm → Archive; chống xóa nhầm → Recycle Bin.
- **Vì sao quan trọng:** Archive không hợp RTO dưới một ngày. Recycle Bin chỉ bảo vệ tài nguyên bị xóa **sau** khi rule tồn tại.

### Instance Store
- **Là gì:** NVMe/SSD gắn vật lý trên host, IOPS cao nhất, đã gồm trong giá instance.
- **Dùng khi nào:** buffer, cache, scratch, dữ liệu đã replicate (Cassandra, Kafka).
- **Vì sao quan trọng:** chỉ sống qua reboot; mất khi **stop, hibernate, terminate** hoặc host lỗi.

### EBS metrics
- **Là gì:** namespace `AWS/EBS`: `VolumeReadOps`, `VolumeWriteOps`, `VolumeQueueLength`, `VolumeIdleTime`, `BurstBalance`.
- **Dùng khi nào:** `VolumeQueueLength` cao kéo dài và `BurstBalance` về 0 → storage là nút cổ chai.
- **Vì sao quan trọng:** sửa bằng gp3/io2, không phải thêm CPU. Metric disk trong `AWS/EC2` chỉ cho instance store.

### Amazon EFS
- **Là gì:** NFS managed, Multi-AZ, tự co giãn, SG điều khiển truy cập, mã hóa at rest bằng KMS (chọn lúc tạo), in transit bằng TLS qua mount helper.
- **Dùng khi nào:** web fleet dùng chung nội dung, home directory, container/Lambda cần file system chung.
- **Vì sao quan trọng:** Windows → FSx for Windows. Giá theo GB cao hơn EBS; giảm bằng lifecycle và One Zone.

### EFS performance & throughput
- **Là gì:** performance mode General Purpose (mặc định) hoặc Max I/O, chọn lúc tạo; throughput mode Elastic, Provisioned, Bursting.
- **Dùng khi nào:** tải khó đoán → Elastic; throughput cao ổn định, dữ liệu nhỏ → Provisioned.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Max I/O cho tải song song cao" → AWS nay khuyên **General Purpose cho mọi file system**; Max I/O không dùng được với Elastic throughput và One Zone.

### EFS storage classes, mount target & Access Points
- **Là gì:** Standard, Infrequent Access, Archive (vài lần mỗi năm) + lifecycle policy; One Zone rẻ hơn, một AZ. Mount target là ENI mỗi AZ; Access Point ép POSIX user/group và thư mục gốc cho từng ứng dụng.
- **Dùng khi nào:** file không truy cập sau N ngày → lifecycle sang IA/Archive; nhiều app dùng chung một file system cô lập → Access Points.
- **Vì sao quan trọng:** "không mount được EFS" gần như luôn do SG không mở **TCP 2049** giữa instance và mount target, AZ của instance chưa có mount target, hoặc thiếu NFS client.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| gp3 | SSD đa dụng, IOPS độc lập dung lượng | "more IOPS without bigger volume" |
| io2 Block Express | SSD IOPS cao nhất | "> 80,000 IOPS", "sub-millisecond" |
| Multi-Attach | io1/io2 gắn nhiều instance cùng AZ | "cluster-aware file system" |
| Elastic Volumes | Sửa volume đang chạy | "modify without detaching" |
| `BurstBalance` | Credit burst của gp2/st1/sc1 | "performance drops after a while" |
| `VolumeQueueLength` | Hàng đợi I/O | "storage bottleneck" |
| Fast Snapshot Restore | Volume đạt hiệu năng ngay | "no first-read latency" |
| DLM | Tự động snapshot/AMI theo tag | "automate EBS backups" |
| Snapshot Archive | Lưu rẻ, restore 24–72 giờ | "retain for years" |
| Recycle Bin | Khôi phục snapshot/AMI bị xóa | "accidental deletion" |
| Instance store | Ổ local tạm | "ephemeral", "highest IOPS" |
| EFS mount target | ENI NFS trong mỗi AZ | "cannot mount EFS" |
| EFS Access Point | POSIX user + thư mục gốc | "isolate apps on one file system" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| EBS | EFS | Một instance, một AZ; nhiều instance, nhiều AZ | "Shared across AZs" → EFS |
| EBS | Instance store | Bền khi stop; mất khi stop nhưng nhanh nhất | "Temporary, highest IOPS" → instance store |
| gp2 | gp3 | IOPS theo size; IOPS độc lập | Thêm IOPS không thêm size → gp3 |
| gp3 | io2 BE | Tới 80k IOPS; tới 256k, độ trễ thấp | > 80k → io2 |
| st1 | sc1 | Throughput, dùng thường xuyên; lạnh, rẻ nhất | "Lowest cost HDD" → sc1 |
| Multi-Attach | EFS | Block, một AZ, cluster-aware FS; file NFS nhiều AZ | Web fleet chia sẻ file → EFS |
| FSR | Pre-warm | Trả phí để khởi tạo sẵn; tự đọc toàn bộ block | Không muốn tốn tiền → pre-warm |
| Snapshot Archive | Recycle Bin | Giảm phí lưu lâu; chống xóa nhầm | "Deleted by mistake" → Recycle Bin |
| DLM | AWS Backup | Chỉ EBS snapshot/AMI; backup tập trung nhiều dịch vụ, Vault Lock | Nhiều dịch vụ, compliance → Backup |
| EFS Elastic | EFS Provisioned | Tự co giãn, trả theo dùng; cố định, đặt trước | Spiky → Elastic |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "move volume to another AZ" | Snapshot → volume ở AZ đích | |
| "encrypt existing unencrypted volume" | Snapshot → copy có mã hóa → volume mới | |
| "volume performance degrades over hours" | `BurstBalance` cạn → gp3 | |
| "restored volume slow on first access" | Fast Snapshot Restore | |
| "modify volume type without downtime" | Elastic Volumes | Tối đa 4 lần / 24 giờ |
| "volume full, increase size" | Elastic Volumes + mở rộng file system | |
| "automate daily EBS snapshots by tag" | DLM | |
| "recover deleted snapshot" | Recycle Bin | |
| "shared POSIX storage across AZs" | EFS | |
| "EFS mount times out" | SG mở TCP 2049, mount target trong AZ | |
| "files idle 30 days, cut EFS cost" | Lifecycle → IA / Archive | |
| "different apps need isolated directories on EFS" | Access Points | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Volume gp2 500 GiB của DB chậm dần sau vài giờ tải cao; `BurstBalance` về 0. Cách khắc phục hiệu quả chi phí nhất?
- A. Tăng instance type
- B. Chuyển sang gp3 và đặt IOPS cần thiết bằng Elastic Volumes
- C. Chuyển sang st1
- D. Bật Fast Snapshot Restore

<details><summary>Đáp án</summary>

**B.** gp3 có baseline 3,000 IOPS, chỉnh thêm độc lập, không phụ thuộc credit.
- A sai: nút cổ chai là storage.
- C sai: HDD, IOPS thấp.
- D sai: chỉ ảnh hưởng volume tạo từ snapshot.
</details>

**Câu 2.** Volume tạo từ snapshot trong lúc DR có độ trễ rất cao ở lần đọc đầu. Cần volume đạt hiệu năng đầy đủ ngay khi tạo. Chọn gì?
- A. Fast Snapshot Restore trên snapshot ở AZ đích
- B. Snapshot Archive
- C. Recycle Bin
- D. Multi-Attach

<details><summary>Đáp án</summary>

**A.**
- B sai: Archive còn chậm hơn, restore 24–72 giờ.
- C sai: chống xóa nhầm.
- D sai: không liên quan khởi tạo volume.
</details>

**Câu 3.** (Chọn 2) Phải giữ snapshot EBS 7 năm cho compliance, hiếm khi restore, và có đường lùi khi lỡ tay xóa. Chọn gì?
- A. Snapshot Archive
- B. Recycle Bin với retention rule
- C. Fast Snapshot Restore
- D. Instance store
- E. Multi-Attach

<details><summary>Đáp án</summary>

**A, B.**
- C sai: tốn tiền, không giảm chi phí lưu.
- D, E sai: không liên quan snapshot.
</details>

**Câu 4.** EC2 ở `us-east-1b` không mount được EFS; instance ở `us-east-1a` mount bình thường. Nguyên nhân khả dĩ nhất?
- A. EFS không hỗ trợ nhiều AZ
- B. Chưa có mount target trong `us-east-1b`, hoặc SG của mount target đó không mở TCP 2049
- C. Thiếu IAM role cho EC2
- D. Phải dùng Max I/O

<details><summary>Đáp án</summary>

**B.**
- A sai: EFS Standard là Multi-AZ.
- C sai: mount NFS mặc định không cần IAM (trừ khi bật IAM authorization).
- D sai: performance mode không ảnh hưởng mount.
</details>

**Câu 5.** Kỹ sư sửa type của một volume 4 lần trong ngày; lần thứ 5 bị lỗi. Vì sao?
- A. Volume phải detach trước
- B. Elastic Volumes giới hạn 4 lần sửa trong 24 giờ trượt
- C. Phải stop instance
- D. gp3 không đổi được type

<details><summary>Đáp án</summary>

**B.** Docs: tối đa 4 lần trong 24 giờ trượt; lỗi trả về thời điểm được sửa tiếp.
- A, C sai: Elastic Volumes sửa khi volume đang gắn và chạy.
- D sai: gp3 đổi được sang type khác.
</details>

**Câu 6.** Công ty muốn tự snapshot mọi volume có tag `Backup=Daily` mỗi ngày, giữ 14 bản, không code. Chọn gì?
- A. DLM policy nhắm theo tag
- B. Lambda + EventBridge
- C. Recycle Bin
- D. Snapshot Archive

<details><summary>Đáp án</summary>

**A.**
- B sai: phải tự viết code.
- C sai: không tạo snapshot.
- D sai: không tự tạo snapshot.
</details>

**Câu 7.** Ứng dụng xử lý dữ liệu tạm cần IOPS cực cao, dữ liệu đã có bản gốc ở S3. Chọn gì rẻ và nhanh nhất?
- A. io2 Block Express
- B. Instance store
- C. EFS Provisioned
- D. gp3 tối đa IOPS

<details><summary>Đáp án</summary>

**B.** Gắn vật lý, đã gồm trong giá instance; mất dữ liệu cũng không sao.
- A, D sai: tốn thêm chi phí, độ trễ mạng.
- C sai: file system mạng.
</details>

**Câu 8.** Ba ứng dụng dùng chung một EFS nhưng mỗi ứng dụng chỉ được thấy thư mục của mình với UID/GID riêng. Chọn gì?
- A. Ba file system riêng
- B. EFS Access Points
- C. Security group riêng cho mỗi app
- D. Max I/O

<details><summary>Đáp án</summary>

**B.** Access Point ép POSIX identity và root directory.
- A sai: tốn kém, không cần thiết.
- C sai: SG kiểm soát mạng, không kiểm soát thư mục.
- D sai: không liên quan phân quyền.
</details>

---

### Nguồn đã đối chiếu
- EBS volume types (gp3 80k IOPS / 2,000 MiB/s / 64 TiB, io2 BE 256k): https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- Modify volume (4 lần trong 24 giờ trượt): https://docs.aws.amazon.com/ebs/latest/userguide/ebs-modify-volume.html
- Snapshot Archive (tới 75%, ≥ 90 ngày): https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-archive.html
- Recycle Bin (volume 1–7 ngày, snapshot/AMI 1–365 ngày): https://docs.aws.amazon.com/ebs/latest/userguide/recycle-bin-concepts.html
- Fast Snapshot Restore: https://docs.aws.amazon.com/ebs/latest/userguide/ebs-fast-snapshot-restore.html
- EBS Multi-Attach: https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html
- EFS performance (khuyên General Purpose): https://docs.aws.amazon.com/efs/latest/ug/performance.html
- EFS storage classes: https://docs.aws.amazon.com/efs/latest/ug/features.html
- EFS access points: https://docs.aws.amazon.com/efs/latest/ug/efs-access-points.html
