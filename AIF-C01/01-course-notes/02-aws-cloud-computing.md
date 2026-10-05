# AIF-C01 — AWS & Cloud Computing

> Chương nền tảng cloud, không phải domain chính của AIF-C01, nhưng là "ngôn ngữ chung" cho các câu hỏi về chi phí, Region, bảo mật (Domain 5) và chọn mức độ managed của dịch vụ AI.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Cloud computing** = thuê tài nguyên IT (compute, storage, database, app...) **on-demand**, trả **pay-as-you-go**; AWS lo phần cứng, bạn chỉ provision và dùng.
- Lợi ích cốt lõi: **CAPEX → OPEX**, không còn đoán capacity, scale theo nhu cầu thật, go global trong vài phút.
- Hạ tầng toàn cầu: **Region** (cụm data center) ⊃ **Availability Zone** (≥ 3 AZ mỗi Region) + **Edge Locations / PoP** (đưa nội dung gần người dùng).
- Chọn Region theo 4 tiêu chí: **Compliance → Latency → Service availability → Pricing**. Với GenAI, "service availability" chính là **model có trong Region đó không**.
- Mức độ managed: **IaaS** (EC2) → **PaaS** (Elastic Beanstalk) → **SaaS** (Rekognition). Càng lên cao, bạn quản lý càng ít.
- **Shared Responsibility Model:** AWS chịu security **OF** the cloud, khách hàng chịu security **IN** the cloud. Với Bedrock, data, IAM và cấu hình là việc của bạn.
- 3 thứ AWS tính tiền cơ bản: **compute, storage, data transfer OUT**; data transfer IN miễn phí.

---

## 2. Core Concepts

### Traditional IT vs Cloud
- **Là gì:** tự xây data center thì phải trả tiền thuê, điện, làm mát, bảo trì, thuê đội vận hành 24/7, tự lo thảm họa; cloud chuyển hết những việc đó cho nhà cung cấp.
- **Dùng khi nào:** khi cần ra sản phẩm nhanh, tải biến động, hoặc không muốn đầu tư phần cứng trước.
- **Vì sao quan trọng:** đề hỏi "lợi ích của cloud" thì đáp án luôn xoay quanh: không trả trước, không đoán capacity, scale nhanh.

### Server components & networking cơ bản
- **Là gì:** một server gồm Compute (CPU), Memory (RAM), Storage, Database, Network; client và server liên lạc qua địa chỉ IP.
- **Dùng khi nào:** router chuyển gói tin **giữa các mạng**, switch chuyển gói tin tới đúng máy **trong cùng mạng**.
- **Vì sao quan trọng:** chỉ là kiến thức nền, AIF-C01 hầu như không hỏi trực tiếp.

### Deployment models: Private / Public / Hybrid
- **Là gì:** Private = cloud riêng của một tổ chức; Public = hạ tầng của bên thứ ba (AWS) qua Internet; Hybrid = giữ một phần on-premises và mở rộng một phần lên cloud.
- **Dùng khi nào:** Private khi cần kiểm soát tuyệt đối; Hybrid khi dữ liệu nhạy cảm phải ở lại on-prem nhưng vẫn muốn sự linh hoạt của public cloud.
- **Vì sao quan trọng:** đề cho bối cảnh "một phần chạy on-premises, một phần trên AWS" thì đáp án là **Hybrid**.

### Five Characteristics of Cloud Computing
- **Là gì:** On-demand self service, Broad network access, Multi-tenancy & resource pooling, Rapid elasticity & scalability, Measured service.
- **Dùng khi nào:** dùng để nhận diện một dịch vụ có đúng là "cloud" hay không (tự phục vụ, đo lường, chia sẻ tài nguyên).
- **Vì sao quan trọng:** hay bị trộn lẫn với "Six Advantages", xem bẫy ở mục 4.

