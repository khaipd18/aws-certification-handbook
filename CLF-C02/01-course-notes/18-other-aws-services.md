# CLF-C02 — Other AWS Services

> Các dịch vụ "lẻ" thỉnh thoảng xuất hiện trong **Domain 3**: end-user computing, IoT, frontend/mobile, backup & DR, migration, chaos engineering, workflow. Chương này có nhiều thay đổi tên và trạng thái.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **WorkSpaces** = **desktop ảo** (DaaS, Windows/Linux) thay VDI on-prem. **AppStream 2.0** (nay tên **WorkSpaces Applications**) = **stream một ứng dụng** desktop tới trình duyệt.
- **IoT Core** = kết nối hàng tỷ thiết bị IoT lên AWS. **AppSync** = GraphQL, đồng bộ dữ liệu real-time cho app. **Amplify** = bộ công cụ build và deploy full-stack web/mobile.
- **AWS Backup** = quản lý backup tập trung, tự động trên nhiều dịch vụ, cross-Region và cross-account.
- **4 chiến lược DR** theo chi phí tăng dần: **Backup & Restore → Pilot Light → Warm Standby → Multi-Site/Hot-Site**. **Elastic Disaster Recovery (DRS)** = replicate server liên tục để khôi phục vào AWS trong vài phút.
- **7Rs migration:** Retire, Retain, Relocate, Rehost (lift & shift), Replatform (lift & reshape), Repurchase (drop & shop), Refactor/Re-architect.
- Công cụ migration: **Application Migration Service (MGN)** (rehost), **Migration Evaluator** (business case), ⚠️ Application Discovery Service và Migration Hub không nhận khách mới → **AWS Transform**.
- **Fault Injection Service (FIS)** = chaos engineering; **Step Functions** = workflow serverless; **Ground Station** = trạm mặt đất vệ tinh; ⚠️ **Pinpoint** hết hỗ trợ 30/10/2026.

---

## 2. Core Concepts

### Amazon WorkSpaces
- **Là gì:** **Desktop as a Service (DaaS)** managed: cấp desktop Windows hoặc Linux cho người dùng.
- **Dùng khi nào:** thay thế hạ tầng **VDI** on-prem; scale nhanh tới hàng nghìn người dùng; đặt WorkSpaces ở Region gần người dùng để giảm latency.
- **Vì sao quan trọng:** dữ liệu bảo mật, tích hợp KMS; trả theo tháng hoặc theo giờ.

### Amazon AppStream 2.0 (WorkSpaces Applications)
- ⚠️ Lỗi thời: tên **Amazon AppStream 2.0** → docs hiện gọi là **Amazon WorkSpaces Applications** (exam guide CLF-C02 vẫn liệt kê tên AppStream 2.0).
- **Là gì:** **stream ứng dụng desktop** tới bất kỳ máy tính nào qua **trình duyệt**, không cần kết nối VDI; cấu hình instance type theo từng ứng dụng (CPU, RAM, GPU).
- **Dùng khi nào:** cần cung cấp một ứng dụng nặng (vd: phần mềm thiết kế) cho người dùng trên mọi thiết bị.

### WorkSpaces vs AppStream 2.0
- **Là gì:** **WorkSpaces**: VDI và **cả desktop** đầy đủ, người dùng kết nối vào desktop rồi mở app; on-demand hoặc always on.
- **Dùng khi nào:** **AppStream 2.0**: chỉ **stream ứng dụng** vào trình duyệt.
- **Vì sao quan trọng:** "full desktop" → WorkSpaces; "single application in browser" → AppStream 2.0.

### AWS IoT Core
- **Là gì:** kết nối thiết bị **IoT** (mạng thiết bị có kết nối Internet thu thập và truyền dữ liệu) lên AWS Cloud; serverless, bảo mật, scale tới hàng tỷ thiết bị và hàng nghìn tỷ message.
- **Dùng khi nào:** app giao tiếp được với thiết bị kể cả khi thiết bị không kết nối; publish & subscribe message; tích hợp Lambda, S3, SageMaker.
- **Vì sao quan trọng:** "connect IoT devices to the cloud" → IoT Core.

