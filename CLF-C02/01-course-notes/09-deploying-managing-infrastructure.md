# CLF-C02 — Deploying & Managing Infrastructure at Scale

> Thuộc **Domain 3** (deployment & operating methods, developer tools, management) và nguyên tắc "operations as code" của Well-Architected.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **CloudFormation** = **Infrastructure as Code** khai báo (declarative): viết template, AWS tạo tài nguyên đúng thứ tự, đúng cấu hình; dùng lặp lại qua nhiều Region và account.
- **AWS CDK** = định nghĩa hạ tầng bằng **ngôn ngữ lập trình** (TypeScript, Python, Java, .NET...), "compile" ra template CloudFormation.
- **Elastic Beanstalk** = **PaaS**: chỉ upload code, Beanstalk lo EC2, ASG, ELB, RDS, deploy, health monitoring. Dịch vụ miễn phí, trả tiền tài nguyên bên dưới.
- Bộ **Code***: **CodeCommit** (Git repo), **CodeBuild** (build & test), **CodeDeploy** (deploy lên EC2/on-prem), **CodePipeline** (điều phối CI/CD), **CodeArtifact** (lưu package dependency).
- **Systems Manager (SSM)** = quản lý EC2 **và on-prem** ở quy mô lớn: patching, run command, **Session Manager** (shell không cần SSH/port 22), **Parameter Store** (cấu hình & secret).
- Dịch vụ **hybrid** (chạy cả on-prem): **CodeDeploy** và **Systems Manager**. CloudFormation và Beanstalk chỉ cho AWS.

---

## 2. Core Concepts

### AWS CloudFormation
- **Là gì:** cách **khai báo** hạ tầng AWS (security group, EC2, S3, ELB...) trong template; CloudFormation tạo chúng đúng thứ tự và đúng cấu hình.
- **Dùng khi nào:** muốn hạ tầng lặp lại được, review qua code, dựng lại môi trường nhanh, triển khai giống nhau ở nhiều Region/account.
- **Vì sao quan trọng:** hỗ trợ gần như mọi tài nguyên AWS; tài nguyên chưa hỗ trợ thì dùng **custom resource**.

### Lợi ích của CloudFormation
- **Là gì:** **IaC**: không tạo tài nguyên thủ công, thay đổi được review qua code; **Cost**: tài nguyên trong stack được tag để biết stack tốn bao nhiêu, ước tính chi phí từ template.
- **Dùng khi nào:** tiết kiệm: tự xóa môi trường dev lúc 17h và tạo lại lúc 8h sáng; **Productivity**: xóa và tạo lại hạ tầng nhanh, tự sinh sơ đồ, không cần tự xử lý thứ tự.
- **Vì sao quan trọng:** không phải phát minh lại: dùng template có sẵn trên web và docs.

### AWS Infrastructure Composer
- **Là gì:** công cụ **trực quan** để thiết kế và xem quan hệ giữa các tài nguyên trong CloudFormation template (vd: stack WordPress).
- **Dùng khi nào:** cần nhìn sơ đồ tài nguyên và liên kết.
- **Vì sao quan trọng:** tên cũ là "AWS Application Composer", hiện exam guide CLF-C02 xếp tên cũ vào out-of-scope. Chi tiết ở chương 18.

### AWS Cloud Development Kit (CDK)
- **Là gì:** định nghĩa hạ tầng bằng ngôn ngữ quen thuộc (JavaScript/TypeScript, Python, Java, .NET...); code được "compile" thành template CloudFormation (JSON/YAML).
- **Dùng khi nào:** deploy **hạ tầng và code ứng dụng cùng lúc**; hợp với Lambda và container trên ECS/EKS.
- **Vì sao quan trọng:** "IaC bằng ngôn ngữ lập trình" → CDK; "IaC bằng JSON/YAML" → CloudFormation.

### Vấn đề của developer trên AWS
- **Là gì:** phải tự quản hạ tầng, deploy code, cấu hình DB/load balancer, lo scaling; hầu hết web app có cùng kiến trúc (ALB + ASG).
- **Dùng khi nào:** kiến trúc 3-tier điển hình: ELB → ASG multi-AZ → ElastiCache (session/cache) + RDS.
- **Vì sao quan trọng:** developer chỉ muốn code chạy, nhất quán qua nhiều môi trường → dẫn tới Elastic Beanstalk.