### Six Advantages of Cloud Computing
- **Là gì:** (1) Trade CAPEX for OPEX, (2) Economies of scale, (3) Stop guessing capacity, (4) Increase speed & agility, (5) Stop spending money running data centers, (6) Go global in minutes.
- **Dùng khi nào:** đề mô tả lợi ích (vd: "không phải mua server trước") và hỏi đó là advantage nào.
- **Vì sao quan trọng:** đây là danh sách chính thức của AWS, đề trích gần như nguyên câu.

### Problems solved by the Cloud
- **Là gì:** Flexibility, Cost-effectiveness, Scalability, Elasticity, High availability & fault tolerance, Agility.
- **Dùng khi nào:** Scalability = chịu được tải lớn hơn (**scale up**: máy mạnh hơn, **scale out**: thêm node); Elasticity = **tự động** scale out/in theo nhu cầu.
- **Vì sao quan trọng:** cặp scalability vs elasticity là bẫy kinh điển.

### Types of Cloud Computing: IaaS / PaaS / SaaS
- **Là gì:** IaaS cho building blocks (network, máy ảo, storage); PaaS lo hạ tầng, bạn chỉ deploy app; SaaS là sản phẩm hoàn chỉnh do nhà cung cấp vận hành.
- **Dùng khi nào:** IaaS = Amazon EC2; PaaS = AWS Elastic Beanstalk; SaaS = nhiều dịch vụ AI của AWS (vd: Amazon Rekognition), Gmail, Dropbox, Zoom. Slide xếp AWS Lambda là **FaaS**.
- **Vì sao quan trọng:** càng lên cao, bạn quản lý càng ít tầng (on-prem tự lo hết; SaaS chỉ dùng). Xem bảng mục 4.

### Pricing fundamentals
- **Là gì:** pay-as-you-go cho 3 thứ: **compute** (thời gian chạy), **storage** (dung lượng lưu), **data transfer OUT** khỏi cloud.
- **Dùng khi nào:** data transfer **IN** vào AWS miễn phí; chi phí "bất ngờ" thường đến từ data transfer OUT.
- **Vì sao quan trọng:** đề có thể hỏi "điều gì **không** bị tính phí", đáp án là data transfer in.

### `[+]` Pricing của GenAI trên Amazon Bedrock
- **Là gì:** on-demand tính theo **input + output tokens**; có các service tier Standard, Priority (nhanh hơn, đắt hơn), Flex (rẻ hơn, chậm hơn), Reserved (cam kết throughput).
- **Dùng khi nào:** batch inference cho khối lượng lớn không gấp; Provisioned Throughput dành cho custom/imported model.
- **Vì sao quan trọng:** pay-as-you-go của cloud áp vào AI chính là "trả theo token". Đề hay hỏi cách giảm chi phí inference.

### AWS Cloud history & use cases
- **Là gì:** AWS chạy nội bộ từ 2002; ra mắt công khai 2004 với SQS; 2006 tái ra mắt với SQS, S3, EC2; 2007 mở sang châu Âu.
- **Dùng khi nào:** use cases gồm Enterprise IT, backup & storage, big data analytics, web hosting, mobile & social apps, gaming. Netflix và Dropbox (ban đầu) chạy trên AWS.
- **Vì sao quan trọng:** gần như không thi. Số liệu thị trường trong slide (doanh thu 2023, thị phần Q1/2024) đã cũ và cũng không thi, nên bỏ qua.

### AWS Regions
- **Là gì:** cụm data center ở một khu vực địa lý (vd: `us-east-1`, `eu-west-3`); các Region tách biệt với nhau và hầu hết dịch vụ là **region-scoped**.
- **Dùng khi nào:** chọn theo thứ tự **Compliance** (data không rời Region nếu bạn không cho phép) → **Latency** (gần khách hàng) → **Service availability** → **Pricing** (giá khác nhau giữa các Region).
- **Vì sao quan trọng:** đề hay cho tình huống có ràng buộc pháp lý thì **compliance luôn thắng** giá rẻ hay latency.

