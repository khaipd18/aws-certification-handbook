# SAA-C03 — Other Storage: Snow Family, FSx, Storage Gateway, Transfer Family & DataSync

> Thuộc **Domain 2: Design Resilient Architectures**, **Domain 3: Design High-Performing Architectures** và **Domain 4: Design Cost-Optimized Architectures**: chuyển dữ liệu lớn, file system managed của bên thứ ba, lưu trữ lai on-premises ↔ AWS.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- ⚠️ Lỗi thời: **Snow Family** (Snowball Edge, Snowcone, Snowmobile) → AWS **không nhận khách mới** đặt thiết bị Snow; thay bằng **DataSync** (online), **AWS Data Transfer Terminal** (mang ổ tới cơ sở của AWS), hoặc giải pháp partner; edge compute → **Outposts**.
- Quy tắc cũ vẫn đúng về tư duy: chuyển qua mạng mất **> 1 tuần** → cân nhắc chuyển vật lý.
- **FSx for Windows File Server**: SMB, NTFS, Active Directory, DFS, Multi-AZ. **FSx for Lustre**: HPC/ML, tích hợp S3, scratch vs persistent. **FSx for NetApp ONTAP**: NFS + SMB + iSCSI, snapshot, clone tức thì. **FSx for OpenZFS**: NFS, snapshot, clone, hiệu năng cao.
- **Storage Gateway** nối on-premises với lưu trữ AWS: **S3 File Gateway** (NFS/SMB → S3), **Volume Gateway** (iSCSI, cached/stored, backup bằng EBS snapshot), **Tape Gateway** (VTL → S3/Glacier).
- ⚠️ Lỗi thời: **FSx File Gateway** đóng với khách mới; **Storage Gateway Hardware Appliance** ngừng bán cho khách mới từ 12/05/2025.
- **Transfer Family**: FTP/FTPS/SFTP/AS2 managed vào S3 hoặc EFS. **DataSync**: đồng bộ/migrate lớn giữa NFS/SMB/HDFS/object storage/cloud khác ↔ S3/EFS/FSx, **giữ metadata và quyền file**, chạy theo lịch.

---

## 2. Core Concepts

### Snow Family (bối cảnh đề cũ)
- **Là gì:** thiết bị vật lý di chuyển dữ liệu offline (Snowball Edge Storage Optimized/Compute Optimized, Snowcone, Snowmobile) và edge computing không cần mạng.
- **Dùng khi nào:** đề cũ: "hàng trăm TB–PB, băng thông hạn chế, chuyển mất hàng tuần" → Snowball; "exabyte" → Snowmobile; import vào Glacier phải qua S3 rồi lifecycle.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: AWS không còn nhận đơn thiết bị Snow từ khách mới. ❓ Snowmobile còn xuất hiện trong whitepaper nhưng thuộc Snow Family nên khả năng không còn cho khách mới. Đề mới nghiêng về DataSync, Direct Connect, Data Transfer Terminal.

### AWS Data Transfer Terminal
- **Là gì:** cơ sở vật lý của AWS, khách mang thiết bị lưu trữ tới và upload tốc độ cao vào AWS.
- **Dùng khi nào:** khách mới cần chuyển offline lượng dữ liệu lớn thay cho Snowball.
- **Vì sao quan trọng:** là phương án thay thế AWS nêu trực tiếp trong thông báo ngừng Snowball Edge.

### FSx for Windows File Server
- **Là gì:** file share Windows managed: **SMB**, **NTFS**, tích hợp **Microsoft AD**, ACL, quota, **DFS Namespaces**; mount được từ Linux; SSD/HDD; Multi-AZ; backup hằng ngày lên S3.
- **Dùng khi nào:** ứng dụng Windows, SharePoint, home directory, file share doanh nghiệp.
- **Vì sao quan trọng:** "Windows + SMB + AD" → FSx for Windows, **không phải EFS** (EFS chỉ NFS/Linux).

### FSx for Lustre
- **Là gì:** file system song song (Linux + cluster) cho HPC, ML, video processing, financial modeling; hàng trăm GB/s, hàng triệu IOPS, độ trễ dưới mili giây.
- **Dùng khi nào:** đọc S3 như file system, ghi kết quả ngược về S3 (data repository); **Scratch** (tạm, không replicate, rẻ, burst cao) vs **Persistent** (lâu dài, replicate trong AZ).
- **Vì sao quan trọng:** từ khóa "HPC", "parallel", "ML training trên dữ liệu S3" → Lustre.

