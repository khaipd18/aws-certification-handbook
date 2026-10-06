# SOA-C03 — Amazon VPC: Networking, Connectivity & Protection

> Thuộc **Domain 5: Networking and Content Delivery (18%)**: CIDR và subnet, IGW/NAT, SG vs NACL, endpoint, peering, Transit Gateway, VPN, Direct Connect, Flow Logs và xử lý sự cố kết nối.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- AWS **giữ 5 IP mỗi subnet** (4 đầu + 1 cuối): `/28` chỉ còn 11 IP dùng được.
- **Không đổi CIDR** của VPC/subnet có sẵn: thêm secondary CIDR hoặc tạo subnet/VPC mới.
- **Security group**: stateful, mức instance/ENI, chỉ allow. **NACL**: stateless, mức subnet, có allow và deny, xét theo số rule. Vì NACL stateless nên phải mở **ephemeral port 1024–65535** cho chiều về.
- **NAT Gateway** nằm trong một AZ, cần IGW, nên đặt mỗi AZ một cái. ⚠️ **NAT instance** là cách cũ, NAT AMI đã hết bảo trì.
- **Endpoint**:
  - **Gateway endpoint**: miễn phí, chỉ S3 và DynamoDB, là target trong route table.
  - **Interface endpoint (PrivateLink)**: ENI có SG, tính phí giờ + GB.
- **VPC peering** không bắc cầu, CIDR không được trùng. **Transit Gateway** là hub-and-spoke cho nhiều VPC, VPN, Direct Connect, share qua RAM.
- **Direct Connect** riêng tư nhưng **không mã hóa**: cần mã hóa thì chạy VPN trên DX. Site-to-Site VPN là backup rẻ cho DX.
- **VPC Flow Logs**:
  - Inbound REJECT → do SG hoặc NACL.
  - Inbound ACCEPT nhưng outbound REJECT → do **NACL**.

---

## 2. Core Concepts

### CIDR, private range & sizing
- **Là gì:** CIDR = IP gốc + mask (`/32` = 1 IP, `/24` = 256, `/16` = 65,536). Dải private: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16.
- **Dùng khi nào:** VPC CIDR từ `/28` tới `/16`, mặc định tối đa 5 CIDR mỗi VPC (điều chỉnh được); VPC mỗi Region có quota mềm.
- **Vì sao quan trọng:** 29 instance cần `/26` (64 − 5 = 59), không phải `/27` (32 − 5 = 27). CIDR trùng với on-prem/VPC khác thì không peering được. Quota thực tế: kiểm tra Service Quotas.

### IPAM & đổi CIDR
- **Là gì:** VPC IP Address Manager quản lý dải IP tập trung qua scope → pool → sub-pool theo Region, cảnh báo CIDR trùng.
- **Dùng khi nào:** hàng trăm account cần cấp CIDR không chồng nhau.
- **Vì sao quan trọng:** hết IP trong subnet → tạo subnet mới lớn hơn; VPC chật → thêm secondary CIDR.

### Internet Gateway & bastion
- **Là gì:** IGW co giãn, HA; một VPC một IGW. Subnet public = route table có `0.0.0.0/0 → IGW`.
- **Dùng khi nào:** bastion ở public subnet, SG chỉ mở 22 từ CIDR công ty; instance private cho SSH từ SG của bastion.
- **Vì sao quan trọng:** chỉ gắn IGW chưa đủ, phải có route. Session Manager hoặc EIC Endpoint bỏ được bastion.

### NAT Gateway & NAT instance
- **Là gì:** NAT Gateway managed, tạo trong một subnet public của một AZ với Elastic IP, cần IGW; không có SG. Tới 100 Gbps, tối đa 55,000 kết nối đồng thời mỗi đích.
- **Dùng khi nào:** instance private cần ra Internet (cập nhật, gọi API ngoài). Mỗi AZ một NAT Gateway để chịu lỗi AZ.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: NAT instance (phải tắt Source/Destination Check, tự lo HA) → NAT AMI dựa trên Amazon Linux 2018.03, hết bảo trì 31/12/2023; AWS khuyên chuyển NAT Gateway. `[+]` Nay có **Regional NAT gateway** gắn với VPC, tự mở rộng qua AZ, không cần public subnet riêng.

