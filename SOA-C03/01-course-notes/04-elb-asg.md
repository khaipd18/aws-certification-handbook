# SOA-C03 — High Availability & Scalability: ELB & Auto Scaling Groups

> Thuộc **Domain 2: Reliability and Business Continuity (22%)** và **Domain 5: Networking and Content Delivery (18%)**: chọn loại load balancer, health check, xử lý mã lỗi, scaling policy, cấu hình và xử lý sự cố ASG.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Vertical scaling** tăng size instance; **horizontal scaling** tăng số instance (ASG + ELB); **HA** = chạy trên ít nhất 2 AZ.
- **ALB** (L7: HTTP/HTTPS/WebSocket, routing theo path/host/header/query); **NLB** (L4: TCP/UDP/TLS, một static IP mỗi AZ, gắn được Elastic IP); **GWLB** (L3, GENEVE port **6081**, cho appliance bảo mật). ⚠️ **CLB** là thế hệ trước, AWS khuyên migrate.
- **Cross-zone**: ALB bật mặc định, không tính phí inter-AZ; NLB/GWLB tắt mặc định, bật thì tính phí inter-AZ.
- **Deregistration delay** (CLB gọi là connection draining) mặc định **300 giây**.
- Mã lỗi phải thuộc:
  - **503**: không có target healthy.
  - **504**: backend chậm, hoặc keep-alive của backend ngắn hơn idle timeout của LB.
  - **400**: request dị dạng.
  - **463**: `X-Forwarded-For` quá 30 IP.
- **Scaling policy**: Target Tracking (đơn giản nhất), Step/Simple (theo alarm), Scheduled (biết trước giờ), Predictive (dự báo và scale trước).
- ASG **không reboot** instance unhealthy, mà terminate rồi launch mới. Launch thất bại liên tục **hơn 24 giờ** thì ASG tự suspend process.
- **Warm Pools** giảm thời gian scale-out. **Lifecycle hooks** chèn bước xử lý ở `Pending:Wait` và `Terminating:Wait`.

---

## 2. Core Concepts

### Bốn loại Elastic Load Balancer
- **Là gì:** ALB (target: EC2, IP, Lambda, ECS task); NLB (EC2, IP, ALB; giữ IP client với target type instance); GWLB (gateway trong suốt + phân tải tới fleet firewall/IDS); CLB (L4+L7 cũ, chỉ một certificate, không SNI).
- **Dùng khi nào:** routing theo URL, microservice, Lambda → ALB; static IP để whitelist, UDP, độ trễ cực thấp → NLB; chuỗi appliance bên thứ ba → GWLB.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: CLB → nay dùng ALB hoặc NLB. Mô hình SG chuẩn: SG của LB mở 80/443, SG của EC2 chỉ cho phép từ SG của LB.

### Health checks của ELB
- **Là gì:** LB gọi định kỳ protocol + port + path của target group; trạng thái `initial`, `healthy`, `unhealthy`, `unused`, `draining`, `unavailable`.
- **Dùng khi nào:** đặt path riêng như `/health` để kiểm tra ứng dụng thật sự chạy.
- **Vì sao quan trọng:** ALB mặc định: interval 30 giây, timeout 5 giây (target instance/IP), **healthy threshold 5, unhealthy threshold 2**. ⚠️ Lỗi thời: bản dán ghi 3/5, không khớp docs. Nếu mọi target đều unhealthy, ALB vẫn gửi tới tất cả (fail-open).

### Sticky sessions & thuật toán routing
- **Là gì:** stickiness đưa client về cùng target. Duration-based cookie do LB sinh (`AWSALB` hết hạn sau 7 ngày, không đổi được; thời gian stickiness cấu hình 1 giây–7 ngày); application-based cookie do target sinh hoặc `AWSALBAPP`.
- **Dùng khi nào:** session lưu cục bộ trên instance. Thuật toán: round robin, **least outstanding requests** (ALB), flow hash (NLB).
- **Vì sao quan trọng:** stickiness gây lệch tải; giải pháp bền là đưa session ra ElastiCache/DynamoDB. Không đặt tên cookie `AWSALB`, `AWSALBAPP`, `AWSALBTG`.

