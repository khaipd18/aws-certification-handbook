# SAA-C03 — EC2 Storage: EBS, AMI, Instance Store & EFS

> Thuộc **Domain 3: Design High-Performing Architectures** và **Domain 4: Design Cost-Optimized Architectures**: chọn loại volume, snapshot, AMI, ổ local và file system dùng chung.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **EBS** là ổ mạng gắn vào **một instance trong cùng AZ** (trừ Multi-Attach); muốn chuyển AZ thì **snapshot** rồi restore.
- Loại EBS: **gp3/gp2** (SSD đa dụng), **io2 Block Express/io1** (SSD IOPS cao, Multi-Attach), **st1** (HDD throughput), **sc1** (HDD lạnh, rẻ nhất). Chỉ SSD làm boot volume được.
- ⚠️ Lỗi thời: "gp3 tối đa 16,000 IOPS / 1,000 MiB/s / 16 TiB" → nay **80,000 IOPS / 2,000 MiB/s / 64 TiB**. io2 Block Express: 256,000 IOPS, 4,000 MiB/s, 64 TiB.
- Snapshot: **Archive tier** (rẻ hơn ~75%, restore 24–72 giờ), **Recycle Bin** (chống xóa nhầm), **Fast Snapshot Restore** (không bị trễ lần đọc đầu).
- **AMI** gắn với Region, copy được sang Region khác; dùng để launch nhanh, golden image.
- **Instance store**: ổ vật lý gắn host, IOPS rất cao, **mất dữ liệu khi stop/terminate** (ephemeral).
- **EFS**: NFS managed, **chỉ Linux (POSIX)**, nhiều AZ, hàng nghìn instance mount cùng lúc, tự co giãn, trả theo dung lượng dùng.
- EFS có storage class **Standard / Infrequent Access / Archive** (+ One Zone), throughput **Elastic / Provisioned / Bursting**.

---

## 2. Core Concepts

### EBS volume
- **Là gì:** ổ lưu trữ mạng, giữ dữ liệu khi instance terminate (nếu tắt Delete on Termination), dung lượng và IOPS cấu hình trước, đổi được về sau.
- **Dùng khi nào:** root volume, database trên EC2, dữ liệu cần bền vững gắn với một instance.
- **Vì sao quan trọng:** khóa vào **một AZ**; mặc định root volume bị xóa khi terminate, volume phụ thì không.

### gp2 vs gp3
- **Là gì:** gp2: IOPS tăng theo dung lượng (3 IOPS/GiB, tối đa 16,000), burst tới 3,000; gp3: baseline 3,000 IOPS và 125 MiB/s, chỉnh IOPS và throughput độc lập.
- **Dùng khi nào:** boot volume, desktop ảo, dev/test, DB vừa và nhỏ.
- **Vì sao quan trọng:** "cần thêm IOPS mà không muốn tăng dung lượng" → gp3. ⚠️ Lỗi thời: giới hạn gp3 16k IOPS / 1,000 MiB/s → nay 80k / 2,000 MiB/s, tới 64 TiB.

### io1 / io2 Block Express & Multi-Attach
- **Là gì:** Provisioned IOPS SSD cho DB cần IOPS ổn định; io2 Block Express độ bền 99.999%, độ trễ dưới 1 ms, tới 256,000 IOPS (cần Nitro).
- **Dùng khi nào:** cần trên 80,000 IOPS hoặc 2,000 MiB/s, DB nhạy độ trễ; **Multi-Attach** gắn một volume io1/io2 vào tối đa 16 instance cùng AZ.
- **Vì sao quan trọng:** Multi-Attach đòi file system **cluster-aware**; không phải giải pháp share file thông thường (đó là EFS).

### st1 & sc1
- **Là gì:** HDD; st1 tối ưu throughput (big data, data warehouse, log), sc1 rẻ nhất cho dữ liệu ít truy cập; 125 GiB–16 TiB.
- **Dùng khi nào:** đọc tuần tự lớn, chi phí là ưu tiên.
- **Vì sao quan trọng:** **không làm boot volume** được.

