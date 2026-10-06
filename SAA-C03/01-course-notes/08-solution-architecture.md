# SAA-C03 — Classic Solutions Architecture & Elastic Beanstalk

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 4: Design Cost-Optimized Architectures**: tiến hóa kiến trúc qua các case study (WhatIsTheTime, MyClothes, MyWordPress), cách khởi tạo ứng dụng nhanh và Elastic Beanstalk.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- Kiến trúc tiến hóa: **1 EC2 + EIP** → scale dọc → nhiều EC2 + DNS A record → **ALB + ASG Multi-AZ** → thêm **reserved capacity** để tiết kiệm.
- **Stateless web app** (WhatIsTheTime): không lưu trạng thái, chỉ cần ALB + ASG Multi-AZ + Route 53 Alias.
- **Stateful web app** (MyClothes giỏ hàng): session → **sticky sessions** hoặc **cookie phía client** hoặc tốt nhất **ElastiCache/DynamoDB**; dữ liệu user → **RDS**, đọc nhiều → **read replica / ElastiCache**; Multi-AZ cho HA.
- **Bảo mật theo tầng**: SG của ALB mở HTTP/HTTPS; SG của EC2 chỉ nhận từ SG ALB; SG của RDS/ElastiCache chỉ nhận từ SG EC2.
- **Ảnh/file dùng chung** (MyWordPress): EBS chỉ một instance → dùng **EFS** cho nhiều instance ở nhiều AZ.
- Khởi tạo nhanh: **Golden AMI**, **user data** cho cấu hình động, kết hợp cả hai; RDS/EBS restore từ **snapshot**.
- **Elastic Beanstalk**: PaaS-like, developer chỉ đưa code; Beanstalk tự dựng EC2, ASG, ELB, RDS...; **dịch vụ miễn phí**, chỉ trả tài nguyên bên dưới. Có **Web Server tier** và **Worker tier** (SQS).

---

## 2. Core Concepts

### WhatIsTheTime.com (stateless)
- **Là gì:** app trả giờ hiện tại, không cần DB; bắt đầu bằng 1 EC2 + Elastic IP, sau đó scale dọc (downtime), rồi scale ngang.
- **Dùng khi nào:** tránh quản lý IP thủ công: Route 53 A record TTL ngắn → thay bằng **ALB + health check** + Route 53 **Alias** → **ASG** → **Multi-AZ**.
- **Vì sao quan trọng:** dùng Reserved Instance cho capacity tối thiểu, On-Demand/Spot cho phần tăng thêm để tối ưu chi phí.

### MyClothes.com (stateful)
- **Là gì:** web bán quần áo, có giỏ hàng và thông tin user, hàng trăm user đồng thời.
- **Dùng khi nào:** giỏ hàng: **sticky sessions** (đơn giản nhưng lệch tải) → **cookie client** (stateless nhưng request nặng, giới hạn 4 KB, có thể bị sửa) → **server session** trong ElastiCache (session ID trong cookie).
- **Vì sao quan trọng:** dữ liệu user lưu RDS; đọc nhiều thì **read replica** hoặc **ElastiCache lazy loading**; RDS Multi-AZ + ElastiCache Multi-AZ cho HA.

### Security group chaining
- **Là gì:** mỗi tầng chỉ nhận traffic từ SG của tầng trước.
- **Dùng khi nào:** ALB (0.0.0.0/0:443) → EC2 (từ SG ALB) → RDS/ElastiCache (từ SG EC2).
- **Vì sao quan trọng:** mẫu "3-tier architecture" chuẩn; đề hỏi cách hạn chế truy cập DB.

### MyWordPress.com
- **Là gì:** blog WordPress lưu ảnh upload và dữ liệu MySQL.
- **Dùng khi nào:** DB → **Aurora MySQL** Multi-AZ + read replica; ảnh → **EFS** để mọi instance trong mọi AZ thấy cùng file.
- **Vì sao quan trọng:** EBS chỉ gắn một instance → ảnh chỉ có trên instance đã nhận upload; EFS giải quyết (đắt hơn EBS, cân nhắc S3 + CloudFront).

### Khởi tạo ứng dụng nhanh
- **Là gì:** EC2 → **Golden AMI** (cài sẵn app, OS deps) + **user data** (cấu hình động lúc boot) hoặc kết hợp (Beanstalk); RDS/EBS → restore từ snapshot (dữ liệu, schema có sẵn).
- **Dùng khi nào:** cần instance sẵn sàng phục vụ nhanh khi ASG scale out.
- **Vì sao quan trọng:** cài mọi thứ bằng user data làm scale out chậm.