### Availability Zones (AZ)
- **Là gì:** mỗi Region có **tối thiểu 3 AZ**; mỗi AZ là một hoặc nhiều data center riêng biệt với power, networking, connectivity dự phòng. Slide ghi tối đa 6 ❓ cần kiểm chứng (docs AWS chỉ nêu mức tối thiểu).
- **Dùng khi nào:** deploy app trên **nhiều AZ** để vẫn chạy khi một AZ gặp sự cố (cháy, mất điện, lũ).
- **Vì sao quan trọng:** các AZ cách ly vật lý nhưng nối với nhau bằng mạng băng thông cao, latency rất thấp. Đó là nền tảng của high availability.

### Edge Locations / Points of Presence
- ⚠️ Lỗi thời: số liệu "400+ PoP, 10+ Regional Caches, 90+ thành phố, 40+ quốc gia" → nay CloudFront có **750+ PoP** ở 100+ thành phố, 50+ quốc gia, và **15 Regional Edge Caches**.
- **Là gì:** điểm phân phối nội dung gần người dùng cuối, dùng bởi CloudFront (CDN).
- **Dùng khi nào:** user ở khắp thế giới cần tải nội dung với **latency thấp**.

### Global vs Region-scoped services
- **Là gì:** global: IAM, Route 53 (DNS), CloudFront (CDN), WAF; region-scoped: EC2, Elastic Beanstalk, Lambda, Rekognition (và hầu hết các dịch vụ khác).
- **Dùng khi nào:** WAF thực ra có **2 scope**: global khi gắn với CloudFront, regional khi gắn với ALB, API Gateway... Slide gộp chung là "global".
- **Vì sao quan trọng:** dịch vụ AI (Bedrock, SageMaker AI, Rekognition...) đều theo Region, nên phải kiểm tra Region có hỗ trợ không.

### `[+]` Region & Bedrock cross-Region inference
- **Là gì:** không phải model Bedrock nào cũng có ở mọi Region; **cross-Region inference** tự định tuyến request sang Region khác để tăng throughput và độ sẵn sàng.
- **Dùng khi nào:** **Geographic** CRI giữ dữ liệu trong một vùng địa lý (US, EU, APAC) nên hợp với data residency; **Global** CRI định tuyến toàn cầu, rẻ hơn khoảng 10%.
- **Vì sao quan trọng:** kết hợp trực tiếp tiêu chí "compliance" khi chọn Region với GenAI. Có ràng buộc residency thì chọn Geographic.

### Shared Responsibility Model
- **Là gì:** AWS chịu trách nhiệm security **OF** the cloud (hạ tầng, phần cứng, data center); khách hàng chịu security **IN** the cloud (data, IAM, cấu hình).
- **Dùng khi nào:** phân vai: cái gì bạn cấu hình hoặc đưa lên thì là trách nhiệm của bạn.
- **Vì sao quan trọng:** xuất hiện ở mọi kỳ thi AWS, kể cả AIF-C01 Domain 5.

### `[+]` Shared Responsibility với dịch vụ AI (Bedrock)
- **Là gì:** AWS vận hành hạ tầng và nơi host model; model provider **không truy cập** được prompt/completion của bạn.
- **Dùng khi nào:** khách hàng chịu trách nhiệm IAM (ai được gọi model), phân loại và bảo vệ data đưa vào prompt, cấu hình Guardrails, data retention, và tuân thủ pháp lý.
- **Vì sao quan trọng:** ❓ cần kiểm chứng: tài liệu cũ hay ghi "Bedrock không lưu prompt"; nay Bedrock có cấu hình **data retention theo Region**, mức mặc định tùy model.

### `[+]` Mức độ managed khi chọn dịch vụ AI
- **Là gì:** AI services có sẵn (Rekognition, Comprehend...) và Bedrock (serverless) gần SaaS; SageMaker AI cho bạn tự train/deploy nên nhiều quyền kiểm soát hơn và nhiều việc vận hành hơn.
- **Dùng khi nào:** đề nói "**least operational overhead**" thì chọn dịch vụ managed/serverless cao nhất đáp ứng được yêu cầu.
- **Vì sao quan trọng:** áp tư duy IaaS/PaaS/SaaS vào AI là cách loại đáp án nhanh nhất.