### DNS attributes
- **Là gì:** `enableDnsSupport` (mặc định true) dùng Resolver ở 169.254.169.253 / CIDR + 2; `enableDnsHostnames` (true ở default VPC, false ở VPC tự tạo) gán DNS name public.
- **Dùng khi nào:** private hosted zone, interface endpoint với private DNS.
- **Vì sao quan trọng:** private hosted zone chỉ chạy khi **cả hai** đều true.

### Network ACL & ephemeral ports
- **Là gì:** mỗi subnet một NACL; rule 1–32766, số nhỏ xét trước, khớp là dừng, cuối có deny ngầm. NACL mới tạo deny hết; NACL mặc định allow hết.
- **Dùng khi nào:** chặn một IP cụ thể (SG không deny được).
- **Vì sao quan trọng:** chiều về dùng ephemeral port (Linux thường 32768–60999, Windows 49152–65535) → mở 1024–65535 để an toàn. "Kết nối được một chiều" → NACL.

### Security group vs NACL & prefix list
- **Là gì:** SG mức instance, chỉ allow, stateful, xét tất cả rule, tham chiếu được SG khác; NACL mức subnet, allow + deny, stateless, xét theo thứ tự, chỉ CIDR.
- **Dùng khi nào:** managed prefix list gom nhiều CIDR để dùng trong SG/route table; customer-managed share được cho account/Organization; AWS-managed (S3, CloudFront…) không sửa được.
- **Vì sao quan trọng:** sửa prefix list là mọi rule tham chiếu cập nhật theo.

### Reachability Analyzer & VPC Block Public Access
- **Là gì:** Reachability Analyzer dựng mô hình cấu hình và kiểm tra đường đi giữa hai điểm, **không gửi gói tin**; VPC Block Public Access chặn Internet theo Region (Bidirectional hoặc Ingress-only), có exclusion.
- **Dùng khi nào:** "vì sao A không tới được B" → Reachability Analyzer chỉ ra thành phần chặn; chặn Internet cho cả Region → Block Public Access.
- **Vì sao quan trọng:** Reachability Analyzer trả đường đi từng hop hoặc tên SG/NACL/route gây chặn.

### VPC peering
- **Là gì:** kết nối riêng giữa hai VPC (khác account, khác Region được).
- **Dùng khi nào:** vài VPC cần nói chuyện trực tiếp.
- **Vì sao quan trọng:** **không bắc cầu**, CIDR **không trùng**, phải sửa route table **cả hai phía**; tham chiếu SG qua peering cùng Region.

### VPC endpoints & PrivateLink
- **Là gì:** gateway endpoint (S3, DynamoDB, route table, miễn phí); interface endpoint (ENI + SG, hầu hết dịch vụ, tính phí); endpoint service (PrivateLink) expose dịch vụ của bạn qua **NLB** (hoặc GWLB) cho VPC/account khác.
- **Dùng khi nào:** Lambda/EC2 private gọi S3/DynamoDB → gateway endpoint thay NAT; truy cập từ on-prem hoặc VPC khác → interface endpoint; SaaS cho hàng nghìn VPC → PrivateLink.
- **Vì sao quan trọng:** endpoint không chạy → kiểm tra DNS attributes, private DNS, route table, SG của endpoint, endpoint policy.

### VPC Flow Logs
- **Là gì:** metadata traffic ở mức VPC, subnet, ENI (kể cả ENI của ELB, RDS, NAT Gateway…), gửi tới S3, CloudWatch Logs hoặc Data Firehose; truy vấn bằng Athena / Logs Insights.
- **Dùng khi nào:** đọc `srcaddr`, `dstaddr`, `srcport`, `dstport`, `action` để biết ai bị chặn.
- **Vì sao quan trọng:** inbound REJECT → SG hoặc NACL; inbound ACCEPT, outbound REJECT → NACL (SG stateful sẽ cho chiều về). Gửi tới CloudWatch Logs cần role có `logs:CreateLogStream`, `logs:PutLogEvents`.