### EBS Snapshots
- **Là gì:** bản sao lưu point-in-time, incremental, lưu ở hạ tầng S3; copy được sang AZ/Region khác.
- **Dùng khi nào:** **Archive tier** cho snapshot giữ lâu (rẻ hơn ~75%, restore 24–72 giờ); **Recycle Bin** giữ snapshot/AMI/volume bị xóa trong khoảng retention; **Fast Snapshot Restore** khởi tạo đầy đủ để không có độ trễ lần đầu (tốn phí).
- **Vì sao quan trọng:** "di chuyển EBS sang AZ khác" → snapshot → restore ở AZ đích; tự động hóa bằng Data Lifecycle Manager hoặc AWS Backup.

### EBS Encryption
- **Là gì:** mã hóa dữ liệu at rest, in-flight giữa instance và volume, snapshot và volume tạo từ snapshot; dùng KMS (AES-256), ảnh hưởng hiệu năng rất nhỏ.
- **Dùng khi nào:** mã hóa volume chưa mã hóa: snapshot → copy snapshot bật mã hóa → tạo volume mới.
- **Vì sao quan trọng:** bật **EBS encryption by default** theo Region để mọi volume mới tự mã hóa.

### AMI
- **Là gì:** image để launch EC2 (OS, phần mềm, cấu hình); loại public, tự tạo, hoặc từ AWS Marketplace; xây theo Region, copy được.
- **Dùng khi nào:** golden image để boot nhanh, chia sẻ cho account khác, chuẩn hóa cấu hình.
- **Vì sao quan trọng:** tạo AMI cũng tạo EBS snapshot; nên stop instance để dữ liệu nhất quán.

### Instance Store
- **Là gì:** ổ đĩa vật lý gắn trên host, I/O rất cao (hàng trăm nghìn tới hàng triệu IOPS).
- **Dùng khi nào:** buffer, cache, scratch data, dữ liệu tạm có bản sao ở nơi khác.
- **Vì sao quan trọng:** dữ liệu **mất khi stop/terminate** hoặc host lỗi; tự chịu trách nhiệm backup/replication.

### Amazon EFS
- **Là gì:** NFSv4.1 managed, chỉ Linux (POSIX), Multi-AZ, tự co giãn, trả theo GB dùng; kiểm soát bằng Security Group, mã hóa bằng KMS.
- **Dùng khi nào:** content management, web serving, home directory, dữ liệu dùng chung cho nhiều EC2/container/Lambda.
- **Vì sao quan trọng:** đắt hơn EBS gp2/gp3 theo GB nhưng chia sẻ nhiều AZ; **Windows → FSx for Windows**, không phải EFS.

### EFS performance & throughput mode
- **Là gì:** performance mode **General Purpose** (mặc định) và **Max I/O** (thế hệ trước); throughput mode **Elastic** (tự co giãn, trả theo dùng), **Provisioned** (đặt trước), **Bursting** (theo dung lượng).
- **Dùng khi nào:** tải khó đoán, đột biến → Elastic; throughput cao ổn định mà dung lượng nhỏ → Provisioned.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Max I/O cho big data, media processing" → AWS nay khuyên dùng **General Purpose** cho mọi file system; Max I/O không hỗ trợ One Zone và Elastic throughput.

