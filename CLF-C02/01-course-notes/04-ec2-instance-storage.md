# CLF-C02 — EC2 Instance Storage

> Thuộc **Domain 3: Cloud Technology and Services** (storage): block storage (EBS, Instance Store), file storage (EFS, FSx), image (AMI, EC2 Image Builder).
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **EBS volume** = "ổ USB qua mạng": gắn vào EC2, **giữ dữ liệu** sau khi instance bị terminate, **gắn chặt với một AZ**.
- Muốn chuyển EBS sang AZ/Region khác → tạo **EBS Snapshot** rồi restore. Snapshot có **Archive tier** (rẻ hơn tới 75%) và **Recycle Bin** (chống xóa nhầm).
- **AMI** = image đóng gói sẵn OS + phần mềm để boot nhanh; build theo Region, copy được sang Region khác. **EC2 Image Builder** tự động tạo, test, phân phối AMI.
- **EC2 Instance Store** = ổ cứng vật lý gắn trên host, **I/O rất cao** nhưng **mất dữ liệu khi stop/terminate** (ephemeral).
- **EFS** = NFS managed, mount cho **hàng trăm EC2 Linux trên nhiều AZ**; trả theo dung lượng dùng. **EFS-IA** cho file ít truy cập.
- **FSx** = file system bên thứ ba managed: **FSx for Windows File Server** (SMB, NTFS, AD), **FSx for Lustre** (HPC), FSx for NetApp ONTAP, `[+]` FSx for OpenZFS.
- Shared Responsibility: AWS lo replication của EBS/EFS và phần cứng; **khách hàng lo backup/snapshot, mã hóa, dữ liệu, rủi ro khi dùng Instance Store**.

---

## 2. Core Concepts

### EBS Volume
- **Là gì:** ổ đĩa **mạng** (không phải ổ vật lý) gắn vào EC2 khi đang chạy; dữ liệu tồn tại sau khi instance terminate; ở mức CLF, mỗi volume gắn **một instance** tại một thời điểm.
- **Dùng khi nào:** làm root volume hoặc ổ dữ liệu cho EC2; tháo ra gắn sang instance khác nhanh chóng.
- **Vì sao quan trọng:** **khóa vào một AZ** (volume ở us-east-1a không gắn được vào instance ở us-east-1b); tính tiền theo **capacity provisioned** (GB, IOPS), không theo dung lượng thực dùng.

### Delete on Termination
- **Là gì:** thuộc tính quyết định EBS có bị xóa khi EC2 terminate không.
- **Dùng khi nào:** mặc định **root volume bị xóa** (bật), **volume gắn thêm không bị xóa** (tắt); đổi được qua Console/CLI.
- **Vì sao quan trọng:** muốn giữ root volume sau khi terminate thì tắt thuộc tính này.

### EBS Snapshots
- **Là gì:** bản backup của EBS tại một thời điểm; không bắt buộc detach volume nhưng nên làm.
- **Dùng khi nào:** backup, hoặc **copy snapshot sang AZ/Region khác** để chuyển volume.
- **Vì sao quan trọng:** snapshot là cách duy nhất để "di chuyển" EBS ra khỏi AZ của nó.

### EBS Snapshots Archive & Recycle Bin
- **Là gì:** **Archive tier** rẻ hơn **tới 75%**, restore mất **tới 72 giờ**; **Recycle Bin** giữ snapshot bị xóa theo rule để khôi phục khi xóa nhầm (snapshot giữ 1 ngày đến 1 năm).
- **Dùng khi nào:** archive cho snapshot cần giữ lâu, ít khi dùng; Recycle Bin chống xóa nhầm.
- **Vì sao quan trọng:** `[+]` archive có thời gian lưu tối thiểu 90 ngày; Recycle Bin nay bảo vệ cả EBS volume (1–7 ngày) và AMI.

### AMI (Amazon Machine Image)
- **Là gì:** bản tùy biến của EC2 (phần mềm, cấu hình, OS, monitoring); tạo AMI cũng tạo EBS snapshot; build theo Region, copy được sang Region khác.
- **Dùng khi nào:** launch từ **Public AMI** (AWS cung cấp), **AMI của bạn**, hoặc **AWS Marketplace AMI** (bên thứ ba làm, có thể bán).
- **Vì sao quan trọng:** boot nhanh hơn vì phần mềm đã đóng gói sẵn. Quy trình: chạy & tùy biến EC2 → stop → tạo AMI → launch instance mới.

