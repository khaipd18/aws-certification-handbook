# SAA-C03 — EC2 Networking: IP, Elastic IP, Placement Groups, ENI & Hibernate

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 3: Design High-Performing Architectures**: địa chỉ IP, cách đặt instance trên phần cứng, network interface và hibernate.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Public IP** đổi mỗi lần stop/start; **private IP** giữ nguyên suốt vòng đời instance; **Elastic IP** là public IPv4 tĩnh, gắn vào một instance/ENI tại một thời điểm.
- ⚠️ Lỗi thời: "EIP miễn phí khi gắn vào instance đang chạy" → từ 02/2024 AWS tính phí **mọi public IPv4**, kể cả EIP đang gắn.
- Tránh EIP: dùng **DNS** (Route 53) trỏ tới instance, hoặc đặt sau **Load Balancer**.
- **Placement group**: **Cluster** (một AZ, độ trễ thấp nhất), **Spread** (mỗi instance một rack, tối đa **7 instance/AZ/group**), **Partition** (tối đa **7 partition/AZ**, cho Hadoop, Cassandra, Kafka).
- **ENI** là card mạng ảo gắn với **một AZ**; tách ra gắn sang instance khác để failover giữ nguyên IP.
- **Hibernate** lưu RAM xuống root EBS (phải **mã hóa**) để khởi động nhanh; RAM < 150 GiB (Linux), không giữ quá **60 ngày**, phải bật lúc launch.

---

## 2. Core Concepts

### Public IP, private IP, Elastic IP
- **Là gì:** public IP định danh trên Internet; private IP chỉ dùng trong mạng riêng, đi ra Internet qua NAT/IGW; EIP là public IPv4 cố định thuộc account cho đến khi release.
- **Dùng khi nào:** EIP khi cần IP tĩnh và phải che lỗi instance bằng cách remap nhanh sang instance khác.
- **Vì sao quan trọng:** EIP là đáp án "kém kiến trúc"; thường nên dùng DNS hoặc Load Balancer. ⚠️ Lỗi thời: "mỗi account 5 EIP" → quota mặc định 5 EIP **theo Region**, xin tăng được.

### Chi phí public IPv4
- **Là gì:** AWS tính phí theo giờ cho mọi public IPv4: EIP đang dùng hoặc đang nhàn rỗi, IP public tự cấp cho instance.
- **Dùng khi nào:** tối ưu chi phí: bỏ public IP không cần, gom outbound qua NAT, cân nhắc IPv6.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "EIP chỉ tính phí khi không gắn hoặc gắn vào instance đã stop" → nay tính phí cả khi đang gắn.

### Cluster placement group
- **Là gì:** gom instance gần nhau trong **một AZ**, cùng phân đoạn mạng băng thông cao.
- **Dùng khi nào:** HPC, big data cần hoàn thành nhanh, ứng dụng cần độ trễ cực thấp giữa node.
- **Vì sao quan trọng:** AZ hoặc rack lỗi thì mất toàn bộ; dùng instance có Enhanced Networking.

### Spread placement group
- **Là gì:** mỗi instance ở một rack phần cứng riêng (mạng, nguồn riêng), trải được nhiều AZ; tối đa **7 instance đang chạy mỗi AZ mỗi group**.
- **Dùng khi nào:** số ít instance quan trọng cần cách ly lỗi phần cứng (vd: primary/standby tự quản).
- **Vì sao quan trọng:** đề "tối đa hóa HA cho vài instance quan trọng" → Spread; "hàng trăm instance" → không phải Spread.

### Partition placement group
- **Là gì:** chia instance thành partition, mỗi partition một nhóm rack riêng; tối đa **7 partition mỗi AZ**, hàng trăm instance, trải nhiều AZ trong Region.
- **Dùng khi nào:** hệ thống phân tán biết topology: HDFS, HBase, Cassandra, Kafka.
- **Vì sao quan trọng:** instance đọc được partition của mình qua metadata để ứng dụng tự đặt replica khác partition.

### Elastic Network Interface (ENI)
- **Là gì:** card mạng ảo: primary private IPv4, các secondary IPv4, mỗi private IP có thể có một EIP, một public IPv4, một hoặc nhiều SG, MAC address.
- **Dùng khi nào:** tạo riêng rồi gắn/tách khỏi instance khi cần failover; ENI gắn với **một AZ**.
- **Vì sao quan trọng:** "giữ nguyên private IP khi failover sang instance dự phòng" → di chuyển ENI.

