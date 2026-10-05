# CLF-C02 — Account Management, Billing & Support

> Gần như toàn bộ **Domain 4: Billing, Pricing, and Support (12%)** cộng phần multi-account governance của **Domain 2**. Chương có nhiều thay đổi nhất so với slide (Support plans, Free Tier, công cụ billing).
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **AWS Organizations** = quản lý nhiều account: **consolidated billing** (một hóa đơn, gộp mức dùng để được giảm giá theo volume, chia sẻ RI/Savings Plans), **SCP** giới hạn quyền tối đa của account/OU.
- **Control Tower** dựng môi trường multi-account chuẩn best practice trong vài click, chạy trên Organizations; **RAM** chia sẻ tài nguyên giữa account; **Service Catalog** cho người dùng tự launch sản phẩm đã được admin duyệt.
- 4 mô hình giá: **Pay as you go, Save when you reserve, Pay less by using more, Pay less as AWS grows**.
- **Savings Plans**: EC2 Instance SP (tới 72%), Compute SP (tới 66%, áp cả Fargate, Lambda), SageMaker AI SP; `[+]` Database Savings Plans.
- Công cụ chi phí: **Pricing Calculator** (ước tính trước), **Billing Dashboard**, **Cost Allocation Tags**, **Cost and Usage Reports** (dữ liệu chi tiết nhất), **Cost Explorer** (xem, phân tích, dự báo), **Budgets** (cảnh báo theo ngân sách), **Cost Anomaly Detection** (ML), **Compute Optimizer** (rightsizing), **Service Quotas**.
- **Trusted Advisor** đánh giá account theo 6 nhóm: Cost optimization, Performance, Security, Fault tolerance, Service limits, Operational excellence.
- **Support plans hiện tại:** **Basic** (miễn phí), **Business Support+**, **Enterprise**, **Unified Operations**. ⚠️ Developer, Business, Enterprise On-Ramp ngừng từ **01/01/2027**.

---

## 2. Core Concepts

### AWS Organizations
- **Là gì:** dịch vụ **global** quản lý nhiều AWS account; account chính là **management account** (⚠️ slide gọi "master account" là tên cũ).
- **Dùng khi nào:** **consolidated billing** với một phương thức thanh toán; giảm giá nhờ gộp mức dùng (EC2, S3...); gộp **Reserved Instances** để tiết kiệm tối đa; có API tự động tạo account.
- **Vì sao quan trọng:** giới hạn quyền account bằng **Service Control Policies (SCP)**.

### Multi-account strategy & OU
- **Là gì:** tạo account theo phòng ban, cost center, dev/test/prod, yêu cầu pháp lý (dùng SCP), để cách ly tài nguyên (VPC), có service limit riêng, account riêng cho logging.
- **Dùng khi nào:** gom account vào **Organizational Unit (OU)** theo Business Unit, theo môi trường (Prod/Dev), hoặc theo dự án.
- **Vì sao quan trọng:** best practice: tag chuẩn cho billing; bật CloudTrail mọi account, gửi log về S3 tập trung; gửi CloudWatch Logs về account logging chung.

### Service Control Policies (SCP)
- **Là gì:** whitelist hoặc blacklist IAM action, áp ở cấp **OU hoặc account**; **không áp lên management account**.
- **Dùng khi nào:** chặn dịch vụ (vd: cấm dùng EMR), thực thi PCI compliance bằng cách tắt dịch vụ.
- **Vì sao quan trọng:** SCP áp lên **mọi user và role của account, kể cả root**; không ảnh hưởng **service-linked role**; SCP **không cấp quyền**, phải có `Allow` tường minh mới cho phép. `[+]` Nay Organizations còn có **Resource Control Policies (RCP)** giới hạn quyền tối đa trên **tài nguyên**.

