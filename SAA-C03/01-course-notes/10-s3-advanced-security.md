# SAA-C03 — Amazon S3: Advanced, Performance, Encryption & Security

> Thuộc **Domain 1: Design Secure Architectures**, **Domain 3: Design High-Performing Architectures** và **Domain 4: Design Cost-Optimized Architectures**: lifecycle, event notification, hiệu năng, mã hóa, khóa object, pre-signed URL, access point.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Lifecycle rule**: transition (chuyển class) và expiration (xóa object, version cũ, multipart dở dang), lọc theo prefix/tag. **Storage Class Analysis** gợi ý thời điểm chuyển Standard → Standard-IA.
- **Event Notifications** gửi tới **SNS, SQS, Lambda**; hoặc mọi event sang **EventBridge** để lọc nâng cao và gửi tới 18+ đích.
- Hiệu năng: mỗi **prefix** ≥ **3,500 PUT/COPY/POST/DELETE** và **5,500 GET/HEAD** mỗi giây; tăng tốc bằng **multipart upload**, **Transfer Acceleration**, **byte-range fetch**.
- Mã hóa: **SSE-S3** (mặc định cho mọi object mới), **SSE-KMS** (audit qua CloudTrail, chịu quota KMS), **DSSE-KMS** (hai lớp), **SSE-C** (khách cung cấp key, bắt buộc HTTPS), **client-side**.
- ⚠️ Lỗi thời: "SSE-C dùng được ngay trên mọi bucket" → từ 04/2026 bucket general purpose mới **mặc định chặn SSE-C**; muốn dùng phải bật lại bằng PutBucketEncryption.
- Bảo mật: ép HTTPS bằng `aws:SecureTransport`, **MFA Delete** (chỉ root, cần versioning), **pre-signed URL**, **Object Lock** (WORM: Compliance/Governance, Legal Hold), **Access Points**.
- ⚠️ Lỗi thời: **S3 Select** và **Glacier vault (Vault Lock)** đóng với khách mới; **S3 Object Lambda** chỉ còn cho khách hiện hữu từ 07/11/2025.

---

## 2. Core Concepts

### Lifecycle rules
- **Là gì:** **transition actions** (vd: sang Standard-IA sau 60 ngày, Glacier sau 6 tháng) và **expiration actions** (xóa log sau 365 ngày, xóa noncurrent version, dọn multipart upload chưa hoàn tất).
- **Dùng khi nào:** dữ liệu già đi theo thời gian biết trước; áp cho cả bucket, một prefix, hoặc theo tag.
- **Vì sao quan trọng:** **Storage Class Analysis** (cập nhật mỗi 24–48 giờ) giúp xác định ngưỡng ngày; không gợi ý cho One Zone-IA hay Glacier.

### Requester Pays
- **Là gì:** người yêu cầu trả phí request và truyền dữ liệu, chủ bucket chỉ trả phí lưu trữ.
- **Dùng khi nào:** chia sẻ dataset lớn cho account khác.
- **Vì sao quan trọng:** requester phải được AWS xác thực (không anonymous).

### S3 Event Notifications
- **Là gì:** sự kiện `ObjectCreated`, `ObjectRemoved`, `ObjectRestore`, `Replication`... lọc theo tên (vd `*.jpg`); đích **SNS, SQS, Lambda** (cần resource policy ở đích), thường trong vài giây.
- **Dùng khi nào:** tạo thumbnail khi upload ảnh, kích hoạt pipeline.
- **Vì sao quan trọng:** gửi sang **EventBridge** để có lọc JSON nâng cao, nhiều đích (Step Functions, Kinesis...), archive, replay.

### S3 performance
- **Là gì:** S3 tự scale, độ trễ 100–200 ms cho byte đầu; ≥ 3,500 write và 5,500 read request/giây **mỗi prefix**, không giới hạn số prefix.
- **Dùng khi nào:** chia dữ liệu ra nhiều prefix để tăng throughput song song.
- **Vì sao quan trọng:** SSE-KMS có thể chạm **quota KMS** (GenerateDataKey/Decrypt) theo Region; xin tăng qua Service Quotas.

