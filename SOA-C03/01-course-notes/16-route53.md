# SOA-C03 — Amazon Route 53

> Thuộc **Domain 5: Networking and Content Delivery (18%)** và **Domain 2: Reliability and Business Continuity (22%)**: record và hosted zone, Alias, routing policy, health check, Resolver cho DNS lai, DNS Firewall, phục hồi ứng dụng.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- Route 53 là **DNS authoritative managed** + **domain registrar** + **health check**; SLA 100%.
- **Alias vs CNAME**:
  - **Alias** dùng được ở **zone apex**, miễn phí, trỏ tới tài nguyên AWS, không đặt TTL.
  - **CNAME** không dùng ở apex.
- **Không tạo Alias tới DNS name của EC2**. Đích hợp lệ: ELB, CloudFront, API Gateway, Elastic Beanstalk, S3 website, VPC interface endpoint, Global Accelerator, record cùng hosted zone.
- **Hosted zone**: public (Internet) và private (chỉ VPC được associate); tính phí theo tháng.
- **Routing policy**: Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, Multivalue, IP-based.
  - ⚠️ Geoproximity nay tạo trực tiếp trong hosted zone, **không bắt buộc Traffic Flow**.
- **Health check** có ba loại: theo endpoint (chỉ public), calculated (gộp check con), theo **CloudWatch alarm** (cách duy nhất cho tài nguyên private).
- Ràng buộc với health check:
  - Simple không gắn health check được.
  - Failover cần health check cho primary.
  - Multivalue trả tối đa 8 record healthy, không thay ELB.
- **Resolver**:
  - **Inbound endpoint**: on-prem hỏi tên trong AWS.
  - **Outbound endpoint** + rule: AWS hỏi tên on-prem.
  - **DNS Firewall**: chặn truy vấn ra domain độc hại.

---

## 2. Core Concepts

### DNS, record & TTL
- **Là gì:** DNS phân cấp root → TLD → SLD → subdomain; record gồm name, type (A, AAAA, CNAME, NS, MX, TXT…), value, routing policy, TTL.
- **Dùng khi nào:** TTL cao giảm truy vấn và chi phí nhưng chậm đổi; TTL thấp đổi nhanh, tốn truy vấn hơn.
- **Vì sao quan trọng:** hạ TTL **trước** khi đổi record hoặc failover có kế hoạch. Alias không cần (không được) đặt TTL.

### Hosted zones
- **Là gì:** public hosted zone cho tên trên Internet; private hosted zone chỉ phân giải trong VPC đã associate.
- **Dùng khi nào:** tên nội bộ cho dịch vụ trong VPC; split-view DNS (cùng tên miền, nội bộ trả IP private, bên ngoài trả IP public).
- **Vì sao quan trọng:** record private không phân giải được → thiếu VPC association hoặc tắt `enableDnsSupport` / `enableDnsHostnames`. Associate cross-account phải cấp quyền ở cả hai phía.

### Alias vs CNAME
- **Là gì:** CNAME trỏ hostname → hostname bất kỳ, không dùng ở apex. Alias là A/AAAA trỏ tới tài nguyên AWS, tự theo IP thay đổi, có evaluate target health.
- **Dùng khi nào:** `example.com` trỏ tới ALB/CloudFront → Alias.
- **Vì sao quan trọng:** query tới Alias trỏ tài nguyên AWS không tính phí. **EC2 DNS name không làm đích Alias** được.

### Simple, Weighted, Latency
- **Là gì:** Simple trả một (hoặc nhiều giá trị, client chọn ngẫu nhiên); Weighted chia theo trọng số tương đối; Latency chọn Region có độ trễ thấp nhất tới người dùng.
- **Dùng khi nào:** canary/blue-green → Weighted (weight 0 dừng traffic); hiệu năng toàn cầu → Latency.
- **Vì sao quan trọng:** Simple **không có health check**. Latency theo độ trễ đo được, không theo địa lý (người Đức có thể vào Region Mỹ).

### Failover, Geolocation, Geoproximity
- **Là gì:** Failover active-passive (primary bắt buộc có health check); Geolocation theo châu lục/quốc gia/bang (khớp chi tiết nhất thắng); Geoproximity theo khoảng cách với **bias** (1 tới 99 mở rộng, -1 tới -99 thu hẹp), đích là Region AWS hoặc tọa độ.
- **Dùng khi nào:** DR active-passive → Failover; bản địa hóa, bản quyền, quy định → Geolocation; dịch chuyển dần traffic theo vùng → Geoproximity.
- **Vì sao quan trọng:** Geolocation phải có **Default record** cho vị trí không khớp. ⚠️ Lỗi thời: "Geoproximity bắt buộc Traffic Flow" → nay tạo được như record thường.

