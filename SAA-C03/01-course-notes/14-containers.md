# SAA-C03 — Containers on AWS: Docker, ECS, ECR, EKS & App Runner

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 3: Design High-Performing Architectures**: chạy container, chọn launch type, IAM role, scaling, registry, Kubernetes managed.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Docker**: đóng gói app + dependency thành image, chạy giống nhau mọi nơi; image lưu ở **ECR** (private/public) hoặc Docker Hub.
- **ECS** chạy container trên **EC2 launch type** (tự quản EC2, có ECS agent) hoặc **Fargate** (serverless, chỉ khai CPU/RAM). Mới: **ECS Managed Instances** (AWS quản EC2 hộ).
- IAM: **EC2 instance profile** (cho ECS agent gọi API, kéo ECR, ghi CloudWatch) và **task role** (quyền riêng mỗi task, gắn trong task definition).
- Tích hợp **ALB** (khuyến nghị, dynamic port), **NLB** (throughput cao, PrivateLink); dữ liệu bền dùng **EFS** (Fargate + EFS = serverless hoàn toàn). S3 **không** mount được làm file system cho task.
- **ECS Service Auto Scaling** theo CPU, RAM, ALB request count; trên EC2 cần thêm **capacity provider** (ASG) để scale instance.
- **EKS** = Kubernetes managed; node: **managed node group**, **self-managed**, **Fargate**, hoặc **EKS Auto Mode**; storage qua **CSI driver** (EBS, EFS, FSx).
- ⚠️ Lỗi thời: **App Runner** đóng với khách mới từ 30/04/2026 → AWS khuyên dùng **ECS Express Mode** (một lệnh tạo Fargate service + ALB + auto scaling).

---

## 2. Core Concepts

### Docker & registry
- **Là gì:** container chia sẻ kernel host nên nhẹ hơn VM; image xây từ Dockerfile, chạy thành container.
- **Dùng khi nào:** microservice, lift-and-shift ứng dụng lên cloud, CI/CD đồng nhất.
- **Vì sao quan trọng:** **ECR** lưu image trên S3, kiểm soát bằng IAM, có **image scanning**, versioning, tag, lifecycle policy.

### ECS — EC2 launch type
- **Là gì:** cluster gồm EC2 chạy ECS agent; bạn provision và bảo trì instance; ECS đặt task lên instance.
- **Dùng khi nào:** cần kiểm soát instance (GPU đặc thù, tối ưu chi phí với RI/Spot).
- **Vì sao quan trọng:** dùng **capacity provider** gắn ASG để cluster tự thêm EC2 khi thiếu chỗ cho task.

### ECS — Fargate
- **Là gì:** không quản server; định nghĩa task với CPU/RAM; AWS chạy task.
- **Dùng khi nào:** muốn serverless container, tải biến động, ít vận hành.
- **Vì sao quan trọng:** scale = tăng số task; mỗi task có ENI riêng (awsvpc), SG riêng.

### `[+]` ECS Managed Instances & Express Mode
- **Là gì:** **Managed Instances**: ECS tự chọn, launch, vá và thay EC2 (chọn được loại instance như GPU) nhưng AWS lo vận hành; **Express Mode**: một API call tạo cả Fargate service, ALB, auto scaling, networking.
- **Dùng khi nào:** cần tính năng EC2 mà không muốn quản EC2 → Managed Instances; deploy web app/API nhanh từ image → Express Mode.
- **Vì sao quan trọng:** Express Mode là đích migrate chính thức cho App Runner.

### IAM roles cho ECS
- **Là gì:** **instance profile** (EC2 launch type) cho agent; **task role** cho code trong container; **task execution role** để kéo image từ ECR, ghi log, đọc secret.
- **Dùng khi nào:** mỗi service một task role riêng theo least privilege (service A đọc S3, service B ghi DynamoDB).
- **Vì sao quan trọng:** đề hỏi "cấp quyền S3 cho một container cụ thể" → **task role**, không phải instance profile.

### Load balancing & storage cho ECS
- **Là gì:** ALB route tới task (dynamic host port trên EC2); NLB cho throughput cao hoặc PrivateLink; EFS mount vào task trên cả EC2 và Fargate, Multi-AZ.
- **Dùng khi nào:** chia sẻ dữ liệu bền giữa task → EFS.
- **Vì sao quan trọng:** S3 không mount được làm file system cho task (dùng SDK/API).

