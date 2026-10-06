# SOA-C03 — Amazon CloudFront & AWS Global Accelerator

> Thuộc **Domain 5: Networking and Content Delivery (18%)** và **Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization (22%)**: origin và bảo mật origin, cache key và TTL, invalidation, log và metric, xử lý mã lỗi, so sánh với Global Accelerator.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **CloudFront** là CDN: cache nội dung tại edge location gần người dùng, giảm tải origin, có sẵn Shield Standard và tích hợp WAF.
- **Ba loại origin**: S3 bucket (bảo vệ bằng **OAC**), **VPC origin** (ALB/NLB/EC2 private, không lộ ra Internet), custom HTTP origin (ALB public, S3 website endpoint…).
- **Cache key** do **cache policy** quyết định (header, cookie, query string). Càng đưa nhiều giá trị vào key thì **cache hit ratio càng thấp**.
- TTL từ 0 giây tới 1 năm, origin điều khiển bằng `Cache-Control` / `Expires`. Không dùng cache policy và origin không gửi header → mặc định **24 giờ**. Làm mới trước hạn → **invalidation**.
- **Origin Shield** thêm một lớp cache tập trung, gộp request trùng tới origin.
- CloudFront **cache cả lỗi 4xx/5xx** của origin (tối thiểu 10 giây mặc định). 403 thường do OAC/bucket policy, 404 do thiếu object, 5xx do origin.
- **Global Accelerator**: 2 IP Anycast tĩnh, proxy TCP/UDP qua mạng AWS tới endpoint theo Region, health check và failover nhanh; **không cache**.

---

## 2. Core Concepts

### CloudFront & origins
- **Là gì:** CDN với hàng trăm Point of Presence; origin là S3, VPC origin (ALB/NLB/EC2 trong private subnet), hoặc custom HTTP origin.
- **Dùng khi nào:** nội dung tĩnh, media, cả nội dung động và API (kết nối kết thúc ở edge gần người dùng).
- **Vì sao quan trọng:** ⚠️ Lỗi thời: OAI → nay dùng **Origin Access Control (OAC)** + bucket policy cho S3 private. S3 website endpoint tính là custom origin, không dùng OAC được.

### Bảo vệ origin public
- **Là gì:** ALB public làm origin thì SG của ALB phải cho phép dải IP edge của CloudFront (dùng managed prefix list của CloudFront); EC2 phía sau chỉ cho phép SG của ALB.
- **Dùng khi nào:** origin bắt buộc public.
- **Vì sao quan trọng:** đề nhấn "origin không được truy cập từ Internet" → **VPC origin**, khỏi phải quản lý dải IP.

### Cache key, cache policy, origin request policy
- **Là gì:** cache policy chọn header/cookie/query string vào cache key và TTL; origin request policy gửi giá trị tới origin **mà không đưa vào cache key**; response headers policy thêm header trả về.
- **Dùng khi nào:** origin cần thêm ngữ cảnh mà vẫn giữ hit ratio cao → origin request policy.
- **Vì sao quan trọng:** đưa `User-Agent`, cookie phiên, query string ngẫu nhiên vào key → cache bị phân mảnh. Thiết lập "forward headers" kiểu cũ là legacy.

### TTL & invalidation
- **Là gì:** TTL 0 giây–1 năm; mặc định 24 giờ khi không có cache policy và origin không gửi header. Invalidation (`CreateInvalidation`) xóa cache theo path hoặc `/*`.
- **Dùng khi nào:** cần cập nhật gấp sau khi đổi origin.
- **Vì sao quan trọng:** 1,000 path invalidation đầu tiên mỗi tháng miễn phí, sau đó tính phí. Lâu dài nên dùng **tên object có version** (`logo-v2.png`).

### Origin Shield & Geo Restriction
- **Là gì:** Origin Shield là lớp cache tập trung giữa regional edge cache và origin; Geo Restriction cho phép allowlist / blocklist theo quốc gia (dựa trên CSDL Geo-IP bên thứ ba).
- **Dùng khi nào:** origin quá tải vì nhiều edge cùng xin một object → Origin Shield; bản quyền theo quốc gia → Geo Restriction.
- **Vì sao quan trọng:** Geo-IP không tuyệt đối (VPN qua được); cần lọc chi tiết hơn → WAF.