### AWS Elastic Beanstalk
- **Là gì:** góc nhìn **developer-centric** để deploy app; dùng EC2, ASG, ELB, RDS trong một giao diện, vẫn toàn quyền cấu hình; là **PaaS**.
- **Dùng khi nào:** Beanstalk lo cấu hình instance/OS, chiến lược deploy, capacity, load balancing, auto-scaling, health monitoring; developer chỉ chịu trách nhiệm **code**.
- **Vì sao quan trọng:** 3 kiến trúc: **Single instance** (dev), **LB + ASG** (production web), **ASG only** (worker không phải web).

### Beanstalk platforms & health monitoring
- **Là gì:** hỗ trợ Go, Java SE, Java/Tomcat, .NET trên Windows Server/IIS, Node.js, PHP, Python, Ruby, Docker (single, multi-container, preconfigured)...
- **Dùng khi nào:** health agent đẩy metric lên CloudWatch, kiểm tra sức khỏe app và phát health event.
- **Vì sao quan trọng:** Beanstalk giới hạn ở các ngôn ngữ/nền tảng được hỗ trợ hoặc Docker.

### AWS CodeDeploy
- **Là gì:** deploy ứng dụng tự động, nâng cấp từ v1 lên v2.
- **Dùng khi nào:** chạy được trên **EC2 và server on-premises** (hybrid); server phải được provision trước và cài **CodeDeploy Agent**.
- **Vì sao quan trọng:** exam guide CLF-C02 hiện xếp CodeDeploy vào **out-of-scope**, vẫn nên biết để loại đáp án.

### AWS CodeCommit
- **Là gì:** dịch vụ source control lưu **Git repository** private, đối thủ của GitHub; fully managed, scalable, HA, bảo mật, tích hợp AWS.
- **Dùng khi nào:** lưu code trước khi build và deploy; thay đổi được version tự động.
- **Vì sao quan trọng:** `[+]` CodeCommit từng ngừng nhận khách mới (07/2024) nhưng đã **mở lại cho khách mới từ 25/11/2025**. Tài liệu cũ ghi "CodeCommit đã đóng" là lỗi thời.

### AWS CodeBuild
- **Là gì:** dịch vụ **build** code trên cloud: compile, chạy test, tạo package sẵn sàng deploy.
- **Dùng khi nào:** sau khi lấy code từ repo, trước khi deploy.
- **Vì sao quan trọng:** fully managed, **serverless**, trả theo **thời gian build**.

### AWS CodePipeline
- **Là gì:** **điều phối** các bước để code tự động lên production: Code → Build → Test → Provision → Deploy.
- **Dùng khi nào:** nền tảng của **CI/CD**; tích hợp CodeCommit, CodeBuild, CodeDeploy, Beanstalk, CloudFormation, GitHub và plugin bên thứ ba.
- **Vì sao quan trọng:** "orchestrate pipeline" → CodePipeline; nó không tự build hay deploy, mà gọi các dịch vụ khác.

### AWS CodeArtifact
- **Là gì:** quản lý **artifact** (package, code dependency) bảo mật, scalable, rẻ.
- **Dùng khi nào:** dùng với Maven, Gradle, npm, yarn, twine, pip, NuGet; developer và CodeBuild lấy dependency trực tiếp từ đây.
- **Vì sao quan trọng:** exam guide hiện xếp CodeArtifact vào **out-of-scope**.

### AWS Systems Manager (SSM)
- **Là gì:** quản lý hệ thống EC2 **và on-premises** ở quy mô lớn (dịch vụ **hybrid**); bộ hơn 10 sản phẩm, cung cấp insight vận hành.
- **Dùng khi nào:** **patching tự động** để tăng compliance, **chạy lệnh trên cả fleet**, lưu cấu hình bằng **Parameter Store**; hỗ trợ Linux, Windows, macOS, Raspberry Pi OS.
- **Vì sao quan trọng:** cần cài **SSM Agent** (có sẵn trên Amazon Linux AMI và một số Ubuntu AMI). Instance không điều khiển được bằng SSM thì thường do agent.

### SSM Session Manager
- **Là gì:** mở **shell bảo mật** vào EC2 và server on-prem **không cần SSH, bastion host hay SSH key**, **không cần mở port 22**.
- **Dùng khi nào:** truy cập instance an toàn hơn SSH; quyền kiểm soát bằng IAM; hỗ trợ Linux, macOS, Windows.
- **Vì sao quan trọng:** gửi log phiên làm việc sang S3 hoặc CloudWatch Logs để audit.