### AWS AppSync
- **Là gì:** lưu và **đồng bộ dữ liệu real-time** giữa app mobile và web, dùng **GraphQL**.
- **Dùng khi nào:** cần real-time subscription, đồng bộ dữ liệu offline (thay Cognito Sync), bảo mật chi tiết; tích hợp DynamoDB, Lambda.
- **Vì sao quan trọng:** Amplify dùng AppSync phía sau. "GraphQL" → AppSync.

### AWS Amplify
- **Là gì:** bộ công cụ và dịch vụ giúp **phát triển và deploy** web/mobile app **full-stack** có khả năng scale.
- **Dùng khi nào:** cần auth, storage, API (REST, GraphQL), CI/CD, PubSub, analytics, AI/ML, monitoring; code từ AWS hoặc GitHub.
- **Vì sao quan trọng:** "build and host full-stack web/mobile app quickly" → Amplify.

### AWS Infrastructure Composer
- **Là gì:** thiết kế và build ứng dụng serverless **trực quan**; deploy hạ tầng mà không cần là chuyên gia AWS.
- **Dùng khi nào:** cấu hình cách tài nguyên tương tác; **sinh IaC bằng CloudFormation**; import template CloudFormation/SAM có sẵn để xem trực quan.
- **Vì sao quan trọng:** tên cũ **AWS Application Composer** (exam guide xếp tên cũ vào out-of-scope).

### AWS Device Farm
- **Là gì:** test web và mobile app trên **trình duyệt desktop, điện thoại và tablet thật** (không phải giả lập).
- **Dùng khi nào:** chạy test song song trên nhiều thiết bị; cấu hình GPS, ngôn ngữ, Wi-Fi, Bluetooth; nhận báo cáo, log, screenshot.
- **Vì sao quan trọng:** exam guide CLF-C02 xếp Device Farm vào **out-of-scope**.

### AWS Backup
- **Là gì:** dịch vụ fully managed để **quản lý và tự động hóa backup tập trung** trên nhiều dịch vụ AWS: EC2, EBS, DynamoDB, RDS, EFS, Aurora, FSx, Storage Gateway, S3.
- **Dùng khi nào:** backup on-demand và theo lịch; **Point-in-time Recovery**; retention, lifecycle, backup policy; **cross-Region** và **cross-account** (qua Organizations).
- **Vì sao quan trọng:** "centralize backup across services and accounts" → AWS Backup.

### Disaster Recovery strategies
- **Là gì:** **Backup & Restore** (rẻ nhất, khôi phục chậm nhất); **Pilot Light** (chạy sẵn **chức năng lõi**, ví dụ DB, sẵn sàng scale); **Warm Standby** (**bản đầy đủ** của app nhưng ở **kích thước tối thiểu**).
- **Dùng khi nào:** **Multi-Site / Hot-Site** (bản đầy đủ ở **kích thước đầy đủ**, đắt nhất, khôi phục gần như tức thì).
- **Vì sao quan trọng:** setup DR điển hình trên cloud: app multi-AZ ở Region chính, failover sang Region khác.

### AWS Elastic Disaster Recovery (DRS)
- **Là gì:** tên cũ **CloudEndure Disaster Recovery**; khôi phục nhanh server vật lý, ảo và cloud vào AWS.
- **Dùng khi nào:** bảo vệ DB quan trọng (Oracle, MySQL, SQL Server), app doanh nghiệp (SAP), chống ransomware.
- **Vì sao quan trọng:** **replicate block-level liên tục** (vài giây) vào staging giá rẻ trên AWS; failover trong vài phút; có failback.

### Cloud Migration Strategies: The 7Rs
- **Là gì:** **Retire** (tắt thứ không cần, giảm bề mặt tấn công, tiết kiệm 10–20%); **Retain** (chưa làm gì: bảo mật, compliance, phụ thuộc chưa giải quyết, mainframe); **Relocate** (chuyển sang phiên bản cloud, vd: VMware SDDC → VMware Cloud on AWS, hoặc chuyển EC2 sang VPC/account/Region khác).
- **Dùng khi nào:** **Rehost** ("lift and shift": đưa nguyên trạng, không tối ưu, có thể tiết kiệm tới 30%, dùng MGN); **Replatform** ("lift and reshape": vd: DB sang RDS, app sang Beanstalk, không đổi kiến trúc lõi); **Repurchase** ("drop and shop": chuyển sang SaaS, vd: CRM → Salesforce).
- **Vì sao quan trọng:** **Refactor / Re-architect**: thiết kế lại theo cloud-native (monolith → microservices, serverless) để thêm tính năng, tăng scale, hiệu năng, bảo mật, agility.

