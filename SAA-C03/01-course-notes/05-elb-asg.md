# SAA-C03 — High Availability & Scalability: ELB & Auto Scaling Groups

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 3: Design High-Performing Architectures**: scale dọc/ngang, các loại Load Balancer, sticky session, cross-zone, TLS/SNI và Auto Scaling.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Vertical scaling** = tăng cỡ instance (DB, có giới hạn phần cứng); **horizontal scaling** = thêm instance (ứng dụng phân tán, web). **High availability** = chạy ở ít nhất 2 AZ.
- **ALB** (Layer 7: HTTP/HTTPS/gRPC, route theo path/host/header/query), **NLB** (Layer 4: TCP/UDP/TLS, triệu request/giây, **IP tĩnh mỗi AZ**, gắn được EIP), **GWLB** (Layer 3, GENEVE port 6081, chèn firewall/IDS bên thứ ba), **CLB** (thế hệ cũ).
- Backend thấy IP client qua header **X-Forwarded-For** (ALB); NLB giữ nguyên IP client.
- **Cross-zone**: ALB luôn bật ở cấp LB (tắt được ở target group), **không phí inter-AZ**; NLB và GWLB **tắt mặc định**, bật thì tính phí inter-AZ.
- **SNI** cho phép một ALB/NLB phục vụ nhiều certificate TLS cho nhiều domain; CLB chỉ một cert.
- **Deregistration delay** (connection draining) cho request đang chạy hoàn tất trước khi gỡ target (mặc định 300 giây).
- **ASG** giữ số instance trong khoảng min–max, thay instance unhealthy; scaling policy: **target tracking**, **step**, **simple**, **scheduled**, **predictive**. Cooldown mặc định **300 giây**.

---

## 2. Core Concepts

### Scalability & High Availability
- **Là gì:** scalability = chịu được tải lớn hơn (scale up/out); HA = sống sót khi mất một AZ/data center.
- **Dùng khi nào:** horizontal + Multi-AZ cho web tier; vertical cho DB hoặc ứng dụng không phân tán.
- **Vì sao quan trọng:** ASG Multi-AZ + ELB Multi-AZ là mẫu kiến trúc chuẩn nhất của SAA.

### Elastic Load Balancing
- **Là gì:** load balancer managed: phân phối traffic, một DNS endpoint, health check, SSL termination, stickiness, tách public/private traffic.
- **Dùng khi nào:** đặt trước EC2, ECS, Lambda (ALB), IP target; tích hợp ASG, ACM, CloudWatch, Route 53, WAF, Global Accelerator.
- **Vì sao quan trọng:** SG của instance nên chỉ cho phép traffic từ **SG của LB**, không mở 0.0.0.0/0.

### Application Load Balancer
- **Là gì:** Layer 7, hỗ trợ HTTP/2, WebSocket, redirect HTTP→HTTPS, route theo path, hostname, query string, header, source IP; target group: EC2, ECS task, Lambda, IP private.
- **Dùng khi nào:** microservice và container (dynamic port mapping với ECS), nhiều ứng dụng sau một LB.
- **Vì sao quan trọng:** hostname cố định nhưng **IP không cố định**; backend lấy IP client qua `X-Forwarded-For`, port qua `X-Forwarded-Port`, giao thức qua `X-Forwarded-Proto`.

### Network Load Balancer
- **Là gì:** Layer 4 TCP/UDP/TLS, độ trễ rất thấp, hàng triệu request/giây; mỗi AZ một IP tĩnh, gắn được Elastic IP; target: EC2, IP private, **ALB**.
- **Dùng khi nào:** cần IP tĩnh để whitelist, giao thức không phải HTTP, hiệu năng cực cao; làm frontend cho PrivateLink.
- **Vì sao quan trọng:** NLB nay **gắn được Security Group** (chỉ khi tạo LB có SG ngay từ đầu); health check hỗ trợ TCP, HTTP, HTTPS.