### Cross-zone load balancing
- **Là gì:** mỗi node LB chia đều cho target ở mọi AZ, không chỉ AZ của nó.
- **Dùng khi nào:** số instance giữa các AZ không đều, AZ ít instance bị quá tải.
- **Vì sao quan trọng:** ALB bật mặc định (tắt được ở target group) và miễn phí; NLB/GWLB tắt mặc định, bật thì tính phí inter-AZ; CLB tắt mặc định nhưng miễn phí.

### SSL/TLS & SNI
- **Là gì:** LB terminate TLS bằng certificate X.509 (ACM hoặc import); SNI cho phép nhiều certificate trên một listener.
- **Dùng khi nào:** nhiều domain trên một LB → ALB/NLB + SNI; client cũ cần TLS thấp → đổi security policy của listener.
- **Vì sao quan trọng:** CLB chỉ một certificate, không SNI → nhiều domain phải nhiều CLB.

### Deregistration delay & slow start
- **Là gì:** deregistration delay chờ request đang dở hoàn tất khi target rời group (mặc định 300 giây, đặt 0 để tắt). Slow start tăng dần traffic vào target mới.
- **Dùng khi nào:** request bị đứt khi deploy/scale-in → tăng delay; request ngắn → giảm delay cho deploy nhanh. App cần warm-up (JIT, cache) → slow start.
- **Vì sao quan trọng:** delay lớn làm Instance Refresh và scale-in chậm. Draining lo lúc **rời**, slow start lo lúc **vào**.

### Listener rules, weighting & logging
- **Là gì:** rule ALB xử lý theo thứ tự, có default rule; action forward / redirect / fixed-response; condition host-header, path-pattern, method, source-ip, header, query-string. Target group weighting chia % traffic.
- **Dùng khi nào:** redirect HTTP → HTTPS; blue/green, canary 90/10. Access logs ghi từng request vào S3 (chỉ trả phí S3); `X-Amzn-Trace-Id` lần vết một request; `X-Forwarded-For` mang IP client.
- **Vì sao quan trọng:** access logs giữ được dữ liệu audit sau khi ELB đã xóa.

### Auto Scaling Group & Launch Template
- **Là gì:** nhóm EC2 giữ trong Min–Desired–Max, tự đăng ký vào target group, thay instance unhealthy; ASG miễn phí, chỉ trả EC2.
- **Dùng khi nào:** mọi fleet web/app cần co giãn và tự phục hồi; trộn On-Demand + Spot, nhiều instance type.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: Launch Configuration → nay dùng **Launch Template** (có version, mixed instances); account mới không tạo được launch configuration.

### Scaling policies
- **Là gì:** Target Tracking giữ metric quanh giá trị đích; Step/Simple theo alarm; Scheduled theo lịch; Predictive dự báo từ lịch sử.
- **Dùng khi nào:** metric hay dùng `CPUUtilization`, `RequestCountPerTarget`, network, custom metric; worker SQS scale theo backlog (`ApproximateNumberOfMessagesVisible` hoặc backlog per instance).
- **Vì sao quan trọng:** đề phân biệt "giữ CPU 40%" (Target Tracking) với "vượt 70% thêm 2, vượt 90% thêm 5" (Step).

### Cooldown, Instance Refresh, Warm Pools, Lifecycle hooks
- **Là gì:** cooldown mặc định 300 giây; Instance Refresh thay instance theo lô (`StartInstanceRefresh`, minimum healthy percentage, warm-up); Warm Pool giữ instance đã khởi tạo ở trạng thái Running / Stopped / Hibernated.
- **Dùng khi nào:** rollout AMI mới → Instance Refresh; bootstrap lâu → Warm Pool hoặc golden AMI; chạy script trước khi vào service hoặc lấy log trước khi terminate → lifecycle hook + EventBridge/SNS/SQS/Lambda.
- **Vì sao quan trọng:** giữ instance để điều tra → Standby hoặc hook `Terminating:Wait`, vì ASG không reboot.