### SSM Parameter Store
- **Là gì:** lưu **cấu hình và secret** (API key, password, config) an toàn; serverless, scalable, bền, SDK dễ dùng.
- **Dùng khi nào:** quản lý quyền bằng IAM, có version tracking và mã hóa tùy chọn bằng KMS.
- **Vì sao quan trọng:** khác Secrets Manager (chương 14): Secrets Manager có **tự xoay vòng secret**.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Infrastructure as Code (IaC) | Định nghĩa hạ tầng bằng code/template | "repeatable", "version-controlled infrastructure" |
| AWS CloudFormation | IaC khai báo bằng JSON/YAML | "template", "stack", "deploy same infra in many Regions" |
| Custom resource | Tài nguyên tự định nghĩa khi CloudFormation chưa hỗ trợ | "resource not natively supported" |
| AWS CDK | IaC bằng ngôn ngữ lập trình, sinh ra CloudFormation | "define infrastructure in Python/TypeScript" |
| Infrastructure Composer | Thiết kế trực quan, xem sơ đồ template | "visualize resources" |
| AWS Elastic Beanstalk | PaaS: upload code, AWS lo hạ tầng | "developer just uploads code" |
| CodeCommit | Git repository private managed | "source control", "Git" |
| CodeBuild | Build & test code, serverless | "compile and run tests" |
| CodeDeploy | Deploy app lên EC2 và on-prem | "deploy to on-premises servers" |
| CodePipeline | Điều phối CI/CD | "orchestrate", "CI/CD pipeline" |
| CodeArtifact | Lưu package dependency | "npm/Maven/pip dependencies" |
| CI/CD | Continuous Integration & Continuous Delivery | "release faster, automated" |
| AWS Systems Manager | Quản lý EC2 + on-prem ở quy mô lớn | "patch fleet", "run commands at scale" |
| SSM Agent | Agent cần cài để SSM điều khiển máy | "instance not managed by SSM" |
| Session Manager | Shell bảo mật không cần SSH/port 22 | "no bastion host", "no SSH keys" |
| Parameter Store | Lưu config và secret, mã hóa tùy chọn | "store configuration values" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CloudFormation | Elastic Beanstalk | IaC cho **mọi** tài nguyên vs PaaS cho **ứng dụng** web/worker | "Upload code only" → Beanstalk |
| CloudFormation | CDK | JSON/YAML vs ngôn ngữ lập trình (sinh ra CloudFormation) | "Python/TypeScript for infra" → CDK |
| CodeCommit | CodeBuild | Lưu code vs build/test code | |
| CodeBuild | CodeDeploy | Build package vs deploy lên server | |
| CodeDeploy | CodePipeline | Một bước deploy vs điều phối toàn bộ pipeline | "Orchestrate" → CodePipeline |
| CodeArtifact | CodeCommit | Lưu **package dependency** vs lưu **source code** | "npm packages" → CodeArtifact |
| Session Manager | SSH / EC2 Instance Connect | Không cần port 22, không key vs cần port 22 | "No inbound port 22" → Session Manager |
| Parameter Store | Secrets Manager | Config + secret đơn giản vs secret có **tự xoay vòng** (RDS) | "Automatic rotation" → Secrets Manager |
| Systems Manager | CodeDeploy | Quản lý/patch fleet vs deploy phiên bản app | "Patch" → SSM |
| Dịch vụ hybrid | Dịch vụ chỉ AWS | CodeDeploy, SSM chạy cả on-prem; CloudFormation, Beanstalk chỉ AWS | "On-premises servers" → CodeDeploy/SSM |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "infrastructure as code, templates, repeatable" | CloudFormation | |
| "define infrastructure using Python/TypeScript" | AWS CDK | |
| "deploy web app, just upload code, AWS handles capacity" | Elastic Beanstalk | PaaS |
| "private Git repositories" | CodeCommit | |
| "compile code and run unit tests, pay per build minute" | CodeBuild | |
| "deploy application to EC2 and on-prem servers" | CodeDeploy | Out-of-scope nhưng nên biết |
| "automate release pipeline end-to-end" | CodePipeline | |
| "store and share software packages" | CodeArtifact | |
| "patch hundreds of EC2 and on-prem servers" | Systems Manager (Patch Manager) | |
| "run a command on all instances" | Systems Manager (Run Command) | |
| "secure shell without SSH keys or open port 22" | SSM Session Manager | |
| "store configuration values, optional encryption" | SSM Parameter Store | |
| "visualize CloudFormation template" | Infrastructure Composer | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Công ty muốn dựng cùng một môi trường (VPC, EC2, RDS) ở 3 Region một cách lặp lại và kiểm soát được qua code. Chọn gì?
- A. AWS Elastic Beanstalk
- B. AWS CloudFormation
- C. AWS Systems Manager
- D. AWS CodeDeploy

<details><summary>Đáp án</summary>

**B.** CloudFormation là IaC, dùng template để tạo cùng hạ tầng ở nhiều Region/account.
- A sai: PaaS cho ứng dụng, không quản lý mọi loại tài nguyên.
- C sai: quản lý vận hành server.
- D sai: deploy code.
</details>

