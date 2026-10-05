# CLF-C02 — Kiến thức tổng hợp từ đề thi thử

> Gộp từ các đề **Practice Test #2, #3, #4, #5, #6** (mỗi đề 65 câu). Bản dán số 1 và số 3 trùng nhau (đều là Practice Test #3), **chưa có Practice Test #1**.
> File chỉ ghi **kiến thức** bóc ra từ đáp án đúng và sai, không chép lại câu hỏi. Khi thêm đề mới: chèn vào đúng mục, cập nhật bảng tra nhanh và 3 phụ lục, không tạo phần riêng cho từng đề.
> Ký hiệu: `⚠️` = đáp án/dịch vụ trong đề đã lỗi thời so với AWS 10/2026 · `❓` = đáp án đáng ngờ hoặc các đề mâu thuẫn nhau · `[Đx]` = Practice Test #x.
> Ánh xạ 4 domain CLF-C02: **D1 Cloud Concepts (24%) · D2 Security & Compliance (30%) · D3 Cloud Technology & Services (34%) · D4 Billing, Pricing & Support (12%)**.

---

## 0. Đọc trước: đáp án đề lệch so với AWS hiện hành

**Khi thi thật:** có đáp án theo tên/gói mới thì chọn cái mới; nếu chỉ có tên cũ thì chọn tên cũ đúng chức năng.

### 0.1 Dịch vụ đổi tên, đóng cửa hoặc sắp ngừng

| Đề dùng | Trạng thái 10/2026 | Thay bằng |
|---|---|---|
| Support plans Developer / Business / Enterprise On-Ramp `[Đ2][Đ4][Đ5][Đ6]` | ⚠️ Ngừng từ **01/01/2027** | **Basic, Business Support+, Enterprise, Unified Operations** |
| "Basic/Developer có 7 core Trusted Advisor checks" `[Đ2][Đ6]` | ⚠️ Mô tả cũ | Basic chỉ có check **Service Limits** + một số check **Security, Fault Tolerance** |
| Trusted Advisor "5 categories" `[Đ3][Đ4]` | ⚠️ Nay **6**: thêm **Operational Excellence** | |
| AWS Snowball / Snowball Edge / OpsHub `[Đ3][Đ5][Đ6]` | ⚠️ **Không nhận khách mới từ 07/11/2025**, không còn bán thiết bị Snow Family cho khách mới | **DataSync** (online), **AWS Data Transfer Terminal** (vận chuyển vật lý), **Outposts** (edge compute) |
| CloudEndure Disaster Recovery `[Đ6]` | ⚠️ Tên cũ | **AWS Elastic Disaster Recovery (DRS)** |
| AWS Fault Injection **Simulator** `[Đ5]` | ⚠️ Tên cũ | AWS Fault Injection **Service** |
| Amazon AppStream 2.0 `[Đ6]` | ⚠️ Docs đổi tên | **Amazon WorkSpaces Applications** (exam guide vẫn ghi AppStream 2.0) |
| Amazon QuickSight `[Đ6]` | ⚠️ Tên cũ | **Amazon Quick Sight** (thuộc Amazon Quick) |
| Amazon Kendra `[Đ3]` | ⚠️ Không nhận khách mới từ 30/07/2026 | Bedrock Knowledge Bases (managed) |
| Amazon Forecast `[Đ6]` | ⚠️ Không nhận khách mới từ 29/07/2024 | SageMaker Canvas (forecast) |
| Amazon Elastic Transcoder `[Đ6]` | ⚠️ **Ngừng hẳn từ 13/11/2025** (nguồn web) | **AWS Elemental MediaConvert** |
| AWS CodeStar `[Đ6]` (đáp án nhiễu) | ⚠️ Ngừng từ 31/07/2024 (nguồn web) | Amazon CodeCatalyst; tài nguyên CodeCommit/CodePipeline/CodeBuild vẫn chạy |
| Amazon CodeGuru `[Đ5]` | ⚠️ Reviewer **không tạo repository association mới từ 07/11/2025**; CodeGuru nằm ngoài phạm vi CLF-C02 | |
| Amazon Pinpoint `[Đ3]` (đáp án nhiễu) | ⚠️ Hết hỗ trợ **30/10/2026** | Amazon Connect (engagement), AWS End User Messaging (SMS, push) |
| Amazon Fraud Detector `[Đ3]` (đáp án nhiễu) | ⚠️ Không nhận khách mới từ 07/11/2025 | |
| AWS Security Hub `[Đ6]` | ⚠️ Bản cũ đổi tên **Security Hub CSPM**; Security Hub mới (GA 12/2025) là giải pháp hợp nhất | |
| Amazon Inspector "rules packages", "assessment templates" `[Đ3][Đ5]` | ⚠️ Mô tả của **Inspector Classic** | Inspector hiện tại: quét liên tục EC2 (agent SSM hoặc agentless), ECR, Lambda |
| CloudWatch Events `[Đ3][Đ4]` | ⚠️ Tên cũ | **Amazon EventBridge** |
| AWS Forums `[Đ5]` (đáp án nhiễu) | ⚠️ Đã thay | **AWS re:Post** |
| AWS Partner Solutions / Quick Starts `[Đ5][Đ6]` | ❓ Quick Starts nay nằm trong **AWS Solutions Library**; tên "Partner Solutions" chưa kiểm chứng được qua docs | |
| Organizations "master account" `[Đ4]` | ⚠️ Tên cũ | **management account** |
| Control Tower "guardrails" `[Đ6]` | ⚠️ Tên cũ | **controls** (preventive, detective, proactive) |
| KMS "CMK", "customer master key" `[Đ6]` | ⚠️ Tên cũ | **KMS key**: customer managed key, AWS managed key, AWS owned key |
| ElastiCache "Redis và Memcached" `[Đ6]` | ⚠️ | Nay hỗ trợ **Valkey, Redis OSS, Memcached** |
| "Amazon WAF" `[Đ3]` | Tên đúng là **AWS WAF** | |

### 0.2 Số liệu, tính năng đã thay đổi

| Đề nói | Hiện hành |
|---|---|
| MFA gồm Virtual, Hardware, **U2F security key**, **SMS** `[Đ3][Đ4]` | ⚠️ IAM nay có 3 loại: **Passkeys & security keys (FIDO)**, **Virtual authenticator app**, **Hardware TOTP token**. SMS không còn là loại MFA của IAM; "U2F" nay gọi là FIDO security key. Tối đa 8 thiết bị MFA/người dùng; **MFA bắt buộc cho root** |
| Lambda tính thời gian "làm tròn lên 100 ms" `[Đ3]` | ⚠️ Nay tính theo **1 ms** |
| Elastic IP "miễn phí khi gắn vào instance đang chạy" `[Đ6]` | ⚠️ Từ 02/2024 AWS **tính phí mọi public IPv4**, kể cả EIP đang dùng (Free Tier có 750 giờ/tháng cho EC2) |
| Savings Plans "chỉ có 2 loại" `[Đ6]` | ⚠️ Nay có **4**: Compute (tới 66%), EC2 Instance (tới 72%), SageMaker AI (tới 64%), Database (tới 35%) |
| RI giảm "tới 75%" `[Đ3][Đ4][Đ5]`; Convertible RI "tới 54%" `[Đ5]` | Docs hiện ghi RI/EC2 Instance SP **tới 72%**. Con số % thay đổi theo thời gian, đề CLF thường chỉ hỏi "RI rẻ hơn On-Demand, Spot rẻ nhất" |
| Cost Explorer "xem 12 tháng, dự báo 12 tháng" `[Đ4][Đ6]` | ⚠️ Dự báo **tới 18 tháng** (Monthly); lịch sử mặc định 13 tháng |
| Cost & Usage Report `[Đ3][Đ5]` | Vẫn đúng chức năng; AWS khuyến nghị **CUR 2.0 qua Data Exports** |
| CloudHSM "FIPS 140-2 Level 3" `[Đ2]` | ⚠️ HSM mới (hsm2m.medium) đạt **FIPS 140-3 Level 3** |
| Aurora "tới 128 TB" `[Đ6]` | Docs mới ghi tới **256 TiB** (giới hạn thực tế tùy engine version) |
| Shield Advanced "DDoS Response Team (DRT)" `[Đ3][Đ4]` | ⚠️ Nay gọi **Shield Response Team (SRT)** |

