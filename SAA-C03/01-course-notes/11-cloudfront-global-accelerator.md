# SAA-C03 — CloudFront & AWS Global Accelerator

> Thuộc **Domain 1: Design Secure Architectures** và **Domain 3: Design High-Performing Architectures**: CDN, origin, bảo vệ origin, geo restriction, price class, invalidation và tăng tốc mạng toàn cầu.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **CloudFront** = CDN: cache nội dung ở hàng trăm **edge location**, giảm độ trễ, giảm tải origin, chống DDoS (tích hợp **Shield**, **WAF**).
- Origin: **S3 bucket** (bảo vệ bằng **OAC**), **custom origin HTTP** (ALB, EC2, S3 website, API Gateway, bất kỳ HTTP backend), **VPC origin** (ALB/NLB/EC2 trong private subnet).
- ⚠️ Lỗi thời: **OAI** (Origin Access Identity) là cơ chế legacy → dùng **OAC** (Origin Access Control), hỗ trợ SSE-KMS, mọi Region, request động.
- **Geo restriction**: allowlist/blocklist theo quốc gia. **Price class**: All, 200, 100 (100 rẻ nhất, ít edge nhất).
- **Cache invalidation**: xóa cache trước khi hết TTL (`/*` hoặc đường dẫn cụ thể); cache key, TTL, behavior theo path.
- **CloudFront vs S3 CRR**: CloudFront cho nội dung tĩnh cache toàn cầu; CRR cho nội dung động cần đọc độ trễ thấp ở vài Region.
- **Global Accelerator**: **2 anycast IP tĩnh**, đi qua mạng AWS từ edge tới endpoint (ALB, NLB, EC2, EIP), **không cache**, TCP/UDP, failover < 1 phút, health check.

---

## 2. Core Concepts

### Amazon CloudFront
- **Là gì:** mạng phân phối nội dung toàn cầu, cache tại edge; kết nối edge → origin qua mạng AWS.
- **Dùng khi nào:** website tĩnh/động, video streaming, API toàn cầu, giảm tải origin.
- **Vì sao quan trọng:** tăng hiệu năng đọc cho user toàn cầu + lớp bảo vệ DDoS (Shield Standard miễn phí) và WAF.

### Origins
- **Là gì:** **S3** (file, tĩnh; upload qua CloudFront được), **MediaStore/MediaPackage**, **custom origin**: ALB, EC2, API Gateway, S3 static website, HTTP backend bất kỳ.
- **Dùng khi nào:** website tĩnh + API động dùng nhiều origin, chọn theo **cache behavior** (path pattern như `/api/*`).
- **Vì sao quan trọng:** custom origin công khai cần cho phép dải IP public của edge location trong SG; dùng VPC origin để giữ backend private.

### OAC & bảo vệ S3 origin
- **Là gì:** **Origin Access Control** cho phép chỉ distribution CloudFront đọc bucket (bucket policy cho principal `cloudfront.amazonaws.com` kèm điều kiện ARN distribution).
- **Dùng khi nào:** bucket S3 private, user chỉ truy cập qua CloudFront.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: OAI → AWS khuyến nghị **OAC** và hướng dẫn migrate OAI → OAC; OAI không hỗ trợ SSE-KMS và một số Region mới.

### VPC origins
- **Là gì:** CloudFront kết nối thẳng tới **ALB, NLB hoặc EC2 trong private subnet**, chỉ distribution được chỉ định mới truy cập được.
- **Dùng khi nào:** không muốn đặt ALB/EC2 public chỉ để CloudFront gọi.
- **Vì sao quan trọng:** thay mẫu cũ "ALB public + SG cho phép IP edge + custom header bí mật".

### Geo restriction & price class
- **Là gì:** geo restriction chặn/cho phép theo quốc gia (dựa trên Geo-IP); price class giới hạn edge location để giảm chi phí: **All**, **200** (bỏ Region đắt nhất), **100** (chỉ Region rẻ nhất).
- **Dùng khi nào:** bản quyền nội dung theo quốc gia; giảm chi phí khi user tập trung ở Bắc Mỹ/châu Âu.
- **Vì sao quan trọng:** "chặn quốc gia ở CDN" → CloudFront geo restriction (hoặc WAF geo match).

### Caching & invalidation
- **Là gì:** cache theo **cache key** (URL + header/cookie/query tùy chọn), TTL; **invalidation** xóa cache cho toàn bộ (`/*`) hoặc đường dẫn cụ thể.
- **Dùng khi nào:** cập nhật nội dung trên origin mà không chờ TTL hết.
- **Vì sao quan trọng:** tách nội dung tĩnh và động bằng behavior để cache hiệu quả; versioned file name tránh phải invalidate.