### Elastic Beanstalk
- **Là gì:** dịch vụ managed giúp triển khai web app; tự lo capacity, load balancing, scaling, health monitoring, cấu hình instance; vẫn cho quyền chỉnh tài nguyên bên dưới.
- **Dùng khi nào:** developer muốn đưa code lên nhanh với kiến trúc chuẩn (ELB + ASG + RDS); hỗ trợ Go, Java, .NET, Node.js, PHP, Python, Ruby, Docker...
- **Vì sao quan trọng:** Beanstalk miễn phí; thành phần: **Application**, **Application Version**, **Environment** (dev/test/prod).

### Beanstalk tiers & deployment mode
- **Là gì:** **Web Server tier** (ELB + ASG xử lý request HTTP); **Worker tier** (EC2 kéo message từ **SQS** để xử lý nền, scale theo số message).
- **Dùng khi nào:** Web tier đẩy job dài vào SQS cho Worker tier xử lý.
- **Vì sao quan trọng:** mode **Single Instance** (dev, có EIP) vs **High Availability with Load Balancer** (production).

### `[+]` Session lưu ở DynamoDB
- **Là gì:** DynamoDB cũng là nơi lưu session phổ biến (serverless, TTL tự xóa session cũ).
- **Dùng khi nào:** không muốn quản lý cluster cache; cần độ bền dữ liệu session.
- **Vì sao quan trọng:** đề có thể cho cả ElastiCache và DynamoDB là đáp án hợp lệ cho session store.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Stateless app | Không lưu trạng thái trên instance | "any instance can serve any request" |
| Stateful app | Có session/giỏ hàng | "shopping cart", "user session" |
| Sticky sessions | Gắn user vào một instance | "session lost when instance changes" |
| Client-side cookie | Lưu dữ liệu ở cookie trình duyệt | "< 4 KB", "may be altered" |
| Server-side session | Session ID ở cookie, dữ liệu ở cache/DB | "ElastiCache session store" |
| SG chaining | SG tầng sau chỉ nhận từ SG tầng trước | "restrict DB access to app tier" |
| Golden AMI | AMI cài sẵn app | "reduce boot time" |
| Bootstrap (user data) | Cấu hình lúc boot | "dynamic configuration" |
| Elastic Beanstalk | Triển khai web app managed | "developer focuses on code" |
| Worker tier | Xử lý nền từ SQS | "long-running background tasks" |
| Environment | Bộ tài nguyên chạy một version | "dev / test / prod" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Sticky sessions | ElastiCache session | Sticky gắn user với instance; ElastiCache cho app stateless thực sự | Scale tốt → ElastiCache |
| Client cookie | Server session | Cookie: dữ liệu ở client, nặng, dễ sửa; Server: chỉ session ID | Bảo mật → server session |
| EBS | EFS (WordPress) | EBS một instance; EFS chia sẻ nhiều AZ | Ảnh upload dùng chung → EFS |
| Golden AMI | User data | AMI cài sẵn, boot nhanh; user data linh hoạt, chậm | Kết hợp cả hai |
| Beanstalk | CloudFormation | Beanstalk là nền tảng ứng dụng; CloudFormation là IaC cho mọi tài nguyên | "Developer only uploads code" → Beanstalk |
| Web Server tier | Worker tier | Web nhận HTTP; Worker xử lý message SQS | "Background jobs" → Worker |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "stateless web app, HA, scale" | ALB + ASG Multi-AZ + Route 53 Alias | |
| "shopping cart lost between requests" | ElastiCache/DynamoDB session (hoặc sticky) | |
| "DB only accessible from app servers" | SG RDS cho phép SG EC2 | |
| "uploaded images visible on all instances" | Amazon EFS (hoặc S3) | |
| "reduce instance boot time" | Golden AMI | |
| "developers deploy code without managing infra" | Elastic Beanstalk | |
| "decouple long-running tasks in Beanstalk" | Worker tier + SQS | |
| "cut cost for baseline capacity" | Reserved Instances/Savings Plans cho min ASG | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Web app chạy trên ASG sau ALB; user mất giỏ hàng khi request rơi vào instance khác. Giải pháp **scale tốt nhất**?
- A. Bật sticky sessions
- B. Lưu giỏ hàng trong ElastiCache, session ID trong cookie
- C. Lưu giỏ hàng trên EBS của instance
- D. Tăng instance type

<details><summary>Đáp án</summary>

**B.** App trở thành stateless; mọi instance đọc cùng session.
- A sai: chạy được nhưng lệch tải và mất session khi instance chết.
- C sai: EBS gắn một instance.
- D sai: không giải quyết session.
</details>