### Consolidated Billing
- **Là gì:** **Combined usage**: gộp mức dùng của mọi account để chia sẻ volume pricing, giảm giá RI và Savings Plans; **One bill**: một hóa đơn cho cả Organization.
- **Dùng khi nào:** account A có 5 RI không dùng hết thì account B được hưởng giảm giá.
- **Vì sao quan trọng:** management account **tắt được** việc chia sẻ RI discount cho bất kỳ account nào, kể cả chính nó.

### AWS Control Tower
- **Là gì:** cách dễ nhất để dựng và quản trị môi trường **multi-account** an toàn, tuân thủ theo best practice; chạy trên AWS Organizations, tự thiết lập Organizations và SCP.
- **Dùng khi nào:** tự động dựng môi trường vài click, phát hiện vi phạm chính sách và khắc phục, theo dõi compliance qua dashboard.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: thuật ngữ "guardrails" → nay gọi là **controls** (preventive, detective, proactive), triển khai bằng SCP, RCP, declarative policy, Config rule hoặc CloudFormation hook.

### AWS Resource Access Manager (RAM)
- **Là gì:** chia sẻ tài nguyên bạn sở hữu với account khác hoặc trong Organization.
- **Dùng khi nào:** tránh tạo trùng tài nguyên: Aurora, **VPC subnet**, Transit Gateway, Route 53, EC2 Dedicated Hosts, License Manager configuration...
- **Vì sao quan trọng:** "share VPC subnets across accounts" → RAM.

### AWS Service Catalog
- **Là gì:** cổng **self-service** để người dùng launch bộ **sản phẩm đã được admin duyệt** (VM, DB, storage...).
- **Dùng khi nào:** admin tạo **product** (CloudFormation template) → gom vào **portfolio** → cấp IAM permission; user launch được sản phẩm đã cấu hình đúng, gắn tag đúng.
- **Vì sao quan trọng:** giúp người mới không tạo stack sai chuẩn tổ chức.

### Pricing models
- **Là gì:** **Pay as you go** (linh hoạt, đáp ứng scale); **Save when you reserve** (EC2 RI, DynamoDB Reserved Capacity, ElastiCache Reserved Nodes, RDS RI, Redshift Reserved Nodes); **Pay less by using more** (giảm giá theo volume); **Pay less as AWS grows**.
- **Dùng khi nào:** đề cho tình huống và hỏi mô hình giá nào.
- **Vì sao quan trọng:** "S3 càng dùng nhiều đơn giá càng rẻ" → Pay less by using more.

### Free Plan & Free Tier
- **Là gì:** account mới nhận **$100 credit**, kiếm thêm **tới $100** qua các hoạt động; chọn **Free plan** hoặc **Paid plan**; cả hai đều có **Always Free** (vd: Lambda 1 triệu request + 400.000 GB-giây/tháng, DynamoDB 25 GB).
- **Dùng khi nào:** **Free plan** không phát sinh phí, hết sau **6 tháng hoặc khi hết credit**; **Paid plan** tính phí khi dùng hết credit, truy cập mọi dịch vụ.
- **Vì sao quan trọng:** Free plan hết hạn thì account **tự đóng** (AWS giữ dữ liệu 90 ngày); Free plan tự nâng lên Paid nếu tham gia Organizations, dựng Control Tower...; Free plan không mua được Savings Plans, RI.

### Compute pricing
- **Là gì:** **EC2**: trả theo số instance, cấu hình (Region, OS, type, size), ELB và data processed, detailed monitoring; On-Demand tối thiểu 60 giây, theo giây (Linux/Windows) hoặc giờ.
- **Dùng khi nào:** RI giảm tới 72% (slide pricing ghi 75%, chương EC2 ghi 72%; docs Savings Plans ghi tới 72%), Spot tới 90%, Dedicated Host On-Demand hoặc Reserved; **Lambda**: theo request + duration; **ECS EC2 launch type**: không phí thêm, trả tài nguyên EC2; **Fargate**: trả theo vCPU + memory cấp cho container.
- **Vì sao quan trọng:** Savings Plans là lựa chọn thay thế RI cho dùng ổn định.

