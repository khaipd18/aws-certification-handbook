# CLF-C02 — AWS Architecting & Ecosystem

> Thuộc **Domain 1** (Well-Architected Framework, design principles, CAF, cloud economics) và **Domain 4** (tài nguyên hỗ trợ: Marketplace, Partner, re:Post, training).
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Well-Architected Framework** có **6 pillars**: **Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability**. Các pillar **cộng hưởng**, không phải đánh đổi.
- Nguyên tắc thiết kế cloud: **Scalability, Disposable resources, Automation, Loose coupling, Services not servers**.
- **Well-Architected Tool** (miễn phí) review workload theo 6 pillar; **Customer Carbon Footprint Tool** đo phát thải carbon từ mức dùng AWS.
- **AWS CAF** có **6 perspectives**: Business, People, Governance (nhóm business) và Platform, Security, Operations (nhóm technical); 4 **transformation domains** (Technology, Process, Organization, Product); 4 **phases**: **Envision → Align → Launch → Scale**.
- **Right sizing** = chọn instance type/size rẻ nhất mà vẫn đáp ứng yêu cầu; **bắt đầu nhỏ**, làm cả trước migration và liên tục sau đó.
- Hệ sinh thái: **Marketplace** (phần mềm bên thứ ba, tính vào hóa đơn AWS), **AWS Partner Network**, **Professional Services**, **re:Post** (thay AWS Forums), **Knowledge Center**, whitepaper, Solutions Library, training.
- **AWS Managed Services (AMS)** = đội ngũ AWS vận hành hạ tầng thay bạn 24/365.

---

## 2. Core Concepts

### General guiding principles
- **Là gì:** ngừng đoán capacity; test ở quy mô production; tự động hóa để thử nghiệm kiến trúc dễ hơn; cho phép kiến trúc tiến hóa.
- **Dùng khi nào:** thiết kế theo yêu cầu thay đổi; ra quyết định kiến trúc dựa trên **dữ liệu**; cải thiện qua **game days** (giả lập ngày flash sale).
- **Vì sao quan trọng:** đề hỏi "nguyên tắc nào của Well-Architected" → các câu này.

### Cloud design principles
- **Là gì:** **Scalability** (vertical & horizontal); **Disposable resources** (server dễ bỏ và dễ cấu hình lại); **Automation** (serverless, IaC, Auto Scaling).
- **Dùng khi nào:** **Loose coupling**: chia monolith thành thành phần nhỏ, lỗi ở một phần không lan sang phần khác; **Services, not servers**: dùng managed service, DB, serverless thay vì chỉ EC2.
- **Vì sao quan trọng:** "thay đổi một thành phần không làm hỏng hệ thống" → loose coupling (SQS, SNS).

### 1) Operational Excellence
- **Là gì:** khả năng **vận hành và giám sát** hệ thống để mang lại giá trị và liên tục cải thiện quy trình.
- **Dùng khi nào:** nguyên tắc: **operations as code** (IaC), thay đổi **nhỏ, thường xuyên, đảo ngược được**, tinh chỉnh quy trình vận hành, dự liệu lỗi, học từ lỗi, dùng managed service, **observability** để có insight hành động được.
- **Vì sao quan trọng:** dịch vụ: CloudFormation, Config, CloudTrail, CloudWatch, X-Ray, CodeBuild, CodeCommit, CodeDeploy, CodePipeline.

### 2) Security
- **Là gì:** bảo vệ thông tin, hệ thống, tài sản qua đánh giá rủi ro và giảm thiểu.
- **Dùng khi nào:** nguyên tắc: **nền tảng danh tính mạnh** (least privilege, giảm credential dài hạn), **traceability**, **bảo mật mọi tầng**, tự động hóa best practice bảo mật, **bảo vệ dữ liệu in transit và at rest**, **giữ con người tránh xa dữ liệu**, chuẩn bị cho sự cố bảo mật.
- **Vì sao quan trọng:** dịch vụ: IAM, STS, MFA, Organizations; Config, CloudTrail, CloudWatch; CloudFront, VPC, Shield, WAF, Inspector; KMS, S3, ELB, EBS, RDS.

