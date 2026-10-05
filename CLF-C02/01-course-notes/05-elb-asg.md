# CLF-C02 — Elastic Load Balancing & Auto Scaling Groups

> Thuộc **Domain 1** (scalability, elasticity, high availability) và **Domain 3** (ELB, ASG). Chương này phân biệt các khái niệm "-ility" mà đề rất hay hỏi.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Scalability** = chịu được tải lớn hơn: **vertical** (scale up/down, máy mạnh hơn) hoặc **horizontal** (scale out/in, thêm máy).
- **Elasticity** = hệ thống scalable **tự động** scale theo tải (pay-per-use). **Agility** là distractor: không liên quan scaling, chỉ là tài nguyên mới "chỉ cách một cú click".
- **High availability** = chạy app trên **ít nhất 2 AZ** để sống sót khi mất một data center.
- **ELB** = load balancer **managed**: phân phối traffic, một điểm truy cập (DNS), health check, SSL termination, multi-AZ.
- Loại ELB: **ALB** (Layer 7, HTTP/HTTPS/gRPC), **NLB** (Layer 4, TCP/UDP, cực nhanh, static IP), **GWLB** (Layer 3, GENEVE, đưa traffic qua firewall/appliance). **CLB** là thế hệ cũ.
- **ASG** = tự scale out/in EC2 theo tải, giữ số instance trong khoảng **min–max**, thay instance hỏng, tự đăng ký vào load balancer.
- Chiến lược scaling: **Manual, Dynamic (Simple/Step, Target Tracking), Scheduled, Predictive (ML)**.

---

## 2. Core Concepts

### Vertical Scalability
- **Là gì:** tăng **kích thước** instance (vd: `t2.micro` → `t2.large`), còn gọi scale up/down.
- **Dùng khi nào:** hệ thống **không phân tán**, ví dụ database.
- **Vì sao quan trọng:** luôn có **giới hạn phần cứng** (từ t2.nano 0.5 GB RAM tới instance hàng chục TB RAM).

### Horizontal Scalability
- **Là gì:** tăng **số lượng** instance (scale out/in), đòi hỏi hệ thống **phân tán**.
- **Dùng khi nào:** web app, ứng dụng hiện đại; thực hiện bằng **ASG + ELB**.
- **Vì sao quan trọng:** horizontal scaling đồng nghĩa với elasticity khi được tự động hóa.

### High Availability
- **Là gì:** chạy cùng một ứng dụng trên **ít nhất 2 AZ**, mục tiêu là sống sót khi mất một data center.
- **Dùng khi nào:** ASG multi-AZ, ELB multi-AZ.
- **Vì sao quan trọng:** HA thường đi cùng horizontal scaling nhưng là khái niệm khác.

### Scalability vs Elasticity vs Agility
- **Là gì:** Scalability = có khả năng chịu tải lớn hơn; Elasticity = **tự động** scale theo tải, "cloud-friendly": trả theo dùng, khớp nhu cầu, tối ưu chi phí.
- **Dùng khi nào:** Agility = rút thời gian có tài nguyên mới từ **vài tuần xuống vài phút**.
- **Vì sao quan trọng:** slide ghi rõ Agility là **distractor** khi đề hỏi về scaling.

### Load balancing & lợi ích của ELB
- **Là gì:** load balancer chuyển traffic từ Internet tới nhiều EC2 phía sau.
- **Dùng khi nào:** dàn tải, một điểm truy cập (DNS), xử lý lỗi instance liền mạch, **health check** định kỳ, **SSL termination** (HTTPS), HA nhiều zone.
- **Vì sao quan trọng:** ELB là **managed**: AWS đảm bảo hoạt động, lo nâng cấp, bảo trì, HA. Tự dựng load balancer rẻ hơn nhưng tốn công hơn nhiều.

### Application Load Balancer (ALB)
- **Là gì:** load balancer **Layer 7**: HTTP, HTTPS, gRPC; có tính năng định tuyến theo HTTP (path, host...).
- **Dùng khi nào:** web app, microservices.
- **Vì sao quan trọng:** có **static DNS (URL)**, không có static IP.