### ECS Service Auto Scaling & event-driven
- **Là gì:** Application Auto Scaling theo CPU, memory, ALBRequestCountPerTarget; target tracking, step, scheduled.
- **Dùng khi nào:** EventBridge rule chạy ECS task khi có object mới trên S3; ECS task kéo message từ SQS; EventBridge bắt sự kiện task dừng để cảnh báo.
- **Vì sao quan trọng:** scale task (service) ≠ scale EC2 (capacity provider); trên Fargate chỉ cần scale task.

### Amazon EKS
- **Là gì:** Kubernetes managed, mã nguồn mở, chạy được đa cloud; control plane do AWS quản.
- **Dùng khi nào:** công ty đã dùng Kubernetes on-prem hoặc đa cloud, muốn migrate giữ API Kubernetes.
- **Vì sao quan trọng:** node: **managed node group** (AWS tạo/quản ASG, hỗ trợ On-Demand/Spot), **self-managed node**, **Fargate**, **EKS Auto Mode** (AWS quản compute, storage, networking, vá node).

### EKS data volumes & pod permissions
- **Là gì:** StorageClass dùng **CSI driver**: EBS, EFS (chạy được với Fargate), FSx for Lustre, FSx for NetApp ONTAP.
- **Dùng khi nào:** pod cần quyền AWS → **EKS Pod Identity** (hoặc IRSA).
- **Vì sao quan trọng:** cấp quyền theo pod thay vì cho cả node.

### ⚠️ App Runner
- **Là gì:** dịch vụ deploy web app/API từ source hoặc image, tự build, scale, HTTPS, load balancing.
- **Dùng khi nào:** đề cũ: "deploy web app nhanh, không cần kinh nghiệm hạ tầng" → App Runner.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: App Runner **đóng với khách mới**, không thêm tính năng → nay dùng **ECS Express Mode**.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Docker image | Gói app + dependency | "package application" |
| Amazon ECR | Registry image managed | "store container images privately" |
| ECS EC2 launch type | Task chạy trên EC2 tự quản | "control over instances" |
| Fargate | Container serverless | "no servers to manage" |
| Task definition | Mô tả container, CPU/RAM, role | "configure container" |
| Task role | Quyền IAM cho container | "container needs S3 access" |
| Capacity provider | Gắn ASG để scale EC2 cho ECS | "scale cluster instances" |
| ECS Managed Instances | AWS quản EC2 cho ECS | "EC2 features without managing EC2" |
| ECS Express Mode | Một lệnh tạo Fargate + ALB + scaling | "simplest deploy from image" |
| Amazon EKS | Kubernetes managed | "already use Kubernetes" |
| Managed node group | Node EKS do AWS quản ASG | "EKS worker nodes managed" |
| EKS Auto Mode | AWS quản compute/storage/network của cluster | "minimal Kubernetes ops" |
| CSI driver | Plugin storage cho Kubernetes | "persistent volume on EKS" |
| EKS Pod Identity | Quyền IAM cho pod | "per-pod AWS permissions" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| ECS EC2 | Fargate | EC2 tự quản instance; Fargate serverless | "Least operational overhead" → Fargate |
| ECS | EKS | ECS là orchestrator riêng AWS; EKS là Kubernetes chuẩn | "Kubernetes" → EKS |
| Instance profile | Task role | Instance profile cho agent/cả host; task role cho từng task | Quyền theo container → task role |
| Task role | Task execution role | Task role: code trong container; execution role: ECS kéo image, ghi log | |
| Service Auto Scaling | Capacity provider | Service scale số task; capacity provider scale EC2 | Fargate chỉ cần service scaling |
| EFS | S3 cho container | EFS mount như file system; S3 gọi qua API | "Shared persistent storage" → EFS |
| App Runner | ECS Express Mode | App Runner đóng với khách mới; Express Mode là thay thế | Khách mới → Express Mode |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "run containers without managing servers" | ECS/EKS on Fargate | |
| "already using Kubernetes on-prem, migrate" | Amazon EKS | |
| "store private Docker images with scanning" | Amazon ECR | |
| "each container needs different AWS permissions" | ECS task role | |
| "persistent shared storage for Fargate tasks" | Amazon EFS | |
| "scale ECS tasks on ALB request count" | ECS Service Auto Scaling | |
| "ECS on EC2 runs out of capacity" | Capacity provider + ASG | |
| "run ECS task when file lands in S3" | EventBridge rule → ECS task | |
| "deploy web app from image with one call (new customer)" | ECS Express Mode | Thay App Runner |
| "EKS pods need IAM permissions" | EKS Pod Identity | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Team muốn chạy container mà không quản lý EC2, patch OS hay scale cluster. Chọn gì?
- A. ECS EC2 launch type
- B. ECS on Fargate
- C. EC2 + Docker tự cài
- D. EKS self-managed nodes

