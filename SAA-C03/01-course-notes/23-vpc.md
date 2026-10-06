# SAA-C03 — Networking: Amazon VPC

> Thuộc **Domain 1: Design Secure Architectures**, **Domain 2: Design Resilient Architectures** và **Domain 4: Design Cost-Optimized Architectures**: CIDR, subnet, IGW, NAT, SG/NACL, peering, endpoint, flow logs, kết nối lai, Transit Gateway, IPv6, chi phí mạng, Network Firewall.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **VPC** theo Region, CIDR IPv4 từ **/16** tới **/28** (nên dùng dải private RFC 1918), tối đa 5 CIDR mỗi VPC; **subnet** theo AZ, AWS giữ **5 IP** mỗi subnet.
- **IGW** cho public subnet (route `0.0.0.0/0 → IGW`); private subnet ra Internet qua **NAT gateway** (managed) hoặc NAT instance (cũ, phải tắt source/destination check).
- ⚠️ Lỗi thời: "NAT gateway phải đặt ở public subnet, mỗi AZ một cái" → nay có thêm **Regional NAT gateway**: không cần public subnet, tự mở rộng đa AZ, một ID dùng chung, tới 32 IP/AZ (zonal 8).
- **SG** (stateful, chỉ Allow, cấp ENI) vs **NACL** (stateless, Allow + Deny, có số thứ tự, cấp subnet, phải mở **ephemeral port 1024–65535** cho chiều về).
- **VPC peering** không bắc cầu, CIDR không trùng; **VPC endpoint**: **Gateway** (S3, DynamoDB, miễn phí, qua route table) và **Interface** (PrivateLink, ENI, tính phí).
- **Flow Logs** gửi tới **S3, CloudWatch Logs, Data Firehose** (⚠️ slide ghi "CloudFront" là sai).
- Lai: **Site-to-Site VPN** (qua Internet, mã hóa, lên nhanh) và **Direct Connect** (đường riêng 1/10/100/400 Gbps dedicated, hosted 50 Mbps–25 Gbps; mất hơn 1 tháng để lắp, **không mã hóa mặc định**). **Transit Gateway** nối hub-and-spoke, có bắc cầu.
- ⚠️ Lỗi thời: **EC2-Classic & ClassicLink** đã ngừng hoàn toàn.

---

## 2. Core Concepts

### VPC, CIDR & subnet
- **Là gì:** mạng riêng ảo theo Region; CIDR không được trùng mạng khác cần kết nối; subnet public/private theo AZ; 5 IP bị giữ: .0 (network), .1 (router), .2 (DNS), .3 (dự phòng), .255 (broadcast).
- **Dùng khi nào:** cần 29 IP cho EC2 → /27 (32 − 5 = 27) **không đủ**, chọn /26.
- **Vì sao quan trọng:** đề tính số IP khả dụng: 2^(32 − prefix) − 5.

### Internet Gateway & route tables
- **Là gì:** IGW cho phép tài nguyên trong VPC ra/vào Internet, scale ngang, HA; một VPC chỉ gắn một IGW.
- **Dùng khi nào:** public subnet = route table có `0.0.0.0/0 → IGW` + instance có public IP/EIP.
- **Vì sao quan trọng:** tạo IGW thôi chưa đủ, phải sửa route table.

### Bastion host & Session Manager
- **Là gì:** EC2 trong public subnet để SSH vào instance private; SG bastion chỉ mở port 22 từ dải IP công ty.
- **Dùng khi nào:** cần truy cập quản trị instance private.
- **Vì sao quan trọng:** thiết kế tốt hơn: **SSM Session Manager** (không mở port 22, có audit) hoặc **EC2 Instance Connect Endpoint**.

### NAT gateway
- **Là gì:** NAT managed, trả theo giờ + GB; zonal: đặt ở public subnet một AZ, dùng EIP, 5 Gbps tự scale tới 100 Gbps; không dùng được từ chính subnet của nó; không cần SG.
- **Dùng khi nào:** private subnet cần ra Internet (cập nhật OS, gọi API) nhưng không cho vào.
- **Vì sao quan trọng:** zonal: HA = mỗi AZ một NAT + route riêng. ⚠️ Lỗi thời: mô hình zonal không còn là lựa chọn duy nhất → **Regional NAT gateway**: không cần public subnet, tự mở rộng theo AZ có workload, có route table riêng; private NAT vẫn chỉ zonal.