### Network Load Balancer (NLB)
- **Là gì:** load balancer **Layer 4**: TCP, UDP; hiệu năng cực cao, **hàng triệu request/giây**.
- **Dùng khi nào:** cần hiệu năng cực cao hoặc giao thức không phải HTTP.
- **Vì sao quan trọng:** có **static IP** qua Elastic IP. NLB cũng là thành phần bắt buộc khi expose dịch vụ qua PrivateLink (chương 13).

### Gateway Load Balancer (GWLB)
- **Là gì:** load balancer **Layer 3**, giao thức **GENEVE** trên IP packet.
- **Dùng khi nào:** đưa traffic qua các **virtual appliance bảo mật** của bên thứ ba (firewall, intrusion detection) chạy trên EC2.
- **Vì sao quan trọng:** từ khóa "third-party firewall appliances", "inspect all traffic" → GWLB.

### Classic Load Balancer (CLB)
- ⚠️ Lỗi thời: slide ghi "retired in 2023" → docs hiện mô tả CLB là **previous generation** và **khuyến nghị migrate** sang ALB/NLB (thứ bị retire năm 2022 là mạng EC2-Classic). ❓ cần kiểm chứng nếu đề hỏi ngày retire cụ thể.
- **Là gì:** Layer 4 & 7 thế hệ cũ.
- **Dùng khi nào:** không chọn cho thiết kế mới.

### Auto Scaling Group (ASG)
- **Là gì:** nhóm EC2 tự **scale out** khi tải tăng, **scale in** khi tải giảm, trong khoảng **minimum – desired – maximum**.
- **Dùng khi nào:** tự đăng ký instance mới vào load balancer, **thay instance unhealthy**, chạy multi-AZ.
- **Vì sao quan trọng:** ASG chính là cách **hiện thực elasticity** và tiết kiệm chi phí (chỉ chạy capacity tối ưu).

### Scaling strategies: Manual, Dynamic, Scheduled
- **Là gì:** **Manual** = tự đổi size ASG; **Dynamic**: **Simple/Step** (CloudWatch alarm kích hoạt, vd: CPU > 70% thì thêm 2) và **Target Tracking** (giữ CPU trung bình quanh 40%).
- **Dùng khi nào:** **Scheduled** = tăng/giảm theo lịch đã biết (vd: min = 10 lúc 5 giờ chiều thứ Sáu).
- **Vì sao quan trọng:** đề cho mô tả và hỏi tên chiến lược.

### Predictive Scaling
- **Là gì:** dùng **ML** dự đoán traffic tương lai và provision trước số EC2 cần thiết.
- **Dùng khi nào:** tải có **pattern theo thời gian** dự đoán được.
- **Vì sao quan trọng:** khác Scheduled ở chỗ không cần bạn biết trước lịch; ML tự học pattern.