### FSx for NetApp ONTAP & OpenZFS
- **Là gì:** **ONTAP**: NFS, SMB, iSCSI; tương thích Linux, Windows, macOS, VMware Cloud on AWS, WorkSpaces, EC2, ECS/EKS; dedup, nén, snapshot, **clone tức thì**. **OpenZFS**: NFS (v3–v4.2), tới hàng triệu IOPS, snapshot, clone tức thì.
- **Dùng khi nào:** chuyển NetApp/ZFS on-premises lên AWS giữ nguyên tính năng; cần nhiều giao thức cùng lúc → ONTAP.
- **Vì sao quan trọng:** "multi-protocol NFS + SMB + iSCSI" → ONTAP; "migrate ZFS" → OpenZFS.

### Storage Gateway — S3 File Gateway
- **Là gì:** gateway (VM on-prem/EC2) cung cấp NFS/SMB share, file lưu thành object trong S3 (Standard, Standard-IA, One Zone-IA, Intelligent-Tiering); cache dữ liệu dùng gần đây.
- **Dùng khi nào:** on-premises cần file share hậu thuẫn bởi S3, có lifecycle sang Glacier; SMB tích hợp AD.
- **Vì sao quan trọng:** truy cập bằng IAM role của gateway; dữ liệu trong S3 dùng được cho analytics.

### Storage Gateway — Volume & Tape Gateway
- **Là gì:** **Volume Gateway**: block storage iSCSI backup bằng EBS snapshot; **cached** (dữ liệu chính trên S3, cache local) hoặc **stored** (dữ liệu chính local, backup bất đồng bộ lên S3). **Tape Gateway**: thư viện băng ảo (VTL) cho phần mềm backup hiện có, lưu vào S3/Glacier.
- **Dùng khi nào:** cached → giảm dung lượng on-prem, độ trễ thấp cho dữ liệu nóng; stored → cần toàn bộ dữ liệu local độ trễ thấp; Tape → thay băng từ vật lý.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: **FSx File Gateway** (truy cập FSx for Windows từ on-prem với cache) đóng với khách mới; Hardware Appliance ngừng bán từ 12/05/2025 (hỗ trợ khách cũ tới 05/2028).

### AWS Transfer Family
- **Là gì:** dịch vụ FTP, FTPS, SFTP, AS2 managed, đọc/ghi vào **S3 hoặc EFS**; trả theo giờ endpoint + GB truyền.
- **Dùng khi nào:** đối tác chỉ gửi file qua SFTP; xác thực qua dịch vụ có sẵn (AD, LDAP, Okta, Cognito, custom).
- **Vì sao quan trọng:** "SFTP vào S3, không quản lý server" → Transfer Family.

### AWS DataSync
- **Là gì:** dịch vụ di chuyển dữ liệu online: NFS, SMB, HDFS, object storage tự quản, cloud khác ↔ S3 (mọi class kể cả Glacier), EFS, FSx; cần **agent** cho nguồn on-prem; chạy theo lịch (không liên tục).
- **Dùng khi nào:** migrate hoặc đồng bộ định kỳ dữ liệu lớn; giữ **permission và metadata** (POSIX, SMB); một task dùng được ~10 Gbps, giới hạn băng thông được.
- **Vì sao quan trọng:** đề phân biệt DataSync (copy/sync dữ liệu) với Storage Gateway (truy cập lai liên tục).