### Security Group vs NACL
- **Là gì:** SG: cấp ENI, stateful (chiều về tự cho phép), chỉ Allow, đánh giá mọi rule; NACL: cấp subnet, stateless, Allow/Deny, đánh giá theo số thứ tự nhỏ → lớn, khớp đầu tiên thắng; NACL mặc định cho phép tất cả, NACL mới tạo chặn tất cả.
- **Dùng khi nào:** chặn một IP cụ thể → **NACL Deny** (SG không Deny được).
- **Vì sao quan trọng:** NACL phải mở **ephemeral port** (1024–65535) cho chiều về; thứ tự rule quan trọng (rule 100 Allow thắng rule 200 Deny).

### VPC Peering
- **Là gì:** nối riêng hai VPC qua mạng AWS như một mạng; được cross-account, cross-Region; SG tham chiếu được SG của VPC peered (cùng Region).
- **Dùng khi nào:** vài VPC cần nói chuyện với nhau.
- **Vì sao quan trọng:** **không bắc cầu** (A–B, B–C không có nghĩa A–C), CIDR **không trùng**, phải cập nhật route table **hai phía**.

### VPC Endpoints & PrivateLink
- **Là gì:** **Gateway endpoint**: S3 và DynamoDB, đích trong route table, **miễn phí**; **Interface endpoint**: ENI có private IP trong subnet, hầu hết dịch vụ AWS (kể cả S3), tính phí theo giờ + GB, dùng SG; **PrivateLink** cũng để expose dịch vụ của bạn cho VPC khác qua **NLB/GWLB**.
- **Dùng khi nào:** EC2 private truy cập S3 không qua NAT/Internet → gateway endpoint; truy cập từ on-prem qua VPN/DX → interface endpoint; cung cấp SaaS cho 1,000 VPC khách hàng → PrivateLink (không cần peering, CIDR trùng vẫn được).
- **Vì sao quan trọng:** endpoint lỗi → kiểm tra DNS resolution của VPC và route table.

### VPC Flow Logs
- **Là gì:** ghi thông tin IP traffic ở cấp VPC, subnet, ENI (kể cả ELB, RDS, ElastiCache, Redshift, NAT, Transit Gateway); đích: **S3, CloudWatch Logs, Data Firehose**.
- **Dùng khi nào:** chẩn đoán kết nối: inbound REJECT → NACL hoặc SG; inbound ACCEPT nhưng outbound REJECT → NACL (vì SG stateful).
- **Vì sao quan trọng:** ⚠️ slide ghi "xuất về CloudFront" → sai, đúng là CloudWatch Logs. Phân tích bằng Athena (S3) hoặc Logs Insights/Contributor Insights.

### Site-to-Site VPN
- **Là gì:** **Virtual Private Gateway** (VGW, phía AWS, gắn VPC) + **Customer Gateway** (CGW, thiết bị/phần mềm phía on-prem); IPsec qua Internet; **route propagation** trong route table.
- **Dùng khi nào:** cần kết nối nhanh, mã hóa, chi phí thấp; CGW sau NAT thì dùng public IP của NAT; ping từ on-prem cần mở ICMP trong SG.
- **Vì sao quan trọng:** **VPN CloudHub**: nhiều site on-prem nói chuyện với nhau qua VGW (hub-and-spoke, dynamic routing). **Client VPN** cho máy cá nhân (OpenVPN).

### AWS Direct Connect
- **Là gì:** kết nối riêng từ data center tới DX location rồi vào AWS; **public VIF** (dịch vụ public như S3), **private VIF** (VPC), **transit VIF** (Transit Gateway); dedicated 1/10/100/400 Gbps, hosted 50 Mbps–25 Gbps qua partner.
- **Dùng khi nào:** băng thông lớn ổn định, độ trễ đều, dữ liệu lớn; **Direct Connect Gateway** nối một DX tới nhiều VPC ở nhiều Region.
- **Vì sao quan trọng:** lắp đặt **hơn 1 tháng**; **không mã hóa** mặc định → chạy **VPN trên DX** hoặc **MACsec**. Resiliency: **high** = 2 kết nối ở 2 DX location; **maximum** = 2 thiết bị × 2 location. DX + **VPN dự phòng** là phương án tiết kiệm.