### Storage, Database, CloudFront pricing
- **Là gì:** **S3**: storage class, số và kích thước object, số và loại request, **data transfer OUT khỏi Region**, Transfer Acceleration, lifecycle transition; **EBS**: loại volume, GB provisioned/tháng, IOPS (gp: đã gồm; io: theo IOPS provisioned), snapshot, data transfer out.
- **Dùng khi nào:** **RDS**: tính theo giờ, engine/size/memory class, On-Demand hoặc RI, backup miễn phí tới 100% dung lượng DB, storage thêm, I/O, Single/Multi-AZ, data transfer out; **CloudFront**: giá khác nhau theo vùng địa lý, data transfer out (volume discount), số request HTTP/HTTPS.
- **Vì sao quan trọng:** **inbound data transfer miễn phí** ở mọi dịch vụ.

### Networking costs
- **Là gì:** traffic **vào** miễn phí; traffic giữa 2 AZ dùng **private IP** khoảng $0.01/GB; dùng **public IP/Elastic IP** giữa AZ khoảng $0.02/GB; **inter-Region** khoảng $0.02/GB (con số theo slide, đơn giản hóa).
- **Dùng khi nào:** dùng **private IP** thay vì public để tiết kiệm và nhanh hơn; dùng **cùng AZ** để rẻ nhất (đổi lại mất HA).
- **Vì sao quan trọng:** "giảm chi phí truyền dữ liệu giữa EC2" → dùng private IP, cùng AZ.

### Savings Plans
- **Là gì:** cam kết **$/giờ** trong 1 hoặc 3 năm, cách dễ nhất để cam kết dài hạn; thiết lập từ **Cost Explorer**.
- **Dùng khi nào:** **EC2 Instance SP**: tới **72%**, khóa vào instance family + Region, linh hoạt AZ, size, OS, tenancy; **Compute SP**: tới **66%**, linh hoạt family, Region, size, OS, tenancy, áp cho **EC2, Fargate, Lambda**; **SageMaker AI SP**: tới 64%.
- **Vì sao quan trọng:** `[+]` nay có thêm **Database Savings Plans** (tới 35%) cho Aurora, RDS, DynamoDB, ElastiCache, DocumentDB, Neptune, DMS...

### AWS Compute Optimizer
- **Là gì:** giảm chi phí, tăng hiệu năng bằng cách đề xuất cấu hình tối ưu (**right-size**: over/under provisioned), dùng **ML** phân tích cấu hình và metric CloudWatch.
- **Dùng khi nào:** tài nguyên hỗ trợ: **EC2, Auto Scaling group, EBS, Lambda**; nay thêm ECS trên Fargate, license thương mại (SQL Server trên EC2), RDS/Aurora.
- **Vì sao quan trọng:** giảm chi phí tới 25% (theo slide); xuất đề xuất ra S3.

### Billing and Costing tools
- **Là gì:** **ước tính**: Pricing Calculator; **theo dõi**: Billing Dashboard, Cost Allocation Tags, Cost and Usage Reports, Cost Explorer; **giám sát theo kế hoạch**: Billing Alarms, Budgets.
- **Dùng khi nào:** chọn công cụ theo giai đoạn: trước khi deploy → Calculator; đang chạy → Explorer/CUR; cảnh báo → Budgets.
- **Vì sao quan trọng:** bảng tổng hợp ở mục 4.

### AWS Pricing Calculator
- **Là gì:** công cụ tại calculator.aws để **ước tính chi phí** cho kiến trúc trước khi triển khai.
- **Dùng khi nào:** lập ngân sách, so sánh phương án.
- **Vì sao quan trọng:** "estimate cost before migrating" → Pricing Calculator (không phải Cost Explorer).