### 0.3 Các đề mâu thuẫn nhau hoặc đáp án đáng ngờ

| Chủ đề | Đề nói | Cách hiểu đúng / khi thi |
|---|---|---|
| Lambda và container | `[Đ2][Đ3]` "Lambda không chạy container" · `[Đ2]` (giải thích khác) Lambda deploy được **container image tới 10 GB** | ❓ Lambda **chạy được** container image theo Lambda Runtime API, nhưng **không điều phối/quản lý container**. Câu hỏi "chạy ứng dụng container" → chọn **ECS/EKS/Fargate** |
| Scheduled Reserved Instances | `[Đ4]` gợi ý dùng Scheduled RI cho job định kỳ · `[Đ5]` "AWS không còn hỗ trợ Scheduled RI" | ❓ Scheduled RI **không còn mua được**. Job định kỳ không bị gián đoạn → On-Demand/Savings Plans; chịu gián đoạn → Spot |
| Read Replica và DR | `[Đ3]` RR **không** tăng availability, **không** chống sự cố Region · `[Đ4]` cả RR lẫn Multi-AZ đều dùng cho DR | ❓ Hai câu khác ngữ cảnh: RR cùng Region = scale đọc; **cross-Region RR** dùng được cho DR khi mất Region; Multi-AZ = HA/failover trong Region. Câu "lợi ích **chính** của RR" → **scale đọc** |
| Tách hóa đơn vs tách chi phí | `[Đ3]` muốn **hóa đơn riêng** dev/prod → **tách AWS account** · `[Đ5]` muốn **phân bổ chi phí** theo phòng ban → **tags** | Không mâu thuẫn: "separate invoice" → account riêng; "cost allocation/tracking" → cost allocation tags |
| So sánh chi phí on-prem với AWS (TCO) | `[Đ4][Đ5]` → AWS Pricing Calculator | ❓ TCO Calculator cũ đã bị bỏ; Pricing Calculator ước tính chi phí **trên AWS**. Business case đầy đủ on-prem vs AWS → **Migration Evaluator**. Nếu không có Migration Evaluator trong đáp án thì chọn Pricing Calculator |
| Right-sizing | `[Đ6]` right-sizing ↔ **Elasticity** | ❓ Theo Well-Architected, right-sizing thuộc **Cost Optimization**/Sustainability. Đề này coi "chỉ dùng đúng lượng cần" là elasticity |
| "Measure overall efficiency" | `[Đ5]` (giải thích) gắn với Reliability | ❓ Theo WA, "Measure overall efficiency" là nguyên tắc của **Cost Optimization**. Giải thích đó cũng nói "dùng Inspector để automate security best practices", không chuẩn |
| Xác định dịch vụ phù hợp khi migrate | `[Đ2]` → APN **và Service Catalog** | ❓ Service Catalog là danh mục sản phẩm **đã được duyệt** để tự launch, không phải công cụ tư vấn chọn dịch vụ. APN (Consulting/Services partner), Professional Services hợp lý hơn |
| WorkSpaces là dịch vụ "global" | `[Đ6]` | ❓ Đề dựa trên trang Global Infrastructure. Trong thực tế bạn tạo WorkSpaces theo Region. Chỉ chọn khi đề hỏi đúng kiểu này |
| WAF là dịch vụ global | `[Đ3]` | WAF có **2 scope**: global (CloudFront) và regional (ALB, API Gateway, AppSync...) |
| Chia sẻ RI trong consolidated billing "chỉ cùng AZ" | `[Đ4]` | ❓ Đúng với **Zonal RI** (ví dụ trong docs). **Regional RI** áp dụng cho mọi AZ trong Region |

---

## Bảng tra nhanh (các cặp hay bị đem làm bẫy)

### EC2 purchasing options

| Lựa chọn | Đặc điểm | Chọn khi |
|---|---|---|
| On-Demand | Trả theo giây (Linux/Windows, sau phút đầu), không cam kết, **không bị gián đoạn** | Ngắn hạn, đột biến, **critical**, không đoán trước |
| Reserved Instances | Cam kết **1 hoặc 3 năm** (không có 6 tháng, 2 năm, 5 năm); có cho **EC2 và RDS** (cả ElastiCache, Redshift, DynamoDB reserved capacity) | Dùng **ổn định, dự đoán được** |
| Convertible RI | Đổi được family, OS, tenancy | Dùng lâu dài nhưng **workload sẽ thay đổi** |
| Savings Plans | Cam kết **$/giờ** 1–3 năm | Muốn linh hoạt hơn RI |
| Spot | Rẻ tới **90%**, có thể bị thu hồi | Batch, phân tích, container chịu gián đoạn, job linh hoạt thời gian |
| Dedicated Host | Server vật lý riêng, **đắt nhất** | **BYOL** theo socket/core, compliance |
| Capacity Reservation | Giữ capacity ở 1 AZ, không giảm giá | Bắt buộc có capacity ở AZ cụ thể |

### S3 storage classes

| Class | Điểm nhận diện | Phí retrieval / tối thiểu |
|---|---|---|
| Standard | Truy cập thường xuyên, **không** thời gian lưu tối thiểu | Không phí retrieval |
| Intelligent-Tiering | **Pattern truy cập không biết trước/ngẫu nhiên** | **Không phí retrieval**, có phí monitoring |
| Standard-IA | Ít truy cập, cần lấy nhanh, nhiều AZ | Có phí retrieval, tối thiểu 30 ngày |
| One Zone-IA | Ít truy cập, **dữ liệu tạo lại được** (thumbnail), **bản backup thứ cấp** của dữ liệu on-prem; **availability thấp nhất** (99.5%) | Có phí, 30 ngày |
| Glacier Instant Retrieval | Archive, lấy trong mili giây | 90 ngày |
| Glacier Flexible Retrieval | Archive, lấy trong phút–giờ, Bulk miễn phí; "data archival" chung chung | 90 ngày |
| Glacier Deep Archive | **Rẻ nhất**, giữ **7–10 năm** vì compliance, lấy 12–48 giờ | 180 ngày |

Mọi class: durability **11 số 9**. S3 Lifecycle = chuyển class/xóa object theo rule thời gian.

### Công cụ chi phí

| Công cụ | Dùng khi |
|---|---|
| AWS Pricing Calculator | **Ước tính trước** khi triển khai |
| AWS Cost Explorer | **Xem, phân tích, dự báo** chi phí; **rightsizing recommendations** EC2; **gợi ý Savings Plans** |
| Cost & Usage Report (CUR) | Dữ liệu **chi tiết nhất**, theo **giờ**, đẩy vào **S3** |
| AWS Budgets | Cảnh báo khi chi phí/usage **vượt hoặc dự báo vượt**; **RI utilization/coverage** thấp hơn ngưỡng. 4 loại: **Cost, Usage, Reservation, Savings Plans** |
| CloudWatch Billing Alarm | Cảnh báo khi **chi phí thực tế** vượt ngưỡng; metric ở **us-east-1** |
| Cost Anomaly Detection | ML phát hiện chi tiêu bất thường |
| Trusted Advisor | Phát hiện **EC2 dùng thấp**, **EBS không gắn/dùng ít**, S3 public, chưa bật CloudTrail, root chưa MFA... |
| Compute Optimizer | Rightsizing bằng ML: **EC2, Auto Scaling group, EBS, Lambda** (không có S3, EFS) |
| Cost allocation tags | Phân bổ chi phí theo phòng ban/dự án; mỗi key một value; **phải activate** cả tag AWS-generated lẫn user-defined |

### Giám sát và audit