### Site-to-Site VPN, CloudHub, Client VPN
- **Là gì:** VGW hoặc Transit Gateway phía AWS, Customer Gateway phía bạn (IP public, hoặc IP của thiết bị NAT khi dùng NAT-T); tunnel chuẩn tới 1.25 Gbps, tunnel large bandwidth tới 5 Gbps. CloudHub nối nhiều site qua một VGW; Client VPN dùng OpenVPN cho từng máy.
- **Dùng khi nào:** kết nối nhanh, rẻ qua Internet; backup cho Direct Connect.
- **Vì sao quan trọng:** hay quên **bật route propagation** trên route table và mở **ICMP** nếu muốn ping từ on-prem. Cần hơn 1.25 Gbps → nhiều tunnel với **ECMP trên Transit Gateway** (VGW không hỗ trợ ECMP).

### Direct Connect
- **Là gì:** đường riêng từ data center tới AWS; dedicated 1, 10, 100, 400 Gbps; hosted 50 Mbps–25 Gbps; truy cập private (VIF private) và public (S3).
- **Dùng khi nào:** băng thông ổn định, độ trễ dự đoán được; **Direct Connect Gateway** nối một DX tới VPC ở nhiều Region.
- **Vì sao quan trọng:** **không mã hóa** → VPN trên DX (hoặc MACsec với kết nối hỗ trợ). Resiliency cao: hai location; tối đa: kết nối riêng trên thiết bị riêng ở nhiều location. Lead time có thể hơn một tháng.

### Transit Gateway & Traffic Mirroring
- **Là gì:** Transit Gateway là hub định tuyến bắc cầu cho VPC, VPN, DX; resource theo Region, peering được giữa Region; route table điều khiển attachment nào nói chuyện với nhau; hỗ trợ multicast. Traffic Mirroring sao chép traffic từ ENI tới ENI/NLB để phân tích.
- **Dùng khi nào:** hàng chục/hàng trăm VPC → Transit Gateway thay mesh peering; IDS, điều tra sâu → Traffic Mirroring.
- **Vì sao quan trọng:** share Transit Gateway qua AWS RAM cho các account trong Organization.