### AWS Application Discovery Service
- ⚠️ Lỗi thời: Application Discovery Service **không nhận khách mới từ 07/11/2025** → AWS Transform cung cấp khả năng discovery tương đương.
- **Là gì:** thu thập thông tin data center on-prem để lập kế hoạch migration: mức dùng server và **sơ đồ phụ thuộc**.
- **Dùng khi nào:** **Agentless** (Agentless Discovery Connector: VM inventory, cấu hình, lịch sử CPU/RAM/disk); **Agent-based** (Discovery Agent: cấu hình, hiệu năng, process đang chạy, kết nối mạng giữa các hệ thống); xem kết quả trong Migration Hub.

### AWS Application Migration Service (MGN)
- **Là gì:** giải pháp **rehost (lift-and-shift)**, phiên bản AWS của CloudEndure Migration, thay AWS Server Migration Service (SMS).
- **Dùng khi nào:** chuyển server vật lý, ảo, cloud khác sang chạy native trên AWS; hỗ trợ nhiều nền tảng, OS, DB.
- **Vì sao quan trọng:** replicate liên tục vào staging rồi **cutover**, downtime tối thiểu, giảm chi phí.

### AWS Migration Evaluator
- **Là gì:** xây dựng **business case dựa trên dữ liệu** cho việc migrate lên AWS; tạo baseline rõ ràng về những gì đang chạy.
- **Dùng khi nào:** cài **Agentless Collector** để discovery diện rộng: snapshot footprint on-prem, phụ thuộc server; import dữ liệu từ công cụ bên thứ ba hoặc Application Discovery Service.
- **Vì sao quan trọng:** cho ra **Quick Insights** (chi phí) và business case; "build a business case for migration" → Migration Evaluator.

### AWS Migration Hub
- ⚠️ Lỗi thời: Migration Hub **không nhận khách mới từ 07/11/2025** → **AWS Transform** (ra mắt 05/2025) gồm toàn bộ khả năng của Migration Hub (Strategy Recommendations, EC2 recommendations, Journeys, Orchestrator) với tự động hóa bằng AI.
- **Là gì:** nơi **tập trung** inventory server/app để đánh giá, lập kế hoạch, theo dõi migration; nhận trạng thái từ MGN và DMS.
- **Dùng khi nào:** Migration Hub Orchestrator có template dựng sẵn cho SAP, SQL Server...

### AWS Fault Injection Service (FIS)
- ⚠️ Lỗi thời: tên "AWS Fault Injection **Simulator**" → nay là **AWS Fault Injection Service**.
- **Là gì:** chạy **fault injection experiment** trên workload AWS theo **Chaos Engineering**: tạo sự kiện gây gián đoạn (tăng CPU/RAM đột ngột), quan sát phản ứng và cải thiện.
- **Dùng khi nào:** tìm bug ẩn, bottleneck hiệu năng, điểm mù giám sát; hỗ trợ EC2, ECS, EKS, RDS; dùng template dựng sẵn; dừng khi hoàn tất hoặc CloudWatch alarm kích hoạt.

### AWS Step Functions
- **Là gì:** xây **workflow trực quan serverless** để điều phối Lambda và dịch vụ khác.
- **Dùng khi nào:** tuần tự, song song, điều kiện, timeout, xử lý lỗi; tích hợp EC2, ECS, on-prem, API Gateway, SQS; có thể thêm bước **phê duyệt thủ công** của con người.
- **Vì sao quan trọng:** use case: xử lý đơn hàng, data processing, mọi workflow. "Orchestrate multiple Lambda functions" → Step Functions.

### AWS Ground Station
- **Là gì:** dịch vụ fully managed để **điều khiển liên lạc vệ tinh**, xử lý dữ liệu, scale vận hành vệ tinh; mạng trạm mặt đất gần các Region AWS.
- **Dùng khi nào:** tải dữ liệu vệ tinh vào VPC trong vài giây, gửi tới S3 hoặc EC2; dự báo thời tiết, chụp ảnh bề mặt, truyền thông, phát sóng video.
- **Vì sao quan trọng:** exam guide CLF-C02 xếp Ground Station vào **out-of-scope**.

