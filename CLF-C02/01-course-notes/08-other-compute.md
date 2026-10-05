# CLF-C02 — Other Compute Services

> Thuộc **Domain 3** (compute, containers, serverless). Trọng tâm: phân biệt container (ECS, EKS, Fargate, ECR), serverless (Lambda, API Gateway), batch và Lightsail.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Docker** đóng gói app vào **container** chạy giống nhau ở mọi nơi; container chia sẻ OS của host nên nhẹ hơn VM.
- **ECR** = registry **private** lưu Docker image. **ECS** = chạy container trên **EC2 do bạn quản lý**. **Fargate** = chạy container **serverless**, không quản EC2. **EKS** = **Kubernetes** managed (cloud-agnostic).
- **Serverless** ≠ không có server: bạn **không quản lý, không provision, không nhìn thấy** server. Ví dụ: Lambda, Fargate, DynamoDB, S3.
- **Lambda** = Function as a Service: chạy theo **event**, tối đa **15 phút**, RAM tới **10 GB**, tính tiền theo **số request + thời gian chạy × RAM**.
- **API Gateway** = tạo, bảo mật, quản lý API (REST, WebSocket) serverless, thường đứng trước Lambda.
- **AWS Batch** = chạy batch job (có điểm đầu và điểm cuối) ở mọi quy mô, **không giới hạn thời gian**, job đóng gói bằng Docker.
- **Lightsail** = VPS giá **thấp và cố định**, đơn giản cho người ít kinh nghiệm cloud; không auto-scaling.

---

## 2. Core Concepts

### Docker & containers
- **Là gì:** nền tảng đóng gói app thành container chạy trên mọi OS, hành vi giống nhau ở mọi máy, không lỗi tương thích.
- **Dùng khi nào:** microservices, cần scale container lên/xuống trong vài giây, chạy được mọi ngôn ngữ và công nghệ.
- **Vì sao quan trọng:** khác VM: container dùng chung OS của host (qua Docker daemon) nên một server chạy được nhiều container; VM mỗi máy có Guest OS riêng trên hypervisor.

### Docker image repositories
- **Là gì:** nơi lưu Docker image: **public** = Docker Hub (base image Ubuntu, MySQL, NodeJS...); **private** = **Amazon ECR**.
- **Dùng khi nào:** lưu image để ECS hoặc Fargate kéo về chạy.
- **Vì sao quan trọng:** "private Docker registry on AWS" → ECR.

### Amazon ECS (Elastic Container Service)
- **Là gì:** chạy Docker container trên AWS; **bạn provision và bảo trì EC2 instance**, AWS lo start/stop container.
- **Dùng khi nào:** cần kiểm soát hạ tầng chạy container; tích hợp Application Load Balancer.
- **Vì sao quan trọng:** khác Fargate ở chỗ bạn vẫn quản lý EC2.

### AWS Fargate
- **Là gì:** chạy container **serverless**: không có EC2 để quản lý, AWS chạy container theo CPU/RAM bạn yêu cầu.
- **Dùng khi nào:** muốn chạy container đơn giản nhất, ít vận hành nhất.
- **Vì sao quan trọng:** Fargate là launch type cho cả ECS và EKS.

### Amazon EKS (Elastic Kubernetes Service)
- **Là gì:** dịch vụ **Kubernetes managed**; Kubernetes là hệ thống open-source quản lý, triển khai, scale container.
- **Dùng khi nào:** đã dùng Kubernetes hoặc cần nền tảng **cloud-agnostic** (chạy được trên Azure, GCP...); pod chạy trên EC2 hoặc Fargate.
- **Vì sao quan trọng:** từ khóa "Kubernetes" → EKS, không phải ECS.

### Serverless
- **Là gì:** mô hình mà developer chỉ deploy code/function, không quản lý server; ban đầu đồng nghĩa FaaS, nay gồm mọi thứ managed (database, messaging, storage).
- **Dùng khi nào:** muốn trả theo dùng, tự scale, không vận hành hạ tầng.
- **Vì sao quan trọng:** ví dụ trong khóa: S3, DynamoDB, Fargate, Lambda.