### EC2 Hibernate
- **Là gì:** khi hibernate, nội dung RAM ghi vào root EBS; khi start lại, RAM được khôi phục, process tiếp tục như chưa từng tắt.
- **Dùng khi nào:** process khởi tạo lâu, muốn giữ trạng thái RAM, muốn boot nhanh.
- **Vì sao quan trọng:** điều kiện: root là **EBS mã hóa**, đủ chỗ chứa RAM; RAM Linux **< 150 GiB**, Windows **≤ 16 GiB**; không dùng instance store làm root; bật **lúc launch**; không giữ hibernate quá **60 ngày**; On-Demand, RI và Spot đều được.

### `[+]` Hibernate không áp dụng trong ASG/ECS
- **Là gì:** instance thuộc Auto Scaling group hoặc dùng bởi ECS không hibernate được.
- **Dùng khi nào:** cần giữ trạng thái RAM trong ASG thì dùng **warm pool** với trạng thái Hibernated thay vì hibernate thủ công.
- **Vì sao quan trọng:** tránh chọn "hibernate instance trong ASG" làm đáp án.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Public IPv4 | IP định tuyến trên Internet, đổi khi stop/start | "IP changes after restart" |
| Private IPv4 | IP trong VPC, giữ nguyên suốt vòng đời | "internal communication" |
| Elastic IP | Public IPv4 tĩnh thuộc account | "fixed public IP", "remap on failure" |
| Public IPv4 charge | Phí theo giờ cho mọi public IPv4 | "reduce IPv4 costs" |
| Cluster PG | Một AZ, độ trễ thấp, băng thông cao | "HPC", "low latency between nodes" |
| Spread PG | Mỗi instance một rack, 7/AZ | "critical instances, isolated hardware" |
| Partition PG | Nhóm rack theo partition, 7 partition/AZ | "Hadoop, Cassandra, Kafka" |
| ENI | Card mạng ảo gắn một AZ | "move IP to standby instance" |
| Enhanced Networking | Mạng hiệu năng cao (ENA) | "high PPS, low latency" |
| Hibernate | Lưu RAM xuống EBS mã hóa | "preserve in-memory state", "fast boot" |
| Warm pool | Nhóm instance đã khởi tạo sẵn cho ASG | "reduce scale-out latency" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Cluster PG | Spread PG | Cluster: gom một chỗ để nhanh; Spread: rải ra để an toàn | Hiệu năng → Cluster; HA → Spread |
| Spread PG | Partition PG | Spread: từng instance một rack, ít instance; Partition: nhóm rack, hàng trăm instance | Big data phân tán → Partition |
| Elastic IP | Public IP | EIP tĩnh, thuộc account; public IP đổi khi stop/start | "Không đổi IP" → EIP |
| EIP | DNS / Load Balancer | EIP che lỗi bằng remap thủ công; DNS/ELB là cách kiến trúc tốt hơn | Ưu tiên ELB |
| Hibernate | Stop | Stop mất RAM, boot lại từ đầu; Hibernate giữ RAM | "Preserve memory" → Hibernate |
| ENI | EIP | ENI là card mạng (có private IP, SG); EIP chỉ là địa chỉ public | Giữ private IP → ENI |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "lowest latency, highest throughput between nodes" | Cluster placement group | Một AZ |
| "few critical instances, avoid simultaneous failure" | Spread placement group | 7/AZ |
| "HDFS / Cassandra / Kafka rack awareness" | Partition placement group | |
| "keep same private IP on failover" | Move secondary ENI | Cùng AZ |
| "static public IP for whitelisting" | Elastic IP (hoặc NLB có EIP) | Có tính phí |
| "preserve in-memory state, fast start" | EC2 Hibernate | Root EBS mã hóa |
| "reduce public IPv4 cost" | Bỏ public IP thừa, dùng NAT/IPv6 | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng HPC cần độ trễ mạng giữa các node thấp nhất có thể. Chọn gì?
- A. Spread placement group
- B. Partition placement group
- C. Cluster placement group
- D. Trải instance ra 3 AZ

<details><summary>Đáp án</summary>

**C.** Cluster gom instance trong một AZ trên cùng phân đoạn mạng băng thông cao.
- A sai: Spread ưu tiên cách ly, không tối ưu độ trễ.
- B sai: Partition dành cho hệ phân tán cần cách ly rack.
- D sai: liên AZ có độ trễ cao hơn.
</details>

**Câu 2.** Công ty chạy 6 instance quan trọng và muốn chắc chắn không có hai instance nào nằm chung phần cứng. Chọn gì?
- A. Cluster placement group
- B. Spread placement group
- C. Dedicated Host
- D. Auto Scaling group

<details><summary>Đáp án</summary>