### AWS Acceptable Use Policy (AUP)
- **Là gì:** quy định cấm dùng AWS cho nội dung phạm pháp, gây hại, xúc phạm; cấm vi phạm bảo mật, lạm dụng mạng, spam email/tin nhắn.
- **Dùng khi nào:** khi đề hỏi hành vi nào bị AWS cấm trên hạ tầng của họ.
- **Vì sao quan trọng:** ít hỏi, chỉ cần nhận diện được tên.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Cloud computing | Cung cấp tài nguyên IT on-demand qua Internet, pay-as-you-go | "on-demand delivery", "over the internet" |
| Pay-as-you-go | Chỉ trả cho phần đã dùng, không cam kết trước | "pay only for what you use" |
| CAPEX | Chi phí đầu tư trả trước (mua server, xây data center) | "upfront investment", "purchase hardware" |
| OPEX | Chi phí vận hành trả theo mức dùng | "variable expense", "operational expense" |
| TCO (Total Cost of Ownership) | Tổng chi phí sở hữu, gồm cả điện, nhân sự, bảo trì | "reduce total cost", "TCO" |
| Economies of scale | AWS mua và vận hành ở quy mô cực lớn nên giá rẻ hơn | "lower prices due to scale" |
| On-demand self service | Tự provision tài nguyên, không cần người của nhà cung cấp can thiệp | "without human interaction", "self-provision" |
| Broad network access | Truy cập tài nguyên qua mạng từ nhiều loại thiết bị | "access from any device" |
| Multi-tenancy / Resource pooling | Nhiều khách hàng dùng chung hạ tầng vật lý nhưng vẫn tách biệt bảo mật | "shared infrastructure", "multiple customers" |
| Measured service | Mức dùng được đo, trả đúng phần đã dùng | "usage is metered" |
| Scalability (scale up / scale out) | Khả năng chịu tải lớn hơn: máy mạnh hơn (vertical) hoặc thêm máy (horizontal) | "handle larger load", "add more nodes" |
| Elasticity | **Tự động** thêm/bớt tài nguyên theo nhu cầu (scale out **và** scale in) | "automatically", "scale in when demand drops" |
| High availability | Hệ thống vẫn chạy khi một phần hạ tầng (vd: 1 AZ) hỏng | "remain available", "multiple AZs" |
| Fault tolerance | Chịu lỗi mà không gián đoạn dịch vụ | "no downtime when a component fails" |
| Agility | Phát triển, thử nghiệm, ra mắt nhanh | "experiment quickly", "faster time to market" |
| IaaS | Hạ tầng thô: VM, network, storage; linh hoạt nhất | "full control over OS", Amazon EC2 |
| PaaS | Nền tảng chạy app, không quản lý hạ tầng | "just deploy code", Elastic Beanstalk |
| SaaS | Sản phẩm hoàn chỉnh, chỉ việc dùng | "ready-to-use", "no management", Rekognition |
| Private / Public / Hybrid cloud | Riêng một tổ chức / của bên thứ ba qua Internet / kết hợp on-prem + cloud | "on-premises and AWS" thì là Hybrid |
| Region | Cụm data center tại một khu vực địa lý | "data residency", "latency to users" |
| Availability Zone (AZ) | 1+ data center độc lập trong một Region | "isolated from failures", "Multi-AZ" |
| Edge Location / PoP | Điểm cache/phân phối nội dung gần user, dùng bởi CloudFront | "global users", "low latency content delivery" |
| Global service | Không gắn với Region: IAM, Route 53, CloudFront | "not region-specific" |
| Shared Responsibility Model | AWS: security **of** the cloud; khách hàng: security **in** the cloud | "who is responsible for..." |
| Data transfer IN / OUT | IN vào AWS miễn phí; OUT ra ngoài bị tính phí | "which is free of charge" |
| `[+]` Token (Bedrock pricing) | Đơn vị tính phí on-demand của FM: input + output tokens | "cost per request depends on prompt length" |
| `[+]` Cross-Region inference | Bedrock định tuyến inference sang Region khác (Geographic hoặc Global) | "higher throughput", "data residency within EU/US" |
| Acceptable Use Policy (AUP) | Quy định hành vi bị cấm khi dùng AWS | "prohibited use", "spam", "network abuse" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Five Characteristics | Six Advantages | Characteristics mô tả **cloud là gì** (self service, measured...); Advantages là **lợi ích** cho doanh nghiệp (CAPEX→OPEX, go global...) | "CAPEX→OPEX" hay "go global" **không** phải characteristic |
| Scalability | Elasticity | Scalability = **có khả năng** chịu tải lớn; Elasticity = **tự động** tăng **và giảm** theo nhu cầu | Có chữ "automatically" + "scale in" thì là Elasticity |
| Scale up (vertical) | Scale out (horizontal) | Up = nâng cấp một máy mạnh hơn; Out = thêm nhiều máy | Out = thêm node, nền tảng của HA và elasticity |
| High availability | Fault tolerance | HA = vẫn chạy, có thể gián đoạn ngắn; FT = không gián đoạn khi có lỗi | FT nghiêm ngặt hơn HA |
| IaaS | PaaS | IaaS: bạn quản lý OS, runtime, app; PaaS: chỉ quản lý app + data | EC2 = IaaS; Elastic Beanstalk = PaaS |
| PaaS | SaaS | PaaS: bạn vẫn viết và deploy app; SaaS: dùng sản phẩm có sẵn | Gọi API Rekognition = dùng SaaS |
| Region | Availability Zone | Region = vùng địa lý (chọn vì compliance/latency); AZ = data center cách ly **trong** Region (chọn để HA) | Sự cố 1 data center → Multi-AZ; thảm họa cả vùng → Multi-Region |
| Availability Zone | Edge Location | AZ chạy workload (EC2, DB...); Edge chỉ cache/phân phối nội dung (CloudFront) | "Low latency cho user toàn cầu" thì là Edge, không phải AZ |
| Private cloud | Hybrid cloud | Private: chỉ hạ tầng riêng; Hybrid: on-prem **kết hợp** public cloud | Có "extend to AWS" thì là Hybrid |
| Security OF the cloud | Security IN the cloud | OF = AWS (phần cứng, data center, hạ tầng host model); IN = khách hàng (data, IAM, cấu hình, Guardrails) | Thứ bạn cấu hình hoặc upload là của bạn |
| Global service | Regional service | IAM, Route 53, CloudFront là global; EC2, Lambda, Bedrock, SageMaker AI theo Region | WAF có cả 2 scope (CloudFront = global; ALB/API Gateway = regional) |
| `[+]` Geographic CRI | `[+]` Global CRI | Geographic giữ data trong vùng (US/EU/APAC); Global định tuyến toàn cầu, rẻ hơn khoảng 10% | Residency → Geographic; tối ưu chi phí, không ràng buộc → Global |
| `[+]` Bedrock (serverless) | `[+]` SageMaker AI | Bedrock: gọi FM qua API, không quản hạ tầng; SageMaker AI: tự train/host model, nhiều quyền kiểm soát hơn | "Least operational overhead" + GenAI → Bedrock |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "no upfront investment", "avoid buying hardware" | Trade CAPEX for OPEX | Six Advantages |
| "don't know how much capacity is needed" | Stop guessing capacity | Six Advantages |
| "lower prices because AWS is large" | Economies of scale | Six Advantages |
| "deploy worldwide quickly" | Go global in minutes | Six Advantages |
| "automatically add/remove resources based on demand" | Elasticity | Không phải scalability |
| "survive a data center failure" | Deploy across multiple AZs | |
| "survive a regional disaster" | Multi-Region | |
| "data must stay in country/EU", "regulation" | Chọn Region theo compliance | Compliance là tiêu chí đầu tiên |
| "users worldwide, low latency static content" | Edge Locations (CloudFront) | |
| "full control of OS / virtual machines" | IaaS (Amazon EC2) | |
| "deploy code, don't manage servers/OS" | PaaS (Elastic Beanstalk) | |
| "ready-to-use API, no infrastructure" | SaaS / managed AI service | vd: Rekognition |
| "some workloads on-premises, some on AWS" | Hybrid cloud | |
| "which is free" | Data transfer IN | OUT bị tính phí |
| "who patches physical hosts / secures data center" | AWS | Security OF the cloud |
| "who controls IAM permissions / data in prompts" | Customer | Security IN the cloud |
| `[+]` "GenAI, least operational overhead" | Amazon Bedrock | Serverless |
| `[+]` "Bedrock model not in our Region / need more throughput + EU residency" | Geographic cross-Region inference | |
| `[+]` "large volume, not time-sensitive, lowest cost inference" | Batch inference / Flex tier | |
| `[+]` "guaranteed throughput for production" | Reserved tier; Provisioned Throughput cho custom/imported model | Đề cũ thường đáp "Provisioned Throughput" |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một startup AI muốn ra mắt sản phẩm mà không phải mua GPU server trước; họ chỉ muốn trả tiền theo mức dùng thực tế. Đây là lợi ích nào của cloud?
- A. Go global in minutes
- B. Trade capital expense for operational expense
- C. Benefit from massive economies of scale
- D. Broad network access