### AWS Lambda vs EC2
- **Là gì:** EC2 = server ảo, giới hạn bởi RAM/CPU, chạy liên tục, scale phải can thiệp; Lambda = function ảo, giới hạn bởi **thời gian**, chạy **on-demand**, **tự scale**.
- **Dùng khi nào:** Lambda cho tác vụ ngắn, theo event; EC2 cho workload chạy lâu, cần OS.
- **Vì sao quan trọng:** đề hay hỏi "chạy code khi có file mới upload lên S3 mà không quản server" → Lambda.

### Lợi ích của Lambda
- **Là gì:** pricing đơn giản (theo request + thời gian chạy), free tier 1.000.000 request và 400.000 GB-giây mỗi tháng; tích hợp toàn bộ AWS; **event-driven**.
- **Dùng khi nào:** monitoring dễ qua CloudWatch; RAM tới **10 GB**, tăng RAM thì CPU và network cũng tăng.
- **Vì sao quan trọng:** thời gian chạy tối đa **15 phút**.

### Lambda language support
- **Là gì:** Node.js, Python, Java, C#/.NET, PowerShell, Ruby; **Custom Runtime API** (vd: Rust, Go); **Lambda Container Image** (image phải implement Lambda Runtime API).
- **Dùng khi nào:** chạy Docker image tùy ý thì ưu tiên **ECS/Fargate**, không phải Lambda.
- **Vì sao quan trọng:** "Lambda chạy mọi ngôn ngữ trừ Docker image tùy ý".

### Lambda use cases
- **Là gì:** **Serverless thumbnail**: ảnh mới trong S3 kích hoạt Lambda tạo thumbnail, lưu S3 và ghi metadata vào DynamoDB.
- **Dùng khi nào:** **Serverless CRON**: EventBridge (CloudWatch Events) kích hoạt Lambda mỗi giờ.
- **Vì sao quan trọng:** hai mẫu kiến trúc kinh điển trong đề.

### Lambda pricing
- **Là gì:** trả theo **số lần gọi** (1 triệu đầu miễn phí, sau đó $0.20/1 triệu request) và **thời lượng** tính theo 1 ms (400.000 GB-giây miễn phí mỗi tháng, sau đó khoảng $1/600.000 GB-giây).
- **Dùng khi nào:** 400.000 GB-giây = 400.000 giây với function 1 GB RAM, hoặc 3.200.000 giây với 128 MB.
- **Vì sao quan trọng:** Lambda thường rất rẻ. Giá cụ thể có thể thay đổi, đề CLF chỉ hỏi **cách tính phí**, không hỏi con số.

### Amazon API Gateway
- **Là gì:** dịch vụ fully managed để tạo, publish, bảo trì, giám sát, bảo mật API; serverless, scalable.
- **Dùng khi nào:** xây serverless API: Client → API Gateway → Lambda → DynamoDB; hỗ trợ RESTful và WebSocket API.
- **Vì sao quan trọng:** có xác thực, throttling, API key, monitoring. "Expose Lambda as HTTP API" → API Gateway.

### AWS Batch
- **Là gì:** batch processing fully managed ở mọi quy mô (100.000+ job); batch job có **điểm bắt đầu và kết thúc**.
- **Dùng khi nào:** Batch tự launch EC2 hoặc **Spot** với lượng compute/RAM phù hợp; job định nghĩa bằng Docker image, chạy trên ECS, EKS, Fargate.
- **Vì sao quan trọng:** giúp tối ưu chi phí và giảm công quản lý hạ tầng.

### Batch vs Lambda
- **Là gì:** **Lambda**: giới hạn thời gian, runtime giới hạn, disk tạm giới hạn, serverless.
- **Dùng khi nào:** **Batch**: không giới hạn thời gian, runtime bất kỳ miễn đóng gói Docker, dùng EBS/instance store làm disk, chạy trên EC2 (có thể do AWS quản lý).
- **Vì sao quan trọng:** job chạy hàng giờ → Batch, không phải Lambda.