**Câu 2.** Developer chỉ muốn upload code Java web app; AWS tự lo EC2, load balancer, auto-scaling, health monitoring. Chọn gì?
- A. AWS Elastic Beanstalk
- B. Amazon EC2
- C. AWS CloudFormation
- D. Amazon ECR

<details><summary>Đáp án</summary>

**A.** Beanstalk là PaaS: developer chỉ chịu trách nhiệm code.
- B sai: IaaS, phải tự quản.
- C sai: phải tự viết template hạ tầng.
- D sai: registry image.
</details>

**Câu 3.** (Chọn 2) Những dịch vụ nào dùng được cho cả server **on-premises**?
- A. AWS CodeDeploy
- B. AWS Elastic Beanstalk
- C. AWS Systems Manager
- D. AWS CloudFormation
- E. AWS Fargate

<details><summary>Đáp án</summary>

**A, C.** Cả hai là dịch vụ hybrid.
- B, D sai: chỉ cho tài nguyên AWS.
- E sai: serverless container trên AWS.
</details>

**Câu 4.** Đội bảo mật muốn admin mở shell vào EC2 mà không mở port 22, không dùng bastion host và có log phiên làm việc. Chọn gì?
- A. EC2 Instance Connect
- B. SSM Session Manager
- C. Security Group mở port 22 cho 0.0.0.0/0
- D. AWS CodeDeploy

<details><summary>Đáp án</summary>

**B.** Session Manager không cần SSH key hay port 22, log sang S3/CloudWatch Logs.
- A sai: vẫn cần port 22.
- C sai: kém an toàn.
- D sai: deploy app.
</details>

**Câu 5.** Team muốn một dịch vụ điều phối toàn bộ quy trình: lấy code từ repo → build → test → deploy lên Beanstalk. Chọn gì?
- A. AWS CodeBuild
- B. AWS CodePipeline
- C. AWS CodeArtifact
- D. AWS CodeCommit

<details><summary>Đáp án</summary>

**B.** CodePipeline là lớp orchestration của CI/CD.
- A sai: chỉ build/test.
- C sai: lưu package.
- D sai: lưu code.
</details>

**Câu 6.** Developer muốn định nghĩa hạ tầng bằng TypeScript thay vì viết JSON/YAML. Chọn gì?
- A. AWS CDK
- B. AWS CloudFormation template viết tay
- C. AWS Systems Manager Parameter Store
- D. AWS Elastic Beanstalk

<details><summary>Đáp án</summary>

**A.** CDK cho phép dùng ngôn ngữ lập trình, rồi sinh template CloudFormation.
- B sai: template viết bằng JSON/YAML.
- C sai: lưu cấu hình.
- D sai: PaaS.
</details>

**Câu 7.** Công ty cần vá bảo mật tự động cho 500 server gồm EC2 và máy ảo on-premises. Chọn gì?
- A. AWS Systems Manager
- B. Amazon Inspector
- C. AWS CodePipeline
- D. AWS Trusted Advisor

<details><summary>Đáp án</summary>

**A.** SSM có patching tự động cho fleet hybrid.
- B sai: Inspector **phát hiện** lỗ hổng, không vá.
- C sai: pipeline CI/CD.
- D sai: khuyến nghị best practice.
</details>

**Câu 8.** (Chọn 2) Lợi ích của CloudFormation là gì?
- A. Ước tính chi phí từ template và theo dõi chi phí theo stack qua tag
- B. Tự động vá OS cho EC2 trong stack
- C. Xóa và tạo lại môi trường nhanh chóng
- D. Thay thế hoàn toàn nhu cầu dùng IAM
- E. Chỉ hỗ trợ tài nguyên EC2

<details><summary>Đáp án</summary>

**A, C.**
- B sai: CloudFormation không vá OS.
- D sai: vẫn cần IAM.
- E sai: hỗ trợ gần như mọi tài nguyên.
</details>

---

### Nguồn đã đối chiếu
- AWS CodeCommit document history (mở lại cho khách mới từ 25/11/2025): https://docs.aws.amazon.com/codecommit/latest/userguide/history.html
- CLF-C02 in-scope services (CodeBuild, CodePipeline, CloudFormation, Systems Manager): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-in-scope-services.html
- CLF-C02 out-of-scope services (CodeDeploy, CodeArtifact, Application Composer): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
- AWS Systems Manager Session Manager: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html
- AWS CDK: https://docs.aws.amazon.com/cdk/v2/guide/home.html
- AWS Elastic Beanstalk: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html