### EC2 Image Builder
- **Là gì:** dịch vụ **tự động** tạo, bảo trì, kiểm thử và phân phối AMI (hoặc container image), chạy theo lịch.
- **Dùng khi nào:** cần AMI luôn được vá và test định kỳ, phân phối sang nhiều Region.
- **Vì sao quan trọng:** miễn phí, chỉ trả tiền tài nguyên bên dưới.

### EC2 Instance Store
- **Là gì:** ổ cứng **vật lý** trên host chạy EC2, I/O và IOPS rất cao.
- **Dùng khi nào:** buffer, cache, scratch data, nội dung tạm.
- **Vì sao quan trọng:** **ephemeral**: mất dữ liệu khi stop/terminate hoặc phần cứng hỏng; backup và replication là trách nhiệm của bạn.

### Amazon EFS (Elastic File System)
- **Là gì:** **NFS managed** mount được trên hàng trăm EC2, chỉ cho **Linux**, dữ liệu **multi-AZ**.
- **Dùng khi nào:** nhiều instance ở nhiều AZ cần dùng chung một file system.
- **Vì sao quan trọng:** highly available, scalable, **trả theo dung lượng dùng**, không cần capacity planning; đắt hơn EBS gp2 khoảng 3 lần (theo slide).

### EFS Infrequent Access (EFS-IA)
- **Là gì:** storage class tối ưu chi phí cho file không truy cập hằng ngày, rẻ hơn tới 92% so với EFS Standard (theo slide).
- **Dùng khi nào:** bật bằng **Lifecycle Policy** (vd: file không truy cập 60 ngày thì chuyển sang IA), trong suốt với ứng dụng.
- **Vì sao quan trọng:** `[+]` nay EFS còn có class **Archive** cho dữ liệu hiếm khi truy cập, latency cỡ chục mili giây.

### Amazon FSx
- **Là gì:** chạy file system hiệu năng cao của **bên thứ ba** dưới dạng fully managed.
- **Dùng khi nào:** **FSx for Windows File Server**: share file native Windows, **SMB + NTFS**, tích hợp **Microsoft Active Directory**, truy cập được từ on-prem; **FSx for Lustre** (Linux + cluster): **HPC**, ML, phân tích, video, tài chính; hàng trăm GB/s, triệu IOPS, độ trễ dưới mili giây, liên kết với S3.
- **Vì sao quan trọng:** còn có FSx for NetApp ONTAP và `[+]` FSx for OpenZFS. Lưu ý: exam guide CLF-C02 xếp **FSx for Lustre vào out-of-scope**, nhưng FSx nói chung vẫn in-scope.

### Shared Responsibility cho EC2 Storage
- **Là gì:** AWS: hạ tầng, **replication** cho EBS và EFS, thay phần cứng hỏng, đảm bảo nhân viên AWS không truy cập dữ liệu của bạn.
- **Dùng khi nào:** khách hàng: thiết lập **backup/snapshot**, **mã hóa**, chịu trách nhiệm dữ liệu trên ổ, hiểu rủi ro khi dùng Instance Store.
- **Vì sao quan trọng:** "ai chịu trách nhiệm snapshot EBS" → **khách hàng**.

