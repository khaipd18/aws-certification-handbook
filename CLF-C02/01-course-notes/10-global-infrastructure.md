# CLF-C02 — Global Infrastructure

> Thuộc **Domain 3** (global infrastructure, networking & content delivery) và **Domain 1** (go global in minutes). Trọng tâm: chọn đúng dịch vụ để giảm latency và tăng khả năng chịu thảm họa toàn cầu.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Global application** = deploy ở nhiều địa lý (Region và/hoặc Edge Location) để **giảm latency**, **disaster recovery** và khó bị tấn công hơn.
- **Route 53** = DNS managed: record **A** (IPv4), **AAAA** (IPv6), **CNAME** (hostname → hostname), **Alias** (→ tài nguyên AWS). Routing: Simple, Weighted, Latency, Failover.
- **CloudFront** = **CDN**: cache nội dung ở edge, giảm latency, chống DDoS (tích hợp Shield, WAF).
- **S3 Transfer Acceleration** = tăng tốc upload/download S3 xa bằng cách đi qua edge rồi mạng riêng AWS.
- **Global Accelerator** = dùng mạng toàn cầu AWS để tăng tốc app TCP/UDP, cấp **2 Anycast IP tĩnh**, **không cache**.
- **Outposts** (rack AWS đặt tại data center của bạn), **Wavelength** (AWS ở rìa mạng **5G**), **Local Zones** (mở rộng Region tới gần người dùng ở thành phố lớn).
- Kiến trúc: Single Region/AZ → Multi-AZ → Multi-Region **Active-Passive** → Multi-Region **Active-Active**; càng về sau càng phức tạp.

---

## 2. Core Concepts

### Vì sao cần global application
- **Là gì:** **latency** = thời gian gói tin tới server; ở gần người dùng thì nhanh hơn. **DR**: khi Region gặp sự cố (động đất, bão, mất điện, chính trị) có thể failover sang Region khác.
- **Dùng khi nào:** người dùng ở nhiều châu lục, hoặc yêu cầu availability cao.
- **Vì sao quan trọng:** hạ tầng phân tán toàn cầu cũng khó bị tấn công hơn.

### Thành phần hạ tầng toàn cầu
- **Là gì:** **Region** (deploy app và hạ tầng), **AZ** (gồm nhiều data center), **Edge Location/PoP** (phân phối nội dung gần người dùng nhất).
- **Dùng khi nào:** Region/AZ cho workload; Edge cho CDN, DNS, tăng tốc mạng.
- **Vì sao quan trọng:** đề hay hỏi "dịch vụ nào dùng Edge Location": CloudFront, Route 53, Global Accelerator, S3 Transfer Acceleration.

### Amazon Route 53
- **Là gì:** **DNS managed**: tập rule/record giúp client biết cách tới server qua URL.
- **Dùng khi nào:** **A** = hostname → IPv4; **AAAA** = hostname → IPv6; **CNAME** = hostname → hostname; **Alias** = hostname → tài nguyên AWS (ELB, CloudFront, S3, RDS...).
- **Vì sao quan trọng:** Route 53 là dịch vụ **global**; dùng để định tuyến người dùng tới deployment gần nhất và hỗ trợ chiến lược DR.

### Route 53 Routing Policies
- **Là gì:** **Simple** (không health check); **Weighted** (chia % traffic, vd: 70/20/10); **Latency** (đưa người dùng tới Region có latency thấp nhất); **Failover** (health check primary, lỗi thì chuyển sang secondary).
- **Dùng khi nào:** Weighted cho thử nghiệm phiên bản mới; Latency cho người dùng toàn cầu; Failover cho DR.
- **Vì sao quan trọng:** CLF chỉ cần biết ở mức cao. `[+]` Ngoài ra còn Geolocation (theo vị trí người dùng, vd: tuân thủ luật quốc gia), Geoproximity, Multivalue answer.

### Amazon CloudFront
- **Là gì:** **CDN**: cache nội dung ở edge, tăng hiệu năng đọc, cải thiện trải nghiệm người dùng; hàng trăm PoP toàn cầu (nay 750+).
- **Dùng khi nào:** phân phối nội dung tĩnh/media cho người dùng toàn cầu.
- **Vì sao quan trọng:** bảo vệ DDoS nhờ phân tán toàn cầu, tích hợp **AWS Shield** và **AWS WAF**.

### CloudFront Origins
- **Là gì:** **S3 bucket** (phân phối và cache file; upload qua CloudFront; bảo mật bằng **Origin Access Control - OAC**); **VPC Origin** (app trong private subnet: ALB/NLB/EC2 private).
- **Dùng khi nào:** **Custom origin (HTTP)**: S3 static website, hoặc bất kỳ backend HTTP public nào (vd: ALB public).
- **Vì sao quan trọng:** OAC + bucket policy cho phép bucket **private**, chỉ CloudFront đọc được.

