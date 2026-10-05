# CLF-C02 — What is Cloud Computing?

> Chương nền của **Domain 1: Cloud Concepts (24%)**: lợi ích của cloud, mô hình triển khai, IaaS/PaaS/SaaS, hạ tầng toàn cầu và Shared Responsibility Model.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Cloud computing** = nhận tài nguyên IT (compute, storage, database, ứng dụng) **on-demand** qua Internet và trả **pay-as-you-go**. AWS sở hữu và vận hành phần cứng, bạn chỉ provision và dùng.
- **Six Advantages** (lợi ích cho doanh nghiệp) khác **Five Characteristics** (định nghĩa thế nào là cloud). Đề rất hay trộn hai danh sách này.
- **Deployment models:** Private (riêng một tổ chức), Public (AWS, qua Internet), Hybrid (on-prem + cloud).
- **IaaS → PaaS → SaaS:** càng lên cao bạn quản lý càng ít. EC2 = IaaS, Elastic Beanstalk = PaaS, Rekognition = SaaS.
- 3 thứ cơ bản bị tính tiền: **compute, storage, data transfer OUT**. Data transfer **IN** miễn phí.
- **Region** (cụm data center) chứa **≥ 3 Availability Zone**; **Edge Location / PoP** đưa nội dung tới gần người dùng.
- Chọn Region theo 4 tiêu chí: **Compliance → Latency → Service availability → Pricing**.
- **Shared Responsibility:** AWS chịu security **OF** the cloud, khách hàng chịu security **IN** the cloud.

---

## 2. Core Concepts

### Traditional IT vs Cloud
- **Là gì:** tự xây data center thì phải trả tiền thuê, điện, làm mát, bảo trì, đội vận hành 24/7, tự lo thảm họa; cloud chuyển hết sang nhà cung cấp.
- **Dùng khi nào:** cần ra sản phẩm nhanh, tải biến động, không muốn đầu tư phần cứng trước.
- **Vì sao quan trọng:** câu "lợi ích của cloud" luôn xoay quanh: không trả trước, không đoán capacity, scale nhanh.

### Server & networking cơ bản
- **Là gì:** server gồm Compute (CPU), Memory (RAM), Storage, Database, Network; client và server liên lạc qua địa chỉ IP.
- **Dùng khi nào:** **router** chuyển gói tin **giữa các mạng**; **switch** chuyển gói tin tới đúng máy **trong cùng mạng**.
- **Vì sao quan trọng:** kiến thức nền, đề hầu như không hỏi trực tiếp.

### Deployment models: Private / Public / Hybrid
- **Là gì:** Private = cloud riêng một tổ chức, kiểm soát toàn bộ; Public = hạ tầng của bên thứ ba qua Internet; Hybrid = giữ một phần on-prem, mở rộng một phần lên cloud.
- **Dùng khi nào:** Hybrid khi dữ liệu nhạy cảm phải ở lại on-prem nhưng vẫn muốn sự linh hoạt và chi phí của public cloud.
- **Vì sao quan trọng:** đề mô tả "một phần chạy on-premises, một phần trên AWS" thì đáp án là **Hybrid**.

### Five Characteristics of Cloud Computing
- **Là gì:** On-demand self service, Broad network access, Multi-tenancy & resource pooling, Rapid elasticity & scalability, Measured service.
- **Dùng khi nào:** để nhận diện một dịch vụ có đúng là cloud hay không (tự phục vụ, đo lường, chia sẻ tài nguyên).
- **Vì sao quan trọng:** hay bị trộn với Six Advantages, xem bẫy ở mục 4.

### Six Advantages of Cloud Computing
- **Là gì:** (1) Trade fixed expense (CAPEX) for variable expense (OPEX), (2) Economies of scale, (3) Stop guessing capacity, (4) Increase speed & agility, (5) Stop spending money running data centers, (6) Go global in minutes.
- **Dùng khi nào:** đề mô tả một lợi ích cụ thể và hỏi đó là advantage nào.
- **Vì sao quan trọng:** AWS whitepaper hiện ghi advantage (1) là **"Trade fixed expense for variable expense"**; slide ghi "CAPEX for OPEX", hai cách gọi cùng nghĩa.