| Dịch vụ | Trả lời câu hỏi |
|---|---|
| CloudWatch | **Hiệu năng**, metric, **resource utilization, application performance**, alarm, log (kể cả **on-prem** qua agent) |
| CloudTrail | **Ai** gọi API gì, khi nào (vd: ai sửa security group). Mặc định ghi **management events**; data events và Insights phải bật thêm |
| CloudTrail Insights | Phát hiện **API activity bất thường** so với baseline |
| AWS Config | **Lịch sử cấu hình** tài nguyên, thay đổi trong 2 tuần qua, compliance, thông báo khi cấu hình đổi |
| X-Ray | Trace request trong ứng dụng phân tán |
| Health Dashboard – Service health | Tình trạng **chung** mọi dịch vụ, có **RSS** |
| Health Dashboard – Your account | Sự kiện AWS **ảnh hưởng tới tài nguyên của bạn** + hướng dẫn khắc phục; có ở **mọi support plan** |
| S3 server access logs | Audit **request** gửi tới bucket |

### Dịch vụ bảo mật

| Dịch vụ | Làm gì | Không làm |
|---|---|---|
| AWS WAF | Lọc request **Layer 7**: SQL injection, XSS, **geo-match chặn quốc gia**, rate-based; chặn tất cả trừ request được phép | Không chống L3/L4; không gắn vào **Route 53** hay **EC2 trực tiếp** |
| Shield Standard | DDoS L3/L4, **miễn phí, tự bật**, AWS quản lý | |
| Shield Advanced | Trả phí cho **mọi** khách (kể cả Enterprise), SRT 24/7, **bảo vệ chi phí tăng do DDoS**; tài nguyên: **EC2, ELB, CloudFront, Route 53, Global Accelerator** | Không bảo vệ S3, Beanstalk, IAM |
| GuardDuty | **Threat detection** liên tục từ CloudTrail, VPC Flow Logs, DNS logs | Không chặn request, không quét lỗ hổng |
| Inspector | **Lỗ hổng phần mềm** và network reachability của EC2/ECR/Lambda | Không theo dõi thay đổi cấu hình |
| Macie | Phát hiện **dữ liệu nhạy cảm (PII)** ở quy mô lớn trong S3 | |
| Detective | Điều tra **root cause**; dùng **CloudTrail, VPC Flow Logs, GuardDuty findings** | |
| Security Hub | Gom finding nhiều dịch vụ, nhiều account | |
| KMS | Quản lý key mã hóa managed | |
| CloudHSM | **HSM phần cứng** riêng, bạn tự quản key, compliance yêu cầu phần cứng | |
| Secrets Manager | Lưu và **tự xoay vòng** secret | Không tạo key mã hóa, không phát hiện PII |
| Artifact | Tải báo cáo compliance (**PCI, SOC, ISO**), thỏa thuận | |

### Kết nối mạng

| Nhu cầu | Chọn |
|---|---|
| VPC truy cập **S3/DynamoDB** riêng tư | **VPC Gateway Endpoint** |
| VPC truy cập **SQS** (và hầu hết dịch vụ khác) riêng tư | **VPC Interface Endpoint** (PrivateLink) |
| Nối **2 VPC** | **VPC Peering** (không bắc cầu) |
| Nối **nhiều VPC + on-prem** | **Transit Gateway** (thường kèm Direct Connect/VPN) |
| On-prem ↔ AWS **riêng tư, băng thông cao, ổn định** | **Direct Connect** (≥ 1 tháng triển khai, không mã hóa sẵn) |
| On-prem ↔ AWS **mã hóa, nhanh dựng**, qua Internet | **Site-to-Site VPN**: **Customer Gateway** (phía công ty) + **Virtual Private Gateway** (phía AWS) |
| Máy cá nhân vào VPC | Client VPN |

### Security Group vs NACL

| | Security Group | Network ACL |
|---|---|---|
| Phạm vi | **Instance** (ENI) | **Subnet** |
| Rule | **Chỉ Allow** | **Allow và Deny** → chặn được IP cụ thể |
| Trạng thái | **Stateful** (traffic trả về tự cho phép) | **Stateless** |
| Đánh giá | Xét **tất cả** rule | Theo **số thứ tự tăng dần** |
| CIDR | Dùng được | Dùng được |

### Tăng tốc và phân phối toàn cầu

| Dịch vụ | Dùng khi | Bẫy |
|---|---|---|
| CloudFront | Website/nội dung **tĩnh** cho người dùng toàn cầu, cache ở edge | Không phải để tăng tốc upload |
| S3 Transfer Acceleration | **Upload/download S3** từ xa, từ nhiều nơi về **một bucket trung tâm** | Không tăng tốc website |
| Global Accelerator | **2 static anycast IP**, TCP/UDP, **non-HTTP** (game, IoT, VoIP), failover nhanh; target được ELB | **Không cache**, không host website; **dùng chung edge location** với CloudFront |
| Route 53 | DNS; **domain registration**, **health check & monitoring** | Không phải load balancer, không IP routing, không transfer acceleration |

Route 53 routing: **Simple** (một tài nguyên) · **Weighted** (tỉ lệ) · **Latency** (Region nhanh nhất) · **Failover** (**active-passive**).

### Chọn database

| Nhu cầu | Chọn |
|---|---|
| Quan hệ, chuyển từ MySQL/PostgreSQL, hiệu năng cao | **Aurora** |
| Quan hệ managed thông thường | RDS |
| NoSQL **schemaless**, key-value/document, **ít vận hành nhất ở mọi quy mô** | **DynamoDB** |
| Cache cho DynamoDB, đọc nhiều mùa cao điểm | **DAX** |
| Cache giảm tải DB đọc nhiều, in-memory | **ElastiCache** |
| **Knowledge graph**, quan hệ phức tạp | **Neptune** |
| Document, MongoDB | DocumentDB (Neptune và DocumentDB đều là **NoSQL**) |
| **OLAP**, data warehouse | **Redshift** |
| SQL trên dữ liệu **trong S3**, serverless | **Athena** |
| Hadoop/Spark cluster | **EMR** |
| **ETL**, chuẩn bị dữ liệu cho analytics, serverless | **Glue** |
| Streaming big data **real-time** | **Kinesis Data Streams** |
| Dashboard BI | **Quick Sight** |

### Support plans

| Tính năng | Gói |
|---|---|
| Account & billing 1-1, forum, service health checks, Health Dashboard, docs | **Basic** (mọi khách) |
| Hỗ trợ kỹ thuật 24x7 qua phone/chat **rẻ nhất** | Đề cũ: **Business** · Hiện tại: **Business Support+** |
| Full Trusted Advisor checks + **Support API** (tạo/đóng case bằng API) | Đề cũ: Business, Enterprise On-Ramp, Enterprise · Hiện tại: **Business Support+, Enterprise, Unified Operations** |
| **TAM** được chỉ định, Concierge (billing) | **Enterprise** (đề cũ: On-Ramp có "pool of TAMs") |
| Cloud Support **Engineers** vs **Associates** | Engineers từ Business trở lên; Associates ở Developer (cũ) |
| Infrastructure Event Management | Đề cũ: Enterprise (On-Ramp 1 lần/năm, Business mua thêm); ❓ hiện liệt kê trong **Unified Operations** |

### DR strategies

| Chiến lược | Chi phí / RTO | Chọn khi |
|---|---|---|
| Backup & Restore | Rẻ nhất, khôi phục chậm | Chấp nhận **mất dữ liệu vài giờ** |
| Pilot Light | Chỉ lõi (DB) chạy sẵn | Phải dựng thêm hạ tầng khi sự cố |
| Warm Standby | Bản đầy đủ ở **quy mô nhỏ**, phục vụ được ngay | App **24x7 nhưng không mission-critical**, chạy tạm ít instance |
| Multi-site active/active | Đắt nhất | Mission-critical |

---

## Domain 1 — Cloud Concepts