<details><summary>Đáp án</summary>

**B.** Không mua phần cứng trước (CAPEX), chỉ trả theo mức dùng (OPEX).
- A sai: nói về triển khai toàn cầu nhanh.
- C sai: nói về giá rẻ nhờ quy mô của AWS.
- D sai: đây là một **characteristic**, không phải advantage.
</details>

**Câu 2.** (Chọn 2) Phát biểu nào thuộc **Five Characteristics of Cloud Computing**?
- A. On-demand self service
- B. Stop guessing capacity
- C. Measured service
- D. Go global in minutes
- E. Increase speed and agility

<details><summary>Đáp án</summary>

**A, C.**
- B, D, E sai: cả ba đều thuộc **Six Advantages**. Đây là bẫy trộn hai danh sách.
</details>

**Câu 3.** Một ngân hàng châu Âu dùng GenAI để tóm tắt hồ sơ khách hàng. Quy định yêu cầu dữ liệu không được xử lý ngoài EU. Tiêu chí nào phải ưu tiên **đầu tiên** khi chọn Region?
- A. Region có giá rẻ nhất
- B. Region gần đội phát triển nhất
- C. Compliance với yêu cầu data governance và pháp lý
- D. Region có nhiều Availability Zone nhất

<details><summary>Đáp án</summary>