### Problems solved by the Cloud
- **Là gì:** Flexibility, Cost-effectiveness, Scalability, Elasticity, High availability & fault tolerance, Agility.
- **Dùng khi nào:** Scalability = chịu được tải lớn hơn (scale up hoặc scale out); Elasticity = **tự động** scale out/in theo nhu cầu.
- **Vì sao quan trọng:** cặp scalability vs elasticity là bẫy kinh điển.

### Types of Cloud Computing: IaaS / PaaS / SaaS
- **Là gì:** IaaS = building blocks (network, VM, storage), linh hoạt nhất; PaaS = AWS lo hạ tầng, bạn deploy app; SaaS = sản phẩm hoàn chỉnh do nhà cung cấp vận hành.
- **Dùng khi nào:** IaaS: EC2; PaaS: Elastic Beanstalk, Heroku; SaaS: Rekognition, Gmail, Dropbox, Zoom.
- **Vì sao quan trọng:** đề hỏi "bạn còn phải quản lý gì" (OS, runtime, app, data). Xem bảng ở mục 4.

### Pricing fundamentals
- **Là gì:** pay-as-you-go cho **compute** (thời gian chạy), **storage** (dung lượng lưu), **data transfer OUT** khỏi cloud.
- **Dùng khi nào:** data transfer **IN** vào AWS miễn phí; chi phí bất ngờ thường đến từ data transfer OUT.
- **Vì sao quan trọng:** đề hay hỏi "cái gì **không** bị tính phí".

### `[+]` Cloud economics (Task 1.4 của exam guide)
- **Là gì:** so sánh **fixed cost** (on-prem: mua server, data center) với **variable cost** (cloud: trả theo dùng); bao gồm chi phí nhân sự, điện, làm mát, license.
- **Dùng khi nào:** đề hỏi lợi ích tài chính khi chuyển lên cloud, hoặc lựa chọn license **BYOL** vs license đã bao gồm.
- **Vì sao quan trọng:** exam guide CLF-C02 nêu rõ fixed vs variable cost, rightsizing, automation và economies of scale.

### AWS history, số liệu & use cases
- **Là gì:** AWS chạy nội bộ từ 2002; công khai năm 2004 với SQS; 2006 tái ra mắt với SQS, S3, EC2; 2007 sang châu Âu.
- **Dùng khi nào:** use cases gồm Enterprise IT, backup & storage, big data, web hosting, mobile & social app, gaming.
- **Vì sao quan trọng:** gần như không thi. ⚠️ Lỗi thời: doanh thu 2023 và thị phần Q1/2024 trong slide → số liệu thị trường thay đổi hằng quý, không thuộc nội dung thi.

### AWS Regions
- **Là gì:** cụm data center ở một khu vực địa lý (vd: `us-east-1`); hầu hết dịch vụ là **region-scoped**.
- **Dùng khi nào:** chọn theo **Compliance** (data không rời Region nếu bạn không cho phép) → **Latency** → **Service availability** → **Pricing**.
- **Vì sao quan trọng:** khi đề có ràng buộc pháp lý thì **compliance luôn thắng** giá rẻ hay latency.

### Availability Zones (AZ)
- **Là gì:** mỗi Region có **tối thiểu 3 AZ**; mỗi AZ là một hoặc nhiều data center riêng biệt có power, networking, connectivity dự phòng. Slide ghi "tối đa 6" ❓ cần kiểm chứng (docs chỉ nêu mức tối thiểu).
- **Dùng khi nào:** deploy trên **nhiều AZ** để app vẫn chạy khi một AZ gặp thảm họa.
- **Vì sao quan trọng:** các AZ cách ly vật lý nhưng nối với nhau bằng mạng băng thông cao, latency rất thấp. Đây là nền tảng của high availability.

### Edge Locations / Points of Presence
- ⚠️ Lỗi thời: "400+ PoP, 10+ Regional Caches, 90+ thành phố, 40+ quốc gia" → nay CloudFront có **750+ PoP** ở 100+ thành phố, 50+ quốc gia và **15 Regional Edge Caches**.
- **Là gì:** điểm phân phối nội dung gần người dùng, dùng bởi CloudFront (CDN), Global Accelerator, Route 53.
- **Dùng khi nào:** người dùng ở khắp thế giới cần tải nội dung với **latency thấp**.

### Global vs Region-scoped services
- **Là gì:** global: IAM, Route 53, CloudFront, WAF (khi gắn CloudFront); region-scoped: EC2, Elastic Beanstalk, Lambda, Rekognition...
- **Dùng khi nào:** WAF có **2 scope**: global với CloudFront, regional với ALB/API Gateway. Slide gộp chung là global.
- **Vì sao quan trọng:** dùng Region Table để biết dịch vụ có ở Region nào.