### Cost Allocation Tags, Tagging & Resource Groups
- **Là gì:** **cost allocation tag** theo dõi chi phí chi tiết: **AWS generated** (tiền tố `aws:`, vd: `aws:createdBy`) và **user-defined** (tiền tố `user:`).
- **Dùng khi nào:** tag tổ chức tài nguyên (Name, Environment, Team...); tài nguyên tạo bởi CloudFormation được tag giống nhau; tạo **Resource Groups** từ tag; quản lý tag bằng **Tag Editor**.
- **Vì sao quan trọng:** "theo dõi chi phí theo dự án/phòng ban" → cost allocation tags.

### AWS Cost and Usage Reports (CUR)
- **Là gì:** bộ dữ liệu **chi tiết nhất** về chi phí và mức dùng: theo dịch vụ, theo giờ hoặc ngày, kèm metadata, giá, reservation và tag đã kích hoạt.
- **Dùng khi nào:** phân tích sâu bằng Athena, Redshift, Quick Sight.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: tạo CUR ở mục Cost and Usage Reports → nay AWS khuyến nghị **CUR 2.0 qua AWS Data Exports**; legacy CUR chưa bị bỏ nhưng trang cũ sẽ dần bị thay.

### AWS Cost Explorer
- **Là gì:** trực quan hóa, hiểu và quản lý chi phí theo thời gian; tạo báo cáo tùy chỉnh; xem tổng quan mọi account hoặc chi tiết theo tháng, giờ, tài nguyên.
- **Dùng khi nào:** chọn **Savings Plan** tối ưu; **dự báo** mức dùng.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "dự báo tới 12 tháng" → nay forecast tới **18 tháng** (granularity Monthly); dữ liệu lịch sử 13 tháng mặc định.

### Billing Alarms (CloudWatch)
- **Là gì:** metric billing lưu ở CloudWatch **us-east-1**, là chi phí **toàn cầu thực tế** (không phải dự báo).
- **Dùng khi nào:** cảnh báo đơn giản khi tổng chi phí vượt ngưỡng.
- **Vì sao quan trọng:** kém mạnh hơn AWS Budgets.

### AWS Budgets
- **Là gì:** tạo ngân sách và **cảnh báo khi chi phí vượt (hoặc dự báo vượt)**; 4 loại: **Usage, Cost, Reservation, Savings Plans**.
- **Dùng khi nào:** theo dõi mức sử dụng RI (EC2, ElastiCache, RDS, Redshift); tới 5 SNS notification mỗi budget; lọc theo service, linked account, tag, purchase option, instance type, Region, AZ...
- **Vì sao quan trọng:** "alert when forecasted cost exceeds budget" → Budgets.

### AWS Cost Anomaly Detection
- **Là gì:** dùng **ML** liên tục theo dõi chi phí để phát hiện **chi tiêu bất thường**; tự học pattern, **không cần đặt ngưỡng**.
- **Dùng khi nào:** giám sát theo dịch vụ, member account, cost allocation tag, cost category; báo cáo kèm **root-cause analysis**.
- **Vì sao quan trọng:** thông báo từng alert hoặc tóm tắt hằng ngày/tuần qua SNS.

### AWS Service Quotas
- **Là gì:** thông báo khi sắp chạm **giới hạn dịch vụ** (quota); tạo CloudWatch Alarm trong console Service Quotas.
- **Dùng khi nào:** ví dụ: Lambda concurrent executions sắp chạm giới hạn.
- **Vì sao quan trọng:** **yêu cầu tăng quota** từ Service Quotas, hoặc tắt bớt tài nguyên trước khi chạm.

### AWS Trusted Advisor
- **Là gì:** đánh giá account ở mức cao, không cài gì; khuyến nghị theo **6 nhóm**: Cost optimization, Performance, Security, Fault tolerance, Service limits, Operational excellence.
- **Dùng khi nào:** **Basic** (và Developer) chỉ có các check về **Service limits** và một số check **Security, Fault tolerance**; **full checks + Support API** cần Business Support+, Enterprise hoặc Unified Operations.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Basic có 7 core checks; full checks cho Business & Enterprise" → mô tả hiện hành như trên; Business Support+ có **hơn 500 check**.

