# SAA-C03 — Amazon Route 53

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 3: Design High-Performing Architectures**: DNS, record, hosted zone, CNAME vs Alias, routing policy và health check.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Route 53** = DNS managed, khả dụng cao, **authoritative** (bạn sửa record được), vừa là **domain registrar**, có health check; SLA 100%.
- Record chính: **A** (IPv4), **AAAA** (IPv6), **CNAME** (tên → tên, **không** dùng cho zone apex), **NS**; mỗi record có **TTL**.
- **Hosted zone**: public (Internet) hoặc private (trong VPC); ~$0.50/tháng mỗi zone.
- **Alias** record trỏ tới tài nguyên AWS (ELB, CloudFront, API Gateway, S3 website, Beanstalk, VPC interface endpoint, Global Accelerator...), **dùng được ở zone apex**, miễn phí truy vấn, tự cập nhật IP. Không alias tới tên DNS của EC2.
- Routing policy: **Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, IP-based, Multi-value**.
- **Health check**: kiểm tra endpoint, **calculated** (gộp nhiều check), hoặc theo **CloudWatch alarm** (cho tài nguyên private).
- Domain registrar ≠ DNS service: mua domain ở nơi khác vẫn dùng Route 53 làm DNS bằng cách sửa NS record ở registrar.

---

## 2. Core Concepts

### DNS cơ bản
- **Là gì:** dịch hostname thân thiện sang IP; phân cấp root → TLD (.com) → SLD (example.com) → subdomain; resolver hỏi lần lượt và cache theo TTL.
- **Dùng khi nào:** mọi ứng dụng public cần tên miền.
- **Vì sao quan trọng:** hiểu FQDN và chuỗi phân giải giúp đọc đề về hybrid DNS.

### Records & TTL
- **Là gì:** mỗi record gồm tên, loại, giá trị, routing policy, TTL; ngoài A/AAAA/CNAME/NS còn có MX, TXT, CAA, SRV...
- **Dùng khi nào:** TTL cao → ít truy vấn, rẻ nhưng đổi chậm; TTL thấp → đổi nhanh nhưng tốn phí truy vấn.
- **Vì sao quan trọng:** trước khi chuyển IP, hạ TTL xuống thấp; TTL bắt buộc với mọi record trừ Alias.

### Hosted Zones
- **Là gì:** container chứa record của domain và subdomain; **public** cho Internet, **private** cho một hoặc nhiều VPC.
- **Dùng khi nào:** private hosted zone cho tên nội bộ (db.internal.example.com).
- **Vì sao quan trọng:** private hosted zone cần bật **enableDnsHostnames** và **enableDnsSupport** trên VPC.

### CNAME vs Alias
- **Là gì:** CNAME trỏ một tên tới tên khác, chỉ cho non-root; Alias là phần mở rộng của Route 53, trỏ thẳng tới tài nguyên AWS, kiểu A/AAAA.
- **Dùng khi nào:** zone apex (example.com) trỏ tới ALB/CloudFront → **Alias**.
- **Vì sao quan trọng:** Alias miễn phí truy vấn, có health check, **không đặt TTL**; không alias tới DNS name của EC2.

### Simple & Weighted
- **Là gì:** Simple trả một hoặc nhiều giá trị, client chọn ngẫu nhiên, không health check; Weighted chia % theo trọng số (0 = không gửi traffic).
- **Dùng khi nào:** Weighted cho blue/green, canary, phân tải giữa Region.
- **Vì sao quan trọng:** Weighted có health check; Simple thì không.

### Latency & Failover
- **Là gì:** Latency gửi user tới Region có độ trễ thấp nhất (đo giữa user và Region); Failover active-passive, primary bắt buộc có health check.
- **Dùng khi nào:** ứng dụng đa Region ưu tiên tốc độ → Latency; DR active-passive → Failover.
- **Vì sao quan trọng:** Latency không phải "gần nhất về địa lý" (Đức có thể được chuyển sang US nếu nhanh hơn).

### Geolocation, Geoproximity, IP-based
- **Là gì:** **Geolocation** theo vị trí user (châu lục, quốc gia, bang Mỹ), cần record **Default**; **Geoproximity** theo khoảng cách user-tài nguyên, chỉnh **bias** (1–99 mở rộng, −1 đến −99 thu hẹp); **IP-based** theo dải CIDR của client.
- **Dùng khi nào:** Geolocation cho localization, hạn chế nội dung theo quốc gia; Geoproximity để dịch chuyển traffic giữa Region; IP-based khi biết dải IP của ISP.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Geoproximity bắt buộc dùng Traffic Flow" → nay tạo geoproximity record trực tiếp được; Traffic Flow vẫn dùng để vẽ policy và xem bản đồ.