### ASG health checks & sự cố
- **Là gì:** health check EC2 (mặc định), ELB (phải bật health check type ELB), custom qua `SetInstanceHealth`.
- **Dùng khi nào:** instance chạy nhưng app chết mà ASG không thay → bật ELB health check.
- **Vì sao quan trọng:** grace period mặc định **300 giây khi tạo bằng console, 0 khi tạo bằng CLI/SDK**; quá ngắn → vòng lặp launch/terminate. "Already running N instances" → chạm Max. Launch fail do SG hoặc key pair đã bị xóa.

### CloudWatch metrics cho ELB & ASG
- **Là gì:** ALB: `HealthyHostCount`, `RequestCountPerTarget`, `TargetResponseTime`, `HTTPCode_ELB_5XX_Count`, `HTTPCode_Target_5XX_Count`, `RejectedConnectionCount`. CLB: `Latency`, `SurgeQueueLength` (tối đa 1,024), `SpilloverCount`.
- **Dùng khi nào:** so sánh `HTTPCode_ELB_5XX` với `HTTPCode_Target_5XX` để biết lỗi ở LB hay ứng dụng.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: tên metric trong slide là của CLB. Metric nhóm ASG (`GroupDesiredCapacity`, `GroupInServiceInstances`…) phải bật metric collection.

### `[+]` NLB: security group, IP client, idle timeout
- **Là gì:** NLB gắn được security group, nhưng **chỉ khi tạo**; tạo không có SG thì không thêm sau được. Target type instance giữ IP client.
- **Dùng khi nào:** cần lọc traffic ngay tại NLB; backend cần IP thật ở tầng TCP.
- **Vì sao quan trọng:** NLB TCP idle timeout mặc định 350 giây (60–6,000); ALB idle timeout mặc định 60 giây (1–4,000). Keep-alive của backend nên dài hơn idle timeout của LB để tránh 502/504.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| ALB | L7, routing theo nội dung | "path-based routing", "Lambda target" |
| NLB | L4, static IP, UDP | "Elastic IP for whitelisting", "UDP" |
| GWLB | Gateway cho appliance, GENEVE 6081 | "third-party firewall fleet" |
| Cross-zone LB | Chia đều qua mọi AZ | "uneven load between AZs" |
| SNI | Nhiều certificate trên một listener | "multiple domains, one LB" |
| Deregistration delay | Chờ request dở khi rời group | "in-flight requests dropped" |
| Slow start | Tăng dần traffic vào target mới | "new instances overwhelmed" |
| Target group weighting | Chia % traffic | "canary 90/10", "blue/green" |
| Launch Template | Cấu hình launch có version | "mixed On-Demand and Spot" |
| Instance Refresh | Thay instance theo lô | "roll out new AMI" |
| Warm Pool | Instance khởi tạo sẵn | "slow scale-out due to bootstrap" |
| Lifecycle hook | Dừng ở Pending/Terminating:Wait | "run script before terminate" |
| Health check grace period | Bỏ qua health check sau launch | "instances terminated before ready" |
| `RequestCountPerTarget` | Request trên mỗi target | "scale by requests per instance" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| ALB | NLB | L7 routing nội dung; L4, static IP, UDP, giữ IP client | Whitelist IP → NLB |
| NLB | GWLB | Phân tải ứng dụng; gateway cho appliance | Firewall/IDS fleet → GWLB |
| Cross-zone ALB | Cross-zone NLB | ALB bật, miễn phí; NLB tắt, bật thì tính phí | NLB lệch tải → bật cross-zone |
| Slow start | Warm Pool | LB tăng dần traffic; ASG giữ instance sẵn | "Overwhelmed" vs "slow scale-out" |
| Deregistration delay | Slow start | Lúc rời; lúc vào | "Dropped requests on deploy" → delay |
| Target Tracking | Step Scaling | Một giá trị đích; alarm + nhiều bậc | "Keep at 40%" → Target Tracking |
| Scheduled | Predictive | Biết trước giờ; AWS tự dự báo | "Every Friday 5 PM" → Scheduled |
| Cooldown | Grace period | Chặn scaling kế tiếp; hoãn đánh giá health | "Terminated before app ready" → grace period |
| EC2 health check | ELB health check | VM sống; ứng dụng trả 200 | "App dead, not replaced" → ELB |
| `HTTPCode_ELB_5XX` | `HTTPCode_Target_5XX` | Lỗi do LB; lỗi do ứng dụng | Tìm thủ phạm → so hai metric |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "route `/users` and `/orders` to different services" | ALB listener rules | |
| "static IP / Elastic IP for partners" | NLB | |
| "third-party firewall appliances inline" | GWLB | GENEVE 6081 |
| "HTTP 503" | Không có target healthy → `HealthyHostCount`, health check, SG | |
| "HTTP 504" | Backend chậm hoặc keep-alive ngắn hơn idle timeout | |
| "multiple TLS certs on one LB" | ALB/NLB + SNI | |
| "redirect HTTP to HTTPS" | ALB rule redirect | |
| "requests cut off during deployment" | Tăng deregistration delay | |
| "canary 10% traffic" | Target group weighting | |
| "audit all requests after LB deleted" | Access logs → S3 | |
| "scale workers on queue backlog" | Metric SQS backlog | |
| "roll out new AMI keeping 60% capacity" | Instance Refresh, min healthy 60% | |
| "scale-out takes 10 minutes to bootstrap" | Warm Pool / golden AMI | |
| "ASG launches and terminates in a loop" | Tăng health check grace period | |
| "ASG stopped launching after a day of failures" | Administrative suspension → sửa lỗi rồi resume | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng sau ALB trả **HTTP 503** cho phần lớn request. Bước chẩn đoán đầu tiên?
- A. Tăng idle timeout của ALB
- B. Kiểm tra `HealthyHostCount` và cấu hình health check của target group
- C. Bật cross-zone load balancing
- D. Bật sticky sessions