### Shared Responsibility Model
- **Là gì:** AWS lo security **OF** the cloud (phần cứng, data center, mạng toàn cầu, hạ tầng dịch vụ); khách hàng lo security **IN** the cloud (data, IAM, cấu hình, OS của EC2).
- **Dùng khi nào:** phân vai: thứ bạn cấu hình hoặc đưa lên là trách nhiệm của bạn.
- **Vì sao quan trọng:** xuất hiện xuyên suốt các chương; chi tiết theo từng dịch vụ ở chương 14.

### AWS Acceptable Use Policy (AUP)
- **Là gì:** cấm dùng AWS cho nội dung phạm pháp/gây hại, vi phạm bảo mật, lạm dụng mạng, spam email/tin nhắn.
- **Dùng khi nào:** đề hỏi hành vi nào bị AWS cấm trên hạ tầng của họ.
- **Vì sao quan trọng:** ít hỏi; liên quan tới AWS Abuse (chương 14).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Cloud computing | Cung cấp tài nguyên IT on-demand qua Internet, pay-as-you-go | "on-demand delivery", "over the internet" |
| Pay-as-you-go | Chỉ trả cho phần đã dùng, không cam kết trước | "pay only for what you use" |
| CAPEX / fixed expense | Chi phí đầu tư trả trước (mua server, xây data center) | "upfront investment", "purchase hardware" |
| OPEX / variable expense | Chi phí vận hành trả theo mức dùng | "variable expense", "operational expense" |
| TCO | Tổng chi phí sở hữu, gồm điện, nhân sự, bảo trì | "reduce total cost of ownership" |
| Economies of scale | AWS mua và vận hành ở quy mô cực lớn nên giá rẻ hơn | "lower prices due to aggregated usage" |
| On-demand self service | Tự provision, không cần người của nhà cung cấp can thiệp | "without human interaction" |
| Broad network access | Truy cập qua mạng từ nhiều loại thiết bị | "access from any device" |
| Multi-tenancy / resource pooling | Nhiều khách hàng dùng chung hạ tầng vật lý nhưng vẫn tách biệt | "shared infrastructure", "multiple customers" |
| Measured service | Mức dùng được đo, trả đúng phần đã dùng | "usage is metered" |
| Scalability | Khả năng chịu tải lớn hơn: scale up (vertical) hoặc scale out (horizontal) | "handle greater load" |
| Elasticity | **Tự động** thêm/bớt tài nguyên theo nhu cầu | "automatically", "scale in when demand drops" |
| High availability | Vẫn chạy khi một phần hạ tầng (vd: 1 AZ) hỏng | "multiple AZs", "remain available" |
| Fault tolerance | Chịu lỗi mà không gián đoạn | "no downtime when a component fails" |
| Agility | Tài nguyên mới chỉ cách vài cú click, phát triển và thử nghiệm nhanh | "experiment quickly", "weeks to minutes" |
| IaaS | Hạ tầng thô: VM, network, storage | "full control over OS", EC2 |
| PaaS | Nền tảng chạy app, không quản lý hạ tầng | "just upload code", Elastic Beanstalk |
| SaaS | Sản phẩm hoàn chỉnh, chỉ việc dùng | "ready-to-use software" |
| Private / Public / Hybrid cloud | Riêng một tổ chức / bên thứ ba qua Internet / kết hợp | "on-premises and AWS" → Hybrid |
| Region | Cụm data center ở một khu vực địa lý | "data residency", "latency to users" |
| Availability Zone | 1+ data center độc lập trong một Region | "isolated from failures", "Multi-AZ" |
| Edge Location / PoP | Điểm cache/phân phối nội dung gần user | "global users", "content delivery" |
| Data transfer IN / OUT | IN miễn phí; OUT bị tính phí | "which is free of charge" |
| Shared Responsibility Model | AWS: security **of** the cloud; khách: security **in** the cloud | "who is responsible for" |
| Acceptable Use Policy | Quy định hành vi bị cấm trên AWS | "prohibited use", "spam" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Five Characteristics | Six Advantages | Characteristics = cloud **là gì**; Advantages = **lợi ích** cho doanh nghiệp | "CAPEX→OPEX", "go global" **không** phải characteristic |
| Scalability | Elasticity | Scalability = **có khả năng** chịu tải lớn; Elasticity = **tự động** tăng **và giảm** | Có "automatically" + "scale in" thì là Elasticity |
| Elasticity | Agility | Elasticity = tự co giãn theo tải; Agility = tạo tài nguyên mới nhanh, rút thời gian từ tuần xuống phút | Slide ghi rõ Agility là **distractor**, không liên quan scaling |
| Scale up (vertical) | Scale out (horizontal) | Up = máy mạnh hơn; Out = thêm máy | Out là nền tảng của HA |
| High availability | Fault tolerance | HA = vẫn chạy, có thể gián đoạn ngắn; FT = không gián đoạn | FT nghiêm ngặt hơn |
| IaaS | PaaS | IaaS: bạn quản OS, runtime, app; PaaS: chỉ app + data | EC2 = IaaS; Beanstalk = PaaS |
| PaaS | SaaS | PaaS: bạn vẫn viết, deploy app; SaaS: dùng sản phẩm có sẵn | Gmail, Zoom, Rekognition = SaaS |
| Region | Availability Zone | Region = vùng địa lý (chọn vì compliance/latency); AZ = data center cách ly **trong** Region (chọn để HA) | 1 data center hỏng → Multi-AZ; cả vùng → Multi-Region |
| Availability Zone | Edge Location | AZ chạy workload; Edge cache/phân phối nội dung | "Low latency cho user toàn cầu" → Edge |
| Private cloud | Hybrid cloud | Private: chỉ hạ tầng riêng; Hybrid: on-prem **kết hợp** public cloud | Có "extend to AWS" → Hybrid |
| Security OF the cloud | Security IN the cloud | OF = AWS (phần cứng, data center); IN = khách (data, IAM, OS EC2, cấu hình) | Thứ bạn cấu hình là của bạn |
| Data transfer IN | Data transfer OUT | IN free; OUT tính phí | "Upload lên AWS" không mất phí truyền |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "no upfront investment", "avoid buying hardware" | Trade fixed (CAPEX) for variable expense (OPEX) | Six Advantages |
| "don't know how much capacity is needed" | Stop guessing capacity | Six Advantages |
| "lower prices because AWS is large" | Benefit from massive economies of scale | Six Advantages |
| "deploy worldwide in minutes" | Go global in minutes | Six Advantages |
| "focus on customers, not racking servers" | Stop spending money running and maintaining data centers | Six Advantages |
| "provision resources without contacting provider" | On-demand self service | Five Characteristics |
| "automatically add/remove resources" | Elasticity | |
| "new resources in minutes instead of weeks" | Agility | |
| "survive a data center failure" | Multi-AZ | |
| "survive a regional disaster" | Multi-Region | |
| "data must stay in country" | Chọn Region theo compliance | Tiêu chí đầu tiên |
| "full control of OS" | IaaS (EC2) | |
| "deploy code, don't manage servers" | PaaS (Elastic Beanstalk) | |
| "some workloads on-premises, some on AWS" | Hybrid cloud | |
| "which is free" | Data transfer IN | |
| "who patches physical hosts" | AWS | Security OF the cloud |
| "who configures firewall/IAM/data encryption" | Customer | Security IN the cloud |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một startup không muốn mua server trước, chỉ muốn trả theo mức dùng thực tế. Đây là lợi ích nào của AWS Cloud?
- A. Go global in minutes
- B. Trade fixed expense for variable expense
- C. Benefit from massive economies of scale
- D. Broad network access

