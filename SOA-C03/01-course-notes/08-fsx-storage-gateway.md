# SOA-C03 — Advanced Storage: Amazon FSx & AWS Storage Gateway

> Thuộc **Domain 2: Reliability and Business Continuity (22%)** và **Domain 5: Networking and Content Delivery (18%)**: chọn loại FSx theo giao thức, độ bền và backup của FSx, các loại Storage Gateway, kích hoạt và theo dõi cache.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **FSx** có bốn loại: **Windows File Server** (SMB, NTFS, AD), **Lustre** (HPC/ML), **NetApp ONTAP** (NFS + SMB + iSCSI), **OpenZFS** (chỉ NFS).
- Chọn theo giao thức: SMB + AD → Windows; chỉ NFS → OpenZFS; cần cả ba giao thức → ONTAP; POSIX song song, throughput cực lớn → Lustre.
- **FSx for Lustre** liên kết trực tiếp với **S3**: đọc bucket như file system, ghi kết quả về S3.
- **Lustre Scratch** không replicate; **Persistent** replicate trong một AZ. ⚠️ Không backup được file system scratch hoặc file system liên kết S3.
- **FSx for Windows Multi-AZ** replicate đồng bộ sang standby ở AZ khác, failover tự động; Single-AZ chỉ trong một AZ.
- **Storage Gateway** nối on-prem với AWS:
  - **S3 File Gateway**: NFS/SMB.
  - **Volume Gateway**: iSCSI, cached hoặc stored.
  - **Tape Gateway**: VTL.
  - ⚠️ **FSx File Gateway** không còn nhận khách mới.
- Hai sự thật khi xử lý sự cố gateway:
  - Kích hoạt cần mở **port 80** tới gateway VM và đồng bộ giờ **NTP**.
  - Sức khỏe cache xem qua `CacheHitPercent` (nên cao) và `CachePercentUsed` (không nên quá cao).

---

## 2. Core Concepts

### FSx for Windows File Server
- **Là gì:** share SMB trên NTFS, tích hợp Active Directory, ACL, user quota, DFS Namespaces; Linux EC2 cũng mount được.
- **Dùng khi nào:** file share Windows, home directory, ứng dụng .NET/SQL Server cần SMB; truy cập từ on-prem qua VPN/Direct Connect.
- **Vì sao quan trọng:** SSD cho tải nhạy độ trễ, HDD cho share thông thường. Backup tự động hằng ngày lên S3, giữ theo retention.

### FSx Windows: Single-AZ vs Multi-AZ
- **Là gì:** Single-AZ replicate trong một AZ; Multi-AZ replicate **đồng bộ** sang file server standby ở AZ khác.
- **Dùng khi nào:** phải chịu được mất cả AZ → Multi-AZ.
- **Vì sao quan trọng:** Single-AZ chỉ chống lỗi phần cứng, không chống mất AZ.

### FSx for Lustre
- **Là gì:** file system song song cho HPC, ML, xử lý video, mô hình tài chính; hàng trăm GB/s, độ trễ dưới mili giây.
- **Dùng khi nào:** dữ liệu gốc trong S3, cần xử lý tốc độ cao rồi ghi kết quả về S3; SSD cho random nhỏ, HDD cho tuần tự lớn.
- **Vì sao quan trọng:** **Scratch** rẻ, không replicate, mất dữ liệu khi file server lỗi; **Persistent** replicate trong AZ, tự thay file server lỗi. Cả hai không Multi-AZ, nên giữ bản gốc ở S3.

### FSx for NetApp ONTAP & OpenZFS
- **Là gì:** ONTAP hỗ trợ đồng thời NFS, SMB, iSCSI, tự co giãn dung lượng, nén, khử trùng lặp, clone tức thời. OpenZFS chỉ NFS (v3–v4.2), tới 1 triệu IOPS, độ trễ dưới 0.5 ms.
- **Dùng khi nào:** chuyển NAS NetApp lên AWS hoặc cần nhiều giao thức → ONTAP; chuyển ZFS/NFS Linux → OpenZFS.
- **Vì sao quan trọng:** đề nhắc SMB hoặc block thì loại OpenZFS; clone môi trường test tức thời → ONTAP/OpenZFS.