### `[+]` AWS Auto Scaling (dịch vụ)
- **Là gì:** dịch vụ quản lý scaling cho nhiều loại tài nguyên (EC2 ASG, ECS, DynamoDB, Aurora replicas...) từ một chỗ.
- **Dùng khi nào:** đề nói "scale nhiều dịch vụ khác nhau theo một scaling plan".
- **Vì sao quan trọng:** "AWS Auto Scaling" nằm trong danh sách in-scope của CLF-C02, khác với "EC2 Auto Scaling group".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Vertical scaling | Tăng kích thước instance | "scale up", "larger instance", "database" |
| Horizontal scaling | Tăng số instance | "scale out", "add more instances" |
| High availability | Chạy trên ≥ 2 AZ | "survive AZ failure" |
| Elasticity | Tự động scale theo tải | "automatically match demand" |
| Agility | Tài nguyên mới trong vài phút | "reduce time to make resources available" |
| Elastic Load Balancing | Load balancer managed | "distribute traffic", "single endpoint" |
| Health check | ELB kiểm tra instance còn khỏe | "route only to healthy instances" |
| SSL termination | ELB xử lý HTTPS thay backend | "offload TLS" |
| ALB | Layer 7, HTTP/HTTPS/gRPC, routing theo HTTP | "path-based routing", "web app" |
| NLB | Layer 4, TCP/UDP, cực nhanh, static IP | "millions of requests per second", "UDP" |
| GWLB | Layer 3, GENEVE, đưa traffic qua appliance | "third-party firewall", "IDS/IPS" |
| Classic Load Balancer | Thế hệ cũ, Layer 4 & 7 | "previous generation" |
| Auto Scaling Group | Tự thêm/bớt EC2 trong min–max, thay instance hỏng | "replace unhealthy", "scale based on demand" |
| Desired capacity | Số instance mục tiêu hiện tại của ASG | "actual size" |
| Target Tracking | Giữ một metric quanh giá trị mục tiêu | "keep average CPU at 40%" |
| Step / Simple scaling | Scale khi CloudWatch alarm kích hoạt | "when CPU > 70% add 2" |
| Scheduled scaling | Scale theo lịch biết trước | "every Friday 5 PM" |
| Predictive scaling | ML dự đoán tải và scale trước | "forecast traffic", "machine learning" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Scalability | Elasticity | Khả năng chịu tải vs **tự động** khớp tải | "Automatically" → Elasticity |
| Elasticity | Agility | Tự co giãn vs có tài nguyên mới nhanh | Agility không liên quan scaling |
| Scalability | High availability | Chịu tải lớn vs sống sót khi hỏng AZ | "Multi-AZ" → HA |
| Vertical | Horizontal | Máy to hơn vs nhiều máy hơn | Database → vertical; web → horizontal |
| ALB | NLB | L7 HTTP, static DNS vs L4 TCP/UDP, static IP, cực nhanh | "UDP" hoặc "static IP" → NLB |
| NLB | GWLB | Phân tải ứng dụng vs đưa traffic qua appliance bảo mật | "Firewall appliances" → GWLB |
| ELB | ASG | ELB phân phối traffic; ASG thêm/bớt instance | Thường dùng cùng nhau |
| Scheduled | Predictive | Lịch do bạn đặt vs ML tự dự đoán | "Machine learning" → Predictive |
| Target Tracking | Step Scaling | Giữ metric ở mục tiêu vs hành động theo ngưỡng alarm | "Keep at 40%" → Target Tracking |
| ELB managed | Tự dựng LB trên EC2 | Managed: AWS lo HA, bảo trì; tự dựng: rẻ hơn nhưng tốn công | "Least operational overhead" → ELB |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "HTTP/HTTPS, route by URL path" | ALB | |
| "TCP/UDP, extreme performance, static IP" | NLB | |
| "deploy third-party virtual firewall appliances" | GWLB | |
| "automatically add instances when load increases" | Auto Scaling Group | Elasticity |
| "replace unhealthy instances automatically" | ASG (+ ELB health check) | |
| "keep average CPU at 50%" | Target Tracking Scaling | |
| "traffic spike every Monday 9 AM" | Scheduled Scaling | |
| "use ML to forecast load and scale ahead" | Predictive Scaling | |
| "run in at least two AZs" | High availability | |
| "move app to bigger instance" | Vertical scaling | |
| "single DNS endpoint for many instances" | ELB | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một ứng dụng web cần định tuyến request `/api` và `/images` tới hai nhóm server khác nhau. Dùng load balancer nào?
- A. Network Load Balancer
- B. Application Load Balancer
- C. Gateway Load Balancer
- D. Classic Load Balancer

<details><summary>Đáp án</summary>

**B.** ALB hoạt động ở Layer 7 và có tính năng định tuyến theo HTTP.
- A sai: Layer 4, không đọc được path HTTP.
- C sai: dùng cho appliance bảo mật.
- D sai: thế hệ cũ, không chọn cho thiết kế mới.
</details>

**Câu 2.** Game online dùng giao thức UDP, cần hàng triệu request mỗi giây và IP tĩnh. Chọn gì?
- A. Application Load Balancer
- B. Network Load Balancer
- C. Amazon CloudFront
- D. Auto Scaling Group

<details><summary>Đáp án</summary>