### Gateway Load Balancer
- **Là gì:** Layer 3, vừa là gateway trong suốt vừa là LB; dùng giao thức **GENEVE port 6081**.
- **Dùng khi nào:** đưa toàn bộ traffic qua fleet appliance bên thứ ba: firewall, IDS/IPS, deep packet inspection.
- **Vì sao quan trọng:** từ khóa "third-party virtual appliances", "GENEVE" → GWLB.

### Sticky Sessions
- **Là gì:** client luôn về cùng một target trong khoảng thời gian cookie còn hạn; cookie **duration-based** (do LB tạo: AWSALB) hoặc **application-based** (cookie tùy chỉnh hoặc AWSALBAPP).
- **Dùng khi nào:** ứng dụng giữ session trong bộ nhớ instance.
- **Vì sao quan trọng:** có thể gây lệch tải; kiến trúc tốt hơn là đưa session ra ElastiCache/DynamoDB (chương 8).

### Cross-Zone Load Balancing
- **Là gì:** mỗi node LB phân phối đều cho target ở **mọi AZ**, không chỉ AZ của nó.
- **Dùng khi nào:** số instance giữa các AZ không đều.
- **Vì sao quan trọng:** ALB: luôn bật ở cấp LB, không tính phí inter-AZ; NLB & GWLB: tắt mặc định, bật thì trả phí inter-AZ; CLB: tắt mặc định qua API/CLI, bật mặc định qua console.

### SSL/TLS & SNI
- **Là gì:** LB dùng X.509 cert (quản lý bằng **ACM**) để kết thúc TLS; **SNI** cho client báo hostname trong handshake để LB chọn đúng cert.
- **Dùng khi nào:** nhiều domain HTTPS sau một ALB/NLB, hoặc CloudFront.
- **Vì sao quan trọng:** CLB chỉ một cert (muốn nhiều domain thì nhiều CLB); ALB/NLB nhiều listener cert nhờ SNI.

### Deregistration Delay
- **Là gì:** thời gian cho request đang xử lý hoàn tất khi target bị gỡ hoặc unhealthy; 1–3600 giây, mặc định 300, đặt 0 để tắt. CLB gọi là Connection Draining.
- **Dùng khi nào:** request ngắn → đặt thấp để gỡ nhanh.
- **Vì sao quan trọng:** đề "tránh cắt ngang request khi scale-in" → deregistration delay.

### Auto Scaling Group
- **Là gì:** nhóm EC2 tự scale out/in trong khoảng **min / desired / max**, tự đăng ký target vào LB, thay instance unhealthy (theo EC2 status hoặc **ELB health check**).
- **Dùng khi nào:** cấu hình qua **launch template** (AMI, instance type, user data, EBS, SG, key pair, IAM role, subnet).
- **Vì sao quan trọng:** ⚠️ Lỗi thời: launch configuration → account tạo từ 01/10/2024 không tạo được launch configuration mới; dùng **launch template**.

### ASG scaling policies
- **Là gì:** **target tracking** (giữ CPU ~40%), **step/simple** (theo CloudWatch alarm), **scheduled** (theo lịch biết trước), **predictive** (dự báo bằng ML theo chu kỳ lịch sử).
- **Dùng khi nào:** metric hay dùng: CPUUtilization, RequestCountPerTarget, network in/out, metric tùy chỉnh.
- **Vì sao quan trọng:** sau mỗi lần scale có **cooldown** (mặc định 300 giây); dùng AMI dựng sẵn để instance sẵn sàng nhanh hơn, rút ngắn cooldown.