### Multipart, Transfer Acceleration, byte-range
- **Là gì:** **multipart** upload song song từng phần; **Transfer Acceleration** đi qua edge location rồi mạng riêng AWS tới bucket; **byte-range fetch** GET song song từng khoảng byte.
- **Dùng khi nào:** upload từ xa qua đại dương → Transfer Acceleration (tương thích multipart); chỉ cần đọc phần đầu file → byte-range.
- **Vì sao quan trọng:** byte-range cũng giúp retry phần lỗi nhanh hơn.

### Batch Operations & Storage Lens
- **Là gì:** **Batch Operations** thực hiện thao tác hàng loạt trên danh sách object (sửa metadata, copy, mã hóa object chưa mã hóa, đổi ACL/tag, restore Glacier, gọi Lambda), có retry và báo cáo; **Storage Lens** dashboard phân tích dung lượng, hoạt động, chi phí ở cấp Organization.
- **Dùng khi nào:** danh sách object lấy từ **S3 Inventory**, lọc bằng Athena rồi đưa vào Batch Operations.
- **Vì sao quan trọng:** "mã hóa toàn bộ object chưa mã hóa" → Inventory + Batch Operations.

### Encryption options
- **Là gì:** **SSE-S3** (key do S3 quản lý, AES-256, header `x-amz-server-side-encryption: AES256`); **SSE-KMS** (`aws:kms`, kiểm soát key, audit CloudTrail); **DSSE-KMS** (hai lớp mã hóa phía server); **SSE-C** (gửi key trong mỗi request, chỉ HTTPS, S3 không lưu key); **client-side** (mã hóa trước khi gửi).
- **Dùng khi nào:** compliance cần audit việc dùng key → SSE-KMS; công ty muốn tự giữ key ngoài AWS → SSE-C hoặc client-side.
- **Vì sao quan trọng:** từ 01/2023 mọi object mới tự mã hóa SSE-S3. ⚠️ Lỗi thời: SSE-C bật sẵn → từ 04/2026 bucket mới mặc định chặn SSE-C (BlockedEncryptionTypes).

### Encryption in transit & bucket policy
- **Là gì:** S3 có endpoint HTTP và HTTPS; ép HTTPS bằng Deny khi `aws:SecureTransport` là false.
- **Dùng khi nào:** ép một loại mã hóa: bucket policy Deny PutObject không có header mã hóa mong muốn.
- **Vì sao quan trọng:** bucket policy được đánh giá **trước** default encryption.

### CORS
- **Là gì:** cơ chế trình duyệt cho phép trang ở origin này gọi tài nguyên ở origin khác; S3 phải trả header `Access-Control-Allow-Origin`.
- **Dùng khi nào:** website ở bucket A tải font/ảnh từ bucket B.
- **Vì sao quan trọng:** cấu hình CORS trên bucket **được gọi** (bucket B), không phải bucket gọi.

### MFA Delete, access logs, pre-signed URL
- **Là gì:** **MFA Delete** bắt nhập MFA khi xóa vĩnh viễn version hoặc tắt versioning (chỉ root bật được); **server access logging** ghi request vào bucket khác; **pre-signed URL** cấp quyền tạm GET/PUT theo quyền của người tạo.
- **Dùng khi nào:** pre-signed URL để user tải file private hoặc upload vào đúng key; hạn: console tới 12 giờ, CLI/SDK tới 7 ngày (với credential dài hạn).
- **Vì sao quan trọng:** **không ghi access log vào chính bucket đang được log** (vòng lặp, phình dữ liệu).