### Lợi ích và mô hình cloud
- **Six advantages**: trade fixed (CAPEX) for variable expense · **massive economies of scale** (gộp mức dùng của hàng trăm nghìn khách → giá pay-as-you-go thấp) · stop guessing capacity · increase speed & agility · stop spending on data centers · go global in minutes. Bẫy: "**trade OPEX for CAPEX**" là đảo ngược.
- **Agility** = đổi mới nhanh, phát triển/thử nghiệm/ra mắt nhanh. **Elasticity** = lấy tài nguyên khi cần, trả lại khi không cần. Triển khai nhanh hạ tầng cho sản phẩm mới → **speed & agility + automatic scaling**.
- **Reliability** = phục hồi sau gián đoạn; **Durability** = dữ liệu không mất/hỏng; **Resiliency** = phục hồi sau lỗi do tải, tấn công, hỏng hóc.
- **Multi-AZ** = **High Availability** (không phải scale out/up). Deploy 2 Region → tăng **availability**, không phải agility/security/scalability `[Đ4]`.
- **Vertical scaling** = thêm CPU/RAM một máy (đổi instance type, nâng DB). **Horizontal** = thêm máy: **ELB, Read Replicas, ASG**.
- **Deployment models**: on-prem + AWS → **Hybrid**; giữ dữ liệu nhạy cảm tại data center vì compliance nhưng vẫn dùng AWS → Hybrid. "Mixed deployment" không tồn tại.
- **IaaS** (EC2) · **PaaS** (Elastic Beanstalk: chỉ lo app và data) · **SaaS** (Gmail, Rekognition) · **FaaS** (Lambda). "Product as a Service" không tồn tại.
- So với hosting truyền thống, EC2: launch/terminate bất cứ lúc nào, scale theo nhu cầu; **không** phải gói cố định theo tháng, **không** chia sẻ tài nguyên giữa người dùng.
- Sau khi migrate, khách hàng vẫn trả **license phần mềm ứng dụng**; điện, phần cứng, bảo vệ data center là của AWS.
- **TCO estimate** nên tính: **server administration** (IT labor), **power/cooling**; không tính quảng cáo, số người dùng, thiết bị văn phòng.

### Cloud design principles
- **Loose coupling** (microservices, SQS/SNS), **automation**, **services not servers**, bảo mật ở mọi tầng. Bẫy: "close coupling", "servers not services", "security comes last", "monolithic", "synchronous".

### Well-Architected Framework (6 pillars)
- **Operational Excellence**: **IaC**, thay đổi **nhỏ, thường xuyên, đảo ngược được**, **anticipate failure**, học từ lỗi.
- **Security**: identity mạnh, **traceability**, bảo vệ dữ liệu at rest/in transit (vd: **KMS mã hóa**).
- **Reliability**: **tự phục hồi khi lỗi**, test recovery, scale ngang.
- **Performance Efficiency**: **chọn đúng loại và kích thước tài nguyên** theo workload, **serverless**, go global.
- **Cost Optimization**: consumption model, **measure overall efficiency**, phân tích chi tiêu.
- **Sustainability**.
- Bẫy: Availability, Scalability, Elasticity **không** phải pillar.
- **Well-Architected Tool** (miễn phí): trả lời câu hỏi về workload → so với best practice. Khác **Trusted Advisor** (check tự động trên account) và **Well-Architected Framework** (bộ nguyên tắc).

### Cloud Adoption Framework (CAF)
- 6 perspectives: **Business, People, Governance, Platform, Security, Operations**. Bẫy: Process, Product, Architecture **không** phải perspective (Process/Product là *transformation domains*).
- **Business**: **strategy management**, IT đầu tư gắn mục tiêu kinh doanh, đo giá trị.
- **Operations**: **performance and capacity management**, observability, incident.
- **Security**: vulnerability management. **Platform**: platform engineering. **Governance**: **application portfolio management**, rủi ro, tài chính cloud.
- 4 phases: **Envision** (cloud giúp tăng tốc kết quả kinh doanh thế nào) → **Align** (khoảng trống năng lực) → **Launch** (pilot) → **Scale**.

### Migration (7Rs)
- **Rehost** = lift and shift, **không đổi code/kiến trúc**, nhanh nhất. **Replatform** = tối ưu nhẹ (vd: DB sang RDS). **Refactor** = viết lại cloud-native. **Repurchase** = sang SaaS.
- Chuyển tài nguyên **giữa 2 AWS Region**: tự tạo tài nguyên ở Region mới rồi chuyển dữ liệu/app; không có dịch vụ "chuyển nguyên khối" (CloudFormation, DMS, AWS Support đều không làm hộ).

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (D1)
- "innovate faster, develop/test/launch quickly" → **Agility**
- "lower pay-as-you-go prices thanks to aggregated usage" → **Economies of scale**
- "acquire and release resources as needed" → **Elasticity**
- "Multi-AZ" → **High availability**; "add CPU/RAM" → **Vertical scaling**
- "IaC, small reversible changes" → **Operational Excellence**; "right resource type and size" → **Performance Efficiency**
- "align IT investment with business strategy" → **CAF Business**; "capacity management" → **CAF Operations**
- "no code changes, fastest migration" → **Rehost**
- "on-prem + AWS" → **Hybrid**; "only upload code, no infrastructure" → **PaaS**

---

## Domain 2 — Security & Compliance

### Shared Responsibility Model
- **AWS (OF the cloud)**: data center, phần cứng, **mạng toàn cầu**, **edge locations**, **AZ**, vá **hạ tầng mạng**, hủy ổ đĩa, đảm bảo nhân viên AWS không truy cập dữ liệu, compliance validation hạ tầng, **Shield Standard**, quản lý DynamoDB/S3 (dịch vụ abstracted: AWS lo hạ tầng, OS, platform).
- **Khách hàng (IN the cloud)**: dữ liệu, **mã hóa** (kể cả **mã hóa DB RDS**, SSE), **IAM user/role**, **bật MFA**, review quyền, **guest OS patch** của EC2, **security group/firewall & network config của EC2**, **WAF**, **Shield Advanced** (tự đăng ký).
- **Lambda**: khách hàng lo **code, dependency, version của function**; AWS lo OS, runtime, mạng, server.
- **RDS**: AWS vá OS và DB engine; khách hàng lo **mã hóa DB**, security group, user trong DB.
- **Shared controls**: **Configuration management** (AWS cấu hình thiết bị hạ tầng, khách cấu hình guest OS/DB/app), **Patch management**, **Awareness & training** (mỗi bên đào tạo nhân viên của mình). **OS configuration** cũng là chia sẻ: host OS → AWS, guest OS → khách.
- Management of **EC2** (IaaS) là của khách; S3, DynamoDB, Beanstalk thì AWS quản phần nền.

### IAM
- **Root**: credential = email + mật khẩu tạo account; **toàn quyền, không giới hạn được bằng IAM policy**; đổi được mật khẩu; bật MFA; **không tạo/xóa access key** của root; không chia sẻ; tạo IAM user admin cho việc quản trị.
- Best practices: **MFA cho mọi user**, **xoay vòng credential**, **least privilege**, không chia sẻ credential, **dùng IAM role thay vì chia sẻ credential giữa account**.
- Đăng nhập Console = **username + password + MFA**. **Access key** (Access Key ID + Secret) gắn với **IAM user** (hoặc root), dùng cho CLI/SDK. X.509 là chứng chỉ của ACM.
- Truy cập AWS bằng 3 cách: **Console, CLI, SDK**.
- EC2 truy cập DynamoDB/S3 → **IAM Role** (credential tạm), không dùng access key.
- **Policy**: phần tử bắt buộc **Effect + Action** (Resource bắt buộc tùy loại policy; Sid, Condition tùy chọn; Principal chỉ trong resource-based policy).
- **Credentials Report** = mọi user + trạng thái password, access key, MFA. **Access Advisor** = **quyền đã cấp** cho user và lần cuối dùng.
- **IAM là global**: nhân viên đi công tác vẫn dùng được, không cần tạo user mỗi Region.
- **STS** = credential **tạm thời**, giới hạn quyền cho user tin cậy/federated. **IAM Identity Center** = SSO nhiều account + app SAML (kế nhiệm AWS SSO). **Cognito** = sign-up/sign-in cho **web/mobile app**.
- MFA không cần thiết bị vật lý → **Virtual MFA app**. Cắm USB → **security key (FIDO)**.

### Mã hóa
- **Mã hóa mặc định, không cần cấu hình**: **S3** (SSE-S3), **S3 Glacier**, **Storage Gateway**, **CloudTrail log files**. **Không** mặc định: EBS, Redshift, EFS, CloudFront, ALB, CloudWatch Logs... (phải bật).
- KMS: **AWS managed key** (`aws/s3`...) = ít công nhất, chỉ dùng trong account của bạn; **customer managed key** = bạn kiểm soát, tốn phí; **AWS owned key** = AWS dùng cho nhiều account, bạn không xem được.
- **CloudHSM** = phần cứng HSM riêng, compliance bắt buộc thiết bị phần cứng, sinh/lưu/quản lý key chỉ bạn truy cập.