### `[+]` EBS Multi-Attach
- **Là gì:** volume **io1/io2** có thể gắn vào nhiều instance trong **cùng AZ**.
- **Dùng khi nào:** ứng dụng cluster đặc thù tự quản lý ghi đồng thời.
- **Vì sao quan trọng:** slide nói "một instance một lúc (ở mức CLF)". Nếu đề CLF hỏi chung chung, vẫn chọn "một instance".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Amazon EBS | Block storage qua mạng, gắn EC2, khóa theo AZ | "persistent block storage for EC2" |
| Delete on Termination | Thuộc tính xóa EBS khi terminate EC2 | "keep root volume after termination" |
| EBS Snapshot | Backup point-in-time của EBS, copy được sang AZ/Region | "move volume to another AZ", "backup" |
| EBS Snapshots Archive | Tier lưu snapshot rẻ hơn tới 75%, restore tới 72 giờ | "long-term, rarely accessed snapshots" |
| Recycle Bin | Giữ tài nguyên bị xóa để khôi phục | "accidental deletion of snapshots" |
| AMI | Image tùy biến để launch EC2 | "pre-configured image", "faster boot" |
| AWS Marketplace AMI | AMI do bên thứ ba làm, có thể trả phí | "third-party pre-built software image" |
| EC2 Image Builder | Tự động build, test, phân phối AMI | "automate AMI creation and patching" |
| EC2 Instance Store | Ổ vật lý trên host, I/O cao, ephemeral | "highest IOPS", "temporary data", "cache" |
| Amazon EFS | NFS managed, Linux, multi-AZ, chia sẻ nhiều instance | "shared file system across AZs" |
| EFS-IA / Lifecycle Policy | Class rẻ cho file ít dùng, chuyển tự động | "files not accessed for 60 days" |
| FSx for Windows File Server | File share Windows, SMB, NTFS, AD | "Windows file share", "SMB" |
| FSx for Lustre | File system HPC hiệu năng cực cao | "HPC", "ML training", "sub-millisecond" |
| FSx for NetApp ONTAP | NetApp ONTAP managed | "migrate NetApp workloads" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| EBS | Instance Store | EBS: qua mạng, bền vững, khóa AZ; Instance Store: vật lý, nhanh nhất, mất khi stop | "Highest I/O + temporary" → Instance Store |
| EBS | EFS | EBS: một instance, một AZ, trả theo provisioned; EFS: nhiều instance, nhiều AZ, trả theo dùng | "Shared by hundreds of instances" → EFS |
| EFS | FSx for Windows | EFS: NFS cho Linux; FSx Windows: SMB/NTFS cho Windows | "Windows" + "SMB" → FSx |
| FSx for Lustre | EFS | Lustre: HPC, cực nhanh, nối S3; EFS: file share đa dụng | "HPC" → Lustre |
| EBS Snapshot | AMI | Snapshot backup một volume; AMI là image hoàn chỉnh để launch instance | Launch nhiều instance giống nhau → AMI |
| AMI | EC2 Image Builder | AMI là kết quả; Image Builder là công cụ tự động tạo AMI | "Automate building and testing" → Image Builder |
| Snapshot Archive | Recycle Bin | Archive giảm chi phí lưu lâu; Recycle Bin khôi phục khi xóa nhầm | |
| EFS Standard | EFS-IA | IA rẻ hơn cho file ít truy cập, chuyển bằng Lifecycle Policy | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "move EBS volume to another AZ/Region" | EBS Snapshot → restore | |
| "preserve root volume after termination" | Tắt Delete on Termination | |
| "very high IOPS, scratch/cache, data loss acceptable" | EC2 Instance Store | |
| "shared file storage for many Linux instances across AZs" | Amazon EFS | |
| "Windows file share, Active Directory, SMB" | FSx for Windows File Server | |
| "HPC, machine learning, high throughput file system" | FSx for Lustre | |
| "pre-installed software, faster launch" | Custom AMI | |
| "buy pre-configured software image from vendor" | AWS Marketplace AMI | |
| "automate creating and testing AMIs on a schedule" | EC2 Image Builder | |
| "keep snapshots for years at lowest cost" | EBS Snapshots Archive | |
| "recover accidentally deleted snapshots" | Recycle Bin | |
| "who sets up EBS backups/encryption" | Customer | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một EBS volume ở `us-east-1a` cần dùng cho instance ở `us-east-1b`. Cách làm đúng?
- A. Detach rồi attach trực tiếp sang instance ở us-east-1b
- B. Tạo snapshot rồi restore thành volume mới ở us-east-1b
- C. Bật Multi-AZ cho EBS
- D. Dùng EC2 Image Builder

<details><summary>Đáp án</summary>

**B.** EBS khóa theo AZ; snapshot là cách chuyển volume.
- A sai: không gắn được EBS sang AZ khác.
- C sai: EBS không có tùy chọn Multi-AZ như RDS.
- D sai: Image Builder dùng để tạo AMI.
</details>

**Câu 2.** Ứng dụng cần lưu dữ liệu tạm với IOPS cao nhất có thể, chấp nhận mất dữ liệu khi instance dừng. Chọn gì?
- A. Amazon EBS
- B. Amazon EFS
- C. EC2 Instance Store
- D. Amazon S3

<details><summary>Đáp án</summary>

**C.** Instance Store là ổ vật lý gắn host, I/O cao nhất, ephemeral.
- A sai: ổ qua mạng, hiệu năng thấp hơn.
- B sai: file system chia sẻ qua mạng.
- D sai: object storage, không phải ổ đĩa của instance.
</details>