### Support plans: Basic
- **Là gì:** miễn phí; truy cập 24x7 **customer service** (account & billing), tài liệu, whitepaper, diễn đàn.
- **Dùng khi nào:** Trusted Advisor ở mức check cơ bản; **AWS Health Dashboard – Your Account**.
- **Vì sao quan trọng:** Basic **không có hỗ trợ kỹ thuật** (technical support case).

### Support plans: Business Support+
- **Là gì:** dành cho **production workload**; phản hồi real-time có ngữ cảnh bằng **GenAI**; Trusted Advisor full + API; **24x7 phone, web, chat** với Cloud Support Engineer; case không giới hạn.
- **Dùng khi nào:** con người phản hồi trong **dưới 30 phút** cho case **business-critical system down**; hỗ trợ **phần mềm bên thứ ba** (vd: OS trên EC2).
- **Vì sao quan trọng:** giá tối thiểu **$29/tháng/account**; thay thế cho Developer và Business cũ.

### Support plans: Enterprise
- **Là gì:** cho workload **production hoặc business-critical**; gồm mọi thứ của Business Support+.
- **Dùng khi nào:** **Technical Account Manager (TAM)** được chỉ định; phản hồi **dưới 15 phút** cho case production-critical; **AWS Security Incident Response**; business review; **AWS Countdown** (hỗ trợ sự kiện quan trọng do TAM dẫn dắt).
- **Vì sao quan trọng:** mức tối thiểu nay là **$5.000** (giảm từ $15.000).

### Support plans: Unified Operations
- **Là gì:** cho workload **mission critical**; hướng dẫn kiến trúc ứng dụng, infrastructure event management.
- **Dùng khi nào:** đội ngũ được chỉ định: **TAM, Domain Specialist Engineer (DSE)**, Senior Billing and Account Specialist, Incident Management Engineer, Migration Specialist, Specialist Support Engineer; Countdown Premium; Customer Incident Response Team (CIRT).
- **Vì sao quan trọng:** review workload quan trọng, runbook và quy trình vận hành cá nhân hóa.

### Support plans cũ (đề thi thử vẫn hay dùng)
- ⚠️ Lỗi thời: **Developer, Business, Enterprise On-Ramp** ngừng từ **01/01/2027** (Enterprise On-Ramp được tự động nâng lên Enterprise trong năm 2026) → nay dùng **Business Support+** hoặc **Enterprise**.
- **Là gì:** mốc phản hồi kiểu cũ: Developer (giờ hành chính, email: general guidance < 24 giờ, system impaired < 12 giờ); Business (24x7: production impaired < 4 giờ, production down < 1 giờ); Enterprise (business-critical down < 15 phút, TAM, Concierge).
- **Dùng khi nào:** đề cũ hỏi "gói rẻ nhất có hỗ trợ 24x7 qua điện thoại" → Business (cũ); đề mới → Business Support+.