### CloudFront vs S3 Cross-Region Replication
- **Là gì:** **CloudFront**: mạng edge toàn cầu, file được cache theo **TTL**, hợp **nội dung tĩnh** cần có ở mọi nơi.
- **Dùng khi nào:** **S3 CRR**: phải thiết lập cho từng Region, cập nhật **gần real-time**, chỉ đọc, hợp **nội dung động** cần latency thấp ở **vài Region**.
- **Vì sao quan trọng:** bẫy so sánh kinh điển.

### S3 Transfer Acceleration
- **Là gì:** tăng tốc truyền file bằng cách gửi tới **edge location** gần nhất, rồi edge chuyển tiếp qua **mạng riêng AWS** tới bucket ở Region đích.
- **Dùng khi nào:** upload/download S3 từ xa (vd: người dùng ở Úc upload vào bucket ở Mỹ).
- **Vì sao quan trọng:** chỉ áp dụng cho **S3**, khác Global Accelerator (cho ứng dụng).

### AWS Global Accelerator
- **Là gì:** tăng availability và hiệu năng của app toàn cầu bằng **mạng nội bộ AWS**; tạo **2 Anycast IP**, traffic vào edge gần nhất rồi đi trên mạng AWS tới app.
- **Dùng khi nào:** app TCP/UDP (game, IoT, VoIP), HTTP cần **IP tĩnh** hoặc **failover nhanh, xác định giữa các Region**.
- **Vì sao quan trọng:** **không cache**: chỉ proxy packet ở edge, khác CloudFront.

### Global Accelerator vs CloudFront
- **Là gì:** cả hai dùng mạng toàn cầu và edge của AWS, cả hai tích hợp **Shield**.
- **Dùng khi nào:** **CloudFront**: nội dung **cache được** (ảnh, video), phục vụ tại edge; **Global Accelerator**: không cache, tăng tốc nhiều loại app **TCP/UDP**.
- **Vì sao quan trọng:** "static IP" hoặc "UDP" → Global Accelerator; "cache images/videos" → CloudFront.

### AWS Outposts
- **Là gì:** **rack server** chạy cùng hạ tầng, dịch vụ, API và công cụ của AWS, đặt **tại data center của bạn**; AWS lắp đặt và quản lý.
- **Dùng khi nào:** hybrid cloud cần latency thấp tới hệ thống on-prem, **xử lý dữ liệu tại chỗ**, **data residency**, migration dễ hơn. Dịch vụ chạy trên Outposts: EC2, EBS, S3, EKS, ECS, RDS, EMR.
- **Vì sao quan trọng:** **bạn chịu trách nhiệm bảo mật vật lý** của rack Outposts (đặt tại nhà bạn).

### AWS Wavelength
- **Là gì:** hạ tầng AWS đặt trong data center của **nhà mạng viễn thông** ở rìa mạng **5G** (EC2, EBS, VPC...).
- **Dùng khi nào:** app **latency cực thấp** qua 5G: smart city, chẩn đoán có ML, xe kết nối, live video tương tác, AR/VR, game real-time.
- **Vì sao quan trọng:** traffic không rời mạng nhà mạng; exam guide CLF-C02 hiện xếp Wavelength vào **out-of-scope**.

### AWS Local Zones
- **Là gì:** đặt compute, storage, database và một số dịch vụ AWS **gần người dùng cuối** ở thành phố lớn; là **phần mở rộng của Region**, mở rộng VPC tới đó.
- **Dùng khi nào:** app nhạy latency cho người dùng ở một thành phố cụ thể (vd: Region us-east-1 có Local Zones ở Boston, Chicago, Dallas...).
- **Vì sao quan trọng:** tương thích EC2, RDS, ECS, EBS, ElastiCache, Direct Connect.

### Kiến trúc ứng dụng toàn cầu
- **Là gì:** **Single Region, single AZ** (đơn giản, không HA); **Single Region, Multi-AZ** (HA, latency toàn cầu chưa tốt); **Multi-Region Active-Passive** (đọc toàn cầu nhanh, ghi vẫn về một Region).
- **Dùng khi nào:** **Multi-Region Active-Active**: đọc và ghi đều nhanh ở nhiều Region (vd: DynamoDB Global Tables).
- **Vì sao quan trọng:** độ khó tăng dần theo thứ tự trên.