### Amazon Pinpoint
- ⚠️ Sắp ngừng: Amazon Pinpoint **hết hỗ trợ ngày 30/10/2026** → tính năng engagement (segment, campaign, journey, analytics) chuyển sang **Amazon Connect**; SMS, voice, push, OTP chuyển sang **AWS End User Messaging**.
- **Là gì:** dịch vụ marketing communication **2 chiều** (gửi và nhận): email, SMS, push, voice, in-app; phân khúc và cá nhân hóa message; scale tới hàng tỷ message/ngày.
- **Dùng khi nào:** khác SNS/SES (bạn tự quản audience, nội dung, lịch gửi từng message): Pinpoint tạo template, lịch gửi, segment và **cả campaign**.

### `[+]` Amazon WorkSpaces Secure Browser & AWS License Manager
- **Là gì:** **WorkSpaces Secure Browser** (tên cũ WorkSpaces Web): trình duyệt managed để truy cập web nội bộ/SaaS an toàn, không lưu dữ liệu trên máy người dùng; **License Manager**: quản lý license phần mềm (Microsoft, Oracle...) trên AWS và on-prem, tránh vượt số license.
- **Dùng khi nào:** nhân viên/đối tác truy cập app nội bộ từ thiết bị không được quản lý; doanh nghiệp BYOL cần theo dõi license.
- **Vì sao quan trọng:** cả hai nằm trong danh sách in-scope CLF-C02.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Amazon WorkSpaces | Desktop ảo managed (DaaS) | "virtual desktops", "replace VDI" |
| AppStream 2.0 / WorkSpaces Applications | Stream ứng dụng desktop vào trình duyệt | "stream application to browser" |
| AWS IoT Core | Kết nối thiết bị IoT lên cloud | "IoT devices", "MQTT messages" |
| AWS AppSync | GraphQL, đồng bộ dữ liệu real-time/offline | "GraphQL", "real-time data sync" |
| AWS Amplify | Build & deploy full-stack web/mobile | "front-end developers", "full-stack app" |
| AWS Infrastructure Composer | Thiết kế serverless trực quan, sinh CloudFormation | "visual designer" |
| AWS Device Farm | Test app trên thiết bị thật | "test on real mobile devices" |
| AWS Backup | Backup tập trung đa dịch vụ, đa account | "centralized backup policies" |
| Backup & Restore | DR rẻ nhất, chậm nhất | "lowest cost DR" |
| Pilot Light | Chỉ chạy thành phần lõi | "core services always running" |
| Warm Standby | Bản đầy đủ ở kích thước nhỏ | "scaled-down full environment" |
| Multi-Site / Hot-Site | Bản đầy đủ ở kích thước đầy đủ | "lowest RTO", "active-active" |
| Elastic Disaster Recovery | Replicate liên tục, failover trong phút | "recover servers into AWS quickly" |
| Rehost | Lift and shift | "migrate as-is" |
| Replatform | Lift and reshape (vd: sang RDS) | "minor optimizations" |
| Repurchase | Chuyển sang SaaS | "move to Salesforce" |
| Refactor / Re-architect | Thiết kế lại cloud-native | "microservices", "serverless" |
| Retire / Retain / Relocate | Tắt / giữ nguyên / dời sang phiên bản cloud | "decommission", "keep on-prem", "VMware Cloud on AWS" |
| Application Migration Service (MGN) | Công cụ rehost | "lift-and-shift servers" |
| Migration Evaluator | Business case migration | "business case", "TCO projection" |
| AWS Transform | Dịch vụ migration/modernization bằng AI, thay Migration Hub | "AI-driven migration" |
| AWS Fault Injection Service | Chaos engineering | "inject faults", "test resiliency" |
| AWS Step Functions | Workflow serverless | "orchestrate Lambda functions", "human approval" |
| AWS Ground Station | Trạm mặt đất vệ tinh | "satellite data" |
| Amazon Pinpoint | Marketing đa kênh 2 chiều (hết hỗ trợ 30/10/2026) | "campaigns", "segments" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| WorkSpaces | AppStream 2.0 | Cả desktop vs một ứng dụng qua trình duyệt | "Full desktop" → WorkSpaces |
| AppSync | Amplify | Dịch vụ GraphQL vs bộ công cụ full-stack (dùng AppSync bên dưới) | "GraphQL API" → AppSync |
| AWS Backup | Elastic Disaster Recovery | Backup theo lịch, PITR vs replicate liên tục, failover trong phút | "RTO minutes for servers" → DRS |
| Pilot Light | Warm Standby | Chỉ lõi (DB) vs cả app ở cỡ nhỏ | |
| Warm Standby | Multi-Site | Cỡ nhỏ vs cỡ đầy đủ | |
| Rehost | Replatform | Nguyên trạng vs tối ưu nhẹ (sang managed service) | "Move DB to RDS" → Replatform |
| Replatform | Refactor | Không đổi kiến trúc lõi vs thiết kế lại | "Monolith to microservices" → Refactor |
| Repurchase | Relocate | Đổi sang sản phẩm SaaS khác vs dời sang phiên bản cloud của cùng nền tảng | "VMware Cloud on AWS" → Relocate |
| Retire | Retain | Tắt bỏ vs giữ nguyên chưa migrate | |
| Application Discovery Service | Migration Evaluator | Thu thập dữ liệu chi tiết server/phụ thuộc vs business case chi phí | "Business case" → Migration Evaluator |
| MGN | DMS | Migrate server (rehost) vs migrate database | |
| Step Functions | EventBridge | Điều phối workflow nhiều bước vs phản ứng event/lịch | "Sequence, parallel, retries" → Step Functions |
| Pinpoint | SNS / SES | Campaign, segment, template vs gửi từng message/email | "Marketing campaigns" → Pinpoint (⚠️) |
| FIS | Inspector | Gây lỗi có chủ đích để test resilience vs quét lỗ hổng | "Chaos engineering" → FIS |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "provide virtual Windows desktops to employees" | Amazon WorkSpaces | |
| "deliver a desktop application through web browser" | AppStream 2.0 (WorkSpaces Applications) | |
| "connect millions of IoT sensors" | AWS IoT Core | |
| "GraphQL API with real-time and offline sync" | AWS AppSync | |
| "quickly build and deploy full-stack web app" | AWS Amplify | |
| "centrally manage backups across services and accounts" | AWS Backup | |
| "lowest-cost DR, longer recovery acceptable" | Backup & Restore | |
| "minimal core always running, scale on disaster" | Pilot Light | |
| "near-zero downtime, full capacity in second Region" | Multi-Site | |
| "continuous replication of on-prem servers for DR" | Elastic Disaster Recovery | |
| "migrate servers as-is with minimal downtime" | Application Migration Service (Rehost) | |
| "build data-driven business case for migration" | Migration Evaluator | |
| "move from commercial CRM to SaaS" | Repurchase | |
| "decompose monolith into microservices" | Refactor | |
| "test system resilience by injecting failures" | AWS Fault Injection Service | |
| "coordinate Lambda functions in a workflow with human approval" | AWS Step Functions | |
| "download satellite data into AWS" | AWS Ground Station | Out-of-scope |