### AWS Transit Gateway
- **Là gì:** hub nối hàng nghìn VPC, on-prem (VPN, DX Gateway) theo mô hình hub-and-spoke, **có bắc cầu**; theo Region, peering được giữa Region; share giữa account bằng **RAM**; route table kiểm soát VPC nào nói với VPC nào.
- **Dùng khi nào:** nhiều VPC + nhiều site; **ECMP** để tăng băng thông VPN (nhiều tunnel song song); hỗ trợ **IP multicast** (duy nhất trong AWS).
- **Vì sao quan trọng:** thay mạng peering lưới phức tạp; chia sẻ một DX cho nhiều account.

### IPv6, Traffic Mirroring, chi phí mạng
- **Là gì:** VPC dual-stack (IPv4 + IPv6 public); **egress-only IGW** cho IPv6 đi ra mà không cho vào (giống NAT cho IPv6); **Traffic Mirroring** sao chép traffic ENI tới NLB/appliance để phân tích.
- **Dùng khi nào:** không launch được EC2 vì hết IPv4 trong subnet (IPv6 không phải nguyên nhân) → thêm CIDR IPv4.
- **Vì sao quan trọng:** chi phí: traffic vào miễn phí; cùng AZ qua private IP miễn phí; khác AZ tính phí; khác Region đắt hơn; dùng private IP thay public IP; S3 qua **gateway endpoint** rẻ hơn NAT; Direct Connect egress rẻ hơn Internet.

### AWS Network Firewall
- **Là gì:** firewall managed bảo vệ cả VPC, Layer 3–7, mọi hướng (VPC ↔ VPC, ↔ Internet, ↔ DX/VPN); dùng GWLB bên dưới; rule: IP/port, giao thức, **lọc domain**, regex; gửi log về S3, CloudWatch, Firehose.
- **Dùng khi nào:** kiểm tra/lọc toàn bộ traffic ra Internet theo domain, IDS/IPS.
- **Vì sao quan trọng:** quản lý tập trung nhiều account bằng **Firewall Manager**.