### `[+]` AWS Outposts form factors
- **Là gì:** Outposts có dạng **rack** (42U) và **server** (1U/2U) cho site nhỏ.
- **Dùng khi nào:** cửa hàng bán lẻ, chi nhánh, nhà máy cần năng lực AWS tại chỗ.
- **Vì sao quan trọng:** đề chỉ cần nhận diện Outposts = "AWS on-premises".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Latency | Thời gian gói tin tới server | "reduce latency for global users" |
| Disaster Recovery (DR) | Chuyển sang Region khác khi sự cố | "regional outage" |
| Amazon Route 53 | DNS managed, global | "domain name", "DNS routing" |
| A / AAAA record | Hostname → IPv4 / IPv6 | "map domain to IP" |
| CNAME | Hostname → hostname | "map one domain to another" |
| Alias record | Hostname → tài nguyên AWS | "point domain to ELB/CloudFront" |
| Weighted routing | Chia % traffic | "send 10% traffic to new version" |
| Latency routing | Tới Region có latency thấp nhất | "lowest latency Region" |
| Failover routing | Chuyển sang secondary khi health check lỗi | "disaster recovery DNS" |
| Amazon CloudFront | CDN, cache ở edge | "cache content globally" |
| Origin Access Control (OAC) | Chỉ CloudFront được đọc S3 private | "restrict S3 to CloudFront only" |
| S3 Transfer Acceleration | Upload/download S3 qua edge | "speed up long-distance S3 uploads" |
| AWS Global Accelerator | 2 Anycast IP, mạng AWS, không cache | "static IP", "TCP/UDP", "fast regional failover" |
| AWS Outposts | Rack AWS tại data center của bạn | "AWS services on-premises" |
| AWS Wavelength | AWS ở rìa mạng 5G | "5G", "telecom", "ultra-low latency mobile" |
| AWS Local Zones | Mở rộng Region tới thành phố gần người dùng | "latency-sensitive users in a specific metro" |
| Active-Passive | Một Region nhận ghi, Region khác dự phòng/đọc | "standby Region" |
| Active-Active | Nhiều Region cùng đọc/ghi | "read/write in multiple Regions" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CloudFront | Global Accelerator | Cache nội dung vs không cache, proxy TCP/UDP, IP tĩnh | "Static IP / UDP" → Global Accelerator |
| CloudFront | S3 Transfer Acceleration | Phân phối (đọc) nội dung vs tăng tốc upload/download S3 | "Upload to S3 from far away" → Transfer Acceleration |
| CloudFront | S3 CRR | Cache theo TTL toàn cầu, nội dung tĩnh vs bản sao gần real-time ở vài Region | "Dynamic content, few Regions" → CRR |
| Outposts | Local Zones | Hạ tầng AWS **tại nhà bạn** vs AWS đặt tại thành phố gần người dùng | "In our own data center" → Outposts |
| Local Zones | Wavelength | Mở rộng Region ở thành phố vs ở rìa mạng 5G của nhà mạng | "5G" → Wavelength |
| Latency routing | Geolocation routing | Theo latency đo được vs theo vị trí người dùng | "Users in Germany must see German site" → Geolocation |
| Failover routing | Weighted routing | Chuyển khi lỗi vs chia tỉ lệ | "DR" → Failover |
| CNAME | Alias | Hostname → hostname vs tới tài nguyên AWS (dùng cả cho root domain) | "Point example.com to ELB" → Alias |
| Active-Passive | Active-Active | Ghi ở một Region vs ghi ở nhiều Region | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "cache static content close to users" | CloudFront | |
| "keep S3 bucket private, serve via CDN" | CloudFront + OAC | |
| "improve upload speed to S3 from other continents" | S3 Transfer Acceleration | |
| "two static anycast IPs, TCP/UDP game" | Global Accelerator | |
| "route users to lowest-latency Region" | Route 53 Latency routing | |
| "send 20% of traffic to new version" | Route 53 Weighted routing | |
| "fail over to DR site when primary unhealthy" | Route 53 Failover routing | |
| "run AWS services in our own data center" | AWS Outposts | |
| "ultra-low latency over 5G networks" | AWS Wavelength | Out-of-scope |
| "low latency for users in a specific large city" | AWS Local Zones | |
| "read and write in multiple Regions" | Multi-Region Active-Active | |
| "DNS service" | Route 53 | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Website có nhiều ảnh và video, người dùng ở khắp thế giới phàn nàn tải chậm. Cách hiệu quả nhất?
- A. Amazon CloudFront
- B. AWS Global Accelerator
- C. Tăng kích thước EC2
- D. Route 53 Weighted routing

<details><summary>Đáp án</summary>