### Multi-Value Answer
- **Là gì:** trả tối đa 8 record **healthy** cho mỗi truy vấn.
- **Dùng khi nào:** phân tải phía client đơn giản kèm health check.
- **Vì sao quan trọng:** **không thay thế ELB**.

### Health Checks
- **Là gì:** ~15 health checker toàn cầu kiểm tra endpoint (HTTP/HTTPS/TCP); healthy khi > 18% checker báo ổn; chỉ 2xx/3xx; có thể đọc 5,120 byte đầu tìm chuỗi.
- **Dùng khi nào:** **calculated health check** gộp nhiều check bằng OR/AND/tối thiểu N; tài nguyên **private** → health check theo **CloudWatch alarm**.
- **Vì sao quan trọng:** firewall phải cho phép dải IP của health checker Route 53.

### `[+]` Route 53 Resolver & ARC
- **Là gì:** **Resolver endpoints** (inbound/outbound) cho DNS lai giữa VPC và on-premises; **Application Recovery Controller (ARC)** hỗ trợ zonal shift và routing control khi failover.
- **Dùng khi nào:** on-prem cần phân giải tên trong private hosted zone (inbound) hoặc VPC cần phân giải tên on-prem (outbound).
- **Vì sao quan trọng:** đề hybrid DNS hay chọn Resolver endpoint (chi tiết chương 23).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Authoritative DNS | Bạn kiểm soát record | "update DNS records" |
| Public hosted zone | Zone cho Internet | "public domain" |
| Private hosted zone | Zone chỉ trong VPC | "internal DNS names" |
| CNAME | Tên → tên, không cho apex | "subdomain to another hostname" |
| Alias | Trỏ tới tài nguyên AWS, dùng được ở apex | "example.com to ALB/CloudFront" |
| Zone apex | Tên gốc không có subdomain | "naked domain" |
| TTL | Thời gian resolver cache record | "reduce DNS query cost" |
| Weighted | Chia % theo trọng số | "canary 10% traffic" |
| Latency-based | Tới Region độ trễ thấp nhất | "best performance for global users" |
| Failover | Active-passive theo health check | "DR to secondary site" |
| Geolocation | Theo vị trí user | "content by country", "restrict distribution" |
| Geoproximity | Theo khoảng cách + bias | "shift more traffic to a Region" |
| IP-based | Theo CIDR của client | "route by ISP IP range" |
| Multi-value | Tối đa 8 record healthy | "client-side load balancing" |
| Calculated health check | Gộp nhiều health check | "combine child checks" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CNAME | Alias | CNAME không ở apex, tính phí truy vấn; Alias ở apex, free, chỉ tài nguyên AWS | Apex → Alias |
| Latency | Geolocation | Latency theo độ trễ; Geolocation theo vị trí user | "Fastest" vs "by country" |
| Geolocation | Geoproximity | Geolocation theo ranh giới hành chính; Geoproximity theo khoảng cách + bias | "Shift traffic" → Geoproximity |
| Multi-value | ELB | Multi-value DNS trả nhiều IP; ELB phân tải thực sự | Không thay ELB |
| Simple (nhiều giá trị) | Multi-value | Simple không health check; Multi-value có | Cần health check → Multi-value |
| Failover | Weighted | Failover active-passive; Weighted chia tỷ lệ | DR → Failover |
| Registrar | DNS service | Mua domain vs quản lý record | Đổi NS để dùng Route 53 |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "example.com (apex) to ALB" | Alias record | |
| "send 10% traffic to new version" | Weighted routing | |
| "users get lowest latency Region" | Latency-based routing | |
| "active-passive DR site" | Failover routing + health check | |
| "show French content to users in France" | Geolocation routing | Thêm Default |
| "restrict content distribution by country" | Geolocation routing | |
| "gradually shift traffic toward a Region" | Geoproximity + bias | |
| "route by client ISP CIDR" | IP-based routing | |
| "health check for private resource" | CloudWatch alarm health check | |
| "domain bought at GoDaddy, DNS on Route 53" | Tạo public hosted zone, đổi NS ở registrar | |
| "internal DNS for VPC only" | Private hosted zone | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Cần trỏ `example.com` (zone apex) tới một ALB. Record nào?
- A. CNAME
- B. Alias A record
- C. MX record
- D. NS record

<details><summary>Đáp án</summary>