<details><summary>Đáp án</summary>

**B.** 503 nghĩa là không có target healthy để nhận request.
- A sai: idle timeout liên quan 504.
- C sai: ALB đã bật cross-zone mặc định.
- D sai: stickiness không tạo thêm target healthy.
</details>

**Câu 2.** Thỉnh thoảng có **HTTP 504** dù CPU EC2 thấp. Nguyên nhân khả dĩ nhất?
- A. SG của ALB chặn 443
- B. Keep-alive timeout trên EC2 ngắn hơn idle timeout của ALB
- C. Cross-zone đang tắt
- D. Target group bật slow start

<details><summary>Đáp án</summary>

**B.** LB gửi request qua kết nối mà backend vừa đóng.
- A sai: chặn 443 thì client không kết nối được, không ra 504 lúc có lúc không.
- C sai: ALB không tắt cross-zone mặc định; lệch tải không gây 504.
- D sai: slow start chỉ điều chỉnh lượng traffic.
</details>

**Câu 3.** (Chọn 2) Đối tác yêu cầu whitelist **IP cố định**, ứng dụng dùng **UDP**. Hai phát biểu nào đúng?
- A. Dùng NLB
- B. Dùng ALB
- C. NLB có một static IP mỗi AZ và gắn được Elastic IP
- D. ALB hỗ trợ UDP
- E. GWLB phù hợp cho traffic UDP của ứng dụng

<details><summary>Đáp án</summary>

**A, C.**
- B, D sai: ALB chỉ HTTP/HTTPS/WebSocket, không có IP tĩnh.
- E sai: GWLB dùng cho appliance bảo mật, không phải phân tải ứng dụng.
</details>