### 3) Reliability
- **Là gì:** hệ thống **phục hồi** sau gián đoạn, tự lấy tài nguyên theo nhu cầu, giảm thiểu lỗi cấu hình hoặc sự cố mạng tạm thời.
- **Dùng khi nào:** nguyên tắc: **test quy trình khôi phục**, **tự phục hồi khi lỗi**, **scale ngang** để tránh điểm lỗi chung, **ngừng đoán capacity**, quản lý thay đổi bằng tự động hóa.
- **Vì sao quan trọng:** dịch vụ: IAM, VPC, Service Quotas, Trusted Advisor; CloudWatch, CloudTrail, Config, Auto Scaling; Backup, CloudFormation, S3, Glacier, Route 53.

### 4) Performance Efficiency
- **Là gì:** dùng tài nguyên compute **hiệu quả** để đáp ứng yêu cầu, giữ hiệu quả khi nhu cầu và công nghệ thay đổi.
- **Dùng khi nào:** nguyên tắc: **dân chủ hóa công nghệ tiên tiến** (dùng như dịch vụ), **go global in minutes**, **dùng kiến trúc serverless**, **thử nghiệm thường xuyên**, **mechanical sympathy** (hiểu rõ dịch vụ AWS).
- **Vì sao quan trọng:** dịch vụ: Auto Scaling, EBS, S3, Lambda, RDS, CloudFormation, CloudWatch, ElastiCache, CloudFront.

### 5) Cost Optimization
- **Là gì:** vận hành hệ thống mang lại giá trị với **chi phí thấp nhất**.
- **Dùng khi nào:** nguyên tắc: **consumption model** (trả theo dùng), đo hiệu quả tổng thể, **ngừng chi tiền cho data center**, **phân tích và phân bổ chi phí** (dùng tag, đo ROI), dùng managed service để giảm chi phí sở hữu.
- **Vì sao quan trọng:** dịch vụ: Budgets, Cost and Usage Report, Cost Explorer, RI reporting; Spot, RI, Glacier; Trusted Advisor; Auto Scaling, Lambda.

### 6) Sustainability
- **Là gì:** giảm **tác động môi trường** của workload trên cloud.
- **Dùng khi nào:** nguyên tắc: hiểu tác động của bạn, đặt mục tiêu bền vững, **tối đa hóa mức sử dụng** (right size, giảm tài nguyên rảnh), áp dụng phần cứng/phần mềm mới hiệu quả hơn, dùng managed service, giảm tác động phía người dùng cuối.
- **Vì sao quan trọng:** dịch vụ: EC2 Auto Scaling, Lambda, Fargate; Cost Explorer, **Graviton**, T instances, Spot; EFS-IA, Glacier, EBS Cold HDD; S3 Lifecycle, Intelligent-Tiering, Data Lifecycle Manager; "Read local, write global": RDS Read Replicas, Aurora Global DB, DynamoDB Global Tables, CloudFront.

### AWS Well-Architected Tool
- **Là gì:** công cụ **miễn phí** để review kiến trúc theo 6 pillar và áp dụng best practice.
- **Dùng khi nào:** chọn workload, trả lời câu hỏi, so sánh với 6 pillar, nhận tư vấn (video, tài liệu), tạo báo cáo, xem dashboard.
- **Vì sao quan trọng:** "review architecture against best practices" → Well-Architected Tool (không phải Trusted Advisor).

### AWS Customer Carbon Footprint Tool
- **Là gì:** theo dõi, đo lường, xem xét và **dự báo phát thải carbon** từ mức dùng AWS.
- **Dùng khi nào:** báo cáo theo địa lý và dịch vụ, theo thời gian; hỗ trợ mục tiêu bền vững của doanh nghiệp.
- **Vì sao quan trọng:** gắn với pillar Sustainability.

### AWS Cloud Adoption Framework (CAF)
- **Là gì:** giúp xây dựng và thực thi kế hoạch **chuyển đổi số** toàn diện với AWS; do chuyên gia AWS tạo từ best practice của hàng nghìn khách hàng.
- **Dùng khi nào:** xác định **năng lực tổ chức** cần có để chuyển đổi cloud thành công.
- **Vì sao quan trọng:** gom năng lực vào **6 perspectives**.