### FSx backup
- **Là gì:** backup tự động hằng ngày + backup thủ công, lưu bền trên S3; quản lý tập trung được bằng AWS Backup.
- **Dùng khi nào:** cần policy backup chung với EBS, RDS, EFS, DynamoDB → AWS Backup.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Lustre persistent và scratch đều backup được" → docs: **không backup được scratch, cũng không backup được file system liên kết với S3 data repository**.

### AWS Storage Gateway
- **Là gì:** cầu nối cho ứng dụng on-prem dùng NFS, SMB, iSCSI tới S3/EBS/Glacier; có local cache, mã hóa in transit.
- **Dùng khi nào:** DR, backup, tiered storage, cache độ trễ thấp cho dữ liệu cloud.
- **Vì sao quan trọng:** chạy dạng VM (ESXi, Hyper-V, KVM) hoặc EC2. ⚠️ Lỗi thời: Hardware Appliance **ngừng bán cho khách mới từ 12/05/2025** (hỗ trợ khách cũ tới 05/2028).

### S3 File Gateway
- **Là gì:** mount bucket S3 qua NFS/SMB, cache dữ liệu dùng gần đây; metadata POSIX lưu trong metadata của object.
- **Dùng khi nào:** server on-prem ghi file, dữ liệu nằm trong S3 để phân tích hoặc lưu trữ.
- **Vì sao quan trọng:** mỗi gateway dùng một IAM role để vào bucket; SMB tích hợp AD; muốn xuống Glacier phải qua **lifecycle policy**.

### Volume Gateway
- **Là gì:** block storage iSCSI, backup thành EBS snapshot. **Cached**: dữ liệu chính trên S3, cache phần hay dùng (tối đa 32 TiB mỗi volume). **Stored**: toàn bộ dữ liệu on-prem, backup async lên S3 (tối đa 16 TiB mỗi volume).
- **Dùng khi nào:** giảm dung lượng on-prem → cached; cần độ trễ thấp cho toàn bộ dữ liệu → stored.
- **Vì sao quan trọng:** tối đa 32 volume mỗi gateway; tổng 1,024 TiB (cached) hoặc 512 TiB (stored) mỗi gateway.

### Tape Gateway
- **Là gì:** Virtual Tape Library qua iSCSI; tape ảo trong S3, eject thì archive sang Glacier / Glacier Deep Archive.
- **Dùng khi nào:** giữ nguyên phần mềm và quy trình backup bằng tape, bỏ thư viện tape vật lý.
- **Vì sao quan trọng:** từ khóa "existing tape backup software" gần như luôn là Tape Gateway.

### Vận hành gateway: kích hoạt, reboot, cache
- **Là gì:** activation key lấy qua CLI của VM hoặc request HTTP tới VM ở **port 80**; giờ VM phải đồng bộ NTP. File Gateway reboot VM là xong; Volume/Tape Gateway phải **stop service → reboot → start service**.
- **Dùng khi nào:** `CacheHitPercent` thấp → cache nhỏ; `CachePercentUsed` cao → thêm cache disk hoặc clone sang volume lớn hơn.
- **Vì sao quan trọng:** `[+]` `CachePercentDirty` cao kéo dài nghĩa là đường uplink không kịp đẩy dữ liệu → tăng băng thông/buffer, kiểm tra bandwidth rate limit.