### S3 Object Lock
- **Là gì:** mô hình **WORM** cho từng object version (cần versioning); **Compliance mode** (không ai xóa/sửa được, kể cả root, không rút ngắn retention); **Governance mode** (user có quyền đặc biệt ghi đè được); **Legal Hold** (giữ vô thời hạn, gỡ bằng quyền `s3:PutObjectLegalHold`).
- **Dùng khi nào:** lưu trữ theo quy định (SEC 17a-4, FINRA), bảo vệ backup khỏi ransomware.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Glacier Vault Lock" → dựa trên Glacier vault, đã **đóng với khách mới**; dùng S3 Object Lock trên class Glacier thay thế.

### Access Points & Multi-Region Access Points
- **Là gì:** **Access Point** có DNS và policy riêng cho từng nhóm (finance/, sales/), có thể giới hạn chỉ truy cập từ VPC; **Multi-Region Access Point** một endpoint toàn cầu định tuyến tới bucket gần nhất, kèm failover.
- **Dùng khi nào:** đơn giản hóa bucket policy khổng lồ; ứng dụng đa Region cần một endpoint S3 duy nhất.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: **S3 Object Lambda** (biến đổi dữ liệu khi GET qua Lambda) chỉ còn cho khách đang dùng từ 07/11/2025; khách mới cân nhắc Lambda + API/CloudFront.

### `[+]` S3 Select đã đóng với khách mới
- **Là gì:** S3 Select/Glacier Select lọc dữ liệu CSV/JSON/Parquet bằng SQL ngay ở server.
- **Dùng khi nào:** khách mới dùng **Athena** hoặc lọc phía client thay thế.
- **Vì sao quan trọng:** đề cũ có thể vẫn chọn S3 Select cho "retrieve subset of data"; hiểu là Athena trong bối cảnh mới.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Lifecycle transition | Chuyển object sang class rẻ hơn theo tuổi | "move to Glacier after 90 days" |
| Lifecycle expiration | Xóa object/version/MPU dở dang | "delete after 365 days" |
| Storage Class Analysis | Gợi ý ngưỡng chuyển sang IA | "determine optimal transition" |
| Requester Pays | Người tải trả phí transfer | "share large dataset, others pay" |
| Event Notifications | Gửi sự kiện tới SNS/SQS/Lambda | "process image after upload" |
| Transfer Acceleration | Upload qua edge location | "global users upload to one bucket" |
| Byte-range fetch | GET song song theo khoảng byte | "read only file header" |
| S3 Batch Operations | Thao tác hàng loạt trên object | "encrypt all existing objects" |
| SSE-S3 | Key do S3 quản lý | "default encryption" |
| SSE-KMS | Key KMS, audit CloudTrail | "audit key usage", "control rotation" |
| DSSE-KMS | Hai lớp mã hóa server-side | "dual-layer encryption" |
| SSE-C | Khách gửi key mỗi request | "customer manages keys outside AWS" |
| MFA Delete | MFA khi xóa version vĩnh viễn | "extra protection on delete" |
| Pre-signed URL | URL tạm với quyền người tạo | "temporary access to private object" |
| Object Lock Compliance | WORM tuyệt đối | "no one, including root, can delete" |
| Access Point | Endpoint + policy theo nhóm | "simplify bucket policy" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| SSE-S3 | SSE-KMS | KMS cho audit, kiểm soát key, chịu quota; S3 đơn giản, miễn phí | "Audit who used key" → KMS |
| SSE-KMS | SSE-C | KMS key nằm trong AWS; SSE-C key do khách gửi, S3 không lưu | "Keys managed outside AWS" → SSE-C |
| SSE-C | Client-side | SSE-C: S3 mã hóa bằng key bạn gửi; client-side: bạn tự mã hóa trước | |
| Compliance mode | Governance mode | Compliance không ai ghi đè; Governance user có quyền đặc biệt ghi đè được | "Even root cannot" → Compliance |
| Retention period | Legal Hold | Retention có hạn; Legal Hold vô thời hạn tới khi gỡ | "Litigation, unknown duration" → Legal Hold |
| Event Notifications | EventBridge | Notifications: SNS/SQS/Lambda; EventBridge: lọc nâng cao, nhiều đích, replay | Nhiều đích → EventBridge |
| Transfer Acceleration | CloudFront | Acceleration tăng tốc **upload/download tới bucket**; CloudFront cache nội dung đọc | Upload xa → Acceleration |
| Pre-signed URL | Bucket policy public | Pre-signed tạm, từng object; public là mọi người, vĩnh viễn | "Premium users only" → pre-signed |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "move to IA after 30 days, delete after 1 year" | Lifecycle rules | |
| "clean up incomplete multipart uploads" | Lifecycle expiration (AbortIncompleteMultipartUpload) | |
| "trigger Lambda when object uploaded" | S3 Event Notification | |
| "advanced filtering, multiple targets, replay" | S3 → EventBridge | |
| "global users upload slowly to one bucket" | Transfer Acceleration | |
| "KMS throttling on S3 uploads" | Tăng quota KMS / dùng S3 Bucket Key | |
| "audit encryption key usage" | SSE-KMS | |
| "keys must stay on-premises" | SSE-C hoặc client-side | SSE-C cần bật lại cho bucket mới |
| "force HTTPS" | Bucket policy Deny `aws:SecureTransport=false` | |
| "temporary download link for private file" | Pre-signed URL | |
| "WORM, regulator, nobody can delete" | Object Lock Compliance mode | |
| "encrypt all existing unencrypted objects" | S3 Inventory + Batch Operations | |
| "website in bucket A loads fonts from bucket B" | Bật CORS trên bucket B | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Bucket chứa ảnh gốc và thumbnail. Thumbnail chỉ cần giữ 60 ngày; ảnh gốc chuyển Glacier sau 90 ngày. Cách làm?
- A. Lifecycle rules theo prefix/tag cho từng loại
- B. Intelligent-Tiering
- C. Lambda chạy hằng ngày xóa file
- D. Replication