**C.** Ràng buộc pháp lý là điều kiện bắt buộc; giá và latency chỉ cân nhắc sau khi đã đáp ứng compliance.
- A, B, D sai: đều là tiêu chí phụ, không được vi phạm yêu cầu pháp lý.
</details>

**Câu 4.** Tiếp câu 3: model ngân hàng cần thường xuyên bị giới hạn throughput ở Region chính. Họ muốn tăng throughput nhưng **vẫn giữ dữ liệu trong EU**. Nên dùng gì?
- A. Global cross-Region inference
- B. Geographic cross-Region inference (EU)
- C. Amazon CloudFront để cache câu trả lời
- D. Chuyển workload sang một Region ở US có nhiều capacity hơn

<details><summary>Đáp án</summary>

**B.** Geographic CRI định tuyến request giữa các Region **trong cùng vùng địa lý** (EU), vừa tăng throughput vừa giữ data residency.
- A sai: Global CRI có thể xử lý ở bất kỳ Region nào trên thế giới, vi phạm yêu cầu.
- C sai: CloudFront là CDN, không giải quyết throughput của model.
- D sai: vi phạm yêu cầu dữ liệu ở EU.
</details>

**Câu 5.** (Chọn 2) Khi dùng Amazon Bedrock, trách nhiệm nào thuộc về **khách hàng**?
- A. Bảo vệ vật lý data center nơi host model
- B. Cấp quyền IAM xác định ai được gọi model
- C. Bảo trì phần cứng GPU chạy inference
- D. Quyết định dữ liệu nhạy cảm nào được đưa vào prompt
- E. Vá lỗi hạ tầng mạng toàn cầu của AWS