### Multivalue, IP-based, Traffic Flow
- **Là gì:** Multivalue trả tối đa 8 record healthy cho mỗi truy vấn; IP-based routing theo CIDR collection của nguồn; Traffic Flow là editor trực quan tạo traffic policy có version.
- **Dùng khi nào:** phân phối phía client đơn giản có loại endpoint lỗi → Multivalue; người dùng một ISP vào endpoint riêng → IP-based; cây routing phức tạp nhiều tầng → Traffic Flow.
- **Vì sao quan trọng:** Multivalue không phải load balancer, không cân bằng theo kết nối.

### Health checks
- **Là gì:** khoảng 15 checker toàn cầu kiểm tra HTTP/HTTPS/TCP; mặc định interval 30 giây (10 giây tính thêm phí), failure threshold 3; healthy khi **hơn 18%** checker báo healthy; chỉ 2xx/3xx là pass; string matching trong 5,120 byte đầu.
- **Dùng khi nào:** calculated health check gộp tới 256 check con bằng AND/OR/NOT; tài nguyên private hoặc on-prem → health check theo **CloudWatch alarm**.
- **Vì sao quan trọng:** checker nằm ngoài VPC → firewall/SG phải mở dải IP health checker của Route 53. Health check gửi metric và alarm được.

### Registrar & S3 website
- **Là gì:** registrar bán tên miền; DNS service lưu record. Mua domain ở nơi khác vẫn dùng Route 53 được bằng cách đổi NS record ở registrar sang name server của hosted zone.
- **Dùng khi nào:** website tĩnh trên S3: tên bucket **trùng** tên record, bật website hosting, Alias A tới S3 website endpoint.
- **Vì sao quan trọng:** S3 website chỉ HTTP; muốn HTTPS → CloudFront phía trước.

### Route 53 Resolver & hybrid DNS
- **Là gì:** Resolver (địa chỉ VPC CIDR + 2, hoặc 169.254.169.253) phân giải tên EC2, private hosted zone, tên public. **Inbound endpoint** nhận truy vấn từ on-prem; **outbound endpoint** + **forwarding rule** gửi truy vấn domain cụ thể về DNS on-prem.
- **Dùng khi nào:** DNS lai qua Direct Connect/VPN; tài khoản trung tâm share Resolver rule cho nhiều account bằng AWS RAM.
- **Vì sao quan trọng:** nhớ chiều: inbound = truy vấn **từ** mạng của bạn **vào** AWS; outbound = AWS hỏi **ra** on-prem.

### Query logging & DNS Firewall
- **Là gì:** public DNS query logging (chỉ public hosted zone, tới CloudWatch Logs); **Resolver query logging** ghi truy vấn từ tài nguyên trong VPC tới CloudWatch Logs, S3 hoặc Data Firehose; **DNS Firewall** chặn/cho phép domain theo danh sách.
- **Dùng khi nào:** điều tra workload bị xâm nhập gọi domain lạ; chặn DNS exfiltration.
- **Vì sao quan trọng:** DNS Firewall quản lý tập trung bằng Firewall Manager; cấu hình query logging share được qua RAM.

### Application Recovery Controller & Route 53 Profiles
- **Là gì:** ARC có readiness check, routing control (công tắc bật/tắt traffic), **zonal shift** (tránh AZ suy giảm), region switch; Profiles chuẩn hóa private hosted zone, Resolver rule, DNS Firewall rule group cho nhiều VPC/account.
- **Dùng khi nào:** ứng dụng mission-critical cần chuyển traffic khỏi AZ/Region có kiểm soát; nhiều VPC cần cùng cấu hình DNS.
- **Vì sao quan trọng:** Profiles share qua AWS RAM. ❓ Phạm vi tính năng ARC thay đổi nhanh; kiểm tra docs khi cần chi tiết.