### Logs, metrics & reports
- **Là gì:** standard logging (v2) gửi tới **CloudWatch Logs, Amazon Data Firehose hoặc S3**; real-time logs qua Kinesis Data Streams; báo cáo Cache Statistics, Popular Objects, Top Referrers, Usage, Viewers.
- **Dùng khi nào:** audit từng request, phân tích hit ratio, tìm object phổ biến.
- **Vì sao quan trọng:** metric CloudFront chỉ xem ở **us-east-1**; metric bổ sung (cache hit rate, origin latency, tỷ lệ theo mã lỗi) phải bật và tính phí. ⚠️ Kinesis Data Firehose → nay **Amazon Data Firehose**.

### Xử lý mã lỗi
- **Là gì:** CloudFront cache phản hồi lỗi 4xx/5xx từ origin; error caching minimum TTL mặc định 10 giây, chỉnh bằng custom error response.
- **Dùng khi nào:** 403 → kiểm tra OAC, bucket policy, KMS key policy (object SSE-KMS); 404 → object không tồn tại; 502/504 → origin không phản hồi, SG, TLS hoặc timeout.
- **Vì sao quan trọng:** lỗi origin chập chờn có thể "dính" ở edge do bị cache. Cần sẵn sàng cao → **origin group** (failover sang origin phụ theo mã lỗi).

### CloudFront + ALB sticky sessions
- **Là gì:** ALB stickiness dùng cookie `AWSALB`; CloudFront phải chuyển cookie này tới origin.
- **Dùng khi nào:** ứng dụng giữ session trên instance và đặt sau CloudFront.
- **Vì sao quan trọng:** TTL của nội dung phải ngắn hơn thời hạn cookie xác thực, nếu không có thể trả nội dung cũ đã xác thực.

### CloudFront vs S3 Cross-Region Replication
- **Là gì:** CloudFront cache toàn cầu theo TTL; CRR sao chép gần real-time sang từng Region đích, bản replica chỉ đọc.
- **Dùng khi nào:** nội dung tĩnh cho người dùng khắp nơi → CloudFront; dữ liệu động cần độ trễ thấp ở vài Region cụ thể → CRR.
- **Vì sao quan trọng:** hai đáp án này hay bị đặt cạnh nhau.

### AWS Global Accelerator
- **Là gì:** 2 IP Anycast tĩnh, đưa traffic vào edge gần nhất rồi đi trên mạng AWS tới endpoint (Elastic IP, EC2, ALB, NLB, public hoặc private), có health check.
- **Dùng khi nào:** UDP (game), IoT MQTT, VoIP, yêu cầu IP tĩnh để whitelist, failover giữa Region nhanh, không phụ thuộc DNS cache.
- **Vì sao quan trọng:** **không cache**. HTTP và nội dung cache được → CloudFront.

### `[+]` HTTPS, chứng chỉ, signed URL
- **Là gì:** viewer protocol policy (HTTP-only / redirect / HTTPS-only) và origin protocol policy; domain tùy chỉnh cần **certificate ACM ở us-east-1**; signed URL (một file) và signed cookie (nhiều file) dùng trusted key group.
- **Dùng khi nào:** nội dung trả phí, private; nhiều domain trên một distribution qua SNI.
- **Vì sao quan trọng:** certificate ở Region khác không gắn được vào distribution. S3 presigned URL thì bỏ qua CloudFront.

### `[+]` CloudFront Functions vs Lambda@Edge
- **Là gì:** CloudFront Functions chạy JavaScript nhẹ tại edge location, chỉ viewer request/response, giá khoảng 1/6 Lambda@Edge; Lambda@Edge chạy cả bốn trigger (viewer/origin request và response), gọi mạng được, đọc được body.
- **Dùng khi nào:** rewrite URL, thêm header, redirect đơn giản → CloudFront Functions; cần gọi dịch vụ khác, xử lý body → Lambda@Edge.
- **Vì sao quan trọng:** đề hỏi "độ trễ thấp nhất, rẻ nhất cho thao tác header" → CloudFront Functions.