### Bảo vệ ứng dụng và mạng
- **WAF** deploy trên **CloudFront, ALB, API Gateway, AppSync** (không trực tiếp trên EC2, không Route 53); chạy ở mọi edge khi gắn CloudFront.
- Chống **DDoS**: **Shield**, **WAF** (rate-based), **CloudFront + Route 53** (edge). Inspector, CloudHSM, Trusted Advisor không chống DDoS.
- Auto Scaling **không** tự deploy Shield khi bị DDoS.
- **NACL** chặn được IP cụ thể; **Security Group** kiểm soát traffic vào EC2.
- **S3 Block Public Access** = cách hiệu quả nhất để giữ dữ liệu luôn private (đè lên policy/ACL).

### Phát hiện, audit, compliance
- **GuardDuty**: phát hiện **hoạt động độc hại & hành vi trái phép**; finding giữ 90 ngày.
- **Inspector**: tự động đánh giá lỗ hổng, network accessibility; **không** "track configuration changes" (đó là Config).
- **Config**: lịch sử cấu hình để **chứng minh compliance**, **thông báo khi cấu hình đổi**.
- **CloudTrail**: ai đổi security group; điều tra thay đổi.
- **Macie**: tìm và bảo vệ **dữ liệu nhạy cảm** ở quy mô lớn.
- **Detective**: phân tích từ **CloudTrail + VPC Flow Logs + GuardDuty findings**.
- **Artifact**: báo cáo **PCI**, SOC, ISO (không phải mua từ bên thứ ba qua Marketplace).
- **Penetration testing**: tự kiểm thử hạ tầng của mình **không cần xin phép** (một số dịch vụ); **network stress test** là khái niệm khác.
- **AWS Abuse team**: IP của AWS đang **port scan**, spam, DDoS... hệ thống của bạn (không phải AWS Support).
- **Acceptable Use Policy**: quy định cách dùng bị cấm. "Fair Use/Applicable Use Policy" không tồn tại.

### Multi-account governance
- **Organizations**: consolidated billing, **một phương thức thanh toán**, **gộp volume discount** EC2/S3, **chia sẻ RI**; tạo account bằng API; best practice: **account theo phòng ban**, **SCP** giới hạn quyền, dùng tags cho billing, bật CloudTrail mọi account.
- **SCP**: **không cấp quyền**, chỉ đặt giới hạn tối đa; **không ảnh hưởng management account**.
- **Control Tower**: dựng **landing zone** multi-account với blueprint và controls; vận hành hằng ngày vẫn là việc của khách.

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (D2)
- "who is responsible for edge locations / network infrastructure patching" → **AWS**
- "encrypt RDS / configure SG / IAM users / guest OS patch / Lambda code & versions" → **Customer**
- "both AWS and customer" → **Configuration management, Patch management, Awareness & training, OS configuration**
- "mandatory policy elements" → **Effect, Action**
- "list all users with password/key/MFA status" → **Credentials Report**; "review permissions granted" → **Access Advisor**
- "temporary credentials for trusted users" → **STS**; "SSO to many accounts" → **IAM Identity Center**; "app user sign-up" → **Cognito**
- "SQL injection / block countries" → **WAF**; "DDoS cost protection" → **Shield Advanced**
- "malicious activity" → **GuardDuty**; "vulnerabilities" → **Inspector**; "sensitive data at scale" → **Macie**; "configuration history/notify changes" → **Config**; "who changed" → **CloudTrail**
- "PCI/SOC report" → **Artifact**; "AWS IP port scanning me" → **AWS Abuse**
- "hardware device for encryption" → **CloudHSM**; "least effort encryption" → **AWS managed key**
- "keep S3 data private always" → **Block Public Access**; "audit S3 requests" → **S3 server access logs**

---

## Domain 3 — Cloud Technology & Services

### Hạ tầng toàn cầu
- **Region** = vị trí vật lý nơi AWS gom các data center; **AZ** = một hoặc nhiều data center **cùng một vị trí** với điện, mạng dự phòng; mọi AZ trong Region cách nhau ≤ 100 km; **mọi traffic giữa các AZ đều được mã hóa** `[Đ6]`.
- **VPC trải trên mọi AZ của một Region**; **subnet nằm trong 1 AZ**. Region là một phần của Global Infrastructure; VPC, subnet, VPN thì không.
- Chọn Region: **compliance/data residency**, **gần người dùng**, (dịch vụ có sẵn, giá). Không phải "5G", "AZ trong 100 km", "availability index".
- Global: **IAM, CloudFront, Route 53, WAF (khi gắn CloudFront)**. Regional: EC2, RDS, Lambda, Rekognition, **S3 bucket** (namespace global), EFS, Snowball.
- **Local Zones**: compute/storage gần người dùng ở thành phố lớn, latency thấp (game). **Wavelength**: AWS ở rìa **mạng 5G**, thiết bị di động. **Outposts**: hạ tầng và API AWS **tại data center của bạn**, latency thấp, xử lý dữ liệu tại chỗ (nhà máy).

### Compute
- **AMI**: thông tin cần để launch EC2; **phải ở cùng Region** với instance (copy AMI sang Region khác để dùng); Region của AMI không ảnh hưởng hiệu năng.
- **User data** = bootstrap script lúc launch; **instance metadata** = thông tin về instance (ami-id, IP...).
- Instance types: **Memory Optimized** (in-memory cache/DB), **Compute Optimized** (HPC, web hiệu năng cao), **Storage Optimized** (I/O tuần tự lớn), **Accelerated** (GPU, FPGA).
- **EC2 Instance Connect**: SSH qua **trình duyệt** từ Mac/Windows/Linux (cần port 22). **SSM Session Manager**: shell **không mở port, không public IP, không SSH key**. PuTTY chỉ Windows.
- **Lambda**: chạy code không quản server, trả theo **request + thời gian chạy** (không theo ngôn ngữ, số dòng code, kích thước package), event-driven (S3 upload, DynamoDB update), tiền xử lý dữ liệu trước khi lưu S3; **không** truy cập OS, **không** cài DB.
- Serverless compute: **Lambda, Fargate**. Không phải serverless: EC2, Beanstalk (tự provision server), Lightsail.
- **ECS** (EC2 launch type) = chạy container **vẫn truy cập server**; **Fargate** = container không quản server; **ECR** = lưu Docker image.
- **AWS Batch**: hàng trăm nghìn batch job, rẻ hơn tự chạy trên Fargate. Lambda có giới hạn thời gian.
- **Lightsail**: VPS giá thấp cố định, cho team mới, **dev/test**, **WordPress**.
- **Elastic Beanstalk**: PaaS; upload code, Beanstalk lo capacity, load balancing, **auto scaling**, health; **vẫn truy cập được tài nguyên/OS bên dưới**; hỗ trợ Java, .NET, PHP, Node.js, Python, Ruby, Go, **Docker**; chạy được cả **non-web app** (worker); **miễn phí**, chỉ trả tài nguyên. Health cơ bản: kiểm tra ASG có ít nhất 1 instance; basic health không đẩy metric riêng lên CloudWatch.
- **EC2 Auto Scaling**: scale out/in theo nhu cầu, **thay instance unhealthy** (fault tolerance), đăng ký instance vào ELB; **không** scale up/down (đó là vertical). **Predictive scaling** = tải lặp lại theo ngày/giờ; **target tracking** = giữ CPU 40%.
- **ELB**: **ALB** cho HTTP/HTTPS, **NLB** cho TCP/UDP/TLS hiệu năng cực cao; lợi ích **HA, fault tolerance**. "System Load Balancer" không tồn tại.

