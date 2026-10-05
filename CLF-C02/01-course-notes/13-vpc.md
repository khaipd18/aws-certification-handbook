# CLF-C02 — Amazon VPC

> Thuộc **Domain 3** (networking) và **Domain 2** (network security). Ở mức CLF chỉ cần hiểu khái niệm (thường 1–2 câu), không cần thiết kế VPC.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **VPC** = mạng riêng ảo để chạy tài nguyên, **theo Region**; **subnet** = phân vùng mạng trong VPC, **theo AZ**. Public subnet ra được Internet, private subnet thì không.
- **Internet Gateway** cho VPC ra Internet; **NAT Gateway** (AWS-managed) / NAT Instance (tự quản) cho **private subnet** ra Internet mà vẫn private.
- **NACL** = firewall cấp **subnet**, **stateless**, có **Allow và Deny**, rule chỉ theo IP. **Security Group** = firewall cấp **instance/ENI**, **stateful**, chỉ **Allow**, rule theo IP hoặc SG khác.
- **VPC Flow Logs** = log traffic IP đi vào/ra interface để troubleshoot kết nối.
- **VPC Peering** nối 2 VPC qua mạng AWS, **không chồng CIDR**, **không bắc cầu** (non-transitive). **Transit Gateway** = hub-and-spoke cho hàng nghìn VPC và on-prem.
- **VPC Endpoint** truy cập dịch vụ AWS qua mạng riêng: **Gateway** (chỉ S3, DynamoDB) và **Interface** (hầu hết dịch vụ). **PrivateLink** = expose dịch vụ cho hàng nghìn VPC khác.
- Kết nối on-prem: **Site-to-Site VPN** (mã hóa, qua Internet công cộng, nhanh dựng), **Direct Connect** (đường vật lý riêng, mất ít nhất 1 tháng), **Client VPN** (từ máy tính cá nhân qua OpenVPN).

---

## 2. Core Concepts

### IP addresses trong AWS
- **Là gì:** **Public IPv4** dùng trên Internet; EC2 nhận public IP **mới** mỗi lần stop/start (mặc định); **Private IPv4** dùng trong mạng riêng, **cố định** dù stop/start; **Elastic IP** = public IPv4 cố định gắn vào EC2.
- **Dùng khi nào:** **IPv6**: không gian địa chỉ khổng lồ, mọi IPv6 trên AWS đều public, miễn phí.
- **Vì sao quan trọng:** AWS **tính phí mọi public IPv4** (kể cả Elastic IP, đang dùng hay để không); slide ghi $0.005/giờ. `[+]` Free Tier được 750 giờ public IPv4 với EC2.

### VPC & Subnets
- **Là gì:** VPC là tài nguyên **Regional**; subnet là tài nguyên **AZ** chia nhỏ mạng của VPC.
- **Dùng khi nào:** public subnet cho tài nguyên cần truy cập từ Internet (load balancer); private subnet cho DB, app nội bộ.
- **Vì sao quan trọng:** **Route Table** định nghĩa đường đi ra Internet và giữa các subnet. AWS tạo sẵn **default VPC** cho bạn.

### Internet Gateway & NAT Gateway
- **Là gì:** **Internet Gateway (IGW)** ở cấp VPC, cho instance kết nối Internet; public subnet có route tới IGW.
- **Dùng khi nào:** **NAT Gateway** (AWS-managed) hoặc **NAT Instance** (tự quản) cho instance trong **private subnet** truy cập Internet (vd: tải bản vá) nhưng **Internet không truy cập ngược vào được**.
- **Vì sao quan trọng:** "private instance cần tải update từ Internet" → NAT Gateway.

### Network ACL (NACL)
- **Là gì:** firewall kiểm soát traffic vào/ra **subnet**; có rule **ALLOW và DENY**; rule chỉ theo **IP**.
- **Dùng khi nào:** chặn một dải IP cụ thể ở cấp subnet.
- **Vì sao quan trọng:** **stateless**: phải cho phép cả chiều vào lẫn chiều ra.

### Security Groups (nhắc lại)
- **Là gì:** firewall cho **EC2 instance** (hoặc ENI); chỉ có **ALLOW**; rule theo IP **và** security group khác.
- **Dùng khi nào:** kiểm soát truy cập tới từng instance.
- **Vì sao quan trọng:** **stateful**: traffic trả về tự động được phép.