### EFS storage classes & lifecycle
- **Là gì:** **Standard** (Multi-AZ), **Infrequent Access**, **Archive** (dữ liệu truy cập vài lần mỗi năm); **One Zone** và **One Zone-IA** lưu trong một AZ.
- **Dùng khi nào:** lifecycle policy tự chuyển file không truy cập sau N ngày sang IA/Archive và chuyển về Standard khi đọc lại.
- **Vì sao quan trọng:** One Zone rẻ hơn nhưng mất khi AZ lỗi (dev, dữ liệu có thể tạo lại). ❓ Con số "One Zone-IA tiết kiệm hơn 90%" trong slide là so với Standard; tỷ lệ thực tùy Region và thời điểm.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| gp3 | SSD đa dụng, IOPS/throughput độc lập dung lượng | "cost-effective SSD", "boot volume" |
| io2 Block Express | SSD IOPS cao nhất, 99.999% | "sub-millisecond latency", "> 80,000 IOPS" |
| Multi-Attach | Một io1/io2 gắn tối đa 16 instance cùng AZ | "shared block storage, cluster-aware" |
| st1 | HDD tối ưu throughput | "big data, log processing, sequential" |
| sc1 | HDD lạnh, rẻ nhất | "infrequently accessed, lowest cost" |
| EBS Snapshot Archive | Lưu snapshot rẻ, restore 24–72 giờ | "retain snapshots for years" |
| Recycle Bin | Khôi phục snapshot/AMI/volume bị xóa nhầm | "accidental deletion" |
| Fast Snapshot Restore | Volume từ snapshot đạt hiệu năng ngay | "no latency on first read" |
| AMI | Image để launch EC2, theo Region | "golden image", "faster boot" |
| Instance store | Ổ local tạm, I/O cao | "ephemeral", "highest IOPS", "cache" |
| EFS | NFS dùng chung, Multi-AZ, Linux | "shared file system across AZs" |
| EFS Elastic throughput | Throughput tự co giãn theo tải | "spiky, unpredictable throughput" |
| EFS IA / Archive | Class rẻ cho file ít truy cập | "lifecycle policy after 30 days" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| EBS | EFS | EBS: một instance, một AZ; EFS: nhiều instance, nhiều AZ | "Shared" → EFS |
| EBS | Instance Store | EBS bền khi stop; Instance store mất dữ liệu nhưng nhanh hơn | "Ephemeral, highest IOPS" → Instance store |
| gp3 | io2 Block Express | gp3 tới 80k IOPS, rẻ; io2 BE tới 256k, độ trễ thấp, Multi-Attach | > 80k IOPS → io2 |
| st1 | sc1 | st1 truy cập thường xuyên; sc1 lạnh, rẻ nhất | "Lowest cost HDD" → sc1 |
| EBS Multi-Attach | EFS | Multi-Attach: block, cùng AZ, cần cluster-aware FS; EFS: file NFS | Share file thông thường → EFS |
| EFS | FSx for Windows | EFS chỉ Linux NFS; FSx for Windows dùng SMB | Windows → FSx |
| Snapshot Archive | Recycle Bin | Archive giảm chi phí lưu lâu; Recycle Bin chống xóa nhầm | "Deleted by mistake" → Recycle Bin |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "move EBS volume to another AZ" | Snapshot → restore ở AZ mới | |
| "encrypt existing unencrypted volume" | Snapshot → copy có mã hóa → volume mới | |
| "more than 80,000 IOPS on one volume" | io2 Block Express | Nitro |
| "attach same volume to multiple instances" | io1/io2 Multi-Attach | Cùng AZ |
| "highest I/O, temporary data, cache" | Instance store | |
| "big data sequential reads, cheap" | st1 | Không boot |
| "shared POSIX file system across AZs" | Amazon EFS | |
| "files not accessed for 30 days, cut cost" | EFS lifecycle → IA/Archive | |
| "recover accidentally deleted snapshot" | Recycle Bin | |
| "launch fully configured instance fast" | Golden AMI | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Database trên EC2 cần 120,000 IOPS ổn định, độ trễ dưới 1 ms. Chọn volume nào?
- A. gp3
- B. io2 Block Express
- C. st1
- D. sc1

<details><summary>Đáp án</summary>

**B.** io2 Block Express tới 256,000 IOPS.
- A sai: gp3 tối đa 80,000 IOPS.
- C, D sai: HDD tối ưu throughput, IOPS thấp.
</details>

**Câu 2.** Nhiều EC2 Linux ở 3 AZ cần cùng đọc/ghi một thư mục chung. Chọn gì?
- A. EBS gp3
- B. Instance store
- C. Amazon EFS
- D. FSx for Windows File Server

<details><summary>Đáp án</summary>

**C.** EFS là NFS Multi-AZ, mount được từ nhiều instance.
- A sai: EBS gắn trong một AZ.
- B sai: ổ local, không chia sẻ.
- D sai: dành cho Windows/SMB.
</details>

**Câu 3.** Cần di chuyển một EBS volume từ `us-east-1a` sang `us-east-1b`. Cách làm?
- A. Detach rồi attach sang instance ở 1b
- B. Tạo snapshot rồi tạo volume từ snapshot ở 1b
- C. Bật Multi-Attach
- D. Đổi subnet của volume