### Amazon Lightsail
- **Là gì:** gói server ảo, storage, database, networking với **giá thấp, dự đoán được**; thay thế đơn giản cho EC2, RDS, ELB, EBS, Route 53.
- **Dùng khi nào:** người ít kinh nghiệm cloud; web app đơn giản (template LAMP, Nginx, MEAN, Node.js), website (WordPress, Magento, Plesk, Joomla), môi trường dev/test.
- **Vì sao quan trọng:** có high availability nhưng **không auto-scaling**, tích hợp AWS hạn chế.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Container | App đóng gói kèm dependency, chạy giống nhau mọi nơi | "package application", "portable" |
| Docker Hub | Registry public cho Docker image | "public base images" |
| Amazon ECR | Registry private cho Docker image | "store private container images" |
| Amazon ECS | Chạy container, bạn quản EC2 | "container orchestration on EC2" |
| AWS Fargate | Chạy container serverless | "containers without managing servers" |
| Amazon EKS | Kubernetes managed | "Kubernetes", "cloud-agnostic" |
| Serverless | Không quản lý/provision/nhìn thấy server | "no servers to manage" |
| FaaS | Function as a Service | "Lambda" |
| AWS Lambda | Function chạy theo event, tối đa 15 phút | "event-driven", "run code on S3 upload" |
| GB-second | Đơn vị tính thời lượng Lambda = RAM × giây | "pricing by memory and duration" |
| Lambda Container Image | Image tuân theo Lambda Runtime API | "deploy Lambda as container" |
| Amazon API Gateway | Tạo và quản lý API serverless | "REST API", "WebSocket", "throttling" |
| AWS Batch | Batch job ở mọi quy mô, không giới hạn thời gian | "batch jobs", "long-running jobs" |
| Amazon Lightsail | VPS giá thấp, cố định, đơn giản | "simple website", "predictable pricing", "little cloud experience" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| ECS | Fargate | ECS (EC2 launch type): bạn quản EC2; Fargate: serverless | "No EC2 to manage" → Fargate |
| ECS | EKS | AWS-native vs Kubernetes | "Kubernetes" → EKS |
| ECR | ECS | ECR lưu image; ECS chạy container | Registry vs orchestration |
| Lambda | EC2 | Function ngắn, event, tự scale vs VM chạy liên tục | "Event-driven, short" → Lambda |
| Lambda | Batch | Tối đa 15 phút vs không giới hạn thời gian | "Job chạy 3 giờ" → Batch |
| Lambda | Fargate | Function code vs container tùy ý | "Arbitrary Docker image" → Fargate/ECS |
| Lightsail | EC2 | Đơn giản, giá cố định vs linh hoạt, nhiều tùy chọn | "Low, predictable pricing, beginners" → Lightsail |
| API Gateway | ELB | Quản lý API (auth, throttling, key) vs cân bằng tải | "Expose Lambda function as REST API" → API Gateway |
| Container | VM | Chia sẻ OS host vs Guest OS riêng | Nhẹ, khởi động nhanh → container |
| Serverless | "Không có server" | Server vẫn có, chỉ là bạn không quản lý | Bẫy định nghĩa |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "run containers without managing servers" | AWS Fargate | |
| "managed Kubernetes" | Amazon EKS | |
| "private Docker image repository" | Amazon ECR | |
| "run Docker containers, control EC2 instances" | Amazon ECS (EC2 launch type) | |
| "run code in response to events, no servers" | AWS Lambda | |
| "create thumbnails when image uploaded to S3" | S3 event → Lambda | |
| "run a script every hour serverless" | EventBridge schedule → Lambda | |
| "expose Lambda via HTTP / REST API" | Amazon API Gateway | |
| "batch jobs, hours long, Docker, Spot" | AWS Batch | |
| "simple WordPress site, fixed monthly price" | Amazon Lightsail | |
| "pay per request and compute duration" | Lambda pricing | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Team muốn chạy Docker container mà không phải provision hay quản lý EC2. Chọn gì?
- A. Amazon ECS với EC2 launch type
- B. AWS Fargate
- C. Amazon EC2 Auto Scaling
- D. Amazon Lightsail

<details><summary>Đáp án</summary>

**B.** Fargate chạy container serverless.
- A sai: phải quản EC2.
- C sai: scale EC2, không phải chạy container.
- D sai: VPS đơn giản.
</details>