### Storage
- **S3**: object storage key-value, **cấu trúc phẳng**, host **website tĩnh** ít công nhất (lỗi truy cập → sửa **bucket policy**/Block Public Access); **Versioning** chống xóa nhầm; **Replication** (giữ metadata, copy sang account test; **CRR** cho yêu cầu lưu xa về địa lý); **Lifecycle** chuyển class; không cài DB lên S3.
- **EBS**: block storage, **khóa 1 AZ**, gắn 1 instance (mức CLF), tồn tại sau khi terminate; tính phí theo **volume type + provisioned IOPS + GB provisioned**; **snapshot lưu trong S3**, **incremental** (tính tiền phần thay đổi).
- **Instance Store**: block tạm thời, **cache/buffer/scratch**.
- **EFS**: NFS dùng chung, **truy cập song song** nhiều EC2, nhiều AZ (và qua VPC/Region), dùng được từ **on-prem** qua Direct Connect/VPN; **HA mặc định**; **EFS-IA** tính phí mỗi lần đọc/ghi, hợp **file cần giữ để audit**; không làm boot volume.
- HA mặc định: **EFS, DynamoDB** (không phải EBS, subnet, instance store).
- **FSx for Windows File Server**: SMB, Windows app, AD.
- **Storage Gateway**: 3 loại **File, Volume, Tape** Gateway. "Object/Block Gateway" không tồn tại.
- **DataSync**: truyền dữ liệu **online**, tự động, **incremental**, liên tục từ on-prem.
- **Snowball** ⚠️ (đề): khối lượng lớn, băng thông hạn chế, offline; nay khách mới dùng **DataSync / Data Transfer Terminal**.
- **AWS Backup**: backup tập trung; trả tiền cả **lượng backup lưu và lượng restore**.

### Database & analytics
- Xem bảng "Chọn database". **RDS Read Replica** = scale đọc (horizontal); **Multi-AZ một standby** = HA, **không** phục vụ đọc; Multi-AZ **hai readable standby** thì đọc được; cross-Region → DR.
- Aurora: chuyển MySQL sang dễ, nhanh hơn MySQL tới 5 lần.
- **Athena** = SQL trên S3 ít công nhất; **Quick Sight** = dashboard tương tác truy cập mọi thiết bị; **Glue** = chuẩn bị dữ liệu (ETL).

### Integration & networking
- **SQS** = decouple, gửi/lưu/nhận message, **pull**, không mất message khi bên nhận không sẵn sàng. **SNS** = pub/sub **push**, có thể mất nếu bên nhận không chạy. **Amazon MQ** = chuyển message broker on-prem (ActiveMQ/RabbitMQ) lên cloud dễ nhất. **Kinesis Data Streams** = streaming real-time (ad-tech, clickstream).
- **API Gateway**: REST, HTTP, WebSocket API; front door gọi Lambda; tích hợp trực tiếp Kinesis; **có caching**; request trả từ cache **vẫn tính là API call**.
- VPC thành phần: **subnet, Internet Gateway**, route table, endpoint. Storage Gateway, API Gateway, S3 object không thuộc VPC.

### Dev tools & IaC
- **CloudFormation**: IaC, **StackSets** triển khai giống nhau qua nhiều account/Region, ước tính chi phí từ template. **CDK**: IaC bằng Python, JavaScript... (sinh CloudFormation).
- **CodeCommit** (Git private, version control), **CodeBuild** (build/test), **CodeDeploy** (deploy tự động lên EC2 **và on-prem**), **CodePipeline** (điều phối, không tự deploy), **CodeArtifact** (package, Maven, npm...).
- **Systems Manager**: quản lý server AWS **và on-prem** (inventory, run command, patch), quản lý node tập trung.
- **EC2 Image Builder**: tự động tạo, cập nhật image cho EC2 và on-prem.
- **Resource Groups**: tổ chức và tự động hóa tác vụ trên nhiều tài nguyên theo tag.

### Monitoring
- Xem bảng "Giám sát và audit". Cảnh báo CPU > 80% qua email → **CloudWatch alarm + SNS**. Cảnh báo khi **root đăng nhập** → **EventBridge (CloudWatch Events) rule + SNS**. Chạy log backup mỗi thứ Hai 2h sáng, 5 phút, serverless → **EventBridge Scheduler + Lambda**.
- Gom log **EC2 và on-prem** → **CloudWatch Logs** (agent), không phải CloudTrail.