### CAF — Business capabilities
- **Là gì:** **Business**: đảm bảo đầu tư cloud thúc đẩy mục tiêu chuyển đổi số và kết quả kinh doanh; **People**: cầu nối công nghệ – kinh doanh, tập trung **văn hóa, cơ cấu tổ chức, lãnh đạo, nhân lực**.
- **Dùng khi nào:** **Governance**: điều phối sáng kiến cloud, tối đa lợi ích, **giảm rủi ro** chuyển đổi.
- **Vì sao quan trọng:** "đào tạo nhân viên, thay đổi văn hóa" → People; "quản lý rủi ro, ngân sách chương trình" → Governance.

### CAF — Technical capabilities
- **Là gì:** **Platform**: xây nền tảng hybrid cloud cấp doanh nghiệp, hiện đại hóa workload, giải pháp cloud-native; **Security**: đảm bảo **confidentiality, integrity, availability** của dữ liệu và workload.
- **Dùng khi nào:** **Operations**: đảm bảo dịch vụ cloud được cung cấp ở mức đáp ứng nhu cầu kinh doanh.
- **Vì sao quan trọng:** "monitoring, incident management" → Operations; "kiến trúc, provisioning" → Platform.

### CAF — Transformation domains
- **Là gì:** **Technology** (migrate và hiện đại hóa hạ tầng, app, data platform); **Process** (số hóa, tự động hóa, tối ưu vận hành; dùng data/ML cải thiện CSKH).
- **Dùng khi nào:** **Organization** (tái định hình mô hình vận hành, tổ chức team theo sản phẩm và value stream, agile); **Product** (tái định hình mô hình kinh doanh: sản phẩm, dịch vụ, nguồn doanh thu mới).
- **Vì sao quan trọng:** chuỗi giá trị chuyển đổi: Technology → Process → Organization → Product.

### CAF — Transformation phases
- **Là gì:** **Envision**: chỉ ra cloud thúc đẩy kết quả kinh doanh thế nào, xác định cơ hội; **Align**: tìm **khoảng trống năng lực** trên 6 perspective → **Action Plan**.
- **Dùng khi nào:** **Launch**: triển khai **pilot** trong production, chứng minh giá trị từng bước; **Scale**: mở rộng pilot tới quy mô mong muốn.
- **Vì sao quan trọng:** thứ tự **Envision → Align → Launch → Scale** hay bị hỏi.

### AWS Right Sizing
- **Là gì:** khớp instance type và size với yêu cầu hiệu năng và capacity của workload ở **chi phí thấp nhất**; cũng là việc rà soát instance đang chạy để bỏ hoặc hạ cấp.
- **Dùng khi nào:** **trước** migration và **liên tục sau** khi lên cloud (yêu cầu thay đổi theo thời gian); scale up dễ nên **luôn bắt đầu nhỏ**.
- **Vì sao quan trọng:** công cụ hỗ trợ: CloudWatch, Cost Explorer, Trusted Advisor, công cụ bên thứ ba (và Compute Optimizer, chương 16).

### AWS Ecosystem — tài nguyên miễn phí
- **Là gì:** AWS Blogs, **AWS Whitepapers & Guides**, **AWS Solutions Library** (tên cũ **Quick Starts**: giải pháp đã được kiểm định, vd: live streaming).
- **Dùng khi nào:** ⚠️ Lỗi thời: "AWS Forums (community)" → đã được thay bằng **AWS re:Post**.
- **Vì sao quan trọng:** "where to find reference architectures/vetted solutions" → Solutions Library, whitepapers.

### AWS Support trong ecosystem
- ⚠️ Lỗi thời: bảng Developer / Business / Enterprise với mốc 24/12/4/1 giờ → Developer và Business **ngừng từ 01/01/2027**; gói hiện tại: Basic, **Business Support+** (< 30 phút cho business-critical down), **Enterprise** (TAM, < 15 phút), **Unified Operations**. Chi tiết ở chương 16.
- **Là gì:** mốc cũ: general guidance < 24 giờ làm việc, system impaired < 12 giờ, production impaired < 4 giờ, production down < 1 giờ, business-critical down < 15 phút.
- **Dùng khi nào:** chỉ dùng mốc cũ khi đề thi thử vẫn hỏi theo gói cũ.