### `[+]` Route 53 Resolver endpoints
- **Là gì:** **inbound endpoint** cho DNS on-prem phân giải tên trong VPC (private hosted zone); **outbound endpoint** + forwarding rule cho VPC phân giải tên on-prem.
- **Dùng khi nào:** hybrid DNS qua VPN/DX.
- **Vì sao quan trọng:** đề hay hỏi đúng chiều inbound/outbound.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| CIDR | Dải IP | "/16 to /28" |
| 5 reserved IPs | AWS giữ 5 IP mỗi subnet | "how many usable IPs" |
| Internet Gateway | Cửa ra/vào Internet của VPC | "public subnet" |
| NAT gateway | Private subnet ra Internet | "outbound only internet" |
| Regional NAT gateway | NAT tự trải đa AZ, không cần public subnet | "simplify NAT HA" |
| Egress-only IGW | IPv6 chỉ đi ra | "IPv6 outbound only" |
| NACL | Firewall stateless cấp subnet | "block specific IP" |
| Ephemeral ports | 1024–65535 cho chiều về | "NACL return traffic" |
| VPC peering | Nối 2 VPC, không bắc cầu | "connect two VPCs" |
| Gateway endpoint | S3/DynamoDB qua route table | "private S3 access, free" |
| Interface endpoint | ENI PrivateLink | "private access to AWS services" |
| PrivateLink | Expose service qua NLB | "share service with many VPCs" |
| Flow Logs | Log IP traffic | "troubleshoot connectivity" |
| VGW / CGW | Hai đầu Site-to-Site VPN | "IPsec to on-prem" |
| VPN CloudHub | Nhiều site qua VGW | "branch offices connect each other" |
| Direct Connect | Đường riêng tới AWS | "consistent bandwidth, private" |
| DX Gateway | DX tới nhiều VPC/Region | "one DX, many VPCs" |
| Transit Gateway | Hub có bắc cầu | "hundreds of VPCs", "transitive" |
| ECMP | Nhiều tunnel song song | "increase VPN bandwidth" |
| Network Firewall | Firewall cả VPC L3–L7 | "domain filtering", "IPS" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Security Group | NACL | SG stateful, Allow, ENI; NACL stateless, Allow/Deny, subnet | Chặn IP → NACL |
| NAT gateway | NAT instance | NAT gateway managed, HA trong AZ; NAT instance tự quản, tắt src/dst check | Luôn chọn NAT gateway |
| NAT gateway zonal | Regional | Zonal: public subnet, mỗi AZ một; Regional: không cần public subnet, tự đa AZ | |
| NAT gateway | Egress-only IGW | NAT cho IPv4; egress-only cho IPv6 | IPv6 ra Internet → egress-only |
| VPC peering | Transit Gateway | Peering 1–1, không bắc cầu; TGW hub, bắc cầu | Nhiều VPC → TGW |
| Gateway endpoint | Interface endpoint | Gateway: S3/DynamoDB, free, route table; Interface: ENI, phí, truy cập từ on-prem được | On-prem → S3 riêng tư → interface |
| VPC peering | PrivateLink | Peering mở cả mạng; PrivateLink chỉ expose một service, CIDR trùng vẫn được | SaaS cho nhiều khách → PrivateLink |
| Site-to-Site VPN | Direct Connect | VPN qua Internet, mã hóa, nhanh; DX riêng, ổn định, lâu lắp, không mã hóa mặc định | "Within a week" → VPN |
| VPN CloudHub | Transit Gateway | CloudHub nối site qua VGW; TGW nối VPC + site | |
| Flow Logs | Traffic Mirroring | Flow Logs metadata; Mirroring copy gói tin đầy đủ | Deep packet inspection → Mirroring |
| Network Firewall | WAF | Network Firewall cả VPC, L3–L7; WAF HTTP gắn ALB/CloudFront | Domain filtering outbound → Network Firewall |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "private instances need internet for updates" | NAT gateway (Regional hoặc mỗi AZ một zonal) | |
| "block a malicious IP" | NACL Deny | |
| "EC2 in private subnet access S3 without internet, cheapest" | Gateway endpoint | |
| "access S3 privately from on-prem via DX/VPN" | S3 interface endpoint | |
| "expose service to 1,000 customer VPCs, overlapping CIDR" | PrivateLink (NLB) | |
| "connect hundreds of VPCs and on-prem, transitive" | Transit Gateway | |
| "increase VPN throughput" | Transit Gateway + ECMP | |
| "IP multicast" | Transit Gateway | |
| "dedicated private connection, consistent latency" | Direct Connect | |
| "connection needed within days" | Site-to-Site VPN | |
| "encrypt Direct Connect traffic" | VPN over DX hoặc MACsec | |
| "DX to VPCs in multiple Regions" | Direct Connect Gateway | |
| "cost-effective backup for DX" | Site-to-Site VPN | |
| "IPv6 outbound only" | Egress-only IGW | |
| "analyze rejected traffic" | VPC Flow Logs | |
| "filter outbound traffic by domain name" | AWS Network Firewall | |
| "on-prem resolves private hosted zone names" | Route 53 Resolver inbound endpoint | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Subnet `/27` cần chứa 29 EC2. Đủ không?
- A. Đủ, /27 có 32 IP
- B. Không, chỉ có 27 IP khả dụng vì AWS giữ 5 IP
- C. Đủ, AWS chỉ giữ 2 IP
- D. Không, /27 chỉ có 16 IP

<details><summary>Đáp án</summary>

**B.** 32 − 5 = 27 < 29; chọn /26.
- A, C sai: quên 5 IP bị giữ.
- D sai: /27 có 32 IP.
</details>

**Câu 2.** Phát hiện một IP liên tục tấn công web server. Cách chặn ở cấp subnet?
- A. Thêm rule Deny vào Security Group
- B. Thêm rule Deny vào NACL với số thứ tự nhỏ hơn rule Allow
- C. Xóa IGW
- D. Dùng NAT gateway

<details><summary>Đáp án</summary>

**B.**
- A sai: SG không có Deny.
- C sai: cắt toàn bộ Internet.
- D sai: NAT cho outbound.
</details>

**Câu 3.** EC2 trong private subnet cần đọc S3 khối lượng lớn; chi phí NAT quá cao. Cách tiết kiệm nhất?
- A. S3 gateway endpoint
- B. Thêm NAT gateway ở mỗi AZ
- C. Đặt EC2 vào public subnet
- D. Interface endpoint cho S3