### CloudFront vs S3 Cross-Region Replication
- **Là gì:** CloudFront: mạng edge toàn cầu, cache có TTL, nội dung tĩnh dùng mọi nơi; CRR: cấu hình cho từng Region, cập nhật gần real-time, read-only.
- **Dùng khi nào:** CRR cho nội dung động cần độ trễ thấp ở **vài Region** cụ thể.
- **Vì sao quan trọng:** đề so sánh trực tiếp hai lựa chọn này.

### AWS Global Accelerator
- **Là gì:** cấp **2 anycast IP tĩnh**; user vào edge gần nhất rồi đi trên mạng riêng AWS tới endpoint; endpoint: ALB, NLB, EC2, Elastic IP, public hoặc private.
- **Dùng khi nào:** ứng dụng TCP/UDP toàn cầu (gaming, IoT, VoIP), HTTP cần IP tĩnh, failover đa Region nhanh (< 1 phút) với health check.
- **Vì sao quan trọng:** không vướng cache client như DNS; tích hợp Shield; **không cache nội dung**.

### `[+]` CloudFront Functions vs Lambda@Edge
- **Là gì:** xử lý logic tại edge: CloudFront Functions (JS nhẹ, viewer request/response, < 1 ms) và Lambda@Edge (Node.js/Python, cả origin request/response, mạnh hơn).
- **Dùng khi nào:** header manipulation, URL rewrite, xác thực token đơn giản → Functions; logic nặng, gọi mạng → Lambda@Edge.
- **Vì sao quan trọng:** chi tiết ở chương 15 (serverless).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Edge location | Điểm cache của CloudFront | "reduce latency globally" |
| Origin | Nguồn nội dung gốc | "S3, ALB, EC2, HTTP server" |
| OAC | Chỉ CloudFront đọc được S3 | "prevent direct S3 access" |
| OAI | Cơ chế cũ, legacy | "origin access identity" |
| VPC origin | Origin trong private subnet | "keep ALB private behind CloudFront" |
| Cache behavior | Quy tắc theo path pattern | "/api/* to ALB, /static/* to S3" |
| Cache invalidation | Xóa cache sớm | "content updated, users see old version" |
| Geo restriction | Chặn/cho phép theo quốc gia | "block users from certain countries" |
| Price Class 100 | Ít edge, rẻ nhất | "reduce CloudFront cost" |
| Global Accelerator | 2 anycast IP tĩnh, mạng AWS | "static IP, global, TCP/UDP" |
| Anycast IP | Một IP quảng bá từ nhiều edge | "same IP served from nearest edge" |
| Endpoint group | Nhóm endpoint theo Region | "traffic dial between Regions" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CloudFront | Global Accelerator | CloudFront cache HTTP tại edge; GA không cache, proxy TCP/UDP, IP tĩnh | UDP / static IP → GA |
| OAC | OAI | OAC mới, hỗ trợ SSE-KMS, mọi Region; OAI legacy | Luôn chọn OAC |
| CloudFront | S3 CRR | CloudFront cache toàn cầu; CRR bản sao đọc ở vài Region | Nội dung động vài Region → CRR |
| Geo restriction | Geolocation routing (Route 53) | Geo restriction chặn ở CDN; Route 53 geolocation chọn endpoint DNS | "Block access" → geo restriction |
| Invalidation | TTL | Invalidation xóa ngay; TTL hết hạn tự nhiên | "Immediately" → invalidation |
| Global Accelerator | Route 53 latency | GA failover không phụ thuộc DNS cache; Route 53 phụ thuộc TTL | "Fast failover, no DNS caching" → GA |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "static content to global users with low latency" | CloudFront + S3 | |
| "users must not access S3 directly, only via CloudFront" | OAC + bucket policy | |
| "ALB should not be public, CloudFront in front" | CloudFront VPC origin | |
| "block content in specific countries" | CloudFront geo restriction | |
| "reduce CloudFront cost, users in US/EU" | Price Class 100 | |
| "users see stale content after deploy" | Cache invalidation | |
| "global UDP game, static IPs, fast failover" | Global Accelerator | |
| "whitelist 2 IPs for a multi-Region app" | Global Accelerator | |
| "dynamic content, low latency read in 2 Regions" | S3 CRR | |
| "DDoS protection at edge" | CloudFront/GA + Shield (+ WAF) | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Website tĩnh trên S3 phục vụ user toàn cầu. Muốn giảm độ trễ và không cho ai truy cập S3 trực tiếp. Chọn gì?
- A. Bật S3 static website hosting public
- B. CloudFront + OAC, bucket policy chỉ cho CloudFront
- C. S3 Transfer Acceleration
- D. Global Accelerator trước S3