### VPC Flow Logs
- **Là gì:** ghi thông tin traffic IP vào interface ở 3 cấp: VPC, Subnet, Elastic Network Interface.
- **Dùng khi nào:** giám sát và troubleshoot kết nối (subnet ↔ Internet, subnet ↔ subnet, Internet ↔ subnet); bắt cả interface của dịch vụ managed (ELB, ElastiCache, RDS, Aurora...).
- **Vì sao quan trọng:** đích lưu: **S3, CloudWatch Logs, Amazon Data Firehose**. GuardDuty cũng dùng Flow Logs làm nguồn.

### VPC Peering
- **Là gì:** nối **hai VPC** riêng tư qua mạng AWS, hoạt động như cùng một mạng.
- **Dùng khi nào:** hai VPC (có thể khác account/Region) cần nói chuyện với nhau.
- **Vì sao quan trọng:** CIDR **không được chồng lấn**; **không bắc cầu**: A–B và B–C không có nghĩa A–C, phải tạo peering riêng A–C.

### VPC Endpoints
- **Là gì:** truy cập dịch vụ AWS qua **mạng riêng** thay vì Internet công cộng: bảo mật hơn, latency thấp hơn.
- **Dùng khi nào:** **Gateway endpoint**: chỉ cho **S3 và DynamoDB**; **Interface endpoint** (ENI): hầu hết dịch vụ (kể cả S3, DynamoDB).
- **Vì sao quan trọng:** "private subnet truy cập S3 mà không qua Internet/NAT" → VPC Gateway Endpoint.

### AWS PrivateLink (VPC Endpoint Services)
- **Là gì:** cách **bảo mật và scalable nhất** để expose một dịch vụ cho **hàng nghìn VPC**.
- **Dùng khi nào:** không cần VPC peering, IGW, NAT hay route table; yêu cầu **Network Load Balancer** ở VPC dịch vụ và **ENI** ở VPC khách hàng.
- **Vì sao quan trọng:** dùng khi bên thứ ba cung cấp dịch vụ cho nhiều khách hàng mà không mở ra Internet.

### Site-to-Site VPN
- **Là gì:** nối VPN on-premises với AWS; kết nối **tự động mã hóa**, đi **qua Internet công cộng**.
- **Dùng khi nào:** cần kết nối nhanh (dựng trong vài phút/giờ), chi phí thấp. On-prem dùng **Customer Gateway (CGW)**, AWS dùng **Virtual Private Gateway (VGW)**.
- **Vì sao quan trọng:** băng thông và latency phụ thuộc Internet.

### AWS Direct Connect (DX)
- **Là gì:** **kết nối vật lý riêng** giữa on-prem và AWS; riêng tư, bảo mật, nhanh; đi qua mạng riêng.
- **Dùng khi nào:** cần băng thông ổn định, latency nhất quán, khối lượng dữ liệu lớn.
- **Vì sao quan trọng:** **mất ít nhất 1 tháng** để thiết lập.

### AWS Client VPN
- **Là gì:** kết nối từ **máy tính cá nhân** bằng **OpenVPN** vào mạng riêng trong AWS và on-prem.
- **Dùng khi nào:** nhân viên làm việc từ xa cần truy cập EC2 qua **private IP**.
- **Vì sao quan trọng:** đi qua Internet công cộng; khác Site-to-Site (nối cả mạng).