### `[+]` IPv6 trong VPC
- **Là gì:** mọi IPv6 trên AWS đều public, định tuyến được trên Internet; VPC chạy dual-stack.
- **Dùng khi nào:** instance IPv6 cần ra Internet mà không cho vào → **egress-only Internet Gateway**.
- **Vì sao quan trọng:** NAT Gateway dành cho IPv4; "không launch được instance" trong subnet dual-stack thường do hết IPv4, không phải IPv6.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| 5 IP reserved | AWS giữ 5 IP mỗi subnet | "how many usable IPs" |
| Secondary CIDR | Mở rộng VPC | "VPC running out of IPs" |
| NAT Gateway | NAT managed theo AZ | "private instances need internet" |
| NACL | Firewall subnet, stateless | "block specific IP" |
| Ephemeral ports | Port chiều về 1024–65535 | "works one way only" |
| Gateway endpoint | S3/DynamoDB miễn phí | "avoid NAT costs for S3" |
| Interface endpoint | ENI PrivateLink | "private access to AWS API" |
| VPC peering | Kết nối hai VPC, không bắc cầu | "A-B, B-C but not A-C" |
| Transit Gateway | Hub định tuyến bắc cầu | "hundreds of VPCs" |
| Route propagation | Đưa route VPN vào route table | "VPN up but no traffic" |
| Direct Connect | Đường riêng, không mã hóa | "consistent bandwidth" |
| Reachability Analyzer | Phân tích đường đi | "why can't instance reach" |
| VPC Flow Logs | Metadata ACCEPT/REJECT | "troubleshoot SG vs NACL" |
| Egress-only IGW | IPv6 chỉ ra ngoài | "IPv6 outbound only" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Security group | NACL | Stateful, allow, instance; stateless, allow + deny, subnet | Chặn IP → NACL |
| NAT Gateway | NAT instance | Managed, HA trong AZ; tự quản, cách cũ | Đáp án mới → NAT Gateway |
| NAT Gateway | Egress-only IGW | IPv4; IPv6 | IPv6 ra ngoài → egress-only |
| Gateway endpoint | Interface endpoint | S3/DynamoDB, miễn phí, route; ENI, tính phí, đa dịch vụ | On-prem dùng → interface |
| VPC peering | Transit Gateway | Một-một, không bắc cầu; hub bắc cầu | Nhiều VPC → TGW |
| PrivateLink | VPC peering | Expose một dịch vụ qua NLB; nối cả mạng | SaaS cho nhiều khách → PrivateLink |
| Site-to-Site VPN | Direct Connect | Internet, mã hóa, nhanh dựng; đường riêng, không mã hóa | Mã hóa trên DX → VPN over DX |
| VGW | Transit Gateway cho VPN | Không ECMP; ECMP gộp băng thông | > 1.25 Gbps → TGW + ECMP |
| Reachability Analyzer | Flow Logs | Phân tích cấu hình, không gửi gói; ghi traffic thật | "Before traffic exists" → Analyzer |
| Default NACL | NACL mới tạo | Allow hết; deny hết | NACL mới chặn mọi thứ |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "how many usable IPs in /27" | 32 − 5 = 27 | |
| "subnet out of IPs" | Subnet mới lớn hơn | Không đổi CIDR subnet cũ |
| "private instances download patches" | NAT Gateway mỗi AZ | |
| "Lambda in VPC to DynamoDB cheapest" | Gateway endpoint | |
| "block one malicious IP" | NACL deny | |
| "response traffic blocked" | NACL thiếu ephemeral port | |
| "Flow Logs: inbound ACCEPT, outbound REJECT" | NACL | |
| "private hosted zone / endpoint DNS not working" | `enableDnsSupport` + `enableDnsHostnames` | |
| "A-B and B-C peered, A cannot reach C" | Peering không bắc cầu → peering A-C / TGW | |
| "VPN connected, instances unreachable" | Route propagation, SG/NACL, ICMP | |
| "encrypt Direct Connect traffic" | VPN over DX | |
| "backup for Direct Connect, low cost" | Site-to-Site VPN | |
| "VPN throughput above 1.25 Gbps" | TGW + ECMP nhiều tunnel / large bandwidth tunnel | |
| "expose service to 1,000 customer VPCs" | PrivateLink (NLB) | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Cần subnet chứa ít nhất 29 instance, mỗi instance một IP. Mask nhỏ nhất phù hợp?
- A. /28
- B. /27
- C. /26
- D. /29

<details><summary>Đáp án</summary>

**C.** /26 có 64 − 5 = 59 IP.
- A sai: 16 − 5 = 11.
- B sai: 32 − 5 = 27, thiếu 2.
- D sai: 8 − 5 = 3.
</details>

**Câu 2.** Instance trong subnet gọi API bên ngoài được, nhưng phản hồi không về. SG cho phép outbound tất cả. Flow Logs cho thấy outbound ACCEPT, inbound REJECT. Nguyên nhân?
- A. Security group thiếu inbound rule
- B. NACL không cho phép inbound ephemeral port
- C. Thiếu IGW
- D. Thiếu NAT Gateway

<details><summary>Đáp án</summary>

**B.** SG stateful tự cho chiều về; NACL stateless phải mở ephemeral port.
- A sai: SG không chặn traffic trả về của kết nối đã cho phép.
- C, D sai: gói đi ra đã được ACCEPT và gửi được.
</details>

**Câu 3.** (Chọn 2) Lambda trong private subnet cần ghi DynamoDB và đọc S3, không qua Internet, chi phí thấp nhất. Chọn gì?
- A. Gateway endpoint cho DynamoDB
- B. Gateway endpoint cho S3
- C. NAT Gateway
- D. Interface endpoint cho DynamoDB qua on-prem
- E. Internet Gateway

<details><summary>Đáp án</summary>

**A, B.** Gateway endpoint miễn phí, là target route table.
- C sai: tốn phí theo giờ và GB.
- D sai: không cần, tốn phí.
- E sai: đi qua Internet.
</details>