### Account best practices
- **Là gì:** dùng Organizations cho nhiều account; SCP giới hạn quyền; Control Tower để dựng nhanh; tag & cost allocation tag; IAM: MFA, least privilege, password policy; Config ghi cấu hình; CloudFormation deploy qua account/Region; Trusted Advisor; CloudTrail ghi API.
- **Dùng khi nào:** gửi service log và access log vào S3 hoặc CloudWatch Logs; Service Catalog cho user tạo stack đã duyệt.
- **Vì sao quan trọng:** **account bị xâm phạm**: đổi mật khẩu root, xóa và xoay vòng mọi mật khẩu/key, **liên hệ AWS Support**.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| AWS Organizations | Quản lý nhiều account | "manage multiple accounts centrally" |
| Management account | Account tạo Organization, trả mọi phí (tên cũ: master) | "payer account" |
| OU | Nhóm account trong Organization | "group accounts by department" |
| SCP | Giới hạn quyền tối đa của account/OU | "prevent accounts from using a service" |
| Consolidated billing | Một hóa đơn, gộp mức dùng, chia sẻ RI | "single bill", "volume discounts" |
| AWS Control Tower | Dựng & quản trị multi-account theo best practice | "landing zone", "controls" |
| AWS RAM | Chia sẻ tài nguyên giữa account | "share subnets / Transit Gateway" |
| AWS Service Catalog | Cổng self-service sản phẩm được duyệt | "approved products", "self-service portal" |
| Free plan / Paid plan | Gói account mới, $100 + tới $100 credit | "new account credits" |
| Always Free | Giới hạn miễn phí hằng tháng (Lambda, DynamoDB...) | "always free tier" |
| Savings Plans | Cam kết $/giờ 1–3 năm | "commit to spend" |
| Compute Savings Plans | Linh hoạt nhất, EC2 + Fargate + Lambda, tới 66% | "flexible across Regions/families" |
| AWS Compute Optimizer | Đề xuất rightsizing bằng ML | "over-provisioned EC2" |
| AWS Pricing Calculator | Ước tính chi phí trước khi triển khai | "estimate cost of architecture" |
| Cost Allocation Tags | Tag để phân bổ chi phí (`aws:`, `user:`) | "track costs by project" |
| Resource Groups / Tag Editor | Nhóm và quản lý tài nguyên theo tag | "manage tags in bulk" |
| Cost and Usage Report | Dữ liệu chi phí chi tiết nhất | "most granular billing data" |
| AWS Cost Explorer | Xem, phân tích, dự báo chi phí | "visualize costs", "forecast" |
| Billing Alarm | CloudWatch alarm trên chi phí thực tế (us-east-1) | "simple alarm on total bill" |
| AWS Budgets | Ngân sách và cảnh báo (cost, usage, RI, SP) | "alert when exceeding budget" |
| Cost Anomaly Detection | ML phát hiện chi tiêu bất thường | "unexpected spend spikes" |
| Service Quotas | Theo dõi và tăng giới hạn dịch vụ | "approaching limit", "request increase" |
| Trusted Advisor | Khuyến nghị theo 6 nhóm | "best practice checks" |
| TAM | Technical Account Manager | "designated technical contact" |
| Business Support+ | Gói hỗ trợ 24x7, GenAI, < 30 phút cho business-critical | "production workloads" |
| Enterprise Support | TAM, < 15 phút, Security Incident Response | "business-critical", "TAM" |
| Unified Operations | Mission critical, TAM + DSE + nhiều chuyên gia | "mission-critical" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Pricing Calculator | Cost Explorer | **Ước tính trước** khi dùng vs xem chi phí **đã/đang** dùng và dự báo | "Before migrating" → Calculator |
| Cost Explorer | Cost and Usage Report | Trực quan, báo cáo, forecast vs dữ liệu thô chi tiết nhất | "Most comprehensive data" → CUR |
| Billing Alarm | AWS Budgets | Ngưỡng chi phí thực tế đơn giản vs nhiều loại budget, cảnh báo cả **dự báo**, RI/SP | "Forecasted to exceed" → Budgets |
| Budgets | Cost Anomaly Detection | Ngưỡng do bạn đặt vs ML tự phát hiện bất thường, không cần ngưỡng | "Unusual spend, no thresholds" → Anomaly Detection |
| Compute Optimizer | Trusted Advisor | Rightsizing chi tiết bằng ML cho compute vs khuyến nghị rộng 6 nhóm | "Right-size EC2/Lambda" → Compute Optimizer |
| Trusted Advisor | Service Quotas | Có check Service limits vs quản lý/tăng quota trực tiếp | "Request quota increase" → Service Quotas |
| Organizations | Control Tower | Nền tảng multi-account vs dựng sẵn môi trường best practice trên Organizations | "Set up governed multi-account quickly" → Control Tower |
| SCP | IAM policy | Giới hạn **tối đa** cho account (không cấp quyền) vs cấp quyền cho user/role | SCP Deny thắng cả root của member account |
| RAM | Service Catalog | Chia sẻ tài nguyên đã có vs cho user tự launch sản phẩm được duyệt | |
| EC2 Instance SP | Compute SP | Tới 72%, khóa family + Region vs tới 66%, linh hoạt, gồm Fargate, Lambda | "Move from EC2 to Fargate" → Compute SP |
| Business Support+ | Enterprise | < 30 phút, không TAM vs < 15 phút, có TAM, Security Incident Response | "TAM" → Enterprise trở lên |
| Enterprise | Unified Operations | Business-critical vs mission-critical, thêm DSE và đội chuyên gia | |
| Basic | Business Support+ | Không có technical support vs 24x7 Cloud Support Engineer | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "single bill for multiple accounts" | Organizations – Consolidated Billing | |
| "prevent all accounts in an OU from using a service" | SCP | |
| "set up secure multi-account environment quickly" | Control Tower | |
| "share a VPC subnet with another account" | AWS RAM | |
| "users launch only pre-approved products" | Service Catalog | |
| "estimate monthly cost before deployment" | Pricing Calculator | |
| "visualize and forecast spending" | Cost Explorer | Forecast tới 18 tháng |
| "most detailed cost and usage data" | Cost and Usage Report (CUR 2.0 / Data Exports) | |
| "alert when cost is forecasted to exceed budget" | AWS Budgets | |
| "detect unusual spend automatically with ML" | Cost Anomaly Detection | |
| "track costs by department/project" | Cost Allocation Tags | |
| "right-size over-provisioned instances" | Compute Optimizer | |
| "approaching Lambda concurrency limit" | Service Quotas (+ CloudWatch alarm) | |
| "best-practice checks: cost, security, fault tolerance, limits" | Trusted Advisor | |
| "designated Technical Account Manager" | Enterprise (hoặc Unified Operations) | |
| "cheapest plan with 24x7 technical support" | Business Support+ | Đề cũ: Business |
| "production down, response < 15 minutes" | Enterprise | |
| "mission-critical, domain specialist engineers" | Unified Operations | |
| "account compromised" | Đổi mật khẩu root, xoay key, liên hệ AWS Support | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Công ty có 20 AWS account và muốn một hóa đơn duy nhất, đồng thời hưởng giảm giá theo tổng mức dùng. Chọn gì?
- A. AWS Organizations với consolidated billing
- B. AWS Budgets
- C. AWS Cost Explorer
- D. AWS Service Catalog