<details><summary>Đáp án</summary>

**B.** Không đầu tư trước (fixed/CAPEX), chỉ trả theo mức dùng (variable/OPEX).
- A sai: nói về triển khai toàn cầu nhanh.
- C sai: nói về giá rẻ nhờ quy mô của AWS.
- D sai: là một **characteristic**, không phải advantage.
</details>

**Câu 2.** (Chọn 2) Phát biểu nào thuộc **Five Characteristics of Cloud Computing**?
- A. On-demand self service
- B. Stop guessing capacity
- C. Measured service
- D. Go global in minutes
- E. Increase speed and agility

<details><summary>Đáp án</summary>

**A, C.**
- B, D, E sai: đều thuộc **Six Advantages**. Đây là bẫy trộn hai danh sách.
</details>

**Câu 3.** Một ngân hàng ở EU phải đảm bảo dữ liệu khách hàng không rời khỏi EU. Tiêu chí nào cần ưu tiên **đầu tiên** khi chọn Region?
- A. Region có giá rẻ nhất
- B. Region gần đội phát triển nhất
- C. Compliance với yêu cầu data governance và pháp lý
- D. Region có nhiều dịch vụ mới nhất

<details><summary>Đáp án</summary>

**C.** Ràng buộc pháp lý là điều kiện bắt buộc; giá, latency, dịch vụ chỉ cân nhắc sau.
- A, B, D sai: đều là tiêu chí phụ, không được vi phạm yêu cầu pháp lý.
</details>