---

## 5. Câu hỏi tình huống

**Câu 1.** Công ty muốn bỏ hạ tầng VDI on-prem và cấp desktop Windows ảo cho 2.000 nhân viên. Chọn gì?
- A. Amazon WorkSpaces
- B. Amazon AppStream 2.0
- C. Amazon EC2 Auto Scaling
- D. AWS Device Farm

<details><summary>Đáp án</summary>

**A.** WorkSpaces là DaaS, thay VDI.
- B sai: stream một ứng dụng, không phải cả desktop.
- C sai: không phải dịch vụ desktop.
- D sai: test app trên thiết bị.
</details>

**Câu 2.** (Chọn 2) Chiến lược DR nào chạy sẵn một phần môi trường trên AWS để khôi phục nhanh hơn Backup & Restore?
- A. Pilot Light
- B. Warm Standby
- C. Retire
- D. Repurchase
- E. Rehost

<details><summary>Đáp án</summary>

**A, B.** Pilot Light chạy thành phần lõi; Warm Standby chạy bản đầy đủ ở kích thước nhỏ.
- C, D, E sai: là chiến lược migration (7Rs), không phải DR.
</details>

**Câu 3.** Công ty chuyển DB on-prem sang Amazon RDS mà không đổi kiến trúc ứng dụng. Đây là chiến lược nào?
- A. Rehost
- B. Replatform
- C. Refactor
- D. Retain