<details><summary>Đáp án</summary>

**B.**
- A, C, D sai: vẫn phải quản lý EC2.
</details>

**Câu 2.** Hai service trên cùng cluster ECS: service A chỉ đọc S3, service B chỉ ghi DynamoDB. Cấp quyền thế nào?
- A. Một instance profile có cả hai quyền
- B. Mỗi task definition một task role riêng
- C. Lưu access key trong biến môi trường
- D. Dùng bucket policy cho cả hai

<details><summary>Đáp án</summary>

**B.** Least privilege theo từng service.
- A sai: mọi task trên host có cả hai quyền.
- C sai: credential dài hạn, rủi ro.
- D sai: bucket policy không cấp quyền DynamoDB.
</details>

**Câu 3.** Công ty đang chạy Kubernetes on-prem, muốn lên AWS và vẫn dùng công cụ Kubernetes. Chọn gì?
- A. ECS
- B. EKS
- C. Elastic Beanstalk
- D. Lambda

<details><summary>Đáp án</summary>

**B.**
- A sai: API khác Kubernetes.
- C sai: PaaS cho web app.
- D sai: serverless function.
</details>

**Câu 4.** (Chọn 2) Nhiều task Fargate cần dùng chung dữ liệu bền giữa các AZ và tự scale theo số request trên ALB. Chọn gì?
- A. Mount Amazon EFS vào task
- B. ECS Service Auto Scaling theo ALBRequestCountPerTarget
- C. Mount S3 bucket làm file system
- D. EBS Multi-Attach
- E. Capacity provider với ASG

<details><summary>Đáp án</summary>

**A, B.**
- C sai: S3 không mount làm file system cho task.
- D sai: một AZ, không dùng với Fargate theo cách này.
- E sai: Fargate không cần capacity provider ASG.
</details>

**Câu 5.** Cluster ECS EC2 launch type không đặt thêm task được vì hết CPU. Cách tự động hóa?
- A. Chuyển sang Lambda
- B. Dùng capacity provider gắn ASG (managed scaling)
- C. Tăng số task desired
- D. Bật Fargate Spot

<details><summary>Đáp án</summary>

**B.** Capacity provider tự thêm EC2 khi task chờ chỗ.
- A sai: đổi kiến trúc.
- C sai: task vẫn không có chỗ chạy.
- D sai: đổi launch type, không giải quyết capacity EC2.
</details>

**Câu 6.** Khi có file mới trong S3, cần chạy một container xử lý rồi kết thúc. Kiến trúc serverless phù hợp?
- A. EventBridge rule bắt sự kiện S3 → chạy ECS task trên Fargate
- B. EC2 chạy cron kiểm tra bucket
- C. CloudFront trigger
- D. Route 53 health check

<details><summary>Đáp án</summary>

**A.**
- B sai: không serverless, tốn tài nguyên.
- C, D sai: không kích hoạt task.
</details>

**Câu 7.** Khách hàng mới muốn deploy web API từ container image với ít thao tác nhất (tự có ALB, auto scaling). Theo hiện trạng 10/2026, chọn gì?
- A. AWS App Runner
- B. Amazon ECS Express Mode
- C. EC2 + Nginx
- D. AWS Batch

<details><summary>Đáp án</summary>

**B.**
- A sai: App Runner đóng với khách mới.
- C sai: nhiều vận hành.
- D sai: dành cho batch job.
</details>

**Câu 8.** Pod trên EKS cần đọc secret từ Secrets Manager; không muốn cấp quyền cho toàn bộ node. Chọn gì?
- A. Gắn quyền vào node IAM role
- B. EKS Pod Identity (hoặc IRSA) cho service account của pod
- C. Lưu access key trong ConfigMap
- D. Security Group cho pod

<details><summary>Đáp án</summary>

**B.** Quyền theo pod.
- A sai: mọi pod trên node có quyền.
- C sai: credential dài hạn không an toàn.
- D sai: SG kiểm soát mạng, không cấp quyền IAM.
</details>

---

### Nguồn đã đối chiếu
- Amazon ECS launch types & capacity: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_types.html
- Amazon ECS Managed Instances: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-cluster-managed-instances.html
- ECS task IAM role: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html
- AWS App Runner availability change (→ ECS Express Mode): https://docs.aws.amazon.com/apprunner/latest/dg/apprunner-availability-change.html
- EKS Auto Mode: https://docs.aws.amazon.com/eks/latest/userguide/automode.html
- EKS Pod Identity: https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html
- Amazon ECR image scanning: https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html