<details><summary>Đáp án</summary>

**A.** Consolidated billing gộp hóa đơn và mức dùng.
- B sai: cảnh báo ngân sách.
- C sai: phân tích chi phí.
- D sai: cổng self-service sản phẩm.
</details>

**Câu 2.** Trước khi migrate, CFO muốn ước tính chi phí hằng tháng của kiến trúc mới trên AWS. Dùng công cụ nào?
- A. AWS Cost Explorer
- B. AWS Pricing Calculator
- C. AWS Cost and Usage Report
- D. AWS Budgets

<details><summary>Đáp án</summary>

**B.** Pricing Calculator ước tính trước khi triển khai.
- A, C sai: dựa trên dữ liệu đã dùng.
- D sai: theo dõi so với ngân sách.
</details>

**Câu 3.** Team muốn nhận cảnh báo khi chi phí **dự báo** của tháng vượt $10.000. Chọn gì?
- A. CloudWatch Billing Alarm
- B. AWS Budgets
- C. AWS Trusted Advisor
- D. AWS Pricing Calculator

<details><summary>Đáp án</summary>

**B.** Budgets cảnh báo cả chi phí thực tế lẫn dự báo.
- A sai: billing alarm dựa trên chi phí thực tế, không phải dự báo.
- C sai: khuyến nghị best practice.
- D sai: ước tính trước khi triển khai.
</details>

**Câu 4.** (Chọn 2) Phát biểu nào về Service Control Policies là **đúng**?
- A. SCP áp dụng cả cho root user của member account
- B. SCP tự cấp quyền cho user
- C. SCP không áp dụng cho management account
- D. SCP chỉ áp dụng cho IAM group
- E. SCP ảnh hưởng tới service-linked role

