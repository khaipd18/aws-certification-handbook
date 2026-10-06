# AWS Certification Handbook

Sổ tay ôn thi chứng chỉ AWS bằng tiếng Việt (giữ nguyên thuật ngữ tiếng Anh). Mỗi chứng chỉ có hai phần: **ghi chú khóa học** theo từng chương và **sổ tay đề thi thử** gom các bẫy và từ khóa hay gặp.

> **Mốc kiến thức: 10/2026.** Nội dung đã đối chiếu với AWS Documentation. Những dịch vụ đã đổi tên, đóng với khách mới hoặc ngừng hoạt động đều được đánh dấu ngay trong bài.

---

## Chứng chỉ

| Mã | Chứng chỉ | Cấp | Ghi chú khóa học | Sổ tay đề thi thử | Trạng thái |
|---|---|---|---|---|---|
| [CLF-C02](CLF-C02/) | AWS Certified Cloud Practitioner | Foundational | 19 chương | [practice-exam-notes](CLF-C02/02-practice-exams/practice-exam-notes.md) | ✅ Hoàn thành |
| [AIF-C01](AIF-C01/) | AWS Certified AI Practitioner | Foundational | 10 chương | [practice-exam-notes](AIF-C01/02-practice-exams/practice-exam-notes.md) | ✅ Hoàn thành |
| [SAA-C03](SAA-C03/) | AWS Certified Solutions Architect – Associate | Associate | 25 chương | [practice-exam-notes](SAA-C03/02-practice-exams/practice-exam-notes.md) | ✅ Hoàn thành |
| [SOA-C03](SOA-C03/) | AWS Certified CloudOps Engineer – Associate | Associate | — | [practice-exam-notes](SOA-C03/02-practice-exams/practice-exam-notes.md) | 🚧 Đã có sổ tay đề thi thử, chưa có ghi chú khóa học |

Lộ trình gợi ý: **CLF-C02** (nền tảng) → **AIF-C01** (tùy chọn, nếu làm AI/ML) → **SAA-C03** → **SOA-C03**. Nhiều chương ở SAA và SOA dùng chung dịch vụ, nên học SAA trước sẽ đỡ cho SOA.

---

## Thông tin kỳ thi

Bốn kỳ thi có cùng định dạng: **65 câu**, gồm 50 câu tính điểm và 15 câu thử nghiệm không tính điểm (không biết câu nào là câu thử nghiệm). Điểm quy đổi theo thang **100–1,000**. Câu hỏi có hai dạng chính: **multiple choice** (chọn 1 trong 4) và **multiple response** (chọn từ 2 đáp án trở lên trong 5 phương án trở lên). Riêng AIF-C01 còn có thể có câu **ordering** (sắp xếp 3–5 bước) và **matching** (ghép 3–7 cặp).

| Mã | Điểm đạt | Domain và tỷ trọng |
|---|---|---|
| CLF-C02 | 700 | Cloud Concepts 24% · Security and Compliance 30% · Cloud Technology and Services 34% · Billing, Pricing, and Support 12% |
| AIF-C01 | 700 | Fundamentals of AI and ML 20% · Fundamentals of GenAI 24% · Applications of Foundation Models 28% · Guidelines for Responsible AI 14% · Security, Compliance, and Governance for AI Solutions 14% |
| SAA-C03 | 720 | Design Secure Architectures 30% · Design Resilient Architectures 26% · Design High-Performing Architectures 24% · Design Cost-Optimized Architectures 20% |
| SOA-C03 | 720 | Monitoring, Logging, Analysis, Remediation, and Performance Optimization 22% · Reliability and Business Continuity 22% · Deployment, Provisioning, and Automation 22% · Security and Compliance 16% · Networking and Content Delivery 18% |