<details><summary>Đáp án</summary>

**B.** Snapshot không gắn AZ, restore được ở AZ khác.
- A sai: volume chỉ gắn được trong cùng AZ.
- C sai: Multi-Attach vẫn trong một AZ.
- D sai: volume không có subnet.
</details>

**Câu 4.** Ứng dụng cần vùng đệm tạm với IOPS cực cao; dữ liệu đã được replicate ở nơi khác. Chọn gì rẻ và nhanh nhất?
- A. io2 Block Express
- B. Instance store
- C. EFS Provisioned throughput
- D. S3

<details><summary>Đáp án</summary>

**B.** Instance store nhanh nhất, phí đã gồm trong giá instance; mất dữ liệu cũng không sao.
- A sai: đắt.
- C sai: file system mạng, độ trễ cao hơn.
- D sai: object storage, không phải ổ local.
</details>

**Câu 5.** (Chọn 2) Công ty phải giữ snapshot EBS 7 năm cho compliance, ít khi restore, và muốn tránh xóa nhầm. Chọn gì?
- A. EBS Snapshot Archive
- B. Fast Snapshot Restore
- C. Recycle Bin với retention rule
- D. Instance store
- E. Multi-Attach

<details><summary>Đáp án</summary>

**A, C.** Archive giảm chi phí lưu lâu; Recycle Bin khôi phục được snapshot bị xóa.
- B sai: tăng tốc restore, tốn thêm tiền.
- D, E sai: không liên quan snapshot.
</details>

**Câu 6.** EFS chứa log, phần lớn file không được đọc sau 30 ngày nhưng thỉnh thoảng cần. Cách giảm chi phí?
- A. Chuyển sang EBS sc1
- B. Bật lifecycle policy chuyển sang EFS Infrequent Access (và Archive)
- C. Đổi performance mode sang Max I/O
- D. Dùng Provisioned throughput

<details><summary>Đáp án</summary>

**B.** Lifecycle tự chuyển file ít dùng sang class rẻ, đọc lại vẫn được.
- A sai: mất khả năng chia sẻ.
- C sai: performance mode không giảm chi phí lưu; Max I/O nay không được khuyến nghị.
- D sai: tăng chi phí.
</details>

**Câu 7.** Volume EBS hiện chưa mã hóa. Làm sao có bản mã hóa?
- A. Bật mã hóa trực tiếp trên volume đang gắn
- B. Snapshot → copy snapshot bật mã hóa → tạo volume mới và gắn vào instance
- C. Dùng S3 SSE-S3
- D. Bật Multi-Attach

<details><summary>Đáp án</summary>

**B.**
- A sai: không bật mã hóa tại chỗ cho volume có sẵn.
- C sai: SSE-S3 dành cho S3 object.
- D sai: không liên quan mã hóa.
</details>

**Câu 8.** Workload EFS có throughput tăng đột biến khó đoán, dung lượng lưu nhỏ. Throughput mode nào phù hợp?
- A. Bursting
- B. Elastic
- C. Max I/O
- D. One Zone

<details><summary>Đáp án</summary>

**B.** Elastic tự co giãn theo tải, trả theo dùng.
- A sai: Bursting phụ thuộc dung lượng, dung lượng nhỏ thì credit ít.
- C sai: là performance mode, không phải throughput mode.
- D sai: là storage class.
</details>

---

### Nguồn đã đối chiếu
- Amazon EBS volume types (gp3 80k IOPS/2,000 MiB/s/64 TiB, io2 BE 256k IOPS, HDD không boot): https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- EBS Multi-Attach: https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html
- EBS Snapshots Archive: https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-archive.html
- Recycle Bin: https://docs.aws.amazon.com/ebs/latest/userguide/recycle-bin.html
- Amazon EFS performance (khuyến nghị General Purpose, throughput mode): https://docs.aws.amazon.com/efs/latest/ug/performance.html
- EFS storage classes: https://docs.aws.amazon.com/efs/latest/ug/storage-classes.html
- Instance store: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html