### Machine Learning & khác
- **Rekognition**: nhận diện vật thể, người, text trong ảnh; **không** resize, greyscale, pose detection.
- **Translate**: dịch phụ đề. **Transcribe**: speech-to-text. **Polly**: text-to-speech. **Lex**: chatbot với NLU. **Comprehend**: NLP. **SageMaker AI**: data scientist build/train/deploy model. **Personalize**: gợi ý cá nhân hóa (out-of-scope). **Kendra** ⚠️: tìm tài liệu nội bộ. **Forecast** ⚠️: dự báo time series (web traffic). "Amazon Customize" không tồn tại.
- **IoT Core**: kết nối thiết bị IoT không quản server ("IoT Gateway" không tồn tại). **Connect**: contact center.
- **WorkSpaces**: desktop ảo (DaaS). **AppStream 2.0** ⚠️: stream ứng dụng desktop qua trình duyệt.
- **Device Farm**: test app trên thiết bị thật (out-of-scope). **FIS**: chaos engineering. **AWS Architecture Center**: kiến trúc tham khảo. **Marketplace**: mua AMI/SaaS bên thứ ba, bán SaaS; không mua RI hay Direct Connect ở đây.
- **Elastic Transcoder** ⚠️: chuyển định dạng media trong S3 (nay MediaConvert).
- **Knowledge Center** (re:Post): câu hỏi phổ biến nhất và lời giải từ AWS.

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (D3)
- "launch same EC2 in another Region" → **copy AMI**; "bootstrap at launch" → **user data**
- "browser-based SSH" → **Instance Connect**; "no open ports, no public IP" → **Session Manager**
- "in-memory caching workload" → **Memory Optimized**
- "containers but access underlying servers" → **ECS (EC2 launch type)**; "no server management" → **Fargate**
- "read-heavy DB" → **ElastiCache**/**Read Replica**; "DynamoDB microseconds/peak reads" → **DAX**
- "schemaless/flexible schema/document" → **DynamoDB**; "knowledge graph" → **Neptune**
- "decouple without losing messages" → **SQS**; "existing message broker" → **Amazon MQ**; "real-time streaming" → **Kinesis Data Streams**
- "many VPCs + on-prem" → **Transit Gateway + Direct Connect**; "two VPCs" → **VPC Peering**
- "VPC to SQS privately" → **Interface endpoint**; "VPC to S3" → **Gateway endpoint**
- "global static website performance" → **CloudFront**; "uploads to central bucket" → **S3 Transfer Acceleration**; "non-HTTP, static IP" → **Global Accelerator**
- "ultra-low latency mobile 5G" → **Wavelength**; "low latency gaming end-users" → **Local Zones**; "AWS in my data center" → **Outposts**
- "ongoing online incremental transfer" → **DataSync**
- "deploy code to EC2 and on-prem" → **CodeDeploy**; "same infra many accounts/Regions" → **CloudFormation StackSets**; "IaC in Python" → **CDK**
- "deploy popular technology quickly per best practice" → **Partner Solutions / Solutions Library**

---

## Domain 4 — Billing, Pricing & Support

### Nguyên tắc giá
- 3 yếu tố chi phí chính: **compute, storage, outbound data transfer**. **Inbound miễn phí**; chỉ **outbound** bị tính.
- S3 miễn phí: **data transfer in từ Internet**, **ra EC2 cùng Region**, ra CloudFront. Storage S3/Glacier luôn tính phí.
- Dịch vụ **miễn phí** (chỉ trả tài nguyên bên dưới): **IAM, AWS Auto Scaling, Elastic Beanstalk, Organizations, CloudFormation**. Không miễn phí: Spot, detailed monitoring, Elastic IP ⚠️.
- **On-Demand Windows/Linux** tính **theo giây**.
- **RI**: 1 hoặc 3 năm; có cho **EC2 và RDS**; CloudFront, S3, IAM không có RI. **Convertible RI** khi workload sẽ tăng/đổi.
- Tối ưu chi phí EC2: **Auto Scaling** khớp nhu cầu + **RI** cho dùng ổn định; Spot cho job chịu gián đoạn. Vertical scaling hay nâng support plan không giảm chi phí.
- **Consolidated billing** ví dụ: A có 5 RI dùng 3, B dùng 6 → **5 instance tính giá RI, 4 tính On-Demand**; B hưởng RI của A nếu cùng AZ (với Zonal RI).

### Công cụ chi phí và tag
- Xem bảng "Công cụ chi phí". Ví dụ đề: tìm EC2 dùng thấp không cần cấu hình → **Trusted Advisor + Cost Explorer (rightsizing)**; báo cáo chi phí theo giờ vào S3 → **CUR**; cảnh báo RI utilization thấp → **Budgets**; chọn Savings Plan tối ưu → **Cost Explorer**; billing alarm → **CloudWatch**.
- Phân bổ chi phí theo phòng ban/dự án → **cost allocation tags** (không phải tạo account riêng, VPC riêng). Muốn **hóa đơn riêng** → **account riêng**.

### Trusted Advisor
- Nhóm: Cost optimization, Performance, Security, Fault tolerance, Service limits (+ **Operational Excellence**). "Elasticity, Documentation, Change Management" không phải nhóm.
- Security check: **S3 bucket public**, **chưa bật CloudTrail**, root chưa MFA, port mở. Không phát hiện việc chia sẻ credential, không yêu cầu tag S3.

### Support plans
- Xem bảng "Support plans" và mục 0. Basic: account & billing 1-1, **service health checks**, forum, docs. Developer (cũ): client-side diagnostic tools, email giờ hành chính.

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (D4)
- "which is free" → **inbound transfer**, IAM, Auto Scaling, Beanstalk
- "fundamental cost drivers" → **compute, storage, outbound data transfer**
- "short-term, spiky, critical" → **On-Demand**; "interruptible" → **Spot**; "predictable 1 year" → **RI**
- "RI terms" → **1 năm, 3 năm**
- "forecast spending" → **Cost Explorer**; "most comprehensive billing data" → **CUR**
- "alert when exceeding budget/forecast" → **Budgets**; "alert on actual charges" → **CloudWatch billing alarm**
- "separate invoices" → **separate accounts**; "cost by department" → **tags**
- "TAM" → **Enterprise**; "cheapest 24x7 phone" → **Business (cũ) / Business Support+**

---

## Danh mục dịch vụ (use case một dòng)

| Dịch vụ | Use case |
|---|---|
| EC2 | Máy ảo, truy cập OS, giá theo giây |
| EC2 Auto Scaling / AWS Auto Scaling | Tự scale theo nhu cầu, thay instance hỏng |
| Elastic Load Balancing (ALB/NLB) | Phân tải HTTP(S) / TCP-UDP, HA |
| Lambda | Code event-driven, serverless |
| ECS / EKS / Fargate / ECR | Chạy container (có/không quản server) / Kubernetes / lưu image |
| AWS Batch | Batch job quy mô lớn |
| Lightsail | VPS giá cố định, WordPress, dev/test |
| Elastic Beanstalk | PaaS deploy web/worker app |
| Outposts / Local Zones / Wavelength | AWS tại on-prem / gần thành phố / rìa 5G |
| S3 / S3 Glacier | Object storage, website tĩnh / archive |
| EBS / Instance Store / EFS / FSx | Block bền vững / block tạm / NFS dùng chung / file system bên thứ ba |
| Storage Gateway | Hybrid storage (File, Volume, Tape) |
| DataSync | Truyền dữ liệu online tự động |
| Snowball ⚠️ | Truyền dữ liệu offline (không nhận khách mới) |
| AWS Backup / Elastic Disaster Recovery | Backup tập trung / DR replicate liên tục |
| RDS / Aurora | DB quan hệ managed / tương thích MySQL-PostgreSQL hiệu năng cao |
| DynamoDB / DAX | NoSQL serverless / cache micro giây |
| ElastiCache | Cache in-memory |
| Neptune / DocumentDB | Graph / document (MongoDB) |
| Redshift / Athena / EMR / Glue / Kinesis / Quick Sight | Warehouse / SQL trên S3 / Hadoop / ETL / streaming / BI |
| VPC, Subnet, IGW, NAT, SG, NACL | Mạng riêng và firewall |
| VPC Endpoint / PrivateLink / Peering / Transit Gateway | Kết nối riêng tới dịch vụ / giữa VPC |
| Direct Connect / Site-to-Site VPN | Kết nối on-prem riêng / mã hóa qua Internet |
| CloudFront / Global Accelerator / Route 53 | CDN / anycast IP / DNS, domain, health check |
| SQS / SNS / Amazon MQ / EventBridge / Step Functions | Queue / pub-sub / broker / event & schedule / workflow |
| API Gateway | Tạo và quản lý API |
| CloudWatch / CloudTrail / Config / X-Ray / Health Dashboard | Metric-log / API audit / cấu hình / trace / tình trạng dịch vụ |
| Systems Manager | Quản lý node AWS + on-prem |
| CloudFormation / CDK | IaC template / IaC bằng ngôn ngữ lập trình |
| CodeCommit / CodeBuild / CodeDeploy / CodePipeline / CodeArtifact | Git / build / deploy / CI-CD / package |
| EC2 Image Builder | Tự động tạo image |
| IAM / STS / IAM Identity Center / Cognito / Directory Service | Quyền / credential tạm / SSO / user của app / AD |
| KMS / CloudHSM / ACM / Secrets Manager | Key / HSM / chứng chỉ TLS / secret |
| WAF / Shield / Firewall Manager | Lọc L7 / DDoS / quản lý rule nhiều account |
| GuardDuty / Inspector / Macie / Detective / Security Hub | Threat / lỗ hổng / PII / root cause / tổng hợp |
| Artifact | Báo cáo compliance |
| Organizations / Control Tower / Service Catalog / RAM | Multi-account / landing zone / sản phẩm được duyệt / chia sẻ tài nguyên |
| Pricing Calculator / Cost Explorer / CUR / Budgets / Cost Anomaly Detection / Compute Optimizer / Trusted Advisor | Công cụ chi phí và best practice |
| Rekognition / Transcribe / Polly / Translate / Lex / Comprehend / SageMaker AI / Personalize | AI/ML |
| Kendra ⚠️ / Forecast ⚠️ | Tìm tài liệu / dự báo time series |
| IoT Core / Connect | IoT / contact center |
| WorkSpaces / AppStream 2.0 ⚠️ | Desktop ảo / stream ứng dụng |
| Device Farm / FIS / Well-Architected Tool | Test thiết bị thật / chaos engineering / review kiến trúc |
| Marketplace / APN / Architecture Center / re:Post / Knowledge Center | Phần mềm bên thứ ba / đối tác / kiến trúc mẫu / cộng đồng / FAQ |
| Elastic Transcoder ⚠️ | Chuyển định dạng media (nay MediaConvert) |

---

## Phụ lục A — Bẫy phải thuộc lòng

1. **Inbound data transfer miễn phí**; tính phí outbound.
2. Auto Scaling **scale out/in**, không scale up/down.
3. **Multi-AZ = HA**; **Read Replica = scale đọc** (cross-Region RR mới hỗ trợ DR Region).
4. **Security Group stateful, chỉ Allow, instance**; **NACL stateless, Allow+Deny, subnet, rule theo số thứ tự**.
5. **Gateway endpoint chỉ cho S3 và DynamoDB**; dịch vụ khác (SQS...) dùng interface endpoint.
6. **VPC Peering không bắc cầu**; nhiều VPC → Transit Gateway.
7. **Site-to-Site VPN = Customer Gateway + Virtual Private Gateway**.
8. **AMI theo Region**; copy để dùng Region khác.
9. **EBS khóa 1 AZ**; snapshot lưu trong **S3**, incremental.
10. **EFS và DynamoDB HA mặc định**; EBS, subnet, instance store thì không.
11. **One Zone-IA availability thấp nhất**; **Standard và Intelligent-Tiering không phí retrieval**; **Standard không có thời gian lưu tối thiểu**.
12. **Deep Archive** = rẻ nhất cho 7–10 năm; Intelligent-Tiering = pattern ngẫu nhiên.
13. **Mã hóa mặc định**: S3, Glacier, Storage Gateway, CloudTrail logs.
14. **Policy bắt buộc: Effect + Action**.
15. **Root không giới hạn được bằng IAM policy**; không tạo access key root.
16. **Access key gắn IAM user**, không gắn role/group/policy.
17. **IAM global**; S3 bucket regional; WAF có 2 scope.
18. **Shield Advanced trả phí cho mọi khách**, có bảo vệ chi phí; Shield Standard thuộc trách nhiệm AWS.
19. **WAF không gắn EC2 hay Route 53** (gắn CloudFront, ALB, API Gateway, AppSync).
20. **Lambda tính theo request + thời gian chạy**.
21. **SCP không cấp quyền**, không ảnh hưởng management account.
22. **Cost allocation tags phải activate** (cả AWS-generated); mỗi key một value.
23. **Budgets 4 loại**: Cost, Usage, Reservation, Savings Plans.
24. **RI chỉ 1 hoặc 3 năm**; có cho EC2 và RDS.
25. **CloudWatch billing metric ở us-east-1**, chỉ chi phí thực tế.
26. **CloudTrail mặc định ghi management events**.
27. **Health Dashboard Your account có ở mọi support plan**; TAM chỉ Enterprise trở lên.
28. **Configuration management, patch management, training** là shared controls.
29. **Lambda customer lo code, dependency, version**.
30. **Beanstalk vẫn truy cập được OS**; Lambda thì không.

## Phụ lục B — Dịch vụ / tính năng không tồn tại hoặc bị dùng sai mục đích

| Đáp án nhiễu trong đề | Vì sao loại |
|---|---|
| Soft Token MFA, AWS Mobile Farm, AWS IoT Gateway, Amazon Customize, System Load Balancer, Amazon S3 Random Access, Amazon DynamoDB Regulator, CloudCover Disaster Recovery, AWS Corporate Support, Adaptable RI, Product as a Service, Mixed deployment | **Không tồn tại** |
| AWS Fair Use Policy, AWS Applicable Use Policy | Không tồn tại; đúng là **Acceptable Use Policy** |
| Object Gateway, Block Gateway | Storage Gateway chỉ có **File, Volume, Tape** |
| Resource / Software / Hardware budget | Budgets chỉ có Cost, Usage, Reservation, Savings Plans |
| Trusted Advisor nhóm Elasticity, Documentation, Change Management | Không phải category |
| Availability, Scalability, Elasticity là pillar WA | Không phải pillar |
| Process, Product, Architecture là CAF perspective | Không phải perspective |
| Storage Savings Plans, RI Savings Plans | Không tồn tại |
| Reserved Instances 6 tháng / 2 năm / 5 năm | Chỉ 1 hoặc 3 năm |
| Scheduled Reserved Instances | Không còn mua được |
| Lambda cài DB/container service, lưu biến môi trường nhạy cảm, truy cập OS | Lambda serverless, không phải nơi cài phần mềm hay lưu trữ |
| S3 là block/file storage, cài DB lên S3 | S3 là object storage |
| Global Accelerator host website tĩnh, dùng edge khác CloudFront | Sai |
| WAF chống L3/L4, có DRT | Đó là Shield/Shield Advanced |
| Inspector theo dõi thay đổi cấu hình | Đó là Config |
| Route 53 cân bằng tải, IP routing, transfer acceleration | Sai dịch vụ |
| CloudFormation di chuyển dữ liệu giữa Region / tăng hiệu năng website | Sai mục đích |
| Organizations / Cost Explorer / tags **tạo hóa đơn riêng** | Hóa đơn riêng chỉ có khi tách account |
| Auto Scaling tự triển khai Shield | Sai |
| Marketplace mua RI, Direct Connect, tài liệu compliance | Sai; compliance ở Artifact |
| Rekognition resize/greyscale ảnh, pose detection | Không có |
| VPC Endpoint nối 2 VPC; Direct Connect/Site-to-Site VPN nối VPC với VPC | Sai |
| Spot cho DB/job critical | Spot có thể bị thu hồi |
| Billing alarm bằng CloudTrail/Organizations/Cost Explorer | Billing alarm là CloudWatch |
| CloudTrail gom log server on-prem | Đó là CloudWatch Logs |

## Phụ lục C — Chủ đề lặp lại nhiều lần

| Chủ đề | Số đề xuất hiện (ước lượng, trên 5 đề) | Ghi nhớ |
|---|---|---|
| Shared Responsibility (EC2, RDS, Lambda, shared controls) | 5/5 | Bảng D2 |
| EC2 purchasing options, RI, Savings Plans | 5/5 | Bảng tra nhanh |
| S3 storage classes, versioning, replication, lifecycle | 5/5 | Bảng tra nhanh |
| Công cụ chi phí (Explorer, CUR, Budgets, Calculator, billing alarm) | 5/5 | Bảng tra nhanh |
| Support plans, TAM, Trusted Advisor | 5/5 | ⚠️ gói đã đổi |
| CloudWatch vs CloudTrail vs Config | 5/5 | Bảng giám sát |
| IAM best practices, root, MFA, Credentials Report/Access Advisor | 5/5 | D2 |
| Security services (GuardDuty, Inspector, Macie, Detective, WAF, Shield) | 5/5 | Bảng bảo mật |
| Kết nối mạng (Endpoint, Peering, TGW, DX, VPN), SG vs NACL | 5/5 | Bảng mạng |
| Well-Architected pillars, CAF perspectives/phases | 5/5 | D1 |
| Global Infra (Region/AZ, Local Zones, Wavelength, Outposts) | 4/5 | D3 |
| Database selection (DynamoDB, Aurora, Neptune, ElastiCache, Redshift) | 5/5 | Bảng DB |
| CloudFront vs S3TA vs Global Accelerator, Route 53 routing | 4/5 | Bảng toàn cầu |
| Containers & serverless (ECS, Fargate, Lambda, Beanstalk) | 5/5 | D3 |
| DR strategies, migration (7Rs, Snowball/DataSync) | 3/5 | Bảng DR, D1 |
| Encryption (KMS key types, CloudHSM, mặc định) | 4/5 | D2 |

---

### Nguồn đã đối chiếu
- AWS Support Plans (gói hiện hành, Developer/Business/On-Ramp ngừng 01/01/2027): https://docs.aws.amazon.com/awssupport/latest/user/aws-support-plans.html
- Trusted Advisor (Basic: Service Limits + một số Security/Fault Tolerance): https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html
- Snowball Edge availability change (không nhận khách mới): https://docs.aws.amazon.com/snowball/latest/developer-guide/snowball-edge-availability-change.html
- IAM MFA (3 loại MFA, MFA bắt buộc cho root): https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html
- Savings Plans types (4 loại): https://docs.aws.amazon.com/savingsplans/latest/userguide/plan-types.html
- Public IPv4 pricing (tính phí mọi public IPv4, kể cả EIP): https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html
- Lambda millisecond metering: https://docs.aws.amazon.com/whitepapers/latest/security-overview-aws-lambda/benefits-of-lambda.html
- Cost Explorer forecast (+18M): https://docs.aws.amazon.com/cost-management/latest/userguide/ce-modify.html
- CloudWatch billing alarm (us-east-1): https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html
- CodeGuru Reviewer availability change: https://docs.aws.amazon.com/codeguru/latest/reviewer-ug/codeguru-reviewer-availability-change.html
- Security Hub (bản hợp nhất mới): https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub-v2.html
- Amazon Forecast availability: https://docs.aws.amazon.com/managedservices/latest/onboardingguide/forecast.html
- Kendra availability change: https://docs.aws.amazon.com/kendra/latest/dg/kendra-availability-change.html
- Pinpoint end of support: https://docs.aws.amazon.com/pinpoint/latest/userguide/migrate.html
- Infrastructure Composer end of support (standalone console tắt 07/12/2026): https://docs.aws.amazon.com/infrastructure-composer/latest/dg/infrastructure-composer-end-of-support.html
- Elastic Transcoder end of support (nguồn web): https://aws.amazon.com/blogs/media/support-for-amazon-elastic-transcoder-ending-soon/
- CodeStar discontinuation (nguồn web): https://repost.aws/questions/QUzfvPNaF6T3CVMRta-qcziA/code-star-vs-code-catalyst
- CLF-C02 in-scope / out-of-scope services: https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-in-scope-services.html · https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