<details><summary>Đáp án</summary>

**A, C.**
- B sai: SCP chỉ giới hạn quyền tối đa, không cấp quyền.
- D sai: áp ở cấp OU/account.
- E sai: không ảnh hưởng service-linked role.
</details>

**Câu 5.** Startup chạy workload production cần hỗ trợ kỹ thuật 24x7 qua điện thoại và chat với chi phí thấp nhất (theo các gói hiện hành). Chọn gói nào?
- A. Basic
- B. Business Support+
- C. Enterprise
- D. Unified Operations

<details><summary>Đáp án</summary>

**B.** Business Support+ là gói rẻ nhất có 24x7 phone/web/chat với Cloud Support Engineer.
- A sai: không có technical support.
- C, D sai: đắt hơn, dành cho workload business/mission-critical.
</details>

**Câu 6.** Ngân hàng cần Technical Account Manager được chỉ định và phản hồi dưới 15 phút cho sự cố production-critical. Gói tối thiểu là gì?
- A. Basic
- B. Business Support+
- C. Enterprise
- D. Developer

<details><summary>Đáp án</summary>

**C.** Enterprise có TAM và phản hồi tới 15 phút.
- A, D sai: không có TAM; Developer còn đang bị ngừng.
- B sai: không có TAM, cam kết dưới 30 phút.
</details>

**Câu 7.** Công ty muốn phát hiện các khoản chi bất thường mà không phải tự đặt ngưỡng, kèm phân tích nguyên nhân. Chọn gì?
- A. AWS Cost Anomaly Detection
- B. AWS Budgets
- C. AWS Compute Optimizer
- D. Cost Allocation Tags

<details><summary>Đáp án</summary>

**A.** Dùng ML học pattern chi tiêu, không cần ngưỡng, có root-cause analysis.
- B sai: cần đặt ngưỡng.
- C sai: rightsizing.
- D sai: phân bổ chi phí.
</details>

**Câu 8.** Công ty sắp chuyển nhiều workload từ EC2 sang Fargate và Lambda trong 3 năm tới, muốn cam kết để giảm giá mà vẫn linh hoạt. Chọn gì?
- A. Standard Reserved Instances
- B. EC2 Instance Savings Plans
- C. Compute Savings Plans
- D. Spot Instances

<details><summary>Đáp án</summary>

**C.** Compute Savings Plans áp cho EC2, Fargate, Lambda, linh hoạt family và Region.
- A sai: gắn với thuộc tính instance EC2.
- B sai: khóa vào một instance family trong một Region, không áp cho Lambda.
- D sai: không phải cam kết, có thể bị thu hồi.
</details>

---

### Nguồn đã đối chiếu
- AWS Support Plans (Basic, Business Support+, Enterprise, Unified Operations; end of support Developer/Business/Enterprise On-Ramp 01/01/2027): https://docs.aws.amazon.com/awssupport/latest/user/aws-support-plans.html
- Trusted Advisor (Basic: Service Limits + một số Security, Fault Tolerance): https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html
- Free account plan ($100 + tới $100 credit, 6 tháng): https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-plans.html
- Savings Plans types (EC2 Instance 72%, Compute 66%, SageMaker AI 64%, Database 35%): https://docs.aws.amazon.com/savingsplans/latest/userguide/plan-types.html
- SCP và management account: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html
- Control Tower controls (tên cũ guardrails): https://docs.aws.amazon.com/controltower/latest/userguide/how-controls-work.html
- Compute Optimizer supported resources: https://docs.aws.amazon.com/compute-optimizer/latest/ug/supported-resources.html
- Cost Explorer forecast (+18M): https://docs.aws.amazon.com/cost-management/latest/userguide/ce-modify.html
- Legacy CUR vs CUR 2.0 in Data Exports: https://docs.aws.amazon.com/cur/latest/userguide/troubleshooting-cur-2-0.html