### `[+]` DNSSEC
- **Là gì:** ký số record của public hosted zone (KSK dựa trên customer managed KMS key asymmetric `ECC_NIST_P256` ở us-east-1) để resolver xác thực câu trả lời.
- **Dùng khi nào:** yêu cầu chống DNS spoofing / cache poisoning.
- **Vì sao quan trọng:** phải thêm bản ghi DS ở zone cha/registrar; KMS key phải ở **us-east-1**.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Alias | Record trỏ tài nguyên AWS, dùng ở apex | "zone apex to ALB" |
| CNAME | Hostname → hostname, không ở apex | "www to another domain" |
| Private hosted zone | DNS chỉ trong VPC | "internal names" |
| Weighted | Chia traffic theo trọng số | "canary 10%" |
| Latency | Region độ trễ thấp nhất | "best performance globally" |
| Failover | Active-passive theo health check | "DR to secondary Region" |
| Geolocation | Theo vị trí người dùng | "users in France see French site" |
| Geoproximity + bias | Theo khoảng cách, chỉnh vùng | "shift traffic gradually between Regions" |
| Multivalue | Tối đa 8 record healthy | "client-side, health-checked" |
| Calculated health check | Gộp check con | "healthy if 2 of 3 healthy" |
| Inbound Resolver endpoint | On-prem → AWS DNS | "on-prem resolve private zone" |
| Outbound endpoint + rule | AWS → on-prem DNS | "resolve corp.local from VPC" |
| DNS Firewall | Chặn domain độc hại | "DNS exfiltration" |
| Zonal shift | Chuyển traffic khỏi AZ | "impaired AZ" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Alias | CNAME | Apex, miễn phí, tài nguyên AWS; không apex, hostname bất kỳ | Apex → Alias |
| Public hosted zone | Private hosted zone | Internet; VPC đã associate | "Internal only" → private |
| Latency | Geolocation | Độ trễ đo được; vị trí người dùng | Bản quyền/ngôn ngữ → Geolocation |
| Geolocation | Geoproximity | Theo vùng hành chính; theo khoảng cách + bias | "Bias" → Geoproximity |
| Failover | Weighted | Active-passive; chia theo trọng số | DR → Failover |
| Multivalue | ELB | Client chọn từ ≤ 8 IP; cân bằng thật | Load balancing → ELB |
| Simple nhiều giá trị | Multivalue | Không health check; có health check | Loại IP lỗi → Multivalue |
| Endpoint health check | CloudWatch alarm health check | Chỉ endpoint public; mọi metric, cả private | Private → alarm-based |
| Inbound endpoint | Outbound endpoint | On-prem → AWS; AWS → on-prem | Theo hướng truy vấn |
| Route 53 failover | Global Accelerator | Phụ thuộc DNS TTL; IP Anycast, không DNS cache | Failover nhanh, IP tĩnh → GA |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "map example.com to ALB" | Alias A record | CNAME không dùng ở apex |
| "map name to EC2 public DNS" | A record với IP / CNAME (không Alias) | |
| "send 10% to new version" | Weighted | |
| "lowest latency for global users" | Latency + health check | |
| "active-passive DR" | Failover + health check primary | |
| "content restricted by country" | Geolocation + Default record | |
| "health check private RDS / on-prem server" | Health check theo CloudWatch alarm | |
| "health check fails though app works" | SG/firewall chưa mở IP health checker | |
| "private hosted zone not resolving" | VPC association, DNS attributes | |
| "on-prem servers resolve AWS private names" | Inbound Resolver endpoint | |
| "EC2 resolve on-prem domain" | Outbound endpoint + forwarding rule | |
| "block malicious domains from VPC" | Route 53 Resolver DNS Firewall | |
| "move traffic away from a bad AZ" | ARC zonal shift | |
| "domain registered at another registrar" | Đổi NS ở registrar | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Cần trỏ `example.com` (apex) tới ALB. Chọn record nào?
- A. CNAME tới DNS name của ALB
- B. Alias A tới ALB
- C. A record với IP hiện tại của ALB
- D. MX record

<details><summary>Đáp án</summary>

**B.**
- A sai: CNAME không dùng ở apex.
- C sai: IP của ALB thay đổi.
- D sai: dành cho email.
</details>

**Câu 2.** Route 53 failover cần kiểm tra sức khỏe một DB chỉ có IP private trong VPC. Cách làm?
- A. Health check endpoint tới IP private
- B. CloudWatch alarm trên metric của DB, health check theo alarm đó
- C. Calculated health check không có check con
- D. Simple routing