**A.** CloudFront cache nội dung tĩnh/media tại edge gần người dùng.
- B sai: không cache nội dung.
- C sai: không giải quyết khoảng cách địa lý.
- D sai: chỉ chia tỉ lệ traffic.
</details>

**Câu 2.** Game online dùng UDP cần 2 IP tĩnh toàn cầu và failover nhanh giữa các Region. Chọn gì?
- A. Amazon CloudFront
- B. AWS Global Accelerator
- C. S3 Transfer Acceleration
- D. AWS Local Zones

<details><summary>Đáp án</summary>

**B.** Global Accelerator cấp 2 Anycast IP, hỗ trợ TCP/UDP, failover nhanh.
- A sai: CDN cho nội dung cache được.
- C sai: chỉ cho S3.
- D sai: mở rộng Region, không phải tăng tốc toàn cầu.
</details>

**Câu 3.** Ngân hàng phải giữ dữ liệu tại data center của mình vì quy định, nhưng muốn dùng cùng API và công cụ AWS. Chọn gì?
- A. AWS Local Zones
- B. AWS Outposts
- C. AWS Wavelength
- D. Amazon CloudFront

<details><summary>Đáp án</summary>

**B.** Outposts đưa hạ tầng AWS vào data center của khách hàng.
- A sai: Local Zones là hạ tầng của AWS ở thành phố.
- C sai: hạ tầng ở nhà mạng 5G.
- D sai: CDN.
</details>

**Câu 4.** Người dùng ở Úc upload file lớn vào bucket S3 ở Mỹ rất chậm. Cách cải thiện đơn giản nhất?
- A. Bật S3 Transfer Acceleration
- B. Dùng S3 Glacier
- C. Bật versioning
- D. Dùng Route 53 Latency routing

<details><summary>Đáp án</summary>

**A.** Transfer Acceleration đưa dữ liệu vào edge gần nhất rồi đi qua mạng AWS.
- B, C sai: không liên quan tốc độ truyền.
- D sai: DNS không tăng tốc upload.
</details>

**Câu 5.** (Chọn 2) Dịch vụ nào sử dụng mạng Edge Locations của AWS?
- A. Amazon CloudFront
- B. Amazon RDS
- C. AWS Global Accelerator
- D. Amazon EBS
- E. AWS Batch

<details><summary>Đáp án</summary>

**A, C.**
- B, D, E sai: chạy trong Region/AZ, không phải edge.
</details>

**Câu 6.** Công ty muốn đưa 10% traffic vào phiên bản mới của ứng dụng để thử nghiệm. Dùng routing policy nào của Route 53?
- A. Simple
- B. Weighted
- C. Failover
- D. Latency

<details><summary>Đáp án</summary>

**B.** Weighted chia traffic theo tỉ lệ.
- A sai: không chia tỉ lệ.
- C sai: dùng cho DR.
- D sai: theo latency.
</details>

**Câu 7.** Ứng dụng AR cho người dùng điện thoại cần latency cực thấp qua mạng 5G. Chọn gì?
- A. AWS Wavelength
- B. AWS Outposts
- C. Amazon CloudFront
- D. AWS Direct Connect

<details><summary>Đáp án</summary>

**A.** Wavelength đặt AWS ở rìa mạng 5G của nhà mạng.
- B sai: đặt tại data center khách hàng.
- C sai: CDN, không chạy compute ở rìa 5G.
- D sai: kết nối riêng từ on-prem tới AWS.
</details>

**Câu 8.** Website tĩnh trên S3 cần phục vụ qua CDN nhưng bucket phải private, chỉ CloudFront được đọc. Cấu hình gì?
- A. Bật public read cho bucket
- B. CloudFront với Origin Access Control và bucket policy tương ứng
- C. S3 Transfer Acceleration
- D. Route 53 Alias trỏ thẳng vào bucket

<details><summary>Đáp án</summary>

**B.** OAC + bucket policy giới hạn quyền đọc cho CloudFront.
- A sai: bucket thành public.
- C sai: tăng tốc truyền.
- D sai: không bảo vệ bucket.
</details>

---

### Nguồn đã đối chiếu
- CloudFront features (750+ PoP, 15 Regional Edge Caches): https://aws.amazon.com/cloudfront/features/
- Route 53 routing policies: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html
- CloudFront origin access control: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html
- AWS Global Accelerator: https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html
- AWS Outposts: https://docs.aws.amazon.com/outposts/latest/userguide/what-is-outposts.html
- AWS Local Zones: https://docs.aws.amazon.com/local-zones/latest/ug/what-is-aws-local-zones.html
- CLF-C02 out-of-scope services (AWS Wavelength): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