<details><summary>Đáp án</summary>

**B, D.** Đây là security **IN** the cloud: quản lý truy cập và dữ liệu của chính bạn.
- A, C, E sai: thuộc security **OF** the cloud, do AWS chịu trách nhiệm.
</details>

**Câu 6.** Một website bán hàng tự động thêm server khi lượng truy cập tăng vọt trong đợt sale, rồi tự động bớt server khi hết sale để tiết kiệm chi phí. Đặc tính này gọi là gì?
- A. Scalability
- B. Elasticity
- C. Fault tolerance
- D. Agility

<details><summary>Đáp án</summary>

**B.** Elasticity là **tự động** scale out **và** scale in theo nhu cầu.
- A sai: scalability chỉ nói khả năng chịu tải lớn, không nhấn mạnh tự động co lại.
- C sai: fault tolerance là chịu lỗi.
- D sai: agility là tốc độ phát triển và ra mắt.
</details>

**Câu 7.** Một công ty muốn ứng dụng inference của họ vẫn hoạt động nếu một data center trong Region gặp sự cố mất điện. Cách nào phù hợp nhất?
- A. Dùng instance có cấu hình mạnh hơn
- B. Triển khai ứng dụng trên nhiều Availability Zone
- C. Cache kết quả tại Edge Locations
- D. Dùng nhiều AWS account trong cùng một AZ

<details><summary>Đáp án</summary>

**B.** Các AZ cách ly vật lý, nên một AZ hỏng thì các AZ khác vẫn phục vụ (high availability).
- A sai: scale up không giúp khi cả data center mất điện.
- C sai: Edge chỉ cache nội dung, không chạy inference workload.
- D sai: cùng một AZ thì vẫn chịu chung sự cố.
</details>

**Câu 8.** Một team muốn thêm tính năng tạo mô tả sản phẩm bằng GenAI với **ít công sức vận hành nhất**: không quản lý server, không tự host model. Lựa chọn nào phù hợp nhất?
- A. Tự cài model open-source trên Amazon EC2
- B. Train và host model trên Amazon SageMaker AI
- C. Gọi foundation model qua Amazon Bedrock
- D. Deploy model lên AWS Elastic Beanstalk

<details><summary>Đáp án</summary>

**C.** Bedrock là serverless: gọi FM qua API, AWS lo toàn bộ hạ tầng, tương đương mức SaaS.
- A sai: IaaS, phải tự quản OS, GPU, scaling.
- B sai: SageMaker AI cho nhiều quyền kiểm soát hơn nhưng nhiều việc vận hành hơn.
- D sai: PaaS cho web app, vẫn phải tự đóng gói và host model.
</details>

---

### Nguồn đã đối chiếu
- AWS Regions & Availability Zones: https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions-availability-zones.html
- AWS Global Infrastructure (≥ 3 AZ mỗi Region, 39 Region / 124 AZ tại thời điểm tra): https://aws.amazon.com/about-aws/global-infrastructure/
- CloudFront features (750+ PoP, 15 Regional Edge Caches): https://aws.amazon.com/cloudfront/features/
- AWS WAF protected resources (global vs regional): https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html
- AWS Elastic Beanstalk: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html
- Bedrock security & shared responsibility: https://docs.aws.amazon.com/bedrock/latest/userguide/security.html
- Bedrock data protection: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html
- Bedrock data retention: https://docs.aws.amazon.com/bedrock/latest/userguide/data-retention.html
- Bedrock cross-Region inference: https://docs.aws.amazon.com/bedrock/latest/userguide/cross-region-inference.html
- Bedrock pricing model & service tiers: https://docs.aws.amazon.com/prescriptive-guidance/latest/gen-ai-inference-architecture-and-best-practices-on-aws/amazon-bedrock.html