### AWS Transit Gateway
- **Là gì:** **transitive peering** giữa **hàng nghìn VPC và on-prem** theo mô hình **hub-and-spoke** (hình sao), qua một gateway duy nhất.
- **Dùng khi nào:** topology mạng phức tạp, quá nhiều VPC peering.
- **Vì sao quan trọng:** hoạt động với Direct Connect Gateway và VPN.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| VPC | Mạng riêng ảo, theo Region | "isolated network" |
| Subnet | Phân vùng mạng trong VPC, theo AZ | "public/private subnet" |
| Route Table | Quy định đường đi của traffic | "define routes" |
| Internet Gateway | Cho VPC ra Internet | "internet access for public subnet" |
| NAT Gateway | Cho private subnet ra Internet một chiều | "private instances download updates" |
| Elastic IP | Public IPv4 cố định, có phí | "fixed public IP" |
| NACL | Firewall subnet, stateless, Allow + Deny | "block an IP range at subnet level" |
| Security Group | Firewall instance, stateful, chỉ Allow | "instance-level firewall" |
| VPC Flow Logs | Log traffic IP | "troubleshoot connectivity", "capture IP traffic" |
| VPC Peering | Nối 2 VPC, non-transitive | "connect two VPCs privately" |
| Gateway Endpoint | Truy cập S3/DynamoDB qua mạng riêng | "S3 without internet" |
| Interface Endpoint | ENI truy cập dịch vụ AWS qua mạng riêng | "private access to AWS services" |
| AWS PrivateLink | Expose dịch vụ cho nhiều VPC qua NLB + ENI | "expose service to thousands of VPCs" |
| Site-to-Site VPN | Kết nối on-prem ↔ VPC mã hóa qua Internet | "quick, encrypted, over internet" |
| Customer Gateway / Virtual Private Gateway | Đầu VPN phía on-prem / phía AWS | "VPN endpoints" |
| Direct Connect | Đường vật lý riêng on-prem ↔ AWS | "dedicated private connection", "consistent performance" |
| Client VPN | Máy cá nhân ↔ VPC qua OpenVPN | "remote employees" |
| Transit Gateway | Hub kết nối hàng nghìn VPC + on-prem | "hub-and-spoke", "transitive" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Security Group | NACL | Instance, stateful, chỉ Allow vs subnet, stateless, Allow + Deny | "Deny specific IP" → NACL |
| Internet Gateway | NAT Gateway | Ra/vào Internet cho public subnet vs chỉ ra Internet cho private subnet | "Private subnet outbound only" → NAT |
| VPC Peering | Transit Gateway | 1–1, non-transitive vs hub nhiều VPC, transitive | "Thousands of VPCs" → Transit Gateway |
| VPC Peering | PrivateLink | Nối hai mạng vs expose **một dịch vụ** cho nhiều VPC | "Expose service, not whole network" → PrivateLink |
| Gateway Endpoint | Interface Endpoint | Chỉ S3/DynamoDB, qua route table vs hầu hết dịch vụ, qua ENI | |
| Site-to-Site VPN | Direct Connect | Qua Internet, mã hóa, nhanh dựng vs đường riêng, ổn định, ≥ 1 tháng | "Need connection this week" → VPN |
| Site-to-Site VPN | Client VPN | Nối **mạng** on-prem vs nối **một máy tính** | "Remote employee laptop" → Client VPN |
| Public IP | Elastic IP | Đổi khi stop/start vs cố định | |
| Private IPv4 | Public IPv4 | Không đổi khi stop/start, không tính phí vs có thể đổi, tính phí | |
| VPC | Subnet | Regional vs AZ-level | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "private instances need to download patches from internet" | NAT Gateway | |
| "allow VPC to access the internet" | Internet Gateway | |
| "block a malicious IP at subnet level" | Network ACL | |
| "capture network traffic information" | VPC Flow Logs | |
| "connect two VPCs privately" | VPC Peering | |
| "connect hundreds/thousands of VPCs and on-prem" | Transit Gateway | |
| "access S3 from private subnet without internet" | VPC Gateway Endpoint | |
| "expose service privately to many customer VPCs" | AWS PrivateLink | |
| "encrypted connection to on-prem quickly over internet" | Site-to-Site VPN | |
| "dedicated, private, consistent network connection" | AWS Direct Connect | |
| "employees connect from laptops to VPC" | AWS Client VPN | |
| "fixed public IPv4 for EC2" | Elastic IP | Có phí |

---

## 5. Câu hỏi tình huống

**Câu 1.** EC2 trong private subnet cần tải bản vá từ Internet nhưng không được nhận kết nối từ Internet. Dùng gì?
- A. Internet Gateway gắn trực tiếp vào instance
- B. NAT Gateway
- C. VPC Peering
- D. Elastic IP

<details><summary>Đáp án</summary>

**B.** NAT Gateway cho phép đi ra Internet một chiều.
- A sai: IGW gắn ở VPC; route trực tiếp tới IGW biến subnet thành public.
- C sai: nối hai VPC.
- D sai: biến instance thành có thể truy cập từ Internet.
</details>