**Câu 4.** ASG tạo bằng CLI liên tục launch rồi terminate instance sau vài phút. Ứng dụng mất 6 phút để khởi động, health check type là ELB. Cách sửa?
- A. Giảm cooldown về 0
- B. Đặt health check grace period dài hơn thời gian khởi động
- C. Tăng MaxCapacity
- D. Bật cross-zone

<details><summary>Đáp án</summary>

**B.** Tạo bằng CLI thì grace period mặc định 0, instance bị đánh unhealthy trước khi app sẵn sàng.
- A sai: cooldown chặn scaling activity, không liên quan health.
- C sai: vấn đề không phải giới hạn số lượng.
- D sai: không liên quan.
</details>

**Câu 5.** Rollout AMI mới cho ASG production, luôn giữ ít nhất 60% capacity phục vụ. Chọn gì?
- A. Instance Refresh, minimum healthy percentage 60%, warm-up phù hợp
- B. Xóa ASG cũ, tạo ASG mới
- C. Terminate tay từng instance
- D. Bật Warm Pool

<details><summary>Đáp án</summary>

**A.**
- B sai: gây gián đoạn.
- C sai: thủ công, không kiểm soát capacity.
- D sai: Warm Pool không thay AMI cho instance đang chạy.
</details>

**Câu 6.** (Chọn 2) NLB trải 2 AZ; AZ-A có 8 instance, AZ-B có 2 instance và bị quá tải. Hai phát biểu nào đúng?
- A. NLB bật cross-zone mặc định
- B. NLB tắt cross-zone mặc định
- C. Bật cross-zone trên NLB phát sinh phí data transfer inter-AZ
- D. Bật cross-zone trên NLB miễn phí
- E. Cách duy nhất là chuyển sang ALB

<details><summary>Đáp án</summary>

**B, C.**
- A sai: chỉ ALB bật mặc định.
- D sai: NLB tính phí inter-AZ khi bật.
- E sai: bật cross-zone hoặc cân bằng số instance là đủ.
</details>

**Câu 7.** Worker đọc SQS; backlog tăng vọt giờ cao điểm nhưng ASG không scale vì CPU thấp. Cách đúng?
- A. Predictive scaling theo CPU
- B. Scale theo metric backlog của queue (số message chờ, hoặc backlog trên mỗi instance)
- C. Bật Detailed Monitoring
- D. Tăng cooldown lên 900 giây

<details><summary>Đáp án</summary>

**B.**
- A sai: CPU không phản ánh backlog.
- C sai: chỉ tăng tần suất metric CPU.
- D sai: làm scale chậm hơn.
</details>

**Câu 8.** (Chọn 2) Kỹ sư xóa nhầm Security Group mà Launch Template tham chiếu; ASG ngừng launch. Hai phát biểu nào đúng?
- A. ASG tự tạo lại SG
- B. Launch thất bại vì SG không còn; cần tạo version Launch Template mới
- C. Launch thất bại liên tục hơn 24 giờ thì ASG tự suspend process
- D. ASG reboot instance hiện có để bù capacity
- E. Lỗi này chỉ xảy ra với Launch Configuration

<details><summary>Đáp án</summary>

**B, C.**
- A sai: ASG không tạo SG.
- D sai: ASG không reboot.
- E sai: Launch Template tham chiếu SG đã xóa cũng lỗi.
</details>

---

### Nguồn đã đối chiếu
- ALB target group health checks (interval 30, timeout 5, healthy 5, unhealthy 2): https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html
- ALB target group attributes (deregistration 300 giây, slow start, `AWSALB` 7 ngày): https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html
- ALB attributes (idle timeout 60 giây, 1–4,000): https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-load-balancer-attributes.html
- NLB (TCP idle timeout 350 giây): https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html
- NLB security groups (chỉ gắn lúc tạo): https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-security-groups.html
- Classic Load Balancer (previous generation): https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html
- ASG health check grace period: https://docs.aws.amazon.com/autoscaling/ec2/userguide/health-check-grace-period.html
- Suspend/resume processes (administrative suspension): https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-suspend-resume-processes.html