**Câu 3.** Hàng trăm EC2 Linux ở 3 AZ cần cùng đọc/ghi một file system chung. Chọn gì?
- A. Amazon EBS
- B. Amazon EFS
- C. FSx for Windows File Server
- D. EC2 Instance Store

<details><summary>Đáp án</summary>

**B.** EFS là NFS managed, multi-AZ, mount được trên nhiều instance Linux.
- A, D sai: gắn với một instance và một AZ (hoặc một host).
- C sai: dành cho Windows/SMB.
</details>

**Câu 4.** Công ty chuyển file server Windows lên AWS, cần SMB và tích hợp Microsoft Active Directory. Chọn gì?
- A. Amazon EFS
- B. FSx for Lustre
- C. FSx for Windows File Server
- D. Amazon S3

<details><summary>Đáp án</summary>

**C.** File share native Windows, SMB, NTFS, AD.
- A sai: NFS cho Linux.
- B sai: cho HPC.
- D sai: object storage.
</details>

**Câu 5.** (Chọn 2) Phát biểu nào về EBS là **đúng**?
- A. Dữ liệu vẫn còn sau khi instance bị terminate nếu volume không bị xóa
- B. Một EBS volume dùng được ở mọi AZ trong Region
- C. Bạn trả tiền cho toàn bộ capacity đã provision
- D. Là ổ vật lý gắn trên host
- E. Mặc định mọi volume đều bị xóa khi terminate

<details><summary>Đáp án</summary>

**A, C.**
- B sai: EBS khóa trong một AZ.
- D sai: EBS là ổ mạng; Instance Store mới là ổ vật lý.
- E sai: chỉ root volume bị xóa mặc định.
</details>

**Câu 6.** Team cần mọi AMI dùng trong công ty được vá bảo mật và kiểm thử hằng tuần một cách tự động, rồi phân phối sang nhiều Region. Dùng dịch vụ nào?
- A. AWS Backup
- B. EC2 Image Builder
- C. EBS Snapshots Archive
- D. AWS Systems Manager Parameter Store

<details><summary>Đáp án</summary>

**B.** Image Builder tự động build, test và phân phối AMI theo lịch.
- A sai: backup tài nguyên.
- C sai: lưu snapshot giá rẻ.
- D sai: lưu cấu hình và secret.
</details>

**Câu 7.** Công ty phải giữ snapshot EBS cuối dự án trong 7 năm để compliance, gần như không bao giờ restore. Cách tiết kiệm nhất?
- A. Giữ ở Snapshot Standard tier
- B. Chuyển sang EBS Snapshots Archive
- C. Giữ volume EBS gắn vào instance đã stop
- D. Copy snapshot sang EC2 Instance Store

<details><summary>Đáp án</summary>

**B.** Archive tier rẻ hơn tới 75%, chấp nhận restore mất tới 72 giờ.
- A sai: đắt hơn.
- C sai: vẫn trả tiền volume provisioned.
- D sai: Instance Store là ephemeral.
</details>

**Câu 8.** (Chọn 2) Theo Shared Responsibility Model, khách hàng chịu trách nhiệm gì với storage của EC2?
- A. Replication dữ liệu bên trong EBS
- B. Thiết lập quy trình backup/snapshot
- C. Thay ổ cứng hỏng
- D. Bật mã hóa dữ liệu
- E. Đảm bảo nhân viên AWS không truy cập dữ liệu

<details><summary>Đáp án</summary>

**B, D.**
- A, C, E sai: thuộc trách nhiệm của AWS.
</details>

---

### Nguồn đã đối chiếu
- Archive Amazon EBS snapshots (rẻ hơn tới 75%, giữ tối thiểu 90 ngày): https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-archive.html
- EBS snapshot archive considerations (restore tới 72 giờ): https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-archive-considerations.html
- How Recycle Bin works (volume 1–7 ngày, snapshot/AMI 1–365 ngày): https://docs.aws.amazon.com/ebs/latest/userguide/recycle-bin-concepts.html
- Amazon EFS storage classes (Standard, IA, Archive): https://docs.aws.amazon.com/efs/latest/ug/features.html
- CLF-C02 out-of-scope services (FSx for Lustre): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
- EC2 Image Builder: https://docs.aws.amazon.com/imagebuilder/latest/userguide/what-is-image-builder.html