**Câu 2.** (Chọn 2) Phát biểu nào về Network ACL là **đúng**?
- A. Hoạt động ở cấp subnet
- B. Chỉ có rule Allow
- C. Có cả rule Allow và Deny
- D. Là stateful
- E. Rule có thể tham chiếu tới security group

<details><summary>Đáp án</summary>

**A, C.**
- B, D, E sai: là đặc điểm của Security Group (chỉ Allow, stateful, tham chiếu SG).
</details>

**Câu 3.** Công ty cần kết nối data center tới AWS với băng thông ổn định, không đi qua Internet công cộng, chấp nhận vài tuần để triển khai. Chọn gì?
- A. Site-to-Site VPN
- B. AWS Direct Connect
- C. AWS Client VPN
- D. VPC Peering

<details><summary>Đáp án</summary>

**B.** Direct Connect là kết nối vật lý riêng, ổn định, mất ít nhất 1 tháng.
- A sai: đi qua Internet.
- C sai: cho máy cá nhân.
- D sai: nối VPC với VPC.
</details>

**Câu 4.** Công ty cần kết nối mã hóa tới VPC ngay trong tuần này, chi phí thấp. Chọn gì?
- A. AWS Direct Connect
- B. Site-to-Site VPN
- C. Transit Gateway riêng lẻ
- D. NAT Gateway

<details><summary>Đáp án</summary>

**B.** VPN dựng nhanh, mã hóa tự động, qua Internet.
- A sai: mất ít nhất 1 tháng.
- C sai: Transit Gateway là hub, vẫn cần VPN hoặc DX để nối on-prem.
- D sai: không nối on-prem.
</details>

**Câu 5.** Có 3 VPC A, B, C. A peering với B, B peering với C. A có nói chuyện được với C không?
- A. Có, vì peering bắc cầu
- B. Không, phải tạo peering A–C riêng (hoặc dùng Transit Gateway)
- C. Có, nếu cùng Region
- D. Có, nếu dùng NAT Gateway

<details><summary>Đáp án</summary>

**B.** VPC Peering không bắc cầu.
- A, C, D sai: peering không transitive bất kể Region hay NAT.
</details>

**Câu 6.** Ứng dụng trong private subnet cần đọc S3 mà không đi qua Internet hay NAT, chi phí thấp nhất. Chọn gì?
- A. VPC Gateway Endpoint cho S3
- B. Internet Gateway
- C. AWS Direct Connect
- D. Elastic IP

<details><summary>Đáp án</summary>

**A.** Gateway Endpoint cho S3/DynamoDB đi qua mạng riêng AWS.
- B sai: đi qua Internet.
- C sai: nối on-prem.
- D sai: public IP.
</details>

**Câu 7.** Nhà cung cấp SaaS muốn cung cấp dịch vụ cho hàng nghìn VPC của khách hàng một cách riêng tư, không peering, không qua Internet. Chọn gì?
- A. AWS PrivateLink
- B. VPC Peering
- C. Internet Gateway
- D. Site-to-Site VPN

<details><summary>Đáp án</summary>

**A.** PrivateLink expose dịch vụ qua NLB + ENI tới nhiều VPC.
- B sai: phải peering từng VPC, không scale.
- C sai: qua Internet.
- D sai: nối on-prem.
</details>

**Câu 8.** Team mạng cần ghi lại thông tin traffic IP đi vào ENI để điều tra vì sao một kết nối bị từ chối. Chọn gì?
- A. VPC Flow Logs
- B. AWS CloudTrail
- C. AWS X-Ray
- D. Amazon Inspector

<details><summary>Đáp án</summary>

**A.** Flow Logs ghi thông tin traffic IP ở cấp VPC/subnet/ENI.
- B sai: log API call.
- C sai: trace ứng dụng.
- D sai: quét lỗ hổng.
</details>

---

### Nguồn đã đối chiếu
- Amazon VPC pricing (tính phí mọi public IPv4, 750 giờ Free Tier): https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html
- Public IPv4 addresses: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html
- Elastic IP address pricing: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html
- Security groups vs network ACLs: https://docs.aws.amazon.com/vpc/latest/userguide/infrastructure-security.html
- VPC Flow Logs: https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html
- AWS PrivateLink: https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html
- AWS Transit Gateway: https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html