**Câu 4.** Website thương mại điện tử tự động thêm server khi lượng truy cập tăng trong đợt sale và tự động bớt server khi hết sale. Đặc tính này gọi là gì?
- A. Scalability
- B. Elasticity
- C. Agility
- D. Fault tolerance

<details><summary>Đáp án</summary>

**B.** Elasticity = **tự động** scale out **và** scale in theo nhu cầu.
- A sai: scalability chỉ nói khả năng chịu tải lớn hơn.
- C sai: agility là tốc độ có tài nguyên mới để phát triển, thử nghiệm.
- D sai: fault tolerance là chịu lỗi.
</details>

**Câu 5.** (Chọn 2) Theo Shared Responsibility Model, việc nào thuộc trách nhiệm của **AWS**?
- A. Bảo vệ vật lý các data center
- B. Cấu hình security group cho EC2
- C. Thay phần cứng hỏng trong hạ tầng
- D. Mã hóa dữ liệu ứng dụng của khách hàng
- E. Vá hệ điều hành của EC2 instance

<details><summary>Đáp án</summary>

**A, C.** Đây là security **OF** the cloud.
- B, D, E sai: khách hàng chịu trách nhiệm cấu hình firewall, mã hóa dữ liệu và vá guest OS của EC2.
</details>

**Câu 6.** Một công ty chỉ muốn upload code, không muốn quản lý OS, load balancer hay scaling. Mô hình nào phù hợp nhất?
- A. IaaS
- B. PaaS
- C. On-premises
- D. Private cloud

<details><summary>Đáp án</summary>

**B.** PaaS (vd: Elastic Beanstalk) lo hạ tầng và runtime, bạn chỉ quản lý app và data.
- A sai: IaaS bắt bạn quản lý OS và phần mềm.
- C, D sai: phải tự vận hành hạ tầng.
</details>

**Câu 7.** Ứng dụng phải tiếp tục chạy nếu một data center trong Region bị mất điện. Cách nào phù hợp nhất?
- A. Dùng instance cấu hình mạnh hơn
- B. Triển khai trên nhiều Availability Zone
- C. Cache nội dung tại Edge Locations
- D. Triển khai trên nhiều AWS account trong cùng một AZ

<details><summary>Đáp án</summary>

**B.** Các AZ cách ly vật lý; một AZ hỏng thì các AZ khác vẫn phục vụ.
- A sai: scale up không giúp khi cả data center mất điện.
- C sai: Edge chỉ cache nội dung, không chạy workload chính.
- D sai: cùng AZ vẫn chịu chung sự cố.
</details>

**Câu 8.** Một công ty giữ hệ thống lõi tại data center của mình nhưng chạy phân tích dữ liệu trên AWS. Đây là mô hình triển khai nào?
- A. Public cloud
- B. Private cloud
- C. Hybrid cloud
- D. Multi-Region

<details><summary>Đáp án</summary>

**C.** Kết hợp on-premises với public cloud là Hybrid.
- A sai: không chạy hoàn toàn trên public cloud.
- B sai: có dùng hạ tầng của AWS.
- D sai: Multi-Region là kiến trúc triển khai, không phải deployment model.
</details>

---

### Nguồn đã đối chiếu
- Six advantages of cloud computing: https://docs.aws.amazon.com/whitepapers/latest/aws-overview/six-advantages-of-cloud-computing.html
- CLF-C02 exam guide (domains & weighting): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/cloud-practitioner-02.html
- AWS Regions & Availability Zones: https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions-availability-zones.html
- CloudFront features (750+ PoP, 15 Regional Edge Caches): https://aws.amazon.com/cloudfront/features/
- AWS WAF protected resources (global vs regional): https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html
- Shared Responsibility Model: https://aws.amazon.com/compliance/shared-responsibility-model/
- AWS Acceptable Use Policy: https://aws.amazon.com/aup/