**B.** NLB hỗ trợ TCP/UDP, hiệu năng cực cao, static IP qua Elastic IP.
- A sai: chỉ HTTP/HTTPS/gRPC.
- C sai: CDN, không phải load balancer.
- D sai: ASG không phân phối traffic.
</details>

**Câu 3.** Hệ thống tự thêm EC2 khi CPU tăng và tự bớt khi CPU giảm để tối ưu chi phí. Khái niệm nào mô tả đúng nhất?
- A. Agility
- B. Elasticity
- C. Vertical scalability
- D. Fault tolerance

<details><summary>Đáp án</summary>

**B.** Elasticity = tự động scale theo tải.
- A sai: agility là tốc độ có tài nguyên mới.
- C sai: vertical là tăng kích thước một máy.
- D sai: chịu lỗi.
</details>

**Câu 4.** Một cửa hàng online biết trước lưu lượng tăng mạnh vào 20 giờ mỗi tối thứ Sáu. Chiến lược scaling phù hợp nhất?
- A. Manual Scaling
- B. Scheduled Scaling
- C. Target Tracking Scaling
- D. Vertical Scaling

<details><summary>Đáp án</summary>

**B.** Lịch đã biết trước → Scheduled Scaling.
- A sai: cần người thao tác.
- C sai: phản ứng sau khi tải tăng, không chuẩn bị trước.
- D sai: không tự động theo lịch.
</details>

**Câu 5.** (Chọn 2) Lợi ích của việc dùng Elastic Load Balancer là gì?
- A. Cung cấp một điểm truy cập (DNS) duy nhất cho ứng dụng
- B. Tự động vá OS cho EC2 phía sau
- C. Health check và chỉ gửi traffic tới instance khỏe
- D. Giảm giá EC2 như Reserved Instances
- E. Lưu trữ dữ liệu session vĩnh viễn

<details><summary>Đáp án</summary>

**A, C.**
- B sai: ELB không vá OS.
- D sai: không liên quan giá EC2.
- E sai: không phải dịch vụ lưu trữ.
</details>

**Câu 6.** Công ty cần kiểm tra toàn bộ traffic mạng bằng firewall của một hãng bảo mật bên thứ ba chạy trên EC2. Chọn gì?
- A. Gateway Load Balancer
- B. Application Load Balancer
- C. Network Load Balancer
- D. AWS WAF

<details><summary>Đáp án</summary>

**A.** GWLB định tuyến traffic qua các virtual appliance bảo mật (Layer 3, GENEVE).
- B, C sai: phân tải cho ứng dụng.
- D sai: WAF là firewall Layer 7 của AWS, không phải appliance bên thứ ba.
</details>

**Câu 7.** Ứng dụng phải tiếp tục chạy khi một AZ gặp sự cố. Thiết kế nào phù hợp nhất?
- A. Một EC2 lớn trong một AZ
- B. ASG trải trên nhiều AZ phía sau một ELB
- C. Nâng instance lên family Memory Optimized
- D. Dùng Scheduled Scaling

<details><summary>Đáp án</summary>

**B.** Multi-AZ ASG + ELB cho high availability.
- A, C sai: vẫn phụ thuộc một AZ.
- D sai: scaling theo lịch không giải quyết sự cố AZ.
</details>

**Câu 8.** Traffic của ứng dụng có pattern hằng ngày rõ ràng. Team muốn AWS tự học pattern và thêm capacity **trước** khi tải tăng. Chọn gì?
- A. Predictive Scaling
- B. Step Scaling
- C. Manual Scaling
- D. Vertical Scaling

<details><summary>Đáp án</summary>

**A.** Predictive Scaling dùng ML dự đoán tải và provision trước.
- B sai: phản ứng theo alarm, sau khi tải đã tăng.
- C sai: thủ công.
- D sai: không tự động theo dự đoán.
</details>

---

### Nguồn đã đối chiếu
- What is a Classic Load Balancer (previous generation, khuyến nghị migrate): https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html
- Migrate your Classic Load Balancer: https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/migrate-classic-load-balancer.html
- Elastic Load Balancing: https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html
- Amazon EC2 Auto Scaling: https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html
- CLF-C02 in-scope services (AWS Auto Scaling): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-in-scope-services.html