### `[+]` Price class & flat-rate pricing plans
- **Là gì:** price class (All / 200 / 100) giới hạn nhóm edge để giảm chi phí; ngoài pay-as-you-go nay có **flat-rate pricing plans** (gộp CloudFront, WAF, chống DDoS… theo giá cố định tháng).
- **Dùng khi nào:** người dùng tập trung một vùng → price class thấp hơn; muốn chi phí CDN cố định, không phí vượt do DDoS bị chặn → flat-rate plan.
- **Vì sao quan trọng:** người xem ngoài price class vẫn được phục vụ, chỉ từ edge xa hơn. Mức plan cụ thể: kiểm tra trang pricing/console.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| OAC | Cho CloudFront đọc S3 private | "restrict S3 access to CloudFront only" |
| VPC origin | Origin trong private subnet | "origin not reachable from internet" |
| Cache policy | Quy định cache key + TTL | "low cache hit ratio" |
| Origin request policy | Gửi giá trị tới origin, không vào key | "origin needs header, keep hit ratio" |
| Invalidation | Xóa cache trước TTL | "updated file not showing" |
| Origin Shield | Lớp cache tập trung | "origin overloaded by edges" |
| Geo Restriction | Allow/block theo quốc gia | "licensing by country" |
| Origin group | Failover origin | "stay available when origin returns 5xx" |
| Signed URL / cookie | Truy cập giới hạn thời gian | "paid content", "private files" |
| CloudFront Functions | Code nhẹ tại edge | "header manipulation at lowest latency" |
| Lambda@Edge | Code đầy đủ hơn, gọi mạng | "call external service at edge" |
| Global Accelerator | 2 IP Anycast, TCP/UDP | "static IPs", "UDP", "fast regional failover" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CloudFront | Global Accelerator | Cache HTTP tại edge; proxy TCP/UDP, IP tĩnh, không cache | UDP / static IP → GA |
| OAC | OAI | Cơ chế mới, hỗ trợ SSE-KMS; cơ chế cũ | Đáp án mới → OAC |
| S3 REST origin | S3 website endpoint | Dùng OAC; là custom origin, không OAC | Website hosting → custom origin |
| Cache policy | Origin request policy | Vào cache key; chỉ gửi tới origin | Giữ hit ratio → origin request policy |
| Invalidation | Versioned object name | Tốn phí, chậm khi nhiều; đổi tên là xong | Lâu dài → versioning |
| CloudFront | S3 CRR | Cache theo TTL toàn cầu; bản sao gần real-time ở Region chọn | Vài Region, dữ liệu động → CRR |
| Signed URL | Signed cookie | Một file; nhiều file | Nhiều file → cookie |
| CloudFront Functions | Lambda@Edge | Nhẹ, viewer event; đủ bốn trigger, gọi mạng | Gọi API ngoài → Lambda@Edge |
| Geo Restriction | WAF geo match | Theo quốc gia toàn distribution; rule chi tiết | Kết hợp điều kiện → WAF |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "S3 accessible only through CloudFront" | OAC + bucket policy | |
| "private ALB as origin, no internet exposure" | VPC origin | |
| "low cache hit ratio" | Bớt header/cookie/query string trong cache key | |
| "updated object still served" | Invalidation hoặc tên có version | |
| "origin overloaded by many edge locations" | Origin Shield | |
| "block viewers from certain countries" | Geo Restriction | |
| "403 from CloudFront with S3 origin" | OAC / bucket policy / KMS key policy | |
| "origin returns 5xx, must stay available" | Origin group failover | |
| "cannot find CloudFront metrics in eu-west-1" | Xem ở us-east-1 | |
| "custom domain HTTPS on CloudFront" | Certificate ACM ở us-east-1 | |
| "gaming UDP, static IPs, global" | Global Accelerator | |
| "customers whitelist exactly two IPs" | Global Accelerator | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Bucket S3 chỉ được truy cập qua CloudFront, không cho truy cập trực tiếp. Cấu hình nào?
- A. Bật S3 website hosting và public bucket
- B. Origin Access Control + bucket policy chỉ cho phép CloudFront
- C. Geo Restriction
- D. Signed URL từ S3

<details><summary>Đáp án</summary>

**B.**
- A sai: public bucket cho phép truy cập trực tiếp.
- C sai: lọc theo quốc gia.
- D sai: S3 presigned URL bỏ qua CloudFront.
</details>

**Câu 2.** Cache hit ratio rất thấp. Cache policy đang đưa toàn bộ header, cookie và query string vào cache key. Cách cải thiện?
- A. Tăng số edge location
- B. Chỉ giữ giá trị thật sự ảnh hưởng nội dung trong cache key; giá trị origin cần thì gửi qua origin request policy
- C. Bật Geo Restriction
- D. Giảm TTL về 0