<details><summary>Đáp án</summary>

**A.** Lifecycle hỗ trợ transition và expiration theo prefix/tag.
- B sai: không xóa object theo tuổi.
- C sai: tự viết code không cần thiết.
- D sai: copy dữ liệu, không đổi class/xóa.
</details>

**Câu 2.** Compliance yêu cầu ghi lại mỗi lần key mã hóa được dùng và có thể kiểm soát quyền dùng key. Chọn gì?
- A. SSE-S3
- B. SSE-KMS
- C. SSE-C
- D. Không mã hóa, dùng HTTPS

<details><summary>Đáp án</summary>

**B.** Mọi lần dùng key KMS được ghi vào CloudTrail; key policy kiểm soát quyền.
- A sai: không có audit theo key.
- C sai: AWS không lưu key, không audit dùng key.
- D sai: chỉ là in-transit.
</details>

**Câu 3.** Upload từ văn phòng ở châu Âu tới bucket `us-east-1` rất chậm. Cải thiện nhanh nhất?
- A. Bật S3 Transfer Acceleration và dùng multipart upload
- B. Chuyển sang Glacier
- C. Dùng byte-range fetch
- D. Bật versioning

<details><summary>Đáp án</summary>

**A.** Dữ liệu vào edge location gần nhất rồi đi trên mạng AWS.
- B sai: class lưu trữ không ảnh hưởng upload.
- C sai: dùng cho download.
- D sai: không liên quan tốc độ.
</details>

**Câu 4.** (Chọn 2) Cơ quan quản lý yêu cầu dữ liệu không bị sửa/xóa trong 7 năm, kể cả bởi root. Chọn gì?
- A. Bật versioning
- B. S3 Object Lock Compliance mode với retention 7 năm
- C. Object Lock Governance mode
- D. MFA Delete là đủ
- E. Lifecycle expiration 7 năm

<details><summary>Đáp án</summary>