**Câu 2.** Công ty đang dùng Kubernetes on-premises và muốn chuyển lên AWS với dịch vụ managed. Chọn gì?
- A. Amazon ECS
- B. Amazon EKS
- C. AWS Batch
- D. AWS Lambda

<details><summary>Đáp án</summary>

**B.** EKS là Kubernetes managed.
- A sai: orchestration riêng của AWS.
- C, D sai: không phải nền tảng Kubernetes.
</details>

**Câu 3.** Mỗi khi có ảnh mới upload lên S3, hệ thống cần tạo thumbnail, không quản lý server. Chọn gì?
- A. EC2 chạy cron job
- B. AWS Lambda kích hoạt bởi S3 event
- C. Amazon Lightsail
- D. AWS Batch chạy thủ công

<details><summary>Đáp án</summary>

**B.** Lambda event-driven, serverless, phù hợp tác vụ ngắn theo sự kiện.
- A sai: phải quản lý server, không theo event.
- C sai: VPS.
- D sai: thừa cho tác vụ ngắn, cần trigger thủ công.
</details>

**Câu 4.** Một job xử lý dữ liệu khoa học chạy 4 giờ mỗi lần, đóng gói bằng Docker, muốn tận dụng Spot để tiết kiệm. Chọn gì?
- A. AWS Lambda
- B. AWS Batch
- C. Amazon API Gateway
- D. Amazon Lightsail

<details><summary>Đáp án</summary>

**B.** Batch không giới hạn thời gian, chạy Docker, dùng được Spot.
- A sai: tối đa 15 phút.
- C sai: quản lý API.
- D sai: không phù hợp batch quy mô lớn.
</details>

**Câu 5.** (Chọn 2) Lambda tính phí dựa trên những yếu tố nào?
- A. Số lượng request
- B. Số EC2 instance đã provision
- C. Thời gian chạy nhân với lượng RAM cấp phát
- D. Số AZ sử dụng
- E. Dung lượng ECR

<details><summary>Đáp án</summary>

**A, C.**
- B sai: Lambda không có EC2 để provision.
- D, E sai: không phải yếu tố tính phí của Lambda.
</details>

**Câu 6.** Một người mới dùng cloud muốn dựng blog WordPress với chi phí hằng tháng thấp và dễ dự đoán. Chọn gì?
- A. Amazon EC2 + RDS + ELB
- B. Amazon Lightsail
- C. Amazon EKS
- D. AWS Batch

<details><summary>Đáp án</summary>

**B.** Lightsail có template WordPress, giá thấp, cố định, đơn giản.
- A sai: phức tạp hơn, chi phí khó dự đoán hơn.
- C, D sai: không dành cho trường hợp này.
</details>

**Câu 7.** Team cần expose các Lambda function thành REST API, có throttling và API key. Chọn gì?
- A. Application Load Balancer
- B. Amazon API Gateway
- C. Amazon CloudFront
- D. Amazon ECR

<details><summary>Đáp án</summary>

**B.** API Gateway tạo, bảo mật, throttling, quản lý API key.
- A sai: cân bằng tải, không quản lý API key/throttling như API Gateway.
- C sai: CDN.
- D sai: registry image.
</details>

**Câu 8.** (Chọn 2) Phát biểu nào về serverless là **đúng**?
- A. Serverless nghĩa là không có server vật lý nào
- B. Bạn không phải provision hay quản lý server
- C. Lambda và Fargate là ví dụ serverless
- D. Serverless chỉ áp dụng cho function, không áp dụng cho database
- E. Serverless luôn chạy liên tục 24/7

<details><summary>Đáp án</summary>

**B, C.**
- A sai: server vẫn tồn tại, chỉ là bạn không quản lý.
- D sai: DynamoDB, Aurora Serverless cũng là serverless.
- E sai: serverless chạy theo nhu cầu.
</details>

---

### Nguồn đã đối chiếu
- Lambda quotas (timeout 15 phút, memory): https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
- Lambda function timeout: https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html
- Lambda pricing: https://aws.amazon.com/lambda/pricing/
- AWS Fargate: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html
- AWS Batch: https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html
- Amazon Lightsail: https://docs.aws.amazon.com/lightsail/latest/userguide/what-is-amazon-lightsail.html