<details><summary>Đáp án</summary>

**B.**
- A sai: không điều khiển được, cũng không giải quyết phân mảnh key.
- C sai: không liên quan.
- D sai: TTL 0 làm hit ratio thấp thêm.
</details>

**Câu 3.** Đội vận hành đặt alarm cho metric CloudFront ở `eu-west-1` nhưng không thấy metric. Vì sao?
- A. Chưa bật standard logging
- B. Metric CloudFront chỉ có ở us-east-1
- C. Phải bật Origin Shield
- D. Phải dùng price class All

<details><summary>Đáp án</summary>

**B.**
- A sai: log khác metric.
- C, D sai: không ảnh hưởng nơi publish metric.
</details>

**Câu 4.** Sau khi chuyển S3 sang mã hóa SSE-KMS, CloudFront (dùng OAC) trả 403. Nguyên nhân khả dĩ nhất?
- A. TTL quá dài
- B. Key policy KMS chưa cho phép CloudFront dùng key để giải mã
- C. Thiếu Geo Restriction
- D. Origin Shield chưa bật

<details><summary>Đáp án</summary>

**B.**
- A sai: TTL không gây 403.
- C, D sai: không liên quan quyền.
</details>

**Câu 5.** (Chọn 2) Game online dùng UDP, người chơi toàn cầu; đối tác cần whitelist số IP ít nhất có thể; cần failover giữa Region trong chưa đầy một phút. Hai phát biểu nào đúng?
- A. Dùng Global Accelerator
- B. Dùng CloudFront
- C. Global Accelerator cấp 2 IP Anycast tĩnh
- D. CloudFront hỗ trợ UDP
- E. Route 53 latency record là cách duy nhất

<details><summary>Đáp án</summary>

**A, C.**
- B, D sai: CloudFront dành cho HTTP(S), không UDP.
- E sai: DNS phụ thuộc cache phía client, failover chậm hơn và không cho IP tĩnh.
</details>

**Câu 6.** Cập nhật `index.html` trên S3 nhưng người dùng vẫn thấy bản cũ, TTL 24 giờ. Cần sửa ngay. Làm gì?
- A. Tạo invalidation cho `/index.html`
- B. Xóa distribution
- C. Bật versioning trên bucket
- D. Đổi price class

<details><summary>Đáp án</summary>

**A.**
- B sai: gây gián đoạn.
- C sai: S3 versioning không làm mới cache CloudFront.
- D sai: không liên quan.
</details>

**Câu 7.** Ứng dụng dùng ALB làm origin; muốn ALB không thể truy cập từ Internet. Chọn gì?
- A. ALB internet-facing, SG mở 0.0.0.0/0
- B. CloudFront VPC origin trỏ tới ALB internal
- C. Geo Restriction
- D. Signed cookies

<details><summary>Đáp án</summary>

**B.**
- A sai: ALB lộ ra Internet.
- C sai: chỉ lọc theo quốc gia tại CloudFront.
- D sai: kiểm soát người xem, không ẩn origin.
</details>

**Câu 8.** Cần thêm security header vào mọi response với độ trễ và chi phí thấp nhất. Chọn gì?
- A. Lambda@Edge origin response
- B. CloudFront Functions viewer response (hoặc response headers policy)
- C. Lambda trong Region gọi qua API Gateway
- D. Global Accelerator

<details><summary>Đáp án</summary>

**B.** Thao tác header đơn giản, chạy tại edge location, rẻ hơn Lambda@Edge.
- A sai: chạy được nhưng đắt và chậm hơn.
- C sai: thêm hop không cần thiết.
- D sai: không xử lý HTTP header.
</details>

---

### Nguồn đã đối chiếu
- Restrict access to S3 origin (OAC): https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html
- VPC origins: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-vpc-origins.html
- Cache expiration (TTL mặc định 24 giờ): https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Expiration.html
- Error caching (minimum TTL 10 giây): https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HTTPStatusCodes.html
- Paying for invalidation (1,000 path miễn phí mỗi tháng): https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PayingForInvalidation.html
- Standard logging v2: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/standard-logging.html
- CloudFront metrics (us-east-1): https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/viewing-cloudfront-metrics.html
- Edge function restrictions: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-restrictions.html
- Flat-rate pricing plans: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/flat-rate-pricing-plan.html