### `[+]` Storage Gateway vs DataSync
- **Là gì:** Storage Gateway là truy cập lai **liên tục** qua cache; DataSync là **chuyển dữ liệu một lần hoặc theo lịch** giữa on-prem và S3/EFS/FSx, có kiểm tra toàn vẹn.
- **Dùng khi nào:** "migrate 200 TB một lần" → DataSync; "server on-prem cần truy cập dữ liệu cloud độ trễ thấp" → Storage Gateway.
- **Vì sao quan trọng:** hai đáp án này hay xuất hiện cạnh nhau làm distractor.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| FSx for Windows | SMB + AD + NTFS | "Windows file share", "DFS" |
| FSx for Lustre | File system HPC liên kết S3 | "HPC", "ML training on S3 data" |
| Lustre Scratch | Tạm, không replicate | "short-term processing, lowest cost" |
| Lustre Persistent | Replicate trong AZ | "long-running, sensitive data" |
| FSx for ONTAP | NFS + SMB + iSCSI | "multi-protocol", "NetApp migration" |
| FSx for OpenZFS | NFS, độ trễ thấp | "ZFS migration", "NFS only" |
| S3 File Gateway | NFS/SMB lên S3 | "on-prem files into S3" |
| Volume Gateway cached | Dữ liệu chính trên S3 | "reduce on-prem storage" |
| Volume Gateway stored | Toàn bộ dữ liệu on-prem | "low latency to entire dataset" |
| Tape Gateway | VTL iSCSI | "replace physical tapes" |
| `CacheHitPercent` | Tỷ lệ đọc trúng cache | "slow reads from gateway" |
| `CachePercentDirty` | Dữ liệu chưa đẩy lên AWS | "uploads falling behind" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| FSx Windows | EFS | SMB/Windows/AD; NFS/Linux | Windows → FSx |
| FSx Lustre | EFS | HPC song song, liên kết S3; NFS dùng chung đa dụng | HPC + S3 → Lustre |
| ONTAP | OpenZFS | Đa giao thức; chỉ NFS | Có SMB/iSCSI → ONTAP |
| Lustre Scratch | Lustre Persistent | Không replicate, không backup; replicate trong AZ | Dữ liệu cần giữ → Persistent |
| FSx Windows Single-AZ | Multi-AZ | Một AZ; standby đồng bộ ở AZ khác | Chịu mất AZ → Multi-AZ |
| Volume cached | Volume stored | Dữ liệu chính trên S3; trên on-prem | "All data local" → stored |
| S3 File Gateway | Volume Gateway | File NFS/SMB; block iSCSI | Ứng dụng cần ổ đĩa → Volume |
| Storage Gateway | DataSync | Truy cập liên tục; chuyển bulk | "One-time migration" → DataSync |
| Reboot File Gateway | Reboot Volume/Tape Gateway | Chỉ restart VM; stop service trước | Volume/Tape → stop service |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "Windows shares with Active Directory" | FSx for Windows | |
| "survive AZ failure for Windows file share" | FSx Windows Multi-AZ | |
| "HPC on data in S3, write results back" | FSx for Lustre | |
| "temporary HPC job, lowest cost" | Lustre Scratch | Không backup được |
| "NFS + SMB + iSCSI" | FSx for NetApp ONTAP | |
| "migrate ZFS NFS workload" | FSx for OpenZFS | |
| "on-prem app writes files that land in S3" | S3 File Gateway | |
| "iSCSI, keep entire dataset on-prem, backup to AWS" | Volume Gateway stored | |
| "replace physical tape library" | Tape Gateway | |
| "gateway activation fails" | Port 80 tới VM, giờ NTP | |
| "low cache hit rate" | Tăng cache disk | |
| "move 200 TB once" | DataSync | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Hàng nghìn node HPC cần đọc dataset trong S3 với throughput hàng trăm GB/s rồi ghi kết quả về S3. Chọn gì?
- A. EFS Elastic throughput
- B. FSx for Lustre liên kết S3
- C. FSx for Windows
- D. S3 File Gateway

<details><summary>Đáp án</summary>

**B.**
- A sai: không có liên kết S3 data repository, throughput thấp hơn.
- C sai: SMB cho Windows.
- D sai: dành cho on-prem, không phải HPC trên AWS.
</details>

**Câu 2.** Công ty cần share SMB cho người dùng Windows, tích hợp AD, phải chịu được mất cả AZ. Chọn gì?
- A. FSx for Windows Single-AZ
- B. FSx for Windows Multi-AZ
- C. EFS Standard
- D. FSx for OpenZFS

<details><summary>Đáp án</summary>

**B.**
- A sai: chỉ replicate trong một AZ.
- C sai: NFS, không SMB.
- D sai: chỉ NFS.
</details>

**Câu 3.** (Chọn 2) Workload cần cả NFS cho Linux và SMB cho Windows trên cùng dữ liệu, đồng thời clone tức thời môi trường test. Hai phát biểu nào đúng?
- A. FSx for NetApp ONTAP hỗ trợ NFS, SMB, iSCSI đồng thời
- B. FSx for OpenZFS hỗ trợ SMB
- C. ONTAP có point-in-time cloning
- D. FSx for Lustre hỗ trợ SMB
- E. EFS hỗ trợ SMB