<details><summary>Đáp án</summary>

**B.** Health checker ở ngoài VPC, không tới được IP private.
- A sai: không truy cập được.
- C sai: calculated cần check con.
- D sai: Simple không có health check.
</details>

**Câu 3.** (Chọn 2) Health check báo unhealthy dù ứng dụng phản hồi bình thường từ trình duyệt. Hai nguyên nhân khả dĩ?
- A. SG/NACL chưa cho phép dải IP health checker của Route 53
- B. Endpoint trả mã 3xx
- C. Endpoint trả 5xx với path được kiểm tra hoặc chuỗi tìm không nằm trong 5,120 byte đầu
- D. TTL record quá cao
- E. Hosted zone là public

<details><summary>Đáp án</summary>

**A, C.**
- B sai: 2xx và 3xx đều pass.
- D sai: TTL không ảnh hưởng health check.
- E sai: health check endpoint dùng cho tài nguyên public là bình thường.
</details>

**Câu 4.** Server on-prem (qua Direct Connect) cần phân giải `db.internal.example` trong private hosted zone. Chọn gì?
- A. Outbound Resolver endpoint
- B. Inbound Resolver endpoint, DNS on-prem forward tới IP của endpoint
- C. Public hosted zone
- D. DNS Firewall

<details><summary>Đáp án</summary>

**B.**
- A sai: chiều ngược lại (AWS → on-prem).
- C sai: lộ tên nội bộ ra Internet.
- D sai: lọc truy vấn, không phân giải.
</details>

**Câu 5.** Người dùng ở Đức thỉnh thoảng được đưa tới Region Mỹ dù có Region Frankfurt. Policy đang dùng là gì?
- A. Geolocation
- B. Latency-based
- C. Failover
- D. Simple

<details><summary>Đáp án</summary>

**B.** Latency chọn theo độ trễ đo được, không theo địa lý.
- A sai: Geolocation sẽ cố định theo quốc gia.
- C sai: chỉ chuyển khi primary lỗi.
- D sai: không chọn theo vị trí.
</details>

**Câu 6.** Cần chuyển 5% traffic sang phiên bản mới ở một ALB khác rồi tăng dần. Chọn gì?
- A. Weighted routing
- B. Geolocation
- C. Multivalue
- D. Failover

<details><summary>Đáp án</summary>

**A.**
- B sai: theo vị trí, không theo tỷ lệ.
- C sai: không điều khiển tỷ lệ.
- D sai: chỉ dùng khi primary lỗi.
</details>

**Câu 7.** (Chọn 2) Đội bảo mật muốn chặn workload trong VPC truy vấn domain độc hại và lưu lại mọi truy vấn DNS để điều tra. Chọn gì?
- A. Route 53 Resolver DNS Firewall
- B. Resolver query logging tới S3/CloudWatch Logs
- C. Public DNS query logging
- D. Weighted routing
- E. Security group outbound chặn port 53

<details><summary>Đáp án</summary>

**A, B.**
- C sai: chỉ ghi truy vấn tới public hosted zone của bạn.
- D sai: routing.
- E sai: chặn hết DNS, làm hỏng phân giải hợp lệ.
</details>

**Câu 8.** Record trong private hosted zone không phân giải được từ EC2 trong VPC mới. Kiểm tra gì trước?
- A. VPC đã associate với private hosted zone chưa, và `enableDnsSupport` / `enableDnsHostnames` đã bật chưa
- B. Bucket S3 có public không
- C. TTL record
- D. Health check

<details><summary>Đáp án</summary>

**A.**
- B sai: không liên quan.
- C sai: TTL không khiến record không phân giải.
- D sai: health check không chặn phân giải record không gắn health check.
</details>

---

### Nguồn đã đối chiếu
- Geoproximity routing: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-geoproximity.html
- Values for geoproximity records (tạo trực tiếp trong hosted zone): https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-values-geoprox.html
- How health checks determine health (18%): https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover-determining-health-of-endpoints.html
- Multivalue answer routing (tối đa 8 record): https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-multivalue.html
- Choosing between alias and non-alias records: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html
- Resolver endpoints & forwarding rules: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html
- DNS Firewall: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver-dns-firewall.html
- DNSSEC signing: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring-dnssec.html
- DNSSEC KMS key (us-east-1, ECC_NIST_P256): https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring-dnssec-cmk-requirements.html