### `[+]` Instance refresh & warm pool
- **Là gì:** **instance refresh** thay dần instance theo launch template mới; **warm pool** giữ sẵn instance đã khởi tạo (stopped/hibernated/running).
- **Dùng khi nào:** cập nhật AMI không downtime; ứng dụng khởi động lâu cần scale nhanh.
- **Vì sao quan trọng:** đáp án cho "giảm thời gian scale-out của app khởi tạo chậm".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| ALB | LB Layer 7, route theo path/host | "path-based routing", "microservices" |
| NLB | LB Layer 4, IP tĩnh, cực nhanh | "static IP", "TCP/UDP", "millions of requests" |
| GWLB | LB Layer 3, GENEVE 6081 | "third-party firewall appliances" |
| Target group | Nhóm target của ALB/NLB | "EC2, IP, Lambda, ECS tasks" |
| X-Forwarded-For | Header chứa IP client | "backend needs client IP" |
| Sticky sessions | Client về cùng target | "user session lost when switching instance" |
| Cross-zone | Phân tải đều mọi AZ | "uneven instances per AZ" |
| SNI | Nhiều cert TLS trên một LB | "multiple domains, one load balancer" |
| Deregistration delay | Chờ request xong trước khi gỡ target | "in-flight requests" |
| Launch template | Cấu hình launch cho ASG, có version | "ASG configuration" |
| Target tracking | Giữ metric quanh giá trị đích | "keep CPU at 50%" |
| Scheduled scaling | Scale theo lịch | "every Friday at 5 PM" |
| Predictive scaling | Dự báo tải theo chu kỳ | "recurring daily pattern" |
| Cooldown | Thời gian chờ sau một lần scale | "prevent rapid scaling" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| ALB | NLB | ALB: HTTP, route nội dung, IP động; NLB: TCP/UDP, IP tĩnh, cực nhanh | IP tĩnh/UDP → NLB |
| NLB | GWLB | NLB phân tải ứng dụng; GWLB đẩy traffic qua appliance bảo mật | "Inspect all traffic" → GWLB |
| Sticky sessions | Session ở ElastiCache | Sticky gắn user vào instance; ElastiCache cho app stateless | Stateless tốt hơn |
| Cross-zone ALB | Cross-zone NLB | ALB luôn bật, không phí; NLB tắt mặc định, có phí | |
| Target tracking | Step scaling | Target tracking tự tạo alarm; step cần alarm và bậc | Dễ nhất → target tracking |
| Scheduled | Predictive | Scheduled theo lịch bạn đặt; Predictive tự học chu kỳ | "Known event" → scheduled |
| EC2 health check | ELB health check (ASG) | EC2 chỉ xem status phần cứng/OS; ELB xem app trả lời | App hỏng nhưng instance chạy → bật ELB health check |
| Launch template | Launch configuration | Template có version, nhiều tính năng; configuration cũ, bị chặn với account mới | Luôn chọn template |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "route /api and /images to different services" | ALB path-based routing | |
| "static IP per AZ, whitelist by partners" | NLB (+ EIP) | |
| "UDP gaming traffic, ultra-low latency" | NLB | |
| "inspect traffic with third-party firewall fleet" | GWLB | GENEVE |
| "backend needs client IP address" | X-Forwarded-For (ALB) | NLB giữ IP gốc |
| "multiple SSL certificates, one LB" | ALB/NLB + SNI | |
| "users logged out when routed to other instance" | Sticky sessions hoặc ElastiCache | |
| "ASG replaces instance when app fails health check" | Bật ELB health check cho ASG | |
| "scale before 9 AM every weekday" | Scheduled scaling | |
| "keep average CPU at 40%" | Target tracking | |
| "finish in-flight requests before removing" | Deregistration delay | |
| "ensure instances only accept LB traffic" | SG instance cho phép SG của LB | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng microservice: `/orders` tới service A, `/users` tới service B, sau một DNS duy nhất. Chọn gì?
- A. NLB
- B. ALB với path-based routing
- C. GWLB
- D. Route 53 weighted routing

<details><summary>Đáp án</summary>

**B.** ALB route theo path tới các target group khác nhau.
- A sai: NLB Layer 4, không đọc path.
- C sai: dành cho appliance bảo mật.
- D sai: Route 53 làm ở mức DNS, không đọc path.
</details>

**Câu 2.** Đối tác chỉ whitelist được IP cố định; ứng dụng chạy TCP tùy chỉnh trên nhiều EC2. Chọn gì?
- A. ALB
- B. NLB với Elastic IP mỗi AZ
- C. CLB
- D. CloudFront

<details><summary>Đáp án</summary>

**B.** NLB có IP tĩnh mỗi AZ, gắn được EIP, hỗ trợ TCP.
- A sai: ALB không có IP tĩnh.
- C sai: thế hệ cũ, không có IP tĩnh.
- D sai: CloudFront cho HTTP và IP thay đổi.
</details>