**B.** Alias dùng được ở apex và trỏ thẳng tới ALB.
- A sai: CNAME không dùng cho apex.
- C sai: MX cho mail.
- D sai: NS chỉ định name server.
</details>

**Câu 2.** Ứng dụng chạy ở `us-east-1` và `ap-southeast-1`; muốn mỗi user tới Region phản hồi nhanh nhất. Chọn gì?
- A. Geolocation
- B. Latency-based
- C. Weighted
- D. Simple

<details><summary>Đáp án</summary>

**B.**
- A sai: theo vị trí, không theo độ trễ thực tế.
- C sai: chia tỷ lệ cố định.
- D sai: không định tuyến thông minh.
</details>

**Câu 3.** Công ty muốn canary 5% traffic tới phiên bản mới, 95% ở bản cũ. Chọn gì?
- A. Weighted routing
- B. Failover routing
- C. Multi-value answer
- D. Geoproximity

<details><summary>Đáp án</summary>

**A.** Đặt trọng số 5 và 95.
- B sai: active-passive.
- C sai: ngẫu nhiên trong các record healthy.
- D sai: theo khoảng cách.
</details>

**Câu 4.** (Chọn 2) Cần DR active-passive: site chính ở `us-east-1`, site phụ ở `us-west-2`, tự chuyển khi site chính lỗi. Cần gì?
- A. Failover routing policy
- B. Health check gắn với record primary
- C. Latency routing
- D. Multi-value answer
- E. TTL 7 ngày

<details><summary>Đáp án</summary>

**A, B.** Failover dựa vào health check của primary.
- C sai: không phải active-passive.
- D sai: không chỉ định thứ tự ưu tiên.
- E sai: TTL dài làm failover chậm.
</details>

**Câu 5.** Do pháp lý, nội dung chỉ được phục vụ cho user ở Đức từ endpoint riêng. Chọn gì?
- A. Latency
- B. Geolocation
- C. Weighted
- D. IP-based

<details><summary>Đáp án</summary>

**B.** Định tuyến theo quốc gia của user.
- A sai: user Đức có thể bị đưa tới Region khác.
- C sai: không theo vị trí.
- D sai: theo dải IP, không theo quốc gia một cách tin cậy.
</details>

**Câu 6.** Cần health check cho một API chạy trong private subnet, không truy cập được từ Internet. Cách làm?
- A. Health check HTTP trực tiếp tới private IP
- B. Tạo CloudWatch alarm theo metric của API và health check theo alarm đó
- C. Calculated health check không có child
- D. Mở API ra Internet

<details><summary>Đáp án</summary>

**B.** Health checker ở ngoài VPC; dùng CloudWatch alarm cho tài nguyên private.
- A sai: health checker không vào được private subnet.
- C sai: calculated cần child check.
- D sai: rủi ro bảo mật.
</details>

**Câu 7.** Domain mua ở registrar bên thứ ba; công ty muốn quản lý DNS bằng Route 53. Làm gì?
- A. Bắt buộc chuyển domain sang Route 53
- B. Tạo public hosted zone, rồi cập nhật NS record ở registrar trỏ về name server của Route 53
- C. Tạo private hosted zone
- D. Tạo CNAME từ registrar sang Route 53

<details><summary>Đáp án</summary>

**B.**
- A sai: không bắt buộc chuyển registrar.
- C sai: private zone chỉ trong VPC.
- D sai: không ủy quyền DNS bằng CNAME.
</details>

**Câu 8.** Công ty sắp đổi IP của web server và muốn user nhận IP mới nhanh nhất. Nên làm gì trước khi đổi?
- A. Tăng TTL lên 1 ngày
- B. Hạ TTL xuống vài giây/phút trước thời điểm đổi
- C. Xóa hosted zone
- D. Đổi sang Geolocation

<details><summary>Đáp án</summary>

**B.** Resolver giữ cache theo TTL; hạ TTL trước để cache hết hạn nhanh.
- A sai: user giữ IP cũ lâu hơn.
- C sai: gây mất DNS.
- D sai: không liên quan.
</details>

---

### Nguồn đã đối chiếu
- Choosing a routing policy: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html
- Geoproximity routing (bias, geoproximity record, bản đồ chỉ có trong Traffic Flow): https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-geoproximity.html
- Choosing between alias and non-alias records: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html
- How Route 53 determines health check status (18%, 2xx/3xx, 5,120 byte): https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover-determining-health-of-endpoints.html
- Private hosted zones: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-private.html
- Route 53 Resolver: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html