<details><summary>Đáp án</summary>

**B.** Replatform ("lift and reshape"): dùng dịch vụ managed, không đổi kiến trúc lõi.
- A sai: rehost giữ nguyên trạng (vd: DB trên EC2).
- C sai: thiết kế lại cloud-native.
- D sai: giữ nguyên on-prem.
</details>

**Câu 4.** Đội vận hành muốn kiểm tra hệ thống phản ứng thế nào khi CPU đột ngột tăng và một EC2 bị dừng, theo phương pháp chaos engineering. Chọn gì?
- A. AWS Fault Injection Service
- B. Amazon Inspector
- C. AWS Trusted Advisor
- D. AWS Backup

<details><summary>Đáp án</summary>

**A.** FIS chạy fault injection experiment.
- B sai: quét lỗ hổng.
- C sai: khuyến nghị best practice.
- D sai: backup.
</details>

**Câu 5.** Công ty cần quản lý backup tập trung cho EBS, RDS, DynamoDB, EFS trên nhiều account với cùng một chính sách. Chọn gì?
- A. AWS Backup
- B. EBS Snapshots thủ công
- C. AWS Elastic Disaster Recovery
- D. S3 Replication

<details><summary>Đáp án</summary>

**A.** AWS Backup quản lý backup tập trung, cross-account qua Organizations.
- B sai: thủ công, chỉ EBS.
- C sai: DR replicate liên tục cho server.
- D sai: chỉ cho S3.
</details>

**Câu 6.** Một quy trình xử lý đơn hàng gồm nhiều Lambda chạy tuần tự, có nhánh song song, retry khi lỗi và bước chờ quản lý duyệt. Chọn gì?
- A. AWS Step Functions
- B. Amazon SQS
- C. Amazon EventBridge
- D. AWS Batch

<details><summary>Đáp án</summary>

**A.** Step Functions điều phối workflow có trình tự, song song, xử lý lỗi, human approval.
- B sai: queue.
- C sai: phản ứng event, không quản lý workflow nhiều bước.
- D sai: batch job.
</details>

**Câu 7.** CFO muốn một **business case** dựa trên dữ liệu thực về footprint on-prem để quyết định có migrate lên AWS không. Chọn gì?
- A. AWS Migration Evaluator
- B. AWS Application Migration Service
- C. AWS DMS
- D. AWS Elastic Disaster Recovery

<details><summary>Đáp án</summary>

**A.** Migration Evaluator xây business case và Quick Insights về chi phí.
- B sai: thực hiện rehost.
- C sai: migrate database.
- D sai: DR.
</details>

**Câu 8.** Công ty muốn chuyển hệ thống CRM tự host sang Salesforce trong lúc lên cloud. Đây là chiến lược nào?
- A. Repurchase
- B. Relocate
- C. Rehost
- D. Retire

<details><summary>Đáp án</summary>

**A.** Repurchase ("drop and shop"): chuyển sang sản phẩm SaaS.
- B sai: dời sang phiên bản cloud của cùng nền tảng.
- C sai: đưa nguyên trạng.
- D sai: tắt bỏ.
</details>

---

### Nguồn đã đối chiếu
- Amazon WorkSpaces Applications (tên mới của AppStream 2.0): https://docs.aws.amazon.com/appstream2/latest/developerguide/what-is-appstream.html
- AWS Application Discovery Service availability change: https://docs.aws.amazon.com/application-discovery/latest/userguide/application-discovery-service-availability-change.html
- AWS Migration Hub availability change → AWS Transform: https://docs.aws.amazon.com/migrationhub/latest/ug/migrationhub-availability-change.html
- What is AWS Fault Injection Service: https://docs.aws.amazon.com/fis/latest/userguide/what-is.html
- Amazon Pinpoint end of support (30/10/2026): https://docs.aws.amazon.com/pinpoint/latest/userguide/migrate.html
- 7 Rs migration strategies: https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-migration/detailed-portfolio-discovery.html
- CLF-C02 in-scope services (WorkSpaces Secure Browser, License Manager, AppStream 2.0): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-in-scope-services.html
- CLF-C02 out-of-scope services (Device Farm, Ground Station, Application Composer): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