<details><summary>Đáp án</summary>

**B.**
- A sai: bucket public, ai cũng truy cập trực tiếp.
- C sai: tăng tốc upload/download nhưng không cache, không chặn truy cập trực tiếp.
- D sai: Global Accelerator không nhận S3 làm endpoint.
</details>

**Câu 2.** Game multiplayer dùng UDP, người chơi toàn cầu, cần IP tĩnh và failover giữa Region nhanh. Chọn gì?
- A. CloudFront
- B. AWS Global Accelerator
- C. Route 53 latency routing
- D. S3 CRR

<details><summary>Đáp án</summary>

**B.** Anycast IP tĩnh, hỗ trợ UDP, failover nhanh.
- A sai: CloudFront cho HTTP/HTTPS.
- C sai: phụ thuộc DNS cache, không có IP tĩnh.
- D sai: replication storage.
</details>

**Câu 3.** Sau khi deploy CSS mới, user vẫn thấy bản cũ vì CloudFront cache 24 giờ. Xử lý nhanh nhất?
- A. Tạo invalidation cho file CSS (hoặc `/*`)
- B. Xóa distribution
- C. Giảm price class
- D. Bật geo restriction

<details><summary>Đáp án</summary>

**A.**
- B sai: phá hủy cấu hình, gián đoạn.
- C, D sai: không liên quan cache.
</details>

**Câu 4.** (Chọn 2) Do bản quyền, video chỉ được xem ở Mỹ và Canada; muốn giảm chi phí CDN vì user chủ yếu ở Bắc Mỹ. Chọn gì?
- A. Geo restriction allowlist US, CA
- B. Price Class 100
- C. Global Accelerator
- D. S3 Object Lock
- E. Route 53 weighted routing

<details><summary>Đáp án</summary>

**A, B.**
- C sai: không chặn theo quốc gia, không giảm phí CDN.
- D sai: chống xóa object.
- E sai: chia tỷ lệ traffic, không chặn quốc gia.
</details>

**Câu 5.** Ứng dụng có ALB trong private subnet. Muốn đặt CloudFront phía trước mà không public ALB. Chọn gì?
- A. Tạo ALB public mới, SG cho phép 0.0.0.0/0
- B. Dùng CloudFront VPC origin trỏ tới ALB private
- C. Dùng OAI
- D. Dùng S3 website endpoint

<details><summary>Đáp án</summary>

**B.**
- A sai: lộ ALB ra Internet.
- C sai: OAI chỉ dành cho S3 và đã legacy.
- D sai: không liên quan ALB.
</details>

**Câu 6.** Nội dung động cần đọc độ trễ thấp ở 2 Region cụ thể, cập nhật gần real-time. Chọn gì?
- A. CloudFront với TTL 1 ngày
- B. S3 Cross-Region Replication
- C. Price Class All
- D. Glacier

<details><summary>Đáp án</summary>

**B.** CRR giữ bản sao gần real-time ở Region đích.
- A sai: TTL dài làm nội dung cũ.
- C sai: chỉ đổi phạm vi edge.
- D sai: archive.
</details>

**Câu 7.** Một distribution phục vụ `/static/*` từ S3 và `/api/*` từ ALB. Cấu hình gì?
- A. Hai distribution riêng
- B. Một distribution, hai origin, hai cache behavior theo path pattern
- C. Route 53 weighted
- D. Global Accelerator endpoint group

<details><summary>Đáp án</summary>

**B.**
- A sai: không cần thiết, phức tạp hơn.
- C sai: DNS không route theo path.
- D sai: GA không route theo path.
</details>

**Câu 8.** Ứng dụng HTTP đa Region; khách hàng doanh nghiệp yêu cầu chỉ whitelist **2 IP cố định** cho mọi Region. Chọn gì?
- A. NLB ở mỗi Region với EIP
- B. AWS Global Accelerator
- C. CloudFront
- D. Route 53 failover

<details><summary>Đáp án</summary>

**B.** GA cấp 2 anycast IP tĩnh dùng chung cho mọi endpoint ở mọi Region.
- A sai: mỗi Region có IP riêng, nhiều hơn 2.
- C sai: IP CloudFront thay đổi.
- D sai: DNS không cố định IP.
</details>

---

### Nguồn đã đối chiếu
- Restricting access to an Amazon S3 origin (OAC khuyến nghị, OAI legacy, migrate): https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html
- Restrict access with VPC origins: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-vpc-origins.html
- Using various origins with CloudFront: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/DownloadDistS3AndCustomOrigins.html
- Restrict geographic distribution: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/georestrictions.html
- Price classes: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PriceClass.html
- Invalidate files: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html
- What is AWS Global Accelerator: https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html