Thời lượng, lệ phí và cách đăng ký có thể thay đổi, nên xem trực tiếp trên trang [AWS Certification](https://aws.amazon.com/certification/) trước khi đặt lịch thi.

Nguồn (exam guide chính thức):
- CLF-C02: https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/cloud-practitioner-02.html
- AIF-C01: https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01.html
- SAA-C03: https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html
- SOA-C03: https://docs.aws.amazon.com/aws-certification/latest/sysops-administrator-associate-03/sysops-administrator-associate-03.html

---

## Cấu trúc repo

- `<MÃ-CHỨNG-CHỈ>/`
  - `01-course-notes/`: mỗi chương một file `NN-ten-chuong.md`, học theo thứ tự số.
  - `02-practice-exams/practice-exam-notes.md`: sổ tay rút ra từ các đề thi thử.

### Ghi chú khóa học (`01-course-notes`)

Mọi chương đều có cùng 5 mục:

1. **TL;DR**: 5–8 ý cần nhớ nhất của chương.
2. **Core Concepts**: mỗi khái niệm gồm tối đa 3 ý: *Là gì*, *Dùng khi nào*, *Vì sao quan trọng*.
3. **Key Terms**: bảng thuật ngữ, giải thích ngắn và **dấu hiệu nhận biết trong đề**.
4. **Dễ nhầm & bẫy đề**: bảng so sánh các cặp khái niệm hay lẫn, và bảng *từ khóa trong đề → đáp án*.
5. **Câu hỏi tình huống**: 8 câu (có câu "Chọn 2"), đáp án ẩn trong mục *Đáp án*, kèm lý do các phương án còn lại sai.

Cuối mỗi chương có mục **Nguồn đã đối chiếu**, dẫn link AWS Documentation.

### Sổ tay đề thi thử (`02-practice-exams`)

- Chia mục (§) theo **nhóm dịch vụ**. Mỗi ý là một dòng ngắn: 🎯 là kiến thức cốt lõi, ⚠️ là chỗ dễ nhầm.
- Các phần ở cuối file dùng để ôn nhanh trước giờ thi:
  - phụ lục distractor (dịch vụ hay bị dùng làm đáp án nhiễu);
  - nguyên tắc làm bài;
  - bảng so sánh dễ nhầm;
  - từ khóa → đáp án;
  - con số cần thuộc và các kết hợp bất khả thi.

---

## Ký hiệu

| Ký hiệu | Ý nghĩa | Dùng ở |
|---|---|---|
| `[+]` | Bổ sung ngoài slide khóa học (tối đa 5 ý mỗi chương) | Course notes |
| ⚠️ Lỗi thời | Slide hoặc đề cũ đã sai so với hiện tại, có ghi cách mới | Course notes |
| ❓ | Chưa kiểm chứng được trong docs, đề mâu thuẫn hoặc đáng ngờ | Cả hai |
| 🎯 | Kiến thức cốt lõi | Practice exam notes |
| ⚠️ | Dễ nhầm, bẫy đề | Practice exam notes |
| 📌 | Cập nhật đến 10/2026 so với đáp án trong đề | Practice exam notes |

---

## Cách ôn đề xuất

1. Đọc **TL;DR** và **Core Concepts** của từng chương theo thứ tự.
2. Tự làm 8 câu tình huống trước khi mở đáp án. Câu sai thì đọc lại mục 4 (Dễ nhầm & bẫy đề).
3. Làm đề thi thử, rồi tra những câu sai trong **sổ tay đề thi thử** theo nhóm dịch vụ.
4. Trước ngày thi, ôn các bảng tổng hợp cuối sổ tay: từ khóa → đáp án, con số cần thuộc, bảng so sánh.
5. Gặp dịch vụ có ⚠️ Lỗi thời: nắm cả tên cũ (đề có thể còn dùng) lẫn tên hoặc cách làm mới.

---

## Mục lục

<details>
<summary><b>CLF-C02 — Cloud Practitioner</b> (19 chương)</summary>

1. [What is Cloud Computing?](CLF-C02/01-course-notes/01-cloud-computing.md)
2. [AWS Identity & Access Management (IAM)](CLF-C02/01-course-notes/02-iam.md)
3. [Amazon EC2](CLF-C02/01-course-notes/03-ec2.md)
4. [EC2 Instance Storage](CLF-C02/01-course-notes/04-ec2-instance-storage.md)
5. [Elastic Load Balancing & Auto Scaling Groups](CLF-C02/01-course-notes/05-elb-asg.md)
6. [Amazon S3](CLF-C02/01-course-notes/06-s3.md)
7. [Databases & Analytics](CLF-C02/01-course-notes/07-databases-analytics.md)
8. [Other Compute Services](CLF-C02/01-course-notes/08-other-compute.md)
9. [Deploying & Managing Infrastructure at Scale](CLF-C02/01-course-notes/09-deploying-managing-infrastructure.md)
10. [Global Infrastructure](CLF-C02/01-course-notes/10-global-infrastructure.md)
11. [Cloud Integration](CLF-C02/01-course-notes/11-cloud-integration.md)
12. [Cloud Monitoring](CLF-C02/01-course-notes/12-cloud-monitoring.md)
13. [Amazon VPC](CLF-C02/01-course-notes/13-vpc.md)
14. [Security & Compliance](CLF-C02/01-course-notes/14-security-compliance.md)
15. [Machine Learning](CLF-C02/01-course-notes/15-machine-learning.md)
16. [Account Management, Billing & Support](CLF-C02/01-course-notes/16-account-billing-support.md)
17. [Advanced Identity](CLF-C02/01-course-notes/17-advanced-identity.md)
18. [Other AWS Services](CLF-C02/01-course-notes/18-other-aws-services.md)
19. [AWS Architecting & Ecosystem](CLF-C02/01-course-notes/19-architecting-ecosystem.md)

📝 [Sổ tay đề thi thử CLF-C02](CLF-C02/02-practice-exams/practice-exam-notes.md)

</details>

<details>
<summary><b>AIF-C01 — AI Practitioner</b> (10 chương)</summary>

1. [Introduction to AI](AIF-C01/01-course-notes/01-intro-to-ai.md)
2. [AWS & Cloud Computing](AIF-C01/01-course-notes/02-aws-cloud-computing.md)
3. [Generative AI with Amazon Bedrock](AIF-C01/01-course-notes/03-genai-bedrock.md)
4. [Prompt Engineering](AIF-C01/01-course-notes/04-prompt-engineering.md)
5. [Amazon Q, Amazon Quick & Kiro](AIF-C01/01-course-notes/05-amazon-q-and-quick.md)
6. [AI & Machine Learning Overview](AIF-C01/01-course-notes/06-ai-ml-overview.md)
7. [AWS Managed AI Services](AIF-C01/01-course-notes/07-aws-managed-ai-services.md)
8. [Amazon SageMaker AI](AIF-C01/01-course-notes/08-sagemaker-ai.md)
9. [Responsible AI, Security, Compliance & Governance](AIF-C01/01-course-notes/09-responsible-ai-security-governance.md)
10. [AWS Services: Security & more](AIF-C01/01-course-notes/10-aws-security-services.md)

📝 [Sổ tay đề thi thử AIF-C01](AIF-C01/02-practice-exams/practice-exam-notes.md)

</details>

<details>
<summary><b>SAA-C03 — Solutions Architect Associate</b> (25 chương)</summary>

1. [IAM & AWS CLI](SAA-C03/01-course-notes/01-iam.md)
2. [EC2 Fundamentals, Purchasing Options & Spot](SAA-C03/01-course-notes/02-ec2.md)
3. [EC2 Networking: IP, Elastic IP, Placement Groups, ENI & Hibernate](SAA-C03/01-course-notes/03-ec2-networking-hibernate.md)
4. [EC2 Storage: EBS, AMI, Instance Store & EFS](SAA-C03/01-course-notes/04-ebs-ami-efs.md)
5. [High Availability & Scalability: ELB & ASG](SAA-C03/01-course-notes/05-elb-asg.md)
6. [RDS, Aurora, RDS Proxy & ElastiCache](SAA-C03/01-course-notes/06-rds-aurora-elasticache.md)
7. [Amazon Route 53](SAA-C03/01-course-notes/07-route53.md)
8. [Classic Solutions Architecture & Elastic Beanstalk](SAA-C03/01-course-notes/08-solution-architecture.md)
9. [Amazon S3: Fundamentals & Storage Classes](SAA-C03/01-course-notes/09-s3.md)
10. [Amazon S3: Advanced, Performance, Encryption & Security](SAA-C03/01-course-notes/10-s3-advanced-security.md)
11. [CloudFront & AWS Global Accelerator](SAA-C03/01-course-notes/11-cloudfront-global-accelerator.md)
12. [Other Storage: Snow Family, FSx, Storage Gateway, Transfer Family & DataSync](SAA-C03/01-course-notes/12-storage-extras.md)
13. [Decoupling: SQS, SNS, Kinesis & Amazon MQ](SAA-C03/01-course-notes/13-decoupling-sqs-sns-kinesis-mq.md)
14. [Containers: Docker, ECS, ECR, EKS & App Runner](SAA-C03/01-course-notes/14-containers.md)
15. [Serverless: Lambda, DynamoDB, API Gateway, Step Functions & Cognito](SAA-C03/01-course-notes/15-serverless.md)
16. [Serverless Solution Architectures](SAA-C03/01-course-notes/16-serverless-architectures.md)
17. [Databases in AWS: Choosing the Right Database](SAA-C03/01-course-notes/17-databases.md)
18. [Data & Analytics](SAA-C03/01-course-notes/18-data-analytics.md)
19. [Machine Learning Services](SAA-C03/01-course-notes/19-machine-learning.md)
20. [Monitoring & Audit: CloudWatch, EventBridge, CloudTrail, Config](SAA-C03/01-course-notes/20-monitoring-audit.md)
21. [Advanced Identity: Organizations, IAM, Identity Center, Directory Service, Control Tower](SAA-C03/01-course-notes/21-advanced-identity.md)
22. [Security & Encryption](SAA-C03/01-course-notes/22-security-encryption.md)
23. [Networking: Amazon VPC](SAA-C03/01-course-notes/23-vpc.md)
24. [Disaster Recovery & Migrations](SAA-C03/01-course-notes/24-disaster-recovery-migrations.md)
25. [Other Services](SAA-C03/01-course-notes/25-other-services.md)

📝 [Sổ tay đề thi thử SAA-C03](SAA-C03/02-practice-exams/practice-exam-notes.md)

</details>

<details>
<summary><b>SOA-C03 — CloudOps Engineer Associate</b> (chưa có ghi chú khóa học)</summary>

Ghi chú khóa học chưa có, sẽ theo cùng cấu trúc với các chứng chỉ ở trên.

📝 [Sổ tay đề thi thử SOA-C03](SOA-C03/02-practice-exams/practice-exam-notes.md)

</details>

---

## Lưu ý

- Đây là tài liệu học cá nhân, **không phải tài liệu chính thức của AWS** và không liên kết với Amazon Web Services.
- Nội dung được diễn giải lại từ khóa học và đề thi thử, không chép nguyên văn câu hỏi thi.
- AWS thay đổi dịch vụ liên tục. Khi thông tin trong sổ tay khác với docs hiện hành, **AWS Documentation là nguồn đúng**. Những gì phụ thuộc account cụ thể (quota thực tế, giá theo Region) thì kiểm tra trên console hoặc AWS CLI.