**A, B.** Object Lock cần versioning; Compliance mode không ai ghi đè được.
- C sai: user có quyền đặc biệt xóa được.
- D sai: root vẫn xóa được với MFA.
- E sai: lifecycle xóa sau 7 năm, không ngăn xóa trước.
</details>

**Câu 5.** Ứng dụng cần cho khách hàng trả phí tải file video private trong 1 giờ, không public bucket. Chọn gì?
- A. Bucket policy public
- B. Pre-signed URL hết hạn sau 1 giờ
- C. CORS
- D. Requester Pays

<details><summary>Đáp án</summary>

**B.**
- A sai: ai cũng tải được.
- C sai: CORS cho trình duyệt gọi cross-origin, không cấp quyền.
- D sai: thay đổi người trả phí, không cấp quyền tạm.
</details>

**Câu 6.** Upload SSE-KMS tốc độ cao bị lỗi throttling. Nguyên nhân và hướng xử lý?
- A. Vượt giới hạn prefix → bật versioning
- B. Chạm quota request của KMS → xin tăng quota hoặc bật S3 Bucket Key
- C. Thiếu CORS
- D. Chuyển sang SSE-C

<details><summary>Đáp án</summary>

**B.** Mỗi upload/download SSE-KMS gọi KMS; Bucket Key giảm số lần gọi.
- A sai: versioning không tăng throughput.
- C sai: không liên quan.
- D sai: đổi mô hình quản lý key chỉ để tránh throttling là không phù hợp; bucket mới còn mặc định chặn SSE-C.
</details>

**Câu 7.** Mỗi khi có ảnh mới, cần gửi sự kiện tới Step Functions và Kinesis với bộ lọc theo metadata. Chọn gì?
- A. S3 Event Notifications tới SNS
- B. Bật gửi sự kiện S3 sang EventBridge, tạo rule tới các đích
- C. S3 Batch Operations
- D. CloudTrail data events

<details><summary>Đáp án</summary>

**B.** EventBridge lọc nâng cao và hỗ trợ nhiều đích.
- A sai: SNS không lọc nâng cao theo nội dung sự kiện S3 như EventBridge.
- C sai: dùng cho thao tác hàng loạt.
- D sai: dùng cho audit, không phải điều phối sự kiện.
</details>

**Câu 8.** Bucket chứa dữ liệu của nhiều phòng ban; bucket policy đã quá dài, khó quản lý. Cách đơn giản hóa?
- A. Tạo S3 Access Point cho từng phòng ban với policy riêng
- B. Tạo bucket riêng cho mỗi người
- C. Dùng ACL cho từng object
- D. Bật Requester Pays

<details><summary>Đáp án</summary>

**A.** Mỗi access point có DNS và policy riêng theo prefix.
- B sai: phức tạp, khó vận hành.
- C sai: ACL không được khuyến nghị và đã tắt mặc định.
- D sai: không liên quan quyền.
</details>

---

### Nguồn đã đối chiếu
- Managing object lifecycle: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html
- Amazon S3 Event Notifications & EventBridge: https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html
- Performance design patterns (3,500/5,500 request mỗi prefix): https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html
- Protecting data with encryption (SSE-S3 mặc định, SSE-KMS, DSSE-KMS, SSE-C): https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html
- Default SSE-C setting for new buckets (chặn SSE-C mặc định từ 04/2026): https://docs.aws.amazon.com/AmazonS3/latest/userguide/default-s3-c-encryption-setting-faq.html
- S3 Object Lock: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html
- Pre-signed URLs: https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html
- Access Points: https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points.html
- S3 Object Lambda availability change: https://docs.aws.amazon.com/AmazonS3/latest/userguide/amazons3-ol-change.html
- S3 Select (đóng với khách mới): https://docs.aws.amazon.com/AmazonS3/latest/API/API_SelectParameters.html
- S3 Glacier vaults (đóng với khách mới): https://docs.aws.amazon.com/amazonglacier/latest/dev/creating-vaults-console.html