### AWS Marketplace
- **Là gì:** **catalog số** với hàng nghìn phần mềm từ **nhà cung cấp độc lập (bên thứ ba)**: AMI tùy biến, CloudFormation template, SaaS, container.
- **Dùng khi nào:** mua phần mềm bên thứ ba chạy trên AWS; bạn cũng có thể **bán** giải pháp của mình.
- **Vì sao quan trọng:** mua qua Marketplace thì **tính vào hóa đơn AWS**.

### AWS Training
- **Là gì:** AWS Digital (online) và Classroom Training (trực tiếp hoặc ảo); Private Training cho tổ chức; chương trình cho chính phủ Mỹ và doanh nghiệp.
- **Dùng khi nào:** **AWS Academy**: giúp trường đại học dạy AWS.
- **Vì sao quan trọng:** `[+]` nền tảng học chính thức hiện nay là **AWS Skill Builder**.

### AWS Professional Services & Partner Network
- **Là gì:** **Professional Services**: đội chuyên gia toàn cầu của AWS làm việc cùng team bạn và đối tác APN; **APN (AWS Partner Network)**: **Technology Partners** (phần cứng, kết nối, phần mềm), **Consulting Partners** (công ty dịch vụ giúp build trên AWS), **Training Partners**.
- **Dùng khi nào:** **AWS Competency Program**: cấp cho đối tác chứng minh năng lực kỹ thuật và thành công với khách hàng ở lĩnh vực chuyên biệt; **AWS Navigate Program**: giúp đối tác trở thành đối tác tốt hơn.
- **Vì sao quan trọng:** ❓ cần kiểm chứng: AWS đã tái cấu trúc APN theo các "path" (Software, Services...), tên "Technology/Consulting Partner" có thể đã đổi. Đề CLF vẫn dùng cách phân loại trong slide.

### AWS re:Post & Knowledge Center
- **Là gì:** **re:Post** = dịch vụ Q&A do AWS quản lý, câu trả lời crowd-sourced và được chuyên gia review, **thay AWS Forums**; thành viên tích điểm reputation.
- **Dùng khi nào:** câu hỏi của khách hàng **Premium Support** không ai trả lời sẽ được chuyển tới AWS Support engineer; **Knowledge Center** chứa câu hỏi và yêu cầu phổ biến nhất.
- **Vì sao quan trọng:** **không** dùng re:Post cho câu hỏi **gấp** hoặc có **thông tin độc quyền/bí mật**.