**B.** Mỗi instance trên một rack riêng; 6 instance nằm trong giới hạn 7/AZ.
- A sai: gom chung một chỗ.
- C sai: Dedicated Host đặt nhiều instance trên cùng server.
- D sai: ASG không quyết định vị trí phần cứng.
</details>

**Câu 3.** Cụm Cassandra 200 node cần đảm bảo các replica không chung rack. Chọn gì?
- A. Spread placement group
- B. Partition placement group
- C. Cluster placement group
- D. Một AZ duy nhất

<details><summary>Đáp án</summary>

**B.** Partition hỗ trợ hàng trăm instance, cho ứng dụng biết partition để đặt replica.
- A sai: tối đa 7 instance/AZ.
- C sai: dồn tất cả vào một nhóm phần cứng.
- D sai: không giải quyết rack.
</details>

**Câu 4.** Ứng dụng mất 10 phút để nạp cache vào RAM khi khởi động. Team muốn tắt instance ban đêm để tiết kiệm nhưng sáng mở lại thật nhanh. Chọn gì?
- A. Stop instance
- B. Hibernate instance (root EBS mã hóa)
- C. Dùng instance store
- D. Tạo AMI mỗi tối

<details><summary>Đáp án</summary>

**B.** Hibernate lưu RAM xuống EBS và khôi phục khi start.
- A sai: Stop mất RAM, phải nạp lại.
- C sai: instance store không giữ dữ liệu khi stop và không dùng làm root khi hibernate.
- D sai: AMI không lưu RAM.
</details>

**Câu 5.** (Chọn 2) Điều kiện nào bắt buộc để EC2 hibernate được?
- A. Root volume là EBS và được mã hóa
- B. Instance thuộc Auto Scaling group
- C. Bật hibernation khi launch instance
- D. RAM lớn hơn 150 GiB
- E. Root volume là instance store

<details><summary>Đáp án</summary>

**A, C.**
- B sai: instance trong ASG không hibernate được.
- D sai: Linux phải dưới 150 GiB.
- E sai: instance store không được làm root khi hibernate.
</details>

**Câu 6.** Instance chính lỗi, cần chuyển lưu lượng sang instance dự phòng trong cùng AZ mà client vẫn dùng private IP cũ. Cách nhanh nhất?
- A. Tạo instance mới từ AMI
- B. Tách secondary ENI khỏi instance lỗi và gắn vào instance dự phòng
- C. Đổi CIDR của subnet
- D. Dùng CloudFront

<details><summary>Đáp án</summary>

**B.** ENI mang theo private IP, SG.
- A sai: instance mới nhận IP khác.
- C sai: không liên quan và gây gián đoạn.
- D sai: CloudFront cho nội dung public, không phải IP nội bộ.
</details>

**Câu 7.** Đối tác yêu cầu whitelist một IP public cố định của web server. Kiến trúc hiện tại là 1 EC2. Giải pháp nào **tốt nhất về lâu dài**?
- A. Gắn Elastic IP vào EC2
- B. Đặt Network Load Balancer có Elastic IP cho mỗi AZ phía trước nhóm EC2
- C. Dùng public IP tự cấp
- D. Dùng private IP

<details><summary>Đáp án</summary>

**B.** NLB cho IP tĩnh theo AZ và vẫn scale/HA được.
- A sai: đúng ngắn hạn nhưng gắn chặt vào một instance.
- C sai: đổi khi stop/start.
- D sai: không truy cập được từ Internet.
</details>

**Câu 8.** Hóa đơn tăng vì nhiều EC2 nội bộ có public IPv4 dù chỉ cần gọi API ra ngoài. Cách giảm chi phí?
- A. Gỡ public IPv4, đặt trong private subnet và đi ra qua NAT gateway
- B. Đổi sang Elastic IP vì EIP miễn phí khi đang gắn
- C. Thêm Security Group
- D. Chuyển sang Spread placement group

<details><summary>Đáp án</summary>

**A.** Ít public IP hơn; một NAT gateway dùng chung cho nhiều instance (cân nhắc thêm phí NAT).
- B sai: EIP đang gắn vẫn tính phí từ 02/2024.
- C sai: SG không ảnh hưởng chi phí.
- D sai: placement group không liên quan.
</details>

---

### Nguồn đã đối chiếu
- Elastic IP addresses (tính phí mọi public IPv4, quota theo Region): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html
- Placement strategies (cluster, spread 7/AZ, partition): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-strategies.html
- Elastic network interfaces: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html
- Hibernation prerequisites (RAM, mã hóa root, 60 ngày, ASG/ECS): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hibernating-prerequisites.html
- How hibernation works: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-hibernate-overview.html
- Warm pools for Auto Scaling: https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-warm-pools.html