<details><summary>Đáp án</summary>

**A.** Miễn phí, đi qua route table, không qua NAT.
- B sai: tăng chi phí.
- C sai: giảm bảo mật.
- D sai: chạy được nhưng có phí; gateway rẻ hơn khi truy cập từ trong VPC.
</details>

**Câu 4.** (Chọn 2) Công ty có 50 VPC và 3 văn phòng; muốn mọi VPC và văn phòng nói chuyện được với nhau, dễ quản lý. Chọn gì?
- A. Transit Gateway
- B. Site-to-Site VPN attachment vào Transit Gateway
- C. VPC peering lưới giữa 50 VPC
- D. Internet Gateway
- E. NAT gateway

<details><summary>Đáp án</summary>

**A, B.**
- C sai: hơn 1,000 kết nối, không bắc cầu tới văn phòng.
- D, E sai: không nối VPC với nhau.
</details>

**Câu 5.** Công ty cần kết nối riêng tới AWS với băng thông ổn định, nhưng ngay tuần sau đã phải chạy. Giải pháp?
- A. Chờ Direct Connect
- B. Site-to-Site VPN trước, chuyển sang Direct Connect khi lắp xong (giữ VPN làm dự phòng)
- C. VPC peering
- D. CloudFront

<details><summary>Đáp án</summary>

**B.** DX mất hơn 1 tháng; VPN lên nhanh.
- A sai: không kịp.
- C sai: nối VPC với VPC.
- D sai: CDN.
</details>

**Câu 6.** Flow Logs cho thấy inbound ACCEPT nhưng outbound REJECT cho cùng kết nối. Nguyên nhân?
- A. Security Group chặn
- B. NACL thiếu rule outbound cho ephemeral port
- C. Thiếu IGW
- D. Route 53 lỗi

<details><summary>Đáp án</summary>

**B.** NACL stateless; SG stateful nên chiều về tự được phép.
- A sai: SG đã cho vào thì tự cho ra.
- C, D sai: không gây mẫu ACCEPT/REJECT này.
</details>

**Câu 7.** Nhà cung cấp SaaS muốn phục vụ API cho hàng trăm khách hàng có CIDR trùng nhau, không đi qua Internet. Chọn gì?
- A. VPC peering với từng khách
- B. AWS PrivateLink (endpoint service sau NLB)
- C. Transit Gateway chung
- D. Public ALB

<details><summary>Đáp án</summary>

**B.**
- A sai: CIDR trùng không peering được.
- C sai: CIDR trùng gây xung đột route, mở cả mạng.
- D sai: đi qua Internet.
</details>

**Câu 8.** Cần đảm bảo HA cho NAT ở 3 AZ với ít cấu hình nhất (10/2026). Chọn gì?
- A. Một NAT gateway zonal ở AZ-a
- B. Regional NAT gateway (hoặc 3 NAT zonal, mỗi AZ một)
- C. NAT instance trong ASG
- D. Internet Gateway cho mọi subnet

<details><summary>Đáp án</summary>

**B.** Regional NAT tự mở rộng theo AZ; cách cũ là mỗi AZ một NAT zonal.
- A sai: AZ-a lỗi thì AZ khác mất Internet.
- C sai: nhiều vận hành.
- D sai: biến subnet private thành public.
</details>

---

### Nguồn đã đối chiếu
- Regional NAT gateways (không cần public subnet, 32 IP/AZ): https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateways-regional.html
- Subnet CIDR blocks (5 IP bị giữ): https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html
- Network ACLs: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html
- VPC peering limitations: https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-basics.html
- Gateway endpoints: https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html
- VPC Flow Logs (đích S3, CloudWatch Logs, Firehose): https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html
- Dedicated Direct Connect connections (1/10/100/400 Gbps): https://docs.aws.amazon.com/directconnect/latest/UserGuide/dedicated_connection.html
- Hosted Direct Connect connections: https://docs.aws.amazon.com/directconnect/latest/UserGuide/hosted_connection.html
- Transit Gateway: https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html
- AWS Network Firewall: https://docs.aws.amazon.com/network-firewall/latest/developerguide/what-is-aws-network-firewall.html
- EC2-Classic retirement / ClassicLink: https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html