### `[+]` So sánh lựa chọn lưu trữ
- **Là gì:** S3 (object), Glacier (archive), EBS (block 1 instance), Instance Store (block local), EFS (NFS Linux), FSx Windows/Lustre/ONTAP/OpenZFS, Storage Gateway (lai), Transfer Family, DataSync.
- **Dùng khi nào:** đọc từ khóa giao thức: SMB → FSx Windows; NFS Linux → EFS; iSCSI → Volume Gateway/ONTAP; HPC → Lustre.
- **Vì sao quan trọng:** đây là bảng tổng kết SAA hay hỏi dưới dạng "chọn storage phù hợp".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Snowball Edge | Thiết bị chuyển dữ liệu offline (không còn cho khách mới) | "petabytes, limited bandwidth" |
| Data Transfer Terminal | Cơ sở AWS để upload ổ cứng tốc độ cao | "physical transfer, new customers" |
| FSx for Windows | SMB, NTFS, AD, DFS | "Windows file share" |
| FSx for Lustre | File system song song cho HPC/ML | "HPC", "S3 data repository" |
| Scratch vs Persistent | Lustre tạm vs lâu dài | "short-term processing" |
| FSx for NetApp ONTAP | NFS + SMB + iSCSI | "multi-protocol", "NetApp" |
| FSx for OpenZFS | NFS, snapshot, clone | "migrate ZFS" |
| S3 File Gateway | NFS/SMB → S3 | "on-prem file share backed by S3" |
| Volume Gateway cached | Dữ liệu chính ở S3, cache local | "minimize on-prem storage" |
| Volume Gateway stored | Dữ liệu chính local, backup S3 | "full dataset local, low latency" |
| Tape Gateway | VTL vào S3/Glacier | "replace physical tapes" |
| Transfer Family | FTP/FTPS/SFTP/AS2 managed | "partners upload via SFTP" |
| DataSync | Migrate/sync dữ liệu online | "preserve metadata and permissions" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| EFS | FSx for Windows | EFS: NFS, Linux; FSx Windows: SMB, AD | Windows → FSx |
| FSx for Lustre | EFS | Lustre: song song, HPC, S3; EFS: file share đa dụng | HPC → Lustre |
| Lustre Scratch | Lustre Persistent | Scratch tạm, không replicate; Persistent lâu dài, replicate trong AZ | Ngắn hạn → Scratch |
| DataSync | Storage Gateway | DataSync copy/sync theo lịch; Gateway truy cập lai liên tục | Migrate → DataSync |
| Volume cached | Volume stored | Cached: dữ liệu chính trên S3; Stored: dữ liệu chính local | "Entire dataset on-prem" → stored |
| Transfer Family | DataSync | Transfer Family: giao thức FTP cho user/đối tác; DataSync: migrate hệ thống | SFTP → Transfer Family |
| Snowball | DataSync | Snowball offline (không còn cho khách mới); DataSync online | Khách mới → DataSync/Terminal |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "Windows file shares, SMB, Active Directory" | FSx for Windows File Server | |
| "HPC, ML, process S3 data as file system" | FSx for Lustre | |
| "NFS + SMB + iSCSI from one system" | FSx for NetApp ONTAP | |
| "on-prem apps need NFS share stored in S3" | S3 File Gateway | |
| "iSCSI, keep frequently accessed data local, rest in S3" | Volume Gateway cached | |
| "replace tape backups, keep backup software" | Tape Gateway | |
| "partners upload files via SFTP into S3" | AWS Transfer Family | |
| "migrate NFS to EFS preserving permissions, scheduled" | AWS DataSync | |
| "hundreds of TB, slow link, offline transfer (new customer)" | Data Transfer Terminal / partner | Snowball không còn cho khách mới |
| "edge compute disconnected (new customer)" | AWS Outposts | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Công ty có ứng dụng Windows dùng file share SMB tích hợp Active Directory, muốn chuyển lên AWS dạng managed. Chọn gì?
- A. Amazon EFS
- B. FSx for Windows File Server
- C. FSx for Lustre
- D. S3 File Gateway

<details><summary>Đáp án</summary>

**B.**
- A sai: EFS chỉ NFS, không tích hợp AD kiểu Windows.
- C sai: Lustre cho HPC Linux.
- D sai: gateway on-prem, không phải file share managed trên AWS.
</details>

**Câu 2.** Cụm HPC cần đọc dữ liệu từ S3 với throughput hàng trăm GB/s, chạy job 2 ngày rồi xóa. Chọn gì rẻ nhất?
- A. FSx for Lustre Scratch liên kết S3
- B. FSx for Lustre Persistent
- C. EFS Provisioned throughput
- D. EBS io2

<details><summary>Đáp án</summary>

**A.** Scratch rẻ, phù hợp xử lý tạm thời.
- B sai: Persistent đắt hơn, dành cho lưu lâu dài.
- C sai: không đạt hiệu năng song song như Lustre.
- D sai: block gắn một instance (hoặc vài instance cùng AZ).
</details>

**Câu 3.** On-premises cần file share NFS mà dữ liệu được lưu trong S3 để sau đó dùng Athena phân tích. Chọn gì?
- A. Volume Gateway stored
- B. S3 File Gateway
- C. Tape Gateway
- D. FSx for OpenZFS

<details><summary>Đáp án</summary>