### AWS Managed Services (AMS)
- **Là gì:** đội chuyên gia AWS **vận hành hạ tầng và ứng dụng** của bạn (bảo mật, độ tin cậy, availability); xử lý change request, monitoring, patch, security, backup.
- **Dùng khi nào:** tổ chức muốn chuyển giao việc vận hành thường ngày để tập trung kinh doanh; hoạt động **24/365**.
- **Vì sao quan trọng:** exam guide CLF-C02 xếp AMS vào **out-of-scope**.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Well-Architected Framework | Bộ best practice 6 pillar | "architectural best practices" |
| Operational Excellence | Vận hành, giám sát, cải tiến | "operations as code", "small reversible changes" |
| Security pillar | Bảo vệ dữ liệu, hệ thống | "least privilege", "traceability" |
| Reliability pillar | Phục hồi, tự lấy tài nguyên theo nhu cầu | "recover from failure", "test recovery" |
| Performance Efficiency | Dùng tài nguyên hiệu quả | "serverless", "go global", "experiment" |
| Cost Optimization | Giá trị với chi phí thấp nhất | "consumption model", "analyze expenditure" |
| Sustainability | Giảm tác động môi trường | "carbon", "maximize utilization" |
| Loose coupling | Thành phần độc lập, lỗi không lan | "failure in one component doesn't cascade" |
| Disposable resources | Server dễ thay thế, cấu hình tự động | "treat servers as disposable" |
| Well-Architected Tool | Review workload theo 6 pillar | "free architecture review tool" |
| Customer Carbon Footprint Tool | Đo/dự báo phát thải carbon | "track carbon emissions" |
| AWS CAF | Khung chuyển đổi cloud | "cloud adoption plan" |
| CAF perspectives | Business, People, Governance, Platform, Security, Operations | "organizational capabilities" |
| Envision / Align / Launch / Scale | 4 phase chuyển đổi CAF | "pilot", "action plan" |
| Right sizing | Chọn cấu hình rẻ nhất đủ yêu cầu | "downsize", "start small" |
| AWS Marketplace | Catalog phần mềm bên thứ ba | "third-party software billed through AWS" |
| AWS Solutions Library | Giải pháp tham khảo đã kiểm định (tên cũ Quick Starts) | "vetted reference deployments" |
| APN | Mạng lưới đối tác AWS | "find a consulting partner" |
| AWS Professional Services | Đội chuyên gia của AWS | "AWS experts work with your team" |
| AWS re:Post | Q&A cộng đồng do AWS quản lý | "community questions", "replaced forums" |
| Knowledge Center | Câu hỏi phổ biến nhất | "common questions and answers" |
| AWS Managed Services (AMS) | AWS vận hành hạ tầng thay bạn | "offload operations", "24/365" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Reliability | Performance Efficiency | Phục hồi, chịu lỗi vs dùng tài nguyên hiệu quả | "Recover automatically" → Reliability |
| Operational Excellence | Reliability | Quy trình vận hành, thay đổi nhỏ đảo ngược vs khôi phục, scale ngang | "Operations as code" → Op Ex |
| Cost Optimization | Sustainability | Giảm tiền vs giảm tác động môi trường | Right sizing thuộc cả hai |
| Well-Architected Tool | Trusted Advisor | Review kiến trúc theo câu hỏi vs check tự động trên account | "Answer questions about workload" → WA Tool |
| CAF People | CAF Governance | Văn hóa, kỹ năng, lãnh đạo vs rủi ro, điều phối chương trình | |
| CAF Platform | CAF Operations | Xây nền tảng vs vận hành dịch vụ ở mức cam kết | |
| Envision | Align | Xác định cơ hội vs tìm khoảng trống → action plan | |
| Launch | Scale | Pilot vs mở rộng | |
| Marketplace | Partner Network | Mua phần mềm vs tìm đối tác dịch vụ/công nghệ | "Buy AMI from vendor" → Marketplace |
| Professional Services | AMS | Chuyên gia giúp triển khai dự án vs đội vận hành liên tục | "Ongoing operations" → AMS |
| re:Post | AWS Support | Cộng đồng, không gấp vs hỗ trợ chính thức có SLA | "Urgent production issue" → Support |
| Solutions Library | Whitepapers | Giải pháp triển khai được vs tài liệu hướng dẫn | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "perform operations as code, small reversible changes" | Operational Excellence | |
| "apply security at all layers, least privilege" | Security | |
| "automatically recover from failure, test recovery" | Reliability | |
| "use serverless, go global in minutes" | Performance Efficiency | |
| "pay only for what you use, analyze expenditure" | Cost Optimization | |
| "minimize environmental impact" | Sustainability | |
| "review workload against 6 pillars" | Well-Architected Tool | |
| "measure carbon emissions of AWS usage" | Customer Carbon Footprint Tool | |
| "organizational change, training, culture" | CAF People | |
| "identify capability gaps, create action plan" | CAF Align phase | |
| "pilot in production" | CAF Launch phase | |
| "buy third-party software billed on AWS invoice" | AWS Marketplace | |
| "find partner to help migrate" | AWS Partner Network (Consulting/Services) | |
| "community Q&A, replaced AWS Forums" | AWS re:Post | |
| "AWS experts operate infrastructure for you" | AWS Managed Services | Out-of-scope |
| "choose cheapest instance meeting requirements" | Right sizing | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Team triển khai thay đổi nhỏ, thường xuyên, có thể rollback và quản lý hạ tầng bằng code. Đây là nguyên tắc của pillar nào?
- A. Operational Excellence
- B. Reliability
- C. Cost Optimization
- D. Sustainability

<details><summary>Đáp án</summary>

**A.** "Operations as code" và "frequent, small, reversible changes" là nguyên tắc của Operational Excellence.
- B sai: tập trung khôi phục khi lỗi.
- C, D sai: chi phí và môi trường.
</details>

**Câu 2.** (Chọn 2) Đâu là pillar của AWS Well-Architected Framework?
- A. Sustainability
- B. Elasticity
- C. Reliability
- D. Agility
- E. Scalability