**Câu 2.** Kiến trúc 3 tầng: ALB, EC2, RDS. Cách cấu hình SG an toàn nhất cho RDS?
- A. Mở port 3306 cho 0.0.0.0/0
- B. Cho phép port 3306 từ SG của EC2
- C. Cho phép port 3306 từ SG của ALB
- D. Không dùng SG cho RDS

<details><summary>Đáp án</summary>

**B.** Chỉ tầng ứng dụng gọi được DB.
- A sai: mở cho Internet.
- C sai: ALB không kết nối DB.
- D sai: RDS luôn cần SG.
</details>

**Câu 3.** WordPress chạy trên 3 EC2 ở 3 AZ; ảnh upload chỉ hiện trên một instance. Sửa thế nào?
- A. Gắn EBS Multi-Attach cho cả 3 instance
- B. Dùng Amazon EFS làm thư mục upload chung
- C. Dùng instance store
- D. Dùng sticky sessions

<details><summary>Đáp án</summary>

**B.** EFS mount được từ mọi AZ.
- A sai: Multi-Attach chỉ trong một AZ, cần cluster-aware FS.
- C sai: ổ local.
- D sai: user khác vẫn không thấy ảnh.
</details>

**Câu 4.** ASG scale out chậm vì user data mất 15 phút cài phần mềm. Cải thiện thế nào?
- A. Dùng Golden AMI đã cài sẵn phần mềm
- B. Tăng cooldown
- C. Dùng instance lớn hơn
- D. Bật cross-zone

<details><summary>Đáp án</summary>

**A.** Instance boot từ AMI có sẵn app, chỉ cần cấu hình nhẹ.
- B sai: chỉ làm scale chậm hơn.
- C sai: cài đặt vẫn mất thời gian.
- D sai: không liên quan.
</details>

**Câu 5.** (Chọn 2) Developer muốn deploy app Python mà không quản lý hạ tầng, có môi trường dev và prod riêng. Phát biểu nào đúng về Elastic Beanstalk?
- A. Beanstalk tính phí theo giờ cho chính dịch vụ
- B. Beanstalk tự tạo ELB, ASG, EC2 và có thể quản lý nhiều environment
- C. Beanstalk không cho chỉnh tài nguyên bên dưới
- D. Chỉ trả tiền cho tài nguyên Beanstalk tạo ra
- E. Beanstalk chỉ hỗ trợ Java

<details><summary>Đáp án</summary>

**B, D.**
- A sai: Beanstalk miễn phí.
- C sai: vẫn chỉnh được.
- E sai: hỗ trợ nhiều nền tảng.
</details>

**Câu 6.** App trên Beanstalk cần xử lý video mất 10 phút mỗi job, không muốn làm chậm web. Chọn gì?
- A. Tăng timeout của ALB
- B. Đẩy job vào SQS và xử lý bằng Beanstalk Worker tier
- C. Dùng sticky sessions
- D. Chạy job trong user data

<details><summary>Đáp án</summary>

**B.** Worker tier kéo message từ SQS và xử lý nền.
- A sai: request vẫn chặn web.
- C sai: không liên quan.
- D sai: user data chỉ chạy lúc boot.
</details>

**Câu 7.** MyClothes đọc thông tin user rất nhiều, RDS quá tải đọc. Chọn gì (ít thay đổi code nhất)?
- A. RDS read replica
- B. ElastiCache lazy loading
- C. DynamoDB
- D. EFS

<details><summary>Đáp án</summary>

**A.** Replica chỉ cần đổi endpoint đọc.
- B sai: hiệu quả nhưng phải sửa code nhiều hơn.
- C sai: đổi DB, sửa code lớn.
- D sai: không phải DB.
</details>

**Câu 8.** WhatIsTheTime chạy 2–10 instance trong ASG, luôn có ít nhất 2. Cách tối ưu chi phí hợp lý?
- A. Mua Reserved Instances/Savings Plans cho 2 instance tối thiểu, phần scale dùng On-Demand/Spot
- B. Mua RI cho 10 instance
- C. Toàn bộ dùng Dedicated Host
- D. Chuyển sang Single Instance Beanstalk

<details><summary>Đáp án</summary>

**A.** Cam kết cho baseline, linh hoạt cho phần biến động.
- B sai: lãng phí khi tải thấp.
- C sai: đắt.
- D sai: mất HA.
</details>

---

### Nguồn đã đối chiếu
- Elastic Beanstalk concepts (application, version, environment, tiers): https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts.html
- Elastic Beanstalk worker environments: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features-managing-env-tiers.html
- Elastic Beanstalk pricing (miễn phí, trả tài nguyên bên dưới): https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html
- ALB target group attributes (sticky sessions): https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html
- Amazon EFS use cases: https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html
- Session management with DynamoDB / ElastiCache (Well-Architected / caching): https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Strategies.html