**B.** File lưu thành object S3 nguyên dạng.
- A sai: iSCSI block, dữ liệu không ở dạng file trong S3.
- C sai: backup băng ảo.
- D sai: file system trên AWS, không phải gateway on-prem tới S3.
</details>

**Câu 4.** (Chọn 2) Công ty muốn migrate 200 TB từ NAS NFS on-prem lên Amazon EFS, giữ nguyên quyền file, chạy hằng đêm cho tới khi cutover. Chọn gì?
- A. AWS DataSync
- B. DataSync agent cài on-prem gần NAS
- C. AWS Transfer Family
- D. Tape Gateway
- E. S3 Transfer Acceleration

<details><summary>Đáp án</summary>

**A, B.** DataSync cần agent cho nguồn on-prem, giữ metadata, chạy theo lịch.
- C sai: dành cho giao thức FTP/SFTP.
- D sai: backup băng.
- E sai: chỉ tăng tốc upload S3, không đích EFS.
</details>

**Câu 5.** Đối tác chỉ gửi file qua SFTP; công ty muốn file vào S3 mà không vận hành server. Chọn gì?
- A. EC2 chạy SFTP server
- B. AWS Transfer Family
- C. DataSync
- D. S3 File Gateway

<details><summary>Đáp án</summary>

**B.**
- A sai: phải tự vận hành.
- C sai: không cung cấp endpoint SFTP cho đối tác.
- D sai: NFS/SMB cho on-prem, không phải SFTP.
</details>

**Câu 6.** Ứng dụng on-prem dùng iSCSI; muốn toàn bộ dữ liệu nằm local với độ trễ thấp, backup lên AWS. Chọn gì?
- A. Volume Gateway stored
- B. Volume Gateway cached
- C. S3 File Gateway
- D. FSx for Windows

<details><summary>Đáp án</summary>

**A.**
- B sai: dữ liệu chính ở S3, chỉ cache local.
- C sai: NFS/SMB.
- D sai: SMB trên AWS.
</details>

**Câu 7.** Khách hàng **mới** của AWS cần chuyển 500 TB nhưng đường truyền rất chậm. Theo hiện trạng 10/2026, lựa chọn nào phù hợp?
- A. Đặt Snowball Edge
- B. AWS Data Transfer Terminal hoặc giải pháp partner (hoặc nâng băng thông + DataSync/Direct Connect)
- C. Snowmobile
- D. Gửi email ổ cứng cho AWS

<details><summary>Đáp án</summary>

**B.** AWS nêu Data Transfer Terminal, DataSync, partner làm phương án thay Snowball.
- A sai: không còn cho khách mới.
- C sai: thuộc Snow Family, không có cho khách mới (❓ trạng thái chính thức chưa rõ).
- D sai: không có quy trình như vậy.
</details>

**Câu 8.** Công ty đang dùng NetApp on-prem với NFS, SMB và iSCSI; muốn chuyển lên AWS giữ nguyên tính năng snapshot, dedup. Chọn gì?
- A. FSx for NetApp ONTAP
- B. EFS
- C. FSx for OpenZFS
- D. EBS Multi-Attach

<details><summary>Đáp án</summary>

**A.**
- B sai: chỉ NFS.
- C sai: chỉ NFS.
- D sai: block storage, không phải NAS.
</details>

---

### Nguồn đã đối chiếu
- AWS Snowball Edge availability change (không còn thiết bị Snow cho khách mới, phương án thay thế): https://docs.aws.amazon.com/snowball/latest/developer-guide/snowball-edge-availability-change.html
- Storage Gateway Hardware Appliance end of availability (12/05/2025): https://docs.aws.amazon.com/filegateway/latest/files3/hardware-appliance.html
- Amazon FSx File Gateway (không còn cho khách mới): https://docs.aws.amazon.com/filegateway/latest/filefsxw/use-fsxw-gateway.html
- FSx for Lustre deployment options: https://docs.aws.amazon.com/fsx/latest/LustreGuide/using-fsx-lustre.html
- FSx for Windows File Server: https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html
- FSx for NetApp ONTAP: https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/what-is-fsx-ontap.html
- AWS DataSync: https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html
- AWS Transfer Family: https://docs.aws.amazon.com/transfer/latest/userguide/what-is-aws-transfer-family.html
- Volume Gateway: https://docs.aws.amazon.com/storagegateway/latest/vgw/WhatIsStorageGateway.html