<details><summary>Đáp án</summary>

**A, C.**
- B sai: OpenZFS chỉ NFS.
- D sai: Lustre dùng client Lustre.
- E sai: EFS chỉ NFS.
</details>

**Câu 4.** Đội cần backup FSx for Lustre bằng AWS Backup nhưng thất bại. File system là loại scratch. Vì sao?
- A. AWS Backup không hỗ trợ FSx
- B. Scratch file system không backup được
- C. Thiếu KMS key
- D. Phải bật Multi-AZ

<details><summary>Đáp án</summary>

**B.** Docs: không backup được scratch, cũng không backup được file system liên kết S3.
- A sai: AWS Backup có hỗ trợ FSx.
- C sai: không phải nguyên nhân.
- D sai: Lustre không có Multi-AZ.
</details>

**Câu 5.** Server on-prem dùng iSCSI cần độ trễ thấp cho **toàn bộ** dữ liệu, đồng thời backup lên AWS. Chọn gì?
- A. Volume Gateway cached
- B. Volume Gateway stored
- C. S3 File Gateway
- D. Tape Gateway

<details><summary>Đáp án</summary>

**B.**
- A sai: chỉ cache phần hay dùng, phần còn lại ở S3.
- C sai: giao thức file, không phải block.
- D sai: dành cho backup bằng tape.
</details>

**Câu 6.** Kích hoạt Storage Gateway mới trên VMware thất bại. (Chọn 2) hai nguyên nhân thường gặp nhất?
- A. Port 80 tới gateway VM bị chặn
- B. Đồng hồ VM sai, chưa đồng bộ NTP
- C. Chưa bật S3 Versioning
- D. Thiếu Elastic IP
- E. Chưa tạo CloudFront distribution

<details><summary>Đáp án</summary>

**A, B.**
- C sai: không liên quan kích hoạt.
- D sai: gateway on-prem không dùng Elastic IP.
- E sai: không liên quan.
</details>

**Câu 7.** Người dùng than đọc file qua Volume Gateway cached chậm. `CacheHitPercent` thấp, `CachePercentUsed` gần 100%. Làm gì?
- A. Tăng dung lượng cache disk
- B. Chuyển sang Tape Gateway
- C. Bật S3 Transfer Acceleration
- D. Giảm upload buffer

<details><summary>Đáp án</summary>

**A.** Cache quá nhỏ nên phải đọc từ S3 liên tục.
- B sai: Tape Gateway không dùng cho truy cập trực tuyến.
- C sai: không áp cho Storage Gateway.
- D sai: làm tình hình xấu hơn.
</details>

**Câu 8.** Công ty muốn bỏ thư viện tape vật lý nhưng giữ nguyên phần mềm backup hiện có; tape cũ đưa vào lưu trữ rẻ nhất. Chọn gì?
- A. S3 File Gateway + lifecycle
- B. Tape Gateway, archive sang Glacier Deep Archive
- C. DataSync
- D. Volume Gateway stored

<details><summary>Đáp án</summary>

**B.**
- A sai: phần mềm backup tape không ghi qua NFS/SMB theo kiểu tape.
- C sai: công cụ chuyển dữ liệu, không giả lập tape.
- D sai: block storage.
</details>

---

### Nguồn đã đối chiếu
- FSx for Lustre backups (không backup scratch và file system liên kết S3): https://docs.aws.amazon.com/fsx/latest/LustreGuide/using-backups-fsx.html
- Volume Gateway quotas: https://docs.aws.amazon.com/storagegateway/latest/vgw/resource-gateway-limits.html
- FSx File Gateway (không còn nhận khách mới): https://docs.aws.amazon.com/filegateway/latest/filefsxw/use-fsxw-gateway.html
- Storage Gateway Hardware Appliance end of availability: https://docs.aws.amazon.com/filegateway/latest/files3/hardware-appliance.html
- FSx for Windows availability: https://docs.aws.amazon.com/fsx/latest/WindowsGuide/high-availability-multiAZ.html
- FSx for ONTAP: https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/what-is-fsx-ontap.html