**Câu 3.** ASG dùng EC2 health check. Ứng dụng treo, trả 500 nhưng instance vẫn "running", ASG không thay. Sửa thế nào?
- A. Tăng max size
- B. Bật ELB health check cho ASG
- C. Bật sticky sessions
- D. Thêm scheduled scaling

<details><summary>Đáp án</summary>

**B.** ELB health check cho ASG biết app không khỏe để thay instance.
- A, D sai: không phát hiện instance hỏng.
- C sai: còn làm user bị kẹt ở instance lỗi.
</details>

**Câu 4.** Lưu lượng tăng mạnh mỗi sáng thứ Hai 8 giờ. Cách scale đơn giản nhất?
- A. Scheduled scaling
- B. Simple scaling theo CPU
- C. Tăng instance type
- D. Bật cross-zone

<details><summary>Đáp án</summary>

**A.** Biết trước thời điểm → scheduled.
- B sai: phản ứng sau khi tải đã tăng.
- C sai: vertical scaling, không tự động theo lịch.
- D sai: không thay đổi số instance.
</details>

**Câu 5.** (Chọn 2) Cần đảm bảo EC2 sau ALB chỉ nhận traffic từ ALB và ALB phục vụ HTTPS cho 3 domain. Chọn gì?
- A. SG của EC2 cho phép inbound từ SG của ALB
- B. SG của EC2 mở 0.0.0.0/0 port 80
- C. Gắn 3 certificate ACM vào listener HTTPS (SNI)
- D. Dùng 3 CLB
- E. Bật sticky sessions

<details><summary>Đáp án</summary>

**A, C.**
- B sai: mở cho cả Internet.
- D sai: tốn kém, không cần thiết khi có SNI.
- E sai: không liên quan bảo mật hay cert.
</details>

**Câu 6.** Khi scale-in, user báo request bị cắt giữa chừng. Cấu hình nào giúp?
- A. Deregistration delay phù hợp
- B. Giảm cooldown
- C. Bật predictive scaling
- D. Dùng Spread placement group

<details><summary>Đáp án</summary>

**A.** Cho request đang chạy hoàn tất trước khi gỡ target.
- B sai: còn làm scale nhanh hơn.
- C sai: dự báo tải, không xử lý request đang chạy.
- D sai: không liên quan.
</details>

**Câu 7.** Công ty phải đưa toàn bộ traffic vào VPC qua fleet firewall của hãng thứ ba, có scale và HA. Chọn gì?
- A. NLB
- B. ALB + WAF
- C. Gateway Load Balancer
- D. Network ACL

<details><summary>Đáp án</summary>

**C.** GWLB phân tải tới appliance qua GENEVE, trong suốt với luồng traffic.
- A sai: NLB không làm gateway trong suốt.
- B sai: WAF là dịch vụ AWS cho HTTP, không phải appliance bên thứ ba.
- D sai: NACL chỉ lọc stateless.
</details>

**Câu 8.** NLB có 2 instance ở AZ-a và 8 instance ở AZ-b; instance ở AZ-a quá tải. Cần làm gì?
- A. Bật cross-zone load balancing trên NLB (chấp nhận phí inter-AZ)
- B. Bật sticky sessions
- C. Đổi sang GWLB
- D. Giảm deregistration delay

<details><summary>Đáp án</summary>

**A.** Mỗi node NLB phân tải cho cả 10 instance.
- B sai: không cân bằng tải giữa AZ.
- C sai: sai mục đích.
- D sai: không liên quan phân tải.
</details>

---

### Nguồn đã đối chiếu
- How Elastic Load Balancing works (cross-zone mặc định theo loại LB, IP tĩnh NLB): https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html
- Security groups for Network Load Balancers: https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-security-groups.html
- Gateway Load Balancers (GENEVE): https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html
- ALB target group attributes (deregistration delay, stickiness): https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html
- Launch configurations (bị chặn với account tạo từ 01/10/2024): https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-configurations.html
- Scaling cooldowns: https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-scaling-cooldowns.html
- Predictive scaling: https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-predictive-scaling.html