**Câu 4.** VPC A peering với B, B peering với C. Instance ở A không tới được C. Cách sửa?
- A. Bật transitive routing trên peering
- B. Tạo peering A–C (và cập nhật route) hoặc dùng Transit Gateway
- C. Thêm NAT Gateway ở B
- D. Đổi SG ở C sang 0.0.0.0/0

<details><summary>Đáp án</summary>

**B.**
- A sai: peering không hỗ trợ bắc cầu.
- C sai: NAT không chuyển tiếp traffic peering.
- D sai: vấn đề là định tuyến, không phải SG.
</details>

**Câu 5.** Site-to-Site VPN báo cả hai tunnel UP nhưng instance không nhận traffic từ on-prem. Kiểm tra gì trước?
- A. Bật route propagation (hoặc route tĩnh) cho route table của subnet
- B. Tạo thêm IGW
- C. Tăng instance type
- D. Bật VPC Block Public Access

<details><summary>Đáp án</summary>

**A.** Không có route về dải on-prem qua VGW thì traffic không đi.
- B sai: VPN không cần IGW cho traffic private.
- C sai: không liên quan.
- D sai: chặn thêm, không sửa.
</details>

**Câu 6.** Công ty dùng Direct Connect, kiểm toán yêu cầu mã hóa traffic giữa on-prem và VPC. Cách làm?
- A. DX tự mã hóa, không cần làm gì
- B. Chạy Site-to-Site VPN (IPsec) trên Direct Connect
- C. Dùng gateway endpoint
- D. Bật VPC Flow Logs

<details><summary>Đáp án</summary>

**B.**
- A sai: DX riêng tư nhưng không mã hóa.
- C sai: không liên quan kết nối on-prem.
- D sai: chỉ ghi log.
</details>

**Câu 7.** Cần tìm nhanh thành phần nào chặn kết nối từ EC2 A tới RDS B mà chưa có traffic thật. Chọn gì?
- A. VPC Flow Logs
- B. Reachability Analyzer
- C. Traffic Mirroring
- D. CloudTrail

<details><summary>Đáp án</summary>

**B.** Phân tích cấu hình, chỉ ra SG/NACL/route gây chặn.
- A sai: cần traffic thật mới có log.
- C sai: sao chép gói tin, không chỉ ra cấu hình chặn.
- D sai: ghi API call.
</details>

**Câu 8.** (Chọn 2) Công ty có 80 VPC ở nhiều account và nhiều kết nối VPN, cần định tuyến tập trung, gộp băng thông VPN vượt 1.25 Gbps. Chọn gì?
- A. Transit Gateway share qua AWS RAM
- B. ECMP với nhiều VPN tunnel trên Transit Gateway
- C. Full mesh VPC peering
- D. Virtual private gateway với ECMP
- E. NAT Gateway mỗi VPC

<details><summary>Đáp án</summary>

**A, B.**
- C sai: mesh 80 VPC rất khó quản lý.
- D sai: VGW không hỗ trợ ECMP.
- E sai: không định tuyến giữa VPC.
</details>

---

### Nguồn đã đối chiếu
- NAT gateway basics (100 Gbps, 55,000 kết nối mỗi đích): https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-basics.html
- Regional NAT gateway: https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateways-regional.html
- NAT instances (NAT AMI hết bảo trì 31/12/2023): https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html
- Site-to-Site VPN quotas (1.25 Gbps / 5 Gbps mỗi tunnel): https://docs.aws.amazon.com/vpn/latest/s2svpn/vpn-limits.html
- Direct Connect dedicated connections: https://docs.aws.amazon.com/directconnect/latest/UserGuide/dedicated_connection.html
- Subnet CIDR (5 IP reserved): https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html
- VPC Flow Logs: https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html
- Reachability Analyzer: https://docs.aws.amazon.com/vpc/latest/reachability/what-is-reachability-analyzer.html
- VPC Block Public Access: https://docs.aws.amazon.com/vpc/latest/userguide/security-vpc-bpa.html