<details><summary>Đáp án</summary>

**A, C.** 6 pillar: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability.
- B, D, E sai: là đặc tính của cloud, không phải pillar.
</details>

**Câu 3.** Công ty muốn review kiến trúc workload bằng bộ câu hỏi theo 6 pillar và nhận báo cáo khuyến nghị miễn phí. Dùng gì?
- A. AWS Well-Architected Tool
- B. AWS Trusted Advisor
- C. AWS Config
- D. AWS Pricing Calculator

<details><summary>Đáp án</summary>

**A.** Well-Architected Tool review workload qua bộ câu hỏi.
- B sai: check tự động trên account, không theo câu hỏi kiến trúc.
- C sai: theo dõi cấu hình.
- D sai: ước tính chi phí.
</details>

**Câu 4.** Trong AWS CAF, perspective nào tập trung vào văn hóa, cơ cấu tổ chức, lãnh đạo và nhân lực?
- A. Business
- B. People
- C. Governance
- D. Platform

<details><summary>Đáp án</summary>

**B.** People perspective.
- A sai: mục tiêu kinh doanh của đầu tư cloud.
- C sai: điều phối và giảm rủi ro.
- D sai: nền tảng kỹ thuật.
</details>

**Câu 5.** Theo CAF, giai đoạn nào xác định khoảng trống năng lực trên 6 perspective và tạo action plan?
- A. Envision
- B. Align
- C. Launch
- D. Scale

<details><summary>Đáp án</summary>

**B.** Align.
- A sai: xác định cơ hội chuyển đổi.
- C sai: chạy pilot.
- D sai: mở rộng pilot.
</details>

**Câu 6.** Công ty muốn mua một firewall của hãng bảo mật bên thứ ba dưới dạng AMI và thanh toán chung trong hóa đơn AWS. Dùng gì?
- A. AWS Marketplace
- B. AWS Partner Network
- C. AWS re:Post
- D. AWS Professional Services

<details><summary>Đáp án</summary>

**A.** Marketplace bán phần mềm bên thứ ba, tính vào hóa đơn AWS.
- B sai: mạng lưới đối tác, không phải nơi mua phần mềm trực tiếp.
- C sai: cộng đồng Q&A.
- D sai: dịch vụ chuyên gia.
</details>

**Câu 7.** (Chọn 2) Hoạt động nào hỗ trợ pillar **Sustainability**?
- A. Right size workload để tối đa hóa mức sử dụng
- B. Dùng managed service và serverless
- C. Luôn chọn instance lớn nhất để an toàn
- D. Giữ server chạy 24/7 kể cả khi không dùng
- E. Lưu mọi dữ liệu ở S3 Standard mãi mãi

<details><summary>Đáp án</summary>

**A, B.**
- C, D sai: lãng phí tài nguyên.
- E sai: nên chuyển dữ liệu ít dùng sang class lạnh (Glacier, Intelligent-Tiering).
</details>

**Câu 8.** Developer có câu hỏi kỹ thuật không gấp, muốn hỏi cộng đồng và chuyên gia AWS miễn phí. Dùng gì?
- A. AWS re:Post
- B. Technical Account Manager
- C. AWS Managed Services
- D. AWS Artifact

<details><summary>Đáp án</summary>

**A.** re:Post là Q&A cộng đồng, thay AWS Forums.
- B sai: chỉ có ở Enterprise trở lên.
- C sai: dịch vụ vận hành trả phí.
- D sai: báo cáo compliance.
</details>

---

### Nguồn đã đối chiếu
- The pillars of the Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html
- Your cloud transformation journey (CAF phases): https://docs.aws.amazon.com/whitepapers/latest/overview-aws-cloud-adoption-framework/your-cloud-transformation-journey.html
- AWS CAF overview (perspectives): https://docs.aws.amazon.com/whitepapers/latest/public-sector-cloud-transformation/aws-cloud-adoption-framework-overview.html
- AWS Support Plans (gói hiện hành, end of support gói cũ): https://docs.aws.amazon.com/awssupport/latest/user/aws-support-plans.html
- CLF-C02 out-of-scope services (AWS Managed Services): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
- AWS re:Post: https://repost.aws/
- Customer Carbon Footprint Tool: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/what-is-ccft.html
