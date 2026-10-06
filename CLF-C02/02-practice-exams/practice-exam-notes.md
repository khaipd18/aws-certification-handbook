# AWS CLF-C02 — Sổ tay cốt lõi & kiến thức dễ nhầm

> Tổng hợp ý tưởng cốt lõi + kiến thức liên quan/dễ nhầm từ các đề **Practice Test #2–#6** (mỗi đề 65 câu), theo nhóm dịch vụ. Chưa có Practice Test #1.
> **🎯 = ý tưởng cốt lõi cần khắc sâu** · **⚠️ = kiến thức liên quan / dễ nhầm trong đề** · **📌 = cập nhật theo docs AWS hiện hành (10/2026)** · **❓ = các đề mâu thuẫn nhau hoặc đáp án đáng ngờ**.
> Xem **§30 (Phụ lục)** để tra nhanh dịch vụ hay xuất hiện ở đáp án SAI và các đáp án **không tồn tại**.
>
> **Ôn cấp tốc trước khi thi:** §31 (Nguyên tắc làm bài) → §32 (Bảng so sánh dễ nhầm) → §33 (Từ khóa → đáp án) → §34 (Con số & đáp án bất khả thi).
>
> 4 domain CLF-C02: **Cloud Concepts 24% · Security & Compliance 30% · Cloud Technology & Services 34% · Billing, Pricing & Support 12%**.
>
> **Khi có đề mới:** chèn kiến thức vào đúng mục dịch vụ (§1–§29), cập nhật §30–§34. Không tạo phần riêng cho từng đề.
>
> **Khi thi thật:** chọn đáp án đúng chức năng theo tên dịch vụ và gói hiện hành.

---

## 1. Lợi ích cloud & mô hình triển khai

🎯 **Six advantages of cloud:** trade fixed expense (CAPEX) for variable expense · **massive economies of scale** (gộp mức dùng của hàng trăm nghìn khách → giá pay-as-you-go thấp) · stop guessing capacity · increase speed & agility · stop spending money running data centers · go global in minutes. Bẫy: "**trade OPEX for CAPEX**" là đảo ngược.

🎯 **Agility** = đổi mới nhanh, phát triển/thử nghiệm/ra mắt nhanh. **Elasticity** = lấy tài nguyên khi cần, trả lại khi không cần. Triển khai nhanh hạ tầng cho sản phẩm mới → **speed & agility + automatic scaling**.

⚠️ **Reliability** = phục hồi sau gián đoạn · **Durability** = dữ liệu không mất/hỏng · **Resiliency** = phục hồi sau lỗi do tải, tấn công, hỏng hóc · **Multi-AZ** = **High Availability** (không phải scale out/up). Deploy 2 Region → tăng **availability**, không phải agility/security/scalability.

⚠️ **Vertical scaling** = thêm CPU/RAM một máy (đổi instance type, nâng DB). **Horizontal** = thêm máy: **ELB, Read Replicas, ASG**.

🎯 **Deployment models:** on-prem + AWS → **Hybrid** (giữ dữ liệu nhạy cảm tại data center vì compliance nhưng vẫn dùng AWS cũng là Hybrid). "Mixed deployment" không tồn tại.

⚠️ **Mô hình dịch vụ:**

| Mô hình | Ví dụ | Khách hàng lo |
| --- | --- | --- |
| **IaaS** | EC2 | OS trở lên |
| **PaaS** | Elastic Beanstalk | Chỉ app + data |
| **SaaS** | Gmail, Rekognition | Chỉ dùng |
| **FaaS** | Lambda | Code |

"Product as a Service" không tồn tại.

⚠️ So với hosting truyền thống, EC2: launch/terminate bất cứ lúc nào, scale theo nhu cầu; **không** phải gói cố định theo tháng, **không** chia sẻ tài nguyên giữa người dùng.

⚠️ Sau khi migrate, khách hàng **vẫn trả license phần mềm ứng dụng**; điện, phần cứng, bảo vệ data center là của AWS. **TCO estimate** nên tính **server administration (IT labor)** và **power/cooling**; không tính quảng cáo, số người dùng, thiết bị văn phòng.

---

## 2. Well-Architected Framework & cloud design principles

🎯 **6 pillars** và từ khóa nhận diện:

| Pillar | Dấu hiệu trong đề |
| --- | --- |
| **Operational Excellence** | **IaC**, thay đổi **nhỏ, thường xuyên, đảo ngược được**, **anticipate failure**, học từ lỗi |
| **Security** | Identity mạnh, **traceability**, bảo vệ dữ liệu at rest/in transit (vd **KMS mã hóa**) |
| **Reliability** | **Tự phục hồi khi lỗi**, test recovery, scale ngang |
| **Performance Efficiency** | **Chọn đúng loại và kích thước tài nguyên**, **serverless**, go global |
| **Cost Optimization** | Consumption model, **measure overall efficiency**, phân tích chi tiêu |
| **Sustainability** | Giảm tác động môi trường |

⚠️ Bẫy: **Availability, Scalability, Elasticity KHÔNG phải pillar.**

❓ Một đề gắn right-sizing với **Elasticity** — theo Well-Architected, right-sizing thuộc **Cost Optimization**/Sustainability. Một đề khác gắn "Measure overall efficiency" với Reliability — đúng ra là **Cost Optimization**.

🎯 **Well-Architected Tool** (miễn phí) = trả lời câu hỏi về workload → so với best practice. Khác **Trusted Advisor** (check tự động trên account) và **Well-Architected Framework** (bộ nguyên tắc).

🎯 **Cloud design principles:** **loose coupling** (microservices, SQS/SNS), **automation**, **services not servers**, bảo mật ở mọi tầng. Bẫy: "close coupling", "servers not services", "security comes last", "monolithic", "synchronous".

---

## 3. Cloud Adoption Framework (CAF) & Migration

🎯 **6 perspectives:** **Business, People, Governance, Platform, Security, Operations**. Bẫy: Process, Product, Architecture **không** phải perspective (Process/Product là *transformation domains*).

| Perspective | Dấu hiệu |
| --- | --- |
| **Business** | **Strategy management**, IT đầu tư gắn mục tiêu kinh doanh, đo giá trị |
| **People** | Văn hóa, đào tạo, tổ chức |
| **Governance** | **Application portfolio management**, rủi ro, tài chính cloud |
| **Platform** | Platform engineering, kiến trúc |
| **Security** | Vulnerability management |
| **Operations** | **Performance and capacity management**, observability, incident |

🎯 **4 phases:** **Envision** (cloud giúp tăng tốc kết quả kinh doanh thế nào) → **Align** (khoảng trống năng lực) → **Launch** (pilot) → **Scale**.

🎯 **7Rs:** **Rehost** = lift and shift, **không đổi code/kiến trúc**, nhanh nhất. **Replatform** = tối ưu nhẹ (vd DB sang RDS). **Refactor** = viết lại cloud-native. **Repurchase** = sang SaaS. (Còn Retire, Retain, Relocate.)

⚠️ Chuyển tài nguyên **giữa 2 AWS Region**: tự tạo tài nguyên ở Region mới rồi chuyển dữ liệu/app; không có dịch vụ "chuyển nguyên khối" (CloudFormation, DMS, AWS Support đều không làm hộ).

⚠️ Xác định dịch vụ phù hợp khi migrate → **APN (Consulting/Services partner)**, **AWS Professional Services**. ❓ Một đề chọn thêm **Service Catalog** — thực ra Service Catalog là danh mục sản phẩm **đã duyệt** để tự launch, không phải công cụ tư vấn.

---

## 4. Global Infrastructure

🎯 **Region** = vị trí vật lý nơi AWS gom các data center. **AZ** = một hoặc nhiều data center **cùng một vị trí** với điện, mạng dự phòng; mọi AZ trong Region cách nhau ≤ 100 km; **mọi traffic giữa các AZ đều được mã hóa**.

🎯 **VPC trải trên mọi AZ của một Region**; **subnet nằm trong 1 AZ**. Region là một phần của Global Infrastructure; VPC, subnet, VPN thì không.

🎯 **Chọn Region:** **compliance/data residency**, **gần người dùng (latency)**, dịch vụ có sẵn, giá. Không phải "5G", "AZ trong 100 km", "availability index".

⚠️ **Global vs Regional:**

| Global | Regional |
| --- | --- |
| **IAM, CloudFront, Route 53, WAF (khi gắn CloudFront)** | EC2, RDS, Lambda, Rekognition, **S3 bucket** (namespace global nhưng bucket thuộc Region), EFS |

❓ Một đề coi **WorkSpaces** là dịch vụ "global" (dựa trên trang Global Infrastructure) — thực tế WorkSpaces tạo theo Region. Một đề coi **WAF** là global — WAF có **2 scope**: global (CloudFront) và regional (ALB, API Gateway, AppSync…).

🎯 **Edge & hybrid:**

| Dịch vụ | Dùng khi |
| --- | --- |
| **Local Zones** | Compute/storage gần người dùng ở thành phố lớn, latency thấp (game) |
| **Wavelength** | AWS ở rìa **mạng 5G**, ứng dụng thiết bị di động latency cực thấp (out-of-scope CLF-C02) |
| **Outposts** | Hạ tầng và API AWS **tại data center của bạn**, xử lý dữ liệu tại chỗ (nhà máy) |
| **Edge locations** | Điểm cache của CloudFront, dùng chung với Global Accelerator |

---

## 5. Shared Responsibility Model

🎯 **AWS = OF the cloud:** data center, phần cứng, **mạng toàn cầu**, **edge locations**, **AZ**, vá **hạ tầng mạng**, hủy ổ đĩa, đảm bảo nhân viên AWS không truy cập dữ liệu, compliance hạ tầng, **Shield Standard**, quản lý nền của dịch vụ abstracted (S3, DynamoDB: AWS lo hạ tầng, OS, platform).

🎯 **Khách hàng = IN the cloud:** dữ liệu, **mã hóa** (kể cả **mã hóa DB RDS**, SSE), **IAM user/role**, **bật MFA**, review quyền, **guest OS patch** của EC2, **security group/firewall & network config của EC2**, **WAF**, **Shield Advanced** (tự đăng ký).

⚠️ **Theo từng dịch vụ:**

| Dịch vụ | AWS lo | Khách lo |
| --- | --- | --- |
| **EC2** (IaaS) | Hạ tầng, host OS | Guest OS, patch, SG, app, data |
| **RDS** | Vá OS và DB engine | **Mã hóa DB**, security group, user trong DB |
| **Lambda** | OS, runtime, mạng, server | **Code, dependency, version của function** |
| **S3 / DynamoDB** | Hạ tầng, OS, platform | Data, quyền truy cập, mã hóa |

🎯 **Shared controls (cả hai bên):** **Configuration management** (AWS cấu hình thiết bị hạ tầng, khách cấu hình guest OS/DB/app), **Patch management**, **Awareness & training** (mỗi bên đào tạo nhân viên của mình). **OS configuration** cũng là chia sẻ: host OS → AWS, guest OS → khách.

---

## 6. IAM

🎯 **Root user:** credential = email + mật khẩu tạo account; **toàn quyền, không giới hạn được bằng IAM policy**; đổi được mật khẩu; bật MFA; **không tạo access key** cho root (có thì xóa); không chia sẻ; tạo IAM user admin cho việc quản trị.

📌 Danh sách tác vụ "chỉ root làm được" **không còn** "đổi/hủy support plan". MFA cho root nay **bắt buộc**.

🎯 **Best practices:** **MFA cho mọi user**, **xoay vòng credential**, **least privilege**, không chia sẻ credential, **dùng IAM role thay vì chia sẻ credential giữa account**.

🎯 Đăng nhập Console = **username + password + MFA**. **Access key** (Access Key ID + Secret) gắn với **IAM user** (hoặc root), dùng cho **CLI/SDK**. 3 cách truy cập AWS: **Console, CLI, SDK**. X.509 là chứng chỉ (ACM).

🎯 EC2 truy cập DynamoDB/S3 → **IAM Role** (credential tạm), không dùng access key.

🎯 **Policy:** phần tử bắt buộc **Effect + Action** (Resource bắt buộc tùy loại policy; Sid, Condition tùy chọn; Principal chỉ trong resource-based policy).

⚠️ **Credentials Report** = mọi user + trạng thái password, access key, MFA. **Access Advisor** = **quyền đã cấp** cho user và lần cuối dùng.

⚠️ **IAM là global**: nhân viên đi công tác vẫn dùng được, không cần tạo user mỗi Region.

🎯 **Identity khác:** **STS** = credential **tạm thời**, giới hạn quyền cho user tin cậy/federated. **IAM Identity Center** = SSO nhiều account + app SAML (kế nhiệm AWS SSO). **Cognito** = sign-up/sign-in cho **web/mobile app**. **Directory Service** = AD trên AWS.

⚠️ **MFA:** không cần thiết bị vật lý → **Virtual MFA app**. Cắm USB → **security key (FIDO)**.

📌 IAM nay có **3 loại MFA**: **Passkeys & security keys (FIDO)**, **Virtual authenticator app**, **Hardware TOTP token**. **SMS không còn** là MFA của IAM; "U2F" nay gọi là FIDO security key. Tối đa **8 thiết bị MFA**/user.

---

## 7. Organizations, SCP & Control Tower

🎯 **Organizations:** **consolidated billing**, **một phương thức thanh toán**, **gộp volume discount** EC2/S3, **chia sẻ RI/Savings Plans**; tạo account bằng API. Best practice: **account theo phòng ban/môi trường**, **SCP** giới hạn quyền, dùng tags cho billing, bật CloudTrail mọi account.

📌 "Master account" nay gọi **management account**.

🎯 **SCP:** **không cấp quyền**, chỉ đặt giới hạn tối đa; **không ảnh hưởng management account**.

🎯 **Control Tower:** dựng **landing zone** multi-account với blueprint và controls; vận hành hằng ngày vẫn là việc của khách.

📌 "Guardrails" của Control Tower nay gọi **controls** (preventive, detective, proactive).

⚠️ **Hóa đơn riêng vs phân bổ chi phí** (không mâu thuẫn): muốn **hóa đơn riêng** dev/prod → **tách AWS account**; muốn **phân bổ chi phí** theo phòng ban → **cost allocation tags**. Organizations/Cost Explorer/tags **không** tạo hóa đơn riêng.

⚠️ **Consolidated billing & RI:** A có 5 RI dùng 3, B dùng 6 → **5 instance tính giá RI, 4 tính On-Demand**. ❓ Một đề nói RI chỉ chia sẻ "trong cùng AZ" — đúng với **Zonal RI**; **Regional RI** áp dụng cho mọi AZ trong Region.

⚠️ **Service Catalog** = danh mục sản phẩm IT **đã được duyệt** cho user tự launch. **RAM** = chia sẻ tài nguyên giữa account.

---

## 8. Mã hóa: KMS, CloudHSM, ACM, Secrets Manager

🎯 **Mã hóa mặc định, không cần cấu hình:** **S3** (SSE-S3), **S3 Glacier**, **Storage Gateway**, **CloudTrail log files**. **Không** mặc định (phải bật): EBS, Redshift, EFS, CloudFront, ALB, CloudWatch Logs…

🎯 **3 loại KMS key:**

| Loại | Đặc điểm |
| --- | --- |
| **AWS managed key** (`aws/s3`…) | **Ít công nhất**, chỉ dùng trong account của bạn |
| **Customer managed key** | Bạn kiểm soát policy/rotation, **tốn phí** |
| **AWS owned key** | AWS dùng cho nhiều account, bạn không xem được |

📌 Tên hiện hành là **KMS key**. KMS chỉnh được **rotation period** và **rotate on-demand**.

🎯 **CloudHSM** = **thiết bị HSM phần cứng riêng**, compliance bắt buộc phần cứng, sinh/lưu/quản lý key chỉ bạn truy cập.

📌 HSM hsm2m.medium đạt **FIPS 140-3 Level 3**.

⚠️ **ACM** = cấp/quản lý chứng chỉ TLS. **Secrets Manager** = lưu và **tự xoay vòng** secret; **không** tạo key mã hóa, **không** phát hiện PII.

---

## 9. Bảo vệ ứng dụng & mạng: WAF, Shield, Firewall Manager

🎯 **AWS WAF** = lọc request **Layer 7**: **SQL injection, XSS, geo-match chặn quốc gia, rate-based**; chặn tất cả trừ request được phép. Gắn vào **CloudFront, ALB, API Gateway, AppSync** (không trực tiếp EC2, không Route 53); chạy ở mọi edge khi gắn CloudFront.

📌 Tên đúng là **AWS WAF** (không phải "Amazon WAF"). WAF nay còn gắn được Cognito user pool, App Runner, Verified Access, Amplify.

🎯 **Shield Standard** = DDoS L3/L4, **miễn phí, tự bật**, AWS quản lý. **Shield Advanced** = **trả phí cho mọi khách (kể cả Enterprise)**, đội hỗ trợ 24/7, **bảo vệ chi phí tăng do DDoS**; tài nguyên: **EC2, ELB, CloudFront, Route 53, Global Accelerator** (không S3, Beanstalk, IAM).

📌 "DDoS Response Team (DRT)" nay gọi **Shield Response Team (SRT)**.

⚠️ **Chống DDoS:** **Shield**, **WAF** (rate-based), **CloudFront + Route 53** (edge). Inspector, CloudHSM, Trusted Advisor **không** chống DDoS. Auto Scaling **không** tự deploy Shield khi bị DDoS.

⚠️ **Firewall Manager** = quản lý rule WAF/Shield/SG tập trung cho nhiều account.

⚠️ **NACL** chặn được IP cụ thể (có Deny); **Security Group** chỉ Allow, kiểm soát traffic vào EC2 (xem §20).

🎯 **S3 Block Public Access** = cách hiệu quả nhất để giữ dữ liệu luôn private (đè lên policy/ACL).

---

## 10. Phát hiện mối đe dọa, audit & compliance

🎯 **Bộ dịch vụ bảo mật — hỏi theo việc cần làm:**

| Dịch vụ | Làm gì | Không làm |
| --- | --- | --- |
| **GuardDuty** | **Threat detection** liên tục (hoạt động độc hại, hành vi trái phép) từ CloudTrail, VPC Flow Logs, DNS logs; finding giữ 90 ngày | Không chặn request, không quét lỗ hổng |
| **Inspector** | **Lỗ hổng phần mềm** và network reachability của EC2/ECR/Lambda, tự động | Không theo dõi thay đổi cấu hình (đó là Config) |
| **Macie** | Phát hiện **dữ liệu nhạy cảm (PII)** ở quy mô lớn trong S3 | Không enforce quyền |
| **Detective** | Điều tra **root cause** từ **CloudTrail + VPC Flow Logs + GuardDuty findings** | |
| **Security Hub** | Gom finding nhiều dịch vụ, nhiều account | Không tự chặn tấn công |
| **Config** | **Lịch sử cấu hình** để chứng minh compliance, **thông báo khi cấu hình đổi** | |
| **Artifact** | Tải báo cáo compliance (**PCI, SOC, ISO**) và thỏa thuận | Không mua qua Marketplace |

📌 **Inspector** "rules packages", "assessment templates" là mô tả của **Inspector Classic**; Inspector hiện tại quét liên tục EC2 (agent SSM hoặc agentless), ECR, Lambda.

📌 **Security Hub CSPM** kiểm tra tư thế bảo mật và gom finding; **Security Hub** (GA 12/2025) là giải pháp hợp nhất.

⚠️ **Penetration testing:** tự kiểm thử hạ tầng của mình **không cần xin phép** (với các dịch vụ được phép); **network stress test** là khái niệm khác.

⚠️ **AWS Abuse team** = IP của AWS đang **port scan**, spam, DDoS… hệ thống của bạn (không phải AWS Support).

⚠️ **Acceptable Use Policy** = quy định cách dùng bị cấm. "Fair Use Policy", "Applicable Use Policy" **không tồn tại**.

---

## 11. Giám sát: CloudWatch, CloudTrail, Config, Health Dashboard

🎯 **Bộ giám sát — hỏi theo "câu đề đặt ra":**

| Câu hỏi | Dịch vụ |
| --- | --- |
| **Hiệu năng**, metric, resource utilization, alarm, log (kể cả **on-prem** qua agent) | **CloudWatch** |
| **Ai** gọi API gì, khi nào (vd ai sửa security group) | **CloudTrail** |
| API activity **bất thường** so với baseline | **CloudTrail Insights** |
| **Lịch sử cấu hình**, thay đổi 2 tuần qua, compliance | **AWS Config** |
| Trace request trong ứng dụng phân tán | **X-Ray** |
| Tình trạng **chung** mọi dịch vụ (có **RSS**) | **Health Dashboard – Service health** |
| Sự kiện AWS **ảnh hưởng tới tài nguyên của bạn** + hướng dẫn khắc phục (mọi support plan) | **Health Dashboard – Your account** |
| Audit **request** gửi tới bucket | **S3 server access logs** |

⚠️ CloudTrail mặc định ghi **management events**; data events và Insights phải bật thêm.

🎯 **Tình huống hay ra:**
- Cảnh báo CPU > 80% qua email → **CloudWatch alarm + SNS**.
- Cảnh báo khi **root đăng nhập** → **EventBridge rule + SNS**.
- Chạy log backup mỗi thứ Hai 2h sáng, 5 phút, serverless → **EventBridge Scheduler + Lambda**.
- Gom log **EC2 và on-prem** → **CloudWatch Logs** (agent), không phải CloudTrail.

📌 **CloudWatch Events** nay là **Amazon EventBridge**. X-Ray SDK/Daemon vào **maintenance mode từ 25/02/2026** → chuyển sang OpenTelemetry.

---

## 12. EC2

🎯 **AMI** = thông tin cần để launch EC2; **phải ở cùng Region** với instance (copy AMI sang Region khác để dùng); Region của AMI không ảnh hưởng hiệu năng.

🎯 **User data** = bootstrap script lúc launch. **Instance metadata** = thông tin về instance (ami-id, IP…).

⚠️ **Instance types:**

| Loại | Dùng khi |
| --- | --- |
| **Memory Optimized** | In-memory cache/DB |
| **Compute Optimized** | HPC, web hiệu năng cao, batch |
| **Storage Optimized** | I/O tuần tự lớn trên local storage |
| **Accelerated Computing** | GPU, FPGA |
| **General Purpose** | Cân bằng |

🎯 **Kết nối vào EC2:** **EC2 Instance Connect** = SSH qua **trình duyệt** từ Mac/Windows/Linux (cần port 22). **SSM Session Manager** = shell **không mở port, không public IP, không SSH key**. PuTTY chỉ cho Windows.

---

## 13. EC2 purchasing options & Savings Plans

🎯 **Bảng chọn:**

| Lựa chọn | Đặc điểm | Chọn khi |
| --- | --- | --- |
| **On-Demand** | Trả theo giây (Linux/Windows, sau phút đầu), không cam kết, **không bị gián đoạn** | Ngắn hạn, đột biến, **critical**, không đoán trước |
| **Reserved Instances** | Cam kết **1 hoặc 3 năm**; có cho **EC2, RDS** (cả ElastiCache, Redshift, DynamoDB reserved capacity) | Dùng **ổn định, dự đoán được** |
| **Convertible RI** | Đổi được family, OS, tenancy | Dài hạn nhưng **workload sẽ thay đổi/tăng** |
| **Savings Plans** | Cam kết **$/giờ** 1–3 năm | Muốn linh hoạt hơn RI |
| **Spot** | Rẻ tới **90%**, có thể bị thu hồi | Batch, phân tích, container chịu gián đoạn, job linh hoạt thời gian |
| **Dedicated Host** | Server vật lý riêng, **đắt nhất** | **BYOL** theo socket/core, compliance |
| **Capacity Reservation** | Giữ capacity ở 1 AZ, **không giảm giá** | Bắt buộc có capacity ở AZ cụ thể |

⚠️ RI **không có** kỳ hạn 6 tháng, 2 năm, 5 năm. CloudFront, S3, IAM **không có** RI.

📌 Savings Plans nay có **4 loại**: **Compute** (tới 66%), **EC2 Instance** (tới 72%), **SageMaker AI** (tới 64%), **Database** (tới 35%). Docs ghi RI/EC2 Instance SP **tới 72%**; CLF thường chỉ hỏi "RI rẻ hơn On-Demand, Spot rẻ nhất".

❓ **Scheduled Reserved Instances**: một đề gợi ý dùng cho job định kỳ, đề khác nói AWS không còn hỗ trợ. Thực tế **không còn mua được** → job định kỳ không được gián đoạn → On-Demand/Savings Plans; chịu gián đoạn → Spot.

📌 Spot bị thu hồi chủ yếu khi **EC2 cần lại capacity** (không còn đấu giá theo giá bid).

🎯 **Tối ưu chi phí EC2:** **Auto Scaling** khớp nhu cầu + **RI/SP** cho phần ổn định + Spot cho job chịu gián đoạn. Vertical scaling hay nâng support plan **không** giảm chi phí.

---

## 14. Auto Scaling & Elastic Load Balancing

🎯 **EC2 Auto Scaling:** scale **out/in** theo nhu cầu, **thay instance unhealthy** (fault tolerance), đăng ký instance vào ELB. **Không** scale up/down (đó là vertical).

⚠️ **Predictive scaling** = tải lặp lại theo ngày/giờ. **Target tracking** = giữ một metric (vd CPU 40%).

🎯 **ELB:** **ALB** cho HTTP/HTTPS (L7), **NLB** cho TCP/UDP/TLS hiệu năng cực cao (L4); lợi ích chính **HA, fault tolerance**. "System Load Balancer" **không tồn tại**.

📌 **Classic Load Balancer** là thế hệ trước (previous generation).

---

## 15. Serverless, containers & PaaS

🎯 **Lambda:** chạy code không quản server, trả theo **request + thời gian chạy** (không theo ngôn ngữ, số dòng code, kích thước package); event-driven (S3 upload, DynamoDB update), tiền xử lý dữ liệu trước khi lưu S3. **Không** truy cập OS, **không** cài DB.

📌 Lambda tính thời gian theo **1 ms**.

❓ **Lambda và container:** đề nói "Lambda không chạy container", đề khác nói Lambda deploy được **container image tới 10 GB**. Thực tế Lambda **chạy được container image** nhưng **không điều phối/quản lý container**. Câu "chạy ứng dụng container" → chọn **ECS/EKS/Fargate**.

⚠️ **Serverless compute:** **Lambda, Fargate**. **Không** serverless: EC2, Elastic Beanstalk (vẫn provision server), Lightsail.

🎯 **Container:** **ECS (EC2 launch type)** = chạy container **vẫn truy cập server** · **Fargate** = container không quản server · **EKS** = Kubernetes · **ECR** = lưu Docker image.

🎯 **AWS Batch** = hàng trăm nghìn batch job, rẻ hơn tự chạy trên Fargate. Lambda có giới hạn thời gian (15 phút).

🎯 **Lightsail** = VPS giá thấp cố định, cho team mới, **dev/test**, **WordPress**.

🎯 **Elastic Beanstalk** = PaaS: upload code, Beanstalk lo capacity, load balancing, **auto scaling**, health. **Vẫn truy cập được tài nguyên/OS bên dưới**; hỗ trợ Java, .NET, PHP, Node.js, Python, Ruby, Go, **Docker**; chạy được cả **non-web app** (worker); **miễn phí**, chỉ trả tài nguyên. Basic health chỉ kiểm tra ASG có ít nhất 1 instance, không đẩy metric riêng lên CloudWatch.

---

## 16. S3

🎯 **S3** = object storage key-value, **cấu trúc phẳng**; host **website tĩnh** ít công nhất (lỗi truy cập → sửa **bucket policy**/Block Public Access). Không cài DB lên S3. Mọi class: durability **11 số 9**.

🎯 **Storage classes:**

| Class | Điểm nhận diện | Phí retrieval / tối thiểu |
| --- | --- | --- |
| **Standard** | Truy cập thường xuyên | Không phí retrieval, **không** thời gian lưu tối thiểu |
| **Intelligent-Tiering** | **Pattern truy cập không biết trước/ngẫu nhiên** | **Không phí retrieval**, có phí monitoring |
| **Standard-IA** | Ít truy cập, cần lấy nhanh, nhiều AZ | Có phí retrieval, tối thiểu 30 ngày |
| **One Zone-IA** | Ít truy cập, **dữ liệu tạo lại được** (thumbnail), **bản backup thứ cấp**; **availability thấp nhất** (99.5%) | Có phí, 30 ngày |
| **Glacier Instant Retrieval** | Archive, lấy trong mili giây | 90 ngày |
| **Glacier Flexible Retrieval** | Archive, lấy trong phút–giờ, Bulk miễn phí | 90 ngày |
| **Glacier Deep Archive** | **Rẻ nhất**, giữ **7–10 năm** vì compliance, lấy 12–48 giờ | 180 ngày |

📌 Nay có thêm **S3 Express One Zone** (1 AZ, nhanh hơn tới 10 lần, chi phí request thấp hơn 80%).

🎯 **Tính năng S3:**
- **Versioning** = chống xóa nhầm.
- **Replication** = giữ metadata, copy sang account test; **CRR** cho yêu cầu lưu xa về địa lý.
- **Lifecycle** = chuyển class/xóa object theo rule thời gian.
- **Block Public Access** = giữ dữ liệu luôn private.
- **Server access logs** = audit request.
- **Transfer Acceleration** = xem §21.

📌 Bucket mới **tắt ACL mặc định** (Object Ownership = bucket owner enforced) → quyền quản lý bằng policy.

---

## 17. Block/file storage, hybrid storage & chuyển dữ liệu

🎯 **EBS** = block storage, **khóa 1 AZ**, gắn 1 instance (mức CLF), tồn tại sau khi terminate (nếu cấu hình); tính phí theo **volume type + provisioned IOPS + GB provisioned**; **snapshot lưu trong S3**, **incremental** (tính tiền phần thay đổi).

📌 EBS snapshot archive rẻ hơn 75%, restore tới 72h. **Recycle Bin** giữ snapshot đã xóa 1–365 ngày, volume 1–7 ngày.

🎯 **Instance Store** = block tạm thời, **cache/buffer/scratch**, mất khi stop/terminate.

🎯 **EFS** = NFS dùng chung, **truy cập song song** nhiều EC2, nhiều AZ (và qua VPC/Region), dùng được từ **on-prem** qua Direct Connect/VPN; **HA mặc định**; **EFS-IA** tính phí mỗi lần đọc/ghi, hợp **file cần giữ để audit**; không làm boot volume.

📌 EFS nay có thêm class **Archive**.

⚠️ **HA mặc định:** **EFS, DynamoDB** (không phải EBS, subnet, instance store).

🎯 **FSx for Windows File Server** = SMB, Windows app, AD.

🎯 **Storage Gateway** = hybrid storage, **3 loại: File, Volume, Tape** Gateway. "Object Gateway", "Block Gateway" **không tồn tại**.

📌 **FSx File Gateway** đóng với khách mới.

🎯 **DataSync** = truyền dữ liệu **online**, tự động, **incremental**, liên tục từ on-prem.

🎯 **Snowball** = khối lượng lớn, băng thông hạn chế, chuyển offline.

📌 **Snowball / Snowball Edge / OpsHub (Snow Family) không nhận khách mới từ 07/11/2025** → khách mới dùng **DataSync** (online), **AWS Data Transfer Terminal** (vận chuyển vật lý), **Outposts** (edge compute).

🎯 **AWS Backup** = backup tập trung; trả tiền cả **lượng backup lưu và lượng restore**.

🎯 **Elastic Disaster Recovery (DRS)** = DR replicate liên tục.

---

## 18. Database

🎯 **Chọn database:**

| Nhu cầu | Chọn |
| --- | --- |
| Quan hệ, chuyển từ MySQL/PostgreSQL, hiệu năng cao (nhanh hơn MySQL tới 5 lần) | **Aurora** |
| Quan hệ managed thông thường | **RDS** |
| NoSQL **schemaless**, key-value/document, **ít vận hành nhất ở mọi quy mô** | **DynamoDB** |
| Cache cho DynamoDB, đọc nhiều mùa cao điểm (micro giây) | **DAX** |
| Cache giảm tải DB đọc nhiều, in-memory | **ElastiCache** |
| **Knowledge graph**, quan hệ phức tạp | **Neptune** |
| Document, MongoDB | **DocumentDB** (Neptune và DocumentDB đều là **NoSQL**) |

📌 Aurora tới **256 TiB** (giới hạn thực tế tùy engine version). ElastiCache nay hỗ trợ **Valkey, Redis OSS, Memcached**.

🎯 **RDS Read Replica** = scale đọc (horizontal). **Multi-AZ một standby** = HA, **không** phục vụ đọc. Multi-AZ **hai readable standby** (DB cluster) thì đọc được. **Cross-Region** read replica → DR.

❓ **Read Replica và DR:** đề nói RR **không** tăng availability, **không** chống sự cố Region; đề khác nói cả RR lẫn Multi-AZ đều dùng cho DR. Hai câu khác ngữ cảnh: RR cùng Region = scale đọc; **cross-Region RR** dùng được cho DR khi mất Region; Multi-AZ = HA trong Region. Câu "lợi ích **chính** của RR" → **scale đọc**.

📌 **Timestream for LiveAnalytics** đóng với khách mới từ 20/06/2025.

---

## 19. Analytics

🎯 **Bảng chọn:**

| Nhu cầu | Chọn |
| --- | --- |
| **OLAP**, data warehouse | **Redshift** |
| SQL trên dữ liệu **trong S3**, serverless, ít công nhất | **Athena** |
| Hadoop/Spark cluster | **EMR** |
| **ETL**, chuẩn bị dữ liệu cho analytics, serverless | **Glue** |
| Streaming big data **real-time** (ad-tech, clickstream) | **Kinesis Data Streams** |
| Dashboard BI tương tác, truy cập mọi thiết bị | **Quick Sight** |
| Search/log analytics | **OpenSearch Service** |

📌 **Amazon QuickSight** nay là **Amazon Quick Sight** (thuộc Amazon Quick).

---

## 20. VPC & kết nối mạng

🎯 **Thành phần VPC:** **subnet, Internet Gateway**, route table, NAT Gateway, endpoint, SG, NACL. Storage Gateway, API Gateway, S3 object **không** thuộc VPC.

🎯 **Security Group vs NACL:**

| | Security Group | Network ACL |
| --- | --- | --- |
| Phạm vi | **Instance** (ENI) | **Subnet** |
| Rule | **Chỉ Allow** | **Allow và Deny** → chặn được IP cụ thể |
| Trạng thái | **Stateful** (traffic trả về tự cho phép) | **Stateless** |
| Đánh giá | Xét **tất cả** rule | Theo **số thứ tự tăng dần** |

🎯 **Bảng chọn kết nối:**

| Nhu cầu | Chọn |
| --- | --- |
| VPC truy cập **S3/DynamoDB** riêng tư | **VPC Gateway Endpoint** |
| VPC truy cập **SQS** (và hầu hết dịch vụ khác) riêng tư | **VPC Interface Endpoint** (PrivateLink) |
| Nối **2 VPC** | **VPC Peering** (không bắc cầu) |
| Nối **nhiều VPC + on-prem** | **Transit Gateway** (thường kèm Direct Connect/VPN) |
| On-prem ↔ AWS **riêng tư, băng thông cao, ổn định** | **Direct Connect** (≥ 1 tháng triển khai, không mã hóa sẵn) |
| On-prem ↔ AWS **mã hóa, dựng nhanh**, qua Internet | **Site-to-Site VPN** = **Customer Gateway** (phía công ty) + **Virtual Private Gateway** (phía AWS) |
| Máy cá nhân vào VPC | **Client VPN** |

📌 Mọi **public IPv4** (kể cả Elastic IP đang gắn vào instance chạy) bị **tính phí từ 02/2024** (Free Tier có 750 giờ/tháng cho EC2).

---

## 21. Content delivery & DNS

🎯 **Tăng tốc & phân phối toàn cầu:**

| Dịch vụ | Dùng khi | Bẫy |
| --- | --- | --- |
| **CloudFront** | Website/nội dung **tĩnh** cho người dùng toàn cầu, cache ở edge | Không phải để tăng tốc upload |
| **S3 Transfer Acceleration** | **Upload/download S3** từ xa, từ nhiều nơi về **một bucket trung tâm** | Không tăng tốc website |
| **Global Accelerator** | **2 static anycast IP**, TCP/UDP, **non-HTTP** (game, IoT, VoIP), failover nhanh; target được ELB | **Không cache**, không host website; **dùng chung edge location** với CloudFront |
| **Route 53** | DNS, **domain registration**, **health check & monitoring** | Không phải load balancer, không IP routing, không transfer acceleration |

🎯 **Route 53 routing:** **Simple** (một tài nguyên) · **Weighted** (tỉ lệ) · **Latency** (Region nhanh nhất) · **Failover** (**active-passive**) · Geolocation · Geoproximity · Multivalue.

---

## 22. Integration: SQS, SNS, MQ, EventBridge, API Gateway

🎯 **Bảng chọn:**

| Dịch vụ | Dùng khi |
| --- | --- |
| **SQS** | Decouple, gửi/lưu/nhận message, **pull**, **không mất message** khi bên nhận chưa sẵn sàng |
| **SNS** | Pub/sub **push**, fan-out; có thể mất nếu bên nhận không chạy |
| **Amazon MQ** | Chuyển message broker on-prem (**ActiveMQ/RabbitMQ**) lên cloud dễ nhất |
| **Kinesis Data Streams** | Streaming real-time |
| **EventBridge** | Event-driven, rule theo sự kiện, lịch (Scheduler) |
| **Step Functions** | Workflow nhiều bước |

🎯 **API Gateway:** tạo REST, HTTP, WebSocket API; front door gọi Lambda; tích hợp trực tiếp Kinesis; **có caching**; request trả từ cache **vẫn tính là API call**.

---

## 23. Developer tools, IaC & quản lý vận hành

🎯 **CloudFormation** = IaC; **StackSets** triển khai giống nhau qua **nhiều account/Region**; ước tính chi phí từ template. **CDK** = IaC bằng Python, JavaScript… (sinh CloudFormation).

📌 Console riêng của **Infrastructure Composer** tắt ngày **07/12/2026** → dùng trình soạn nhúng trong console CloudFormation hoặc AWS Toolkit for VS Code.

🎯 **Bộ Code\*:**

| Dịch vụ | Làm gì |
| --- | --- |
| **CodeCommit** | Git private, version control (📌 mở lại cho khách mới từ 25/11/2025) |
| **CodeBuild** | Build/test |
| **CodeDeploy** | Deploy tự động lên **EC2 và on-prem** (out-of-scope CLF-C02 nhưng vẫn hay gặp) |
| **CodePipeline** | Điều phối CI/CD, **không tự deploy** |
| **CodeArtifact** | Package (Maven, npm…) |

📌 **CodeStar** ngừng 31/07/2024 (nguồn web). **CodeGuru Reviewer** không tạo repository association mới từ 07/11/2025.

🎯 **Systems Manager** = quản lý server AWS **và on-prem** (inventory, run command, patch, Session Manager), quản lý node tập trung.

🎯 **EC2 Image Builder** = tự động tạo, cập nhật image cho EC2 và on-prem. **Resource Groups** = tổ chức và tự động hóa tác vụ trên nhiều tài nguyên theo tag.

⚠️ Triển khai nhanh công nghệ phổ biến theo best practice → **AWS Solutions Library**. ❓ Tên "Partner Solutions" trong đề chưa kiểm chứng được qua docs.

---

## 24. Machine Learning & dịch vụ khác

🎯 **AI/ML:**

| Dịch vụ | Làm gì |
| --- | --- |
| **Rekognition** | Nhận diện vật thể, người, text trong ảnh/video (**không** resize, greyscale, pose detection) |
| **Translate** | Dịch (vd phụ đề) |
| **Transcribe** | Speech-to-text |
| **Polly** | Text-to-speech |
| **Lex** | Chatbot với NLU |
| **Comprehend** | NLP, sentiment |
| **SageMaker AI** | Data scientist build/train/deploy model |
| **Personalize** | Gợi ý cá nhân hóa (out-of-scope) |
| **Kendra** | Tìm tài liệu nội bộ (📌 đóng với khách mới từ 30/07/2026 → Bedrock Knowledge Bases) |
| **Forecast** | Dự báo time series (📌 đóng với khách mới từ 29/07/2024 → SageMaker Canvas) |
| **Fraud Detector** | Phát hiện gian lận (📌 không nhận khách mới từ 07/11/2025) |

🎯 **Khác:**

| Dịch vụ | Làm gì |
| --- | --- |
| **IoT Core** | Kết nối thiết bị IoT không quản server ("IoT Gateway" không tồn tại) |
| **Connect** | Contact center |
| **WorkSpaces** | Desktop ảo (DaaS) |
| **WorkSpaces Applications** | Stream ứng dụng desktop qua trình duyệt |
| **Device Farm** | Test app trên thiết bị thật (out-of-scope) |
| **Fault Injection Service (FIS)** | Chaos engineering |
| **Elastic Transcoder** | Chuyển định dạng media trong S3 (📌 ngừng hẳn 13/11/2025, nguồn web → **MediaConvert**) |
| **Pinpoint** | Marketing engagement (📌 hết hỗ trợ 30/10/2026 → Connect, End User Messaging) |
| **Marketplace** | Mua AMI/SaaS bên thứ ba, bán SaaS; **không** mua RI hay Direct Connect |
| **AWS Architecture Center** | Kiến trúc tham khảo |
| **re:Post / Knowledge Center** | Hỏi đáp cộng đồng / câu hỏi phổ biến và lời giải từ AWS (📌 thay AWS Forums) |

---

## 25. Nguyên tắc giá & dịch vụ miễn phí

🎯 **3 yếu tố chi phí chính:** **compute, storage, outbound data transfer**. **Inbound miễn phí**; chỉ **outbound** bị tính.

🎯 **S3 miễn phí:** **data transfer in từ Internet**, **ra EC2 cùng Region**, ra CloudFront. Storage S3/Glacier luôn tính phí.

🎯 **Dịch vụ miễn phí** (chỉ trả tài nguyên bên dưới): **IAM, AWS Auto Scaling, Elastic Beanstalk, Organizations, CloudFormation**. **Không** miễn phí: Spot, detailed monitoring, Elastic IP/public IPv4.

⚠️ **On-Demand Windows/Linux** tính **theo giây**.

📌 **AWS Free Tier** cho account mới nay là **Free plan**: credit $100 + tối đa $100 thêm, dùng trong 6 tháng.

---

## 26. Công cụ chi phí & cost allocation tags

🎯 **Bảng chọn công cụ:**

| Công cụ | Dùng khi |
| --- | --- |
| **Pricing Calculator** | **Ước tính trước** khi triển khai |
| **Cost Explorer** | **Xem, phân tích, dự báo** chi phí; **rightsizing recommendations** EC2; **gợi ý Savings Plans** |
| **Cost & Usage Report (CUR)** | Dữ liệu **chi tiết nhất**, theo **giờ**, đẩy vào **S3** |
| **Budgets** | Cảnh báo khi chi phí/usage **vượt hoặc dự báo vượt**; **RI/SP utilization/coverage** thấp. **4 loại: Cost, Usage, Reservation, Savings Plans** |
| **CloudWatch billing alarm** | Cảnh báo khi **chi phí thực tế** vượt ngưỡng; metric ở **us-east-1** |
| **Cost Anomaly Detection** | ML phát hiện chi tiêu bất thường |
| **Trusted Advisor** | Phát hiện **EC2 dùng thấp**, **EBS không gắn/dùng ít**, S3 public, chưa bật CloudTrail, root chưa MFA… |
| **Compute Optimizer** | Rightsizing bằng ML: **EC2, Auto Scaling group, EBS, Lambda** (không S3, EFS) |
| **Cost allocation tags** | Phân bổ chi phí theo phòng ban/dự án; mỗi key một value; **phải activate** cả tag AWS-generated lẫn user-defined |

📌 Cost Explorer dự báo **tới 18 tháng**; lịch sử mặc định 13 tháng. **CUR** nên dùng **CUR 2.0 qua Data Exports**.

❓ **So sánh chi phí on-prem với AWS (TCO):** đề chọn **Pricing Calculator**. Pricing Calculator ước tính chi phí **trên AWS**. Business case đầy đủ on-prem vs AWS → **Migration Evaluator**. Không có Migration Evaluator trong đáp án thì chọn Pricing Calculator.

🎯 **Tình huống hay ra:** tìm EC2 dùng thấp không cần cấu hình → **Trusted Advisor + Cost Explorer (rightsizing)**; báo cáo chi phí theo giờ vào S3 → **CUR**; cảnh báo RI utilization thấp → **Budgets**; chọn Savings Plan tối ưu → **Cost Explorer**; billing alarm → **CloudWatch**.

---

## 27. Trusted Advisor

🎯 **Category:** Cost optimization, Performance, Security, Fault tolerance, Service limits, **Operational Excellence**. "Elasticity, Documentation, Change Management" **không** phải category.

📌 Trusted Advisor có **6 category**. Gói **Basic** chỉ có check **Service Limits** + một số check **Security, Fault Tolerance**.

⚠️ **Security check:** **S3 bucket public**, **chưa bật CloudTrail**, root chưa MFA, port mở. Không phát hiện việc chia sẻ credential, không yêu cầu tag S3.

---

## 28. Support plans

🎯 **4 gói hiện hành:** **Basic** → **Business Support+** → **Enterprise** → **Unified Operations**.

| Gói | Giá tối thiểu | Phản hồi sự cố nặng nhất | Điểm nhận diện |
| --- | --- | --- | --- |
| **Basic** | Miễn phí, mọi account | Không có hỗ trợ kỹ thuật | Hỏi đáp account & billing 1-1, tăng service quota, forum, service health checks, docs |
| **Business Support+** | **$29/tháng** mỗi account | **< 30 phút** (business-critical system down) | Trợ lý **AI (generative AI)** theo ngữ cảnh; **24x7 phone/web/chat** với **Cloud Support Engineers**; **500+ check Trusted Advisor**; **Support API**; hỗ trợ phần mềm bên thứ ba; mở case không giới hạn |
| **Enterprise** | **$5,000/tháng** | **≤ 15 phút** (production-critical) | Mọi thứ của Business Support+ **+ TAM được chỉ định** + **AWS Security Incident Response** (không tính thêm) + strategic business review + **AWS Countdown** (hỗ trợ sự kiện quan trọng) |
| **Unified Operations** | Theo yêu cầu & độ phức tạp workload | **5 phút** (critical incident) | **TAM + Domain Specialist Engineer** được chỉ định; **Infrastructure Event Management**; tư vấn kiến trúc ứng dụng; giám sát bảo mật & hiệu năng 24/7; **AWS Countdown Premium**; review workload quan trọng, runbook riêng |

🎯 **Chọn nhanh:**

| Yêu cầu trong đề | Gói |
| --- | --- |
| Chỉ cần hỏi billing/account, tăng quota | **Basic** |
| Hỗ trợ kỹ thuật 24x7 qua phone/chat **rẻ nhất** | **Business Support+** |
| Toàn bộ check Trusted Advisor, **Support API** (tự động tạo/đóng case) | **Business Support+** trở lên |
| Hỗ trợ cấu hình OS EC2, phần mềm bên thứ ba phổ biến | **Business Support+** trở lên |
| **TAM** được chỉ định, phản hồi ≤ 15 phút | **Enterprise** |
| Hỗ trợ ứng phó sự cố bảo mật (chiếm account, ransomware) đi kèm gói | **Enterprise** trở lên |
| Phản hồi **5 phút**, kỹ sư chuyên môn theo workload, quản lý sự kiện hạ tầng (ra mắt, migration lớn) | **Unified Operations** |

⚠️ **Trusted Advisor theo gói:** Basic chỉ có toàn bộ check **Service Limits** + một số check **Security** và **Fault Tolerance**, check Security phải **refresh thủ công**. Business Support+ trở lên: mọi check + Trusted Advisor API.

⚠️ **Developer Support, Business Support, Enterprise On-Ramp ngừng từ 01/01/2027** (vẫn còn ở AWS GovCloud (US)). Developer/Business tự chọn nâng lên Business Support+; Enterprise On-Ramp được tự động nâng lên Enterprise trong năm 2026.

---

## 29. Disaster Recovery strategies

🎯 **4 chiến lược (chi phí tăng dần, RTO giảm dần):**

| Chiến lược | Chi phí / RTO | Chọn khi |
| --- | --- | --- |
| **Backup & Restore** | Rẻ nhất, khôi phục chậm | Chấp nhận **mất dữ liệu vài giờ** |
| **Pilot Light** | Chỉ lõi (DB) chạy sẵn | Phải dựng thêm hạ tầng khi sự cố |
| **Warm Standby** | Bản đầy đủ ở **quy mô nhỏ**, phục vụ được ngay | App **24x7 nhưng không mission-critical** |
| **Multi-site active/active** | Đắt nhất, RTO ~0 | Mission-critical |

---

## 30. Phụ lục — Tra cứu dịch vụ & bẫy distractor

> Những dịch vụ/đáp án hay xuất hiện ở **đáp án SAI**. Nhớ "nó thực chất làm gì" để loại nhanh.

### Đáp án KHÔNG TỒN TẠI (loại ngay)

| Đáp án nhiễu | Thực tế |
| --- | --- |
| Soft Token MFA, AWS Mobile Farm, AWS IoT Gateway, Amazon Customize, System Load Balancer, Amazon S3 Random Access, Amazon DynamoDB Regulator, CloudCover Disaster Recovery, AWS Corporate Support, Adaptable RI, Product as a Service, Mixed deployment | **Không tồn tại** |
| AWS Fair Use Policy, AWS Applicable Use Policy | Đúng là **Acceptable Use Policy** |
| Object Gateway, Block Gateway | Storage Gateway chỉ có **File, Volume, Tape** |
| Resource / Software / Hardware budget | Budgets chỉ có Cost, Usage, Reservation, Savings Plans |
| Trusted Advisor category Elasticity, Documentation, Change Management | Không phải category |
| Pillar Availability, Scalability, Elasticity | Không phải pillar Well-Architected |
| CAF perspective Process, Product, Architecture | Không phải perspective |
| Storage Savings Plans, RI Savings Plans | Không tồn tại |
| RI 6 tháng / 2 năm / 5 năm | Chỉ 1 hoặc 3 năm |
| Scheduled Reserved Instances | Không còn mua được |
| "Amazon WAF" | Tên đúng: **AWS WAF** |

### Compute

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Lambda** | Code event-driven, ≤15 phút, không truy cập OS | Cài DB/container service, lưu biến môi trường nhạy cảm, truy cập OS |
| **Elastic Beanstalk** | PaaS, vẫn truy cập OS, không serverless | Đề hỏi "serverless" (đúng là Lambda/Fargate) |
| **Lightsail** | VPS giá cố định, nhỏ | Workload enterprise cần scale |
| **Spot** | Rẻ nhất, bị thu hồi | DB / job critical, không được gián đoạn |
| **Auto Scaling** | Scale out/in, thay instance hỏng | "Scale up/down", tự deploy Shield khi bị DDoS |

### Storage & Database

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **S3** | Object storage | Đề gọi là block/file storage, cài DB lên S3 |
| **EBS** | Block, 1 AZ | Cần dùng chung nhiều instance/AZ (→ EFS) |
| **Instance Store** | Block tạm | Cần dữ liệu bền |
| **One Zone-IA** | 1 AZ, availability thấp nhất | Dữ liệu quan trọng không tạo lại được |
| **Read Replica** | Scale đọc | "Lợi ích chính là HA" (đó là Multi-AZ) |

### Networking & Content Delivery

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **VPC Endpoint** | Kết nối riêng tới **dịch vụ AWS** | Nối 2 VPC (→ Peering) |
| **Direct Connect / Site-to-Site VPN** | Kết nối **on-prem ↔ AWS** | Nối VPC với VPC |
| **Global Accelerator** | Static anycast IP, không cache | Host website tĩnh, "dùng edge khác CloudFront" |
| **Route 53** | DNS, domain, health check | Cân bằng tải, IP routing, transfer acceleration |
| **CloudFront** | CDN cache nội dung | Tăng tốc upload về bucket (→ S3TA) |

### Security & Compliance

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **WAF** | Lọc L7 | Chống L3/L4, có DRT/SRT (đó là Shield); gắn EC2/Route 53 |
| **Inspector** | Lỗ hổng phần mềm | "Theo dõi thay đổi cấu hình" (→ Config) |
| **Secrets Manager** | Lưu/rotate secret | Tạo key mã hóa (→ KMS), phát hiện PII (→ Macie) |
| **Trusted Advisor / Inspector / CloudHSM** | Khuyến nghị / quét lỗ hổng / HSM | Chống DDoS |
| **Marketplace** | Phần mềm bên thứ ba | Mua RI, Direct Connect, tài liệu compliance (→ Artifact) |
| **Service Catalog** | Danh mục sản phẩm đã duyệt | Tư vấn chọn dịch vụ khi migrate |

### Monitoring & Cost

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **CloudTrail** | Log API | Gom log server on-prem (→ CloudWatch Logs), billing alarm |
| **CloudFormation** | IaC | Di chuyển dữ liệu giữa Region, tăng hiệu năng website |
| **Organizations / Cost Explorer / tags** | Quản lý account / phân tích / phân bổ chi phí | "Tạo hóa đơn riêng" (chỉ có khi tách account) |
| **Cost Explorer / Organizations** | Phân tích / multi-account | Billing alarm (→ CloudWatch) |

### ML & khác

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Rekognition** | Nhận diện trong ảnh/video | Resize, greyscale, pose detection |
| **Personalize, Device Farm, Wavelength, CodeDeploy, CodeArtifact** | Dịch vụ có thật | Out-of-scope CLF-C02 — chỉ hay xuất hiện làm đáp án nhiễu |
| **Pinpoint, Fraud Detector, Kendra, Forecast, CodeStar, Elastic Transcoder** | Dịch vụ đã/sắp ngừng | Xem 📌 ở §23, §24 |

---

## 31. Nguyên tắc làm bài chung

🎯 **"Least effort / least operational overhead"** → dịch vụ **managed/serverless** (S3 website, Athena, DynamoDB, Fargate, Lambda, AWS managed key), không tự dựng trên EC2.

🎯 **"Most cost-effective"** → cái **rẻ nhất vừa đủ**: Spot nếu chịu gián đoạn, RI/SP nếu ổn định, On-Demand nếu ngắn hạn/critical; storage class theo tần suất truy cập; giữ traffic inbound/cùng Region.

🎯 **Phân biệt câu hỏi "ai chịu trách nhiệm"** → AWS lo **vật lý + hạ tầng + dịch vụ managed**; khách lo **mọi thứ mình cấu hình** (data, IAM, mã hóa, SG, guest OS, code). "Cả hai" → configuration/patch management, training.

🎯 **Đọc động từ trong đề để chọn dịch vụ:** "detect threat" → GuardDuty · "find vulnerabilities" → Inspector · "discover PII" → Macie · "investigate root cause" → Detective · "track configuration" → Config · "who did" → CloudTrail · "performance/alarm" → CloudWatch · "download compliance report" → Artifact.

🎯 **Cách loại đáp án SAI nhanh:** (1) **tên không tồn tại** (§30 bảng đầu); (2) **sai loại dịch vụ** (Inspector theo dõi cấu hình, Route 53 cân bằng tải, CloudTrail gom log on-prem); (3) **đảo ngược khái niệm** ("trade OPEX for CAPEX", "scale up bằng Auto Scaling", "servers not services"); (4) **sai con số cố định** (RI 2 năm, Budgets 5 loại).

🎯 **Global vs regional, HA mặc định, mã hóa mặc định** là ba nhóm câu "thuộc lòng" — ôn bảng ở §4, §17, §8.

⚠️ **Chủ đề lặp lại ở cả 5 đề:** Shared Responsibility · EC2 purchasing & Savings Plans · S3 storage classes · Công cụ chi phí · Support plans & Trusted Advisor · CloudWatch vs CloudTrail vs Config · IAM best practices · Dịch vụ bảo mật · Kết nối mạng & SG vs NACL · Well-Architected & CAF · Chọn database · Containers & serverless. Ít hơn (3–4/5 đề): Global Infrastructure, CloudFront vs S3TA vs GA, DR & migration, mã hóa.

---

## 32. Bảng so sánh dịch vụ dễ nhầm (tổng hợp)

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| Agility vs Elasticity vs Economies of scale | Agility: ra mắt nhanh. Elasticity: lấy/trả tài nguyên theo nhu cầu. Economies of scale: giá rẻ nhờ gộp khách | "innovate faster" → Agility; "lower prices from aggregated usage" → Economies of scale |
| High availability vs Scalability vs Elasticity | HA: Multi-AZ, chịu lỗi. Scalability: tăng được quy mô. Elasticity: tự co giãn | "Multi-AZ" → HA |
| Vertical vs Horizontal scaling | Vertical: to máy hơn. Horizontal: thêm máy (ASG, ELB, Read Replica) | "add CPU/RAM" → Vertical |
| Well-Architected Tool vs Trusted Advisor vs Framework | Tool: tự trả lời câu hỏi về workload. Trusted Advisor: check tự động account. Framework: bộ nguyên tắc | "review workload against best practices" → WA Tool |
| CAF Business vs Governance vs Operations | Business: chiến lược, giá trị. Governance: portfolio, rủi ro, tài chính. Operations: capacity, observability | "align IT investment with business" → Business |
| Rehost vs Replatform vs Refactor | Rehost: không đổi. Replatform: tối ưu nhẹ. Refactor: viết lại | "no code changes, fastest" → Rehost |
| Local Zones vs Wavelength vs Outposts | Local Zones: gần thành phố. Wavelength: rìa 5G. Outposts: trong data center khách | "AWS in my data center" → Outposts |
| Root vs IAM user vs IAM role | Root: toàn quyền, không giới hạn được. User: credential dài hạn. Role: credential tạm | "EC2 truy cập S3" → Role |
| Credentials Report vs Access Advisor | Report: trạng thái password/key/MFA của mọi user. Advisor: quyền đã cấp và lần cuối dùng | "list users + MFA status" → Report |
| STS vs IAM Identity Center vs Cognito | STS: credential tạm. Identity Center: SSO nhiều account. Cognito: user của app | "app sign-up" → Cognito |
| SCP vs IAM policy | SCP: trần quyền cho account, không cấp quyền. IAM policy: cấp quyền | "giới hạn account member" → SCP |
| Organizations vs Control Tower | Organizations: quản lý account, billing, SCP. Control Tower: landing zone + controls | "set up multi-account environment theo best practice" → Control Tower |
| AWS managed key vs Customer managed key vs CloudHSM | AWS managed: ít công. Customer managed: tự kiểm soát. CloudHSM: phần cứng riêng | "hardware device" → CloudHSM |
| WAF vs Shield Standard vs Shield Advanced | WAF: L7. Standard: L3/L4 free. Advanced: trả phí, SRT, bảo vệ chi phí | "SQL injection" → WAF; "DDoS cost protection" → Shield Advanced |
| GuardDuty vs Inspector vs Macie vs Detective | Threat / lỗ hổng / PII / root cause | Theo động từ trong đề |
| CloudWatch vs CloudTrail vs Config | Hiệu năng / ai gọi API / lịch sử cấu hình | "who changed SG" → CloudTrail |
| Health Dashboard Service health vs Your account | Chung mọi dịch vụ / ảnh hưởng tài nguyên của bạn | "events affecting my resources" → Your account |
| Instance Connect vs Session Manager | Instance Connect: SSH qua browser, cần port 22. Session Manager: không port, không key | "no open ports" → Session Manager |
| On-Demand vs RI vs Savings Plans vs Spot | Không cam kết / 1–3 năm theo instance / 1–3 năm theo $/giờ / rẻ nhất, bị thu hồi | "interruptible" → Spot |
| Standard RI vs Convertible RI | Standard: rẻ hơn, cố định. Convertible: đổi được family/OS | "workload sẽ thay đổi" → Convertible |
| Dedicated Host vs Dedicated Instance | Host: thấy socket/core (BYOL). Instance: hardware riêng | "BYOL per socket" → Dedicated Host |
| ALB vs NLB | ALB: HTTP/HTTPS L7. NLB: TCP/UDP L4, hiệu năng cực cao | "TCP/UDP extreme performance" → NLB |
| Lambda vs Fargate vs ECS on EC2 vs Beanstalk | Lambda: function. Fargate: container serverless. ECS on EC2: container + truy cập server. Beanstalk: PaaS, có OS | "containers but access servers" → ECS (EC2 launch type) |
| S3 Standard-IA vs One Zone-IA vs Intelligent-Tiering | IA: nhiều AZ. One Zone: 1 AZ. Int-Tiering: pattern không biết | "recreatable" → One Zone-IA; "unknown pattern" → Int-Tiering |
| Glacier Flexible vs Deep Archive | Phút–giờ / 12–48h, rẻ nhất | "7–10 năm compliance" → Deep Archive |
| EBS vs EFS vs Instance Store vs S3 | Block 1 AZ / NFS dùng chung, HA / block tạm / object | "nhiều EC2 cùng truy cập" → EFS |
| Storage Gateway vs DataSync vs Snowball | Hybrid lâu dài / chuyển online / chuyển offline | "ongoing online incremental" → DataSync |
| RDS Multi-AZ vs Read Replica | HA, standby không đọc / scale đọc | "read-heavy" → Read Replica |
| DynamoDB vs RDS vs Aurora | NoSQL schemaless / SQL managed / SQL hiệu năng cao | "schemaless, least ops" → DynamoDB |
| DAX vs ElastiCache | Cache riêng DynamoDB / cache chung | "DynamoDB microseconds" → DAX |
| Neptune vs DocumentDB | Graph / document (MongoDB) | "knowledge graph" → Neptune |
| Redshift vs Athena vs EMR vs Glue | Warehouse / SQL trên S3 / Hadoop / ETL | "query S3 serverless" → Athena |
| SG vs NACL | Instance, Allow, stateful / subnet, Allow+Deny, stateless | "block specific IP" → NACL |
| Gateway vs Interface Endpoint | S3/DynamoDB / dịch vụ khác | "VPC to SQS privately" → Interface |
| VPC Peering vs Transit Gateway | 2 VPC, không bắc cầu / nhiều VPC + on-prem | "many VPCs + on-prem" → TGW |
| Direct Connect vs Site-to-Site VPN vs Client VPN | Riêng, ổn định / mã hóa qua Internet, nhanh / máy cá nhân | "encrypted, quick" → VPN |
| CloudFront vs S3TA vs Global Accelerator | Cache nội dung / upload về bucket / static IP non-HTTP | "uploads to central bucket" → S3TA |
| SQS vs SNS vs Amazon MQ | Pull, không mất / push fan-out / broker chuẩn | "existing message broker" → MQ |
| CloudFormation vs CDK vs StackSets | Template / code / nhiều account+Region | "IaC in Python" → CDK |
| CodeCommit vs CodeBuild vs CodeDeploy vs CodePipeline | Git / build / deploy / điều phối | "deploy to EC2 and on-prem" → CodeDeploy |
| Pricing Calculator vs Cost Explorer vs CUR vs Budgets | Ước tính trước / phân tích, dự báo / dữ liệu chi tiết / cảnh báo | "forecast" → Cost Explorer |
| Budgets vs CloudWatch billing alarm | Thực tế + dự báo, RI/SP / chỉ thực tế | "forecasted to exceed" → Budgets |
| Account riêng vs cost allocation tags | Hóa đơn riêng / phân bổ chi phí | "separate invoices" → account riêng |
| Pricing Calculator vs Migration Evaluator | Ước tính trên AWS / business case on-prem vs AWS | "TCO on-prem vs AWS" → Migration Evaluator (nếu có) |
| Backup & Restore vs Pilot Light vs Warm Standby vs Multi-site | Rẻ nhất → đắt nhất | "scaled-down full copy" → Warm Standby |

---

## 33. Từ khóa → đáp án (keyword mapping nhanh)

| Từ khóa trong đề | Đáp án |
| --- | --- |
| innovate faster, develop/test/launch quickly | Agility |
| lower pay-as-you-go prices thanks to aggregated usage | Economies of scale |
| acquire and release resources as needed | Elasticity |
| Multi-AZ | High availability |
| add CPU/RAM | Vertical scaling |
| on-prem + AWS | Hybrid |
| only upload code, no infrastructure | PaaS (Elastic Beanstalk) |
| IaC, small reversible changes | Operational Excellence |
| right resource type and size | Performance Efficiency |
| review workload against best practices | Well-Architected Tool |
| align IT investment with business strategy | CAF Business |
| capacity management | CAF Operations |
| application portfolio management | CAF Governance |
| no code changes, fastest migration | Rehost |
| AWS in my data center / factory | Outposts |
| ultra-low latency mobile 5G | Wavelength |
| low latency gaming end-users in a city | Local Zones |
| who is responsible for edge locations / network patching | AWS |
| encrypt RDS / configure SG / IAM users / guest OS patch / Lambda code | Customer |
| both AWS and customer | Configuration management, Patch management, Awareness & training |
| mandatory policy elements | Effect, Action |
| list all users with password/key/MFA status | Credentials Report |
| review permissions granted / last used | Access Advisor |
| EC2 needs access to S3/DynamoDB | IAM Role |
| temporary credentials for trusted users | STS |
| SSO to many accounts | IAM Identity Center |
| app user sign-up/sign-in | Cognito |
| restrict member accounts | SCP |
| set up multi-account landing zone | Control Tower |
| one bill, volume discount, share RI | Organizations consolidated billing |
| SQL injection / XSS / block countries | WAF |
| DDoS cost protection / 24x7 DDoS team | Shield Advanced |
| malicious activity | GuardDuty |
| vulnerabilities | Inspector |
| sensitive data / PII at scale | Macie |
| root cause investigation | Detective |
| configuration history / notify changes | Config |
| who changed / API calls | CloudTrail |
| unusual API activity | CloudTrail Insights |
| PCI / SOC / ISO report | Artifact |
| AWS IP port scanning me | AWS Abuse team |
| hardware device for encryption keys | CloudHSM |
| least effort encryption | AWS managed key |
| keep S3 data private always | S3 Block Public Access |
| audit S3 requests | S3 server access logs |
| performance metrics / alarm | CloudWatch |
| alert CPU > 80% by email | CloudWatch alarm + SNS |
| alert when root logs in | EventBridge rule + SNS |
| collect logs from EC2 and on-prem | CloudWatch Logs (agent) |
| events affecting my resources | Health Dashboard – Your account |
| launch same EC2 in another Region | Copy AMI |
| bootstrap at launch | User data |
| browser-based SSH | EC2 Instance Connect |
| no open ports, no public IP, no SSH key | Session Manager |
| in-memory caching workload | Memory Optimized |
| short-term, spiky, critical | On-Demand |
| interruptible / flexible start time | Spot |
| predictable 1–3 years | RI / Savings Plans |
| workload will change, long term | Convertible RI |
| BYOL per socket/core | Dedicated Host |
| scale out/in, replace unhealthy | EC2 Auto Scaling |
| recurring daily load pattern | Predictive scaling |
| HTTP/HTTPS routing | ALB |
| TCP/UDP extreme performance | NLB |
| run code without servers, per request | Lambda |
| containers but access underlying servers | ECS (EC2 launch type) |
| containers, no server management | Fargate |
| store Docker images | ECR |
| hundreds of thousands of batch jobs | AWS Batch |
| low fixed-price VPS, WordPress | Lightsail |
| static website, least effort | S3 |
| unknown/changing access pattern | S3 Intelligent-Tiering |
| recreatable / secondary backup copy | S3 One Zone-IA |
| 7–10 years compliance, cheapest | Glacier Deep Archive |
| prevent accidental deletion | S3 Versioning |
| copy to another Region / account | S3 Replication (CRR) |
| shared file system, many EC2 | EFS |
| Windows file share, SMB, AD | FSx for Windows File Server |
| hybrid storage, tape replacement | Storage Gateway (Tape) |
| ongoing online incremental transfer | DataSync |
| centralized backup | AWS Backup |
| relational, MySQL/PostgreSQL compatible, high performance | Aurora |
| schemaless / flexible schema / least ops at any scale | DynamoDB |
| DynamoDB microseconds / peak reads | DAX |
| read-heavy DB | ElastiCache / Read Replica |
| knowledge graph | Neptune |
| MongoDB compatible | DocumentDB |
| data warehouse / OLAP | Redshift |
| SQL on S3, serverless | Athena |
| Hadoop / Spark | EMR |
| ETL / prepare data | Glue |
| real-time streaming | Kinesis Data Streams |
| interactive BI dashboard | Quick Sight |
| block a specific IP | NACL |
| VPC to S3 privately | Gateway endpoint |
| VPC to SQS privately | Interface endpoint |
| two VPCs | VPC Peering |
| many VPCs + on-prem | Transit Gateway (+ Direct Connect) |
| private, consistent, high bandwidth to on-prem | Direct Connect |
| encrypted, quick, over Internet | Site-to-Site VPN |
| global static website performance | CloudFront |
| uploads to central bucket | S3 Transfer Acceleration |
| non-HTTP, static IP, gaming/IoT | Global Accelerator |
| domain registration, health check | Route 53 |
| active-passive failover | Route 53 Failover routing |
| decouple without losing messages | SQS |
| pub/sub, push | SNS |
| existing message broker (ActiveMQ/RabbitMQ) | Amazon MQ |
| front door for Lambda / REST API | API Gateway |
| same infra in many accounts/Regions | CloudFormation StackSets |
| IaC in Python/JavaScript | CDK |
| deploy code to EC2 and on-prem | CodeDeploy |
| manage EC2 and on-prem servers, patch | Systems Manager |
| automate image creation | EC2 Image Builder |
| approved products self-service | Service Catalog |
| deploy popular technology quickly per best practice | AWS Solutions Library |
| identify objects/text in images | Rekognition |
| speech-to-text / text-to-speech / translate | Transcribe / Polly / Translate |
| chatbot | Lex |
| contact center | Connect |
| virtual desktop | WorkSpaces |
| chaos engineering | Fault Injection Service |
| buy third-party software | Marketplace |
| which is free | Inbound data transfer, IAM, Auto Scaling, Beanstalk, CloudFormation, Organizations |
| fundamental cost drivers | Compute, storage, outbound data transfer |
| estimate before deploying | Pricing Calculator |
| forecast spending | Cost Explorer |
| most comprehensive / hourly billing data | CUR |
| alert when exceeding budget / forecast | Budgets |
| alert on actual charges | CloudWatch billing alarm |
| separate invoices | Separate accounts |
| cost by department | Cost allocation tags |
| low-utilization EC2 without configuration | Trusted Advisor / Cost Explorer rightsizing |
| rightsizing EC2, EBS, Lambda by ML | Compute Optimizer |
| TAM | Enterprise |
| cheapest 24x7 phone support | Business Support+ |
| Support API, full Trusted Advisor | Business Support+ trở lên |
| security incident response included, 15-minute response | Enterprise |
| 5-minute critical response, infrastructure event management | Unified Operations |
| accept hours of data loss, cheapest DR | Backup & Restore |
| scaled-down full copy always on | Warm Standby |

---

## 34. Con số cần thuộc & đáp án bất khả thi

### Con số phải nhớ

| Hạng mục | Con số |
| --- | --- |
| Well-Architected pillars | 6 |
| CAF perspectives / phases | 6 / 4 (Envision, Align, Launch, Scale) |
| Migration strategies | 7Rs |
| AZ trong Region | Cách nhau ≤ 100 km, traffic giữa AZ được mã hóa |
| RI / Savings Plans | Kỳ hạn 1 hoặc 3 năm |
| Spot | Rẻ tới 90% |
| Savings Plans | 4 loại: Compute ~66%, EC2 Instance ~72%, SageMaker AI ~64%, Database ~35% |
| On-Demand Linux/Windows | Tính theo giây (sau phút đầu) |
| Lambda | Tính phí 1 ms; tối đa 15 phút; container image tới 10 GB |
| S3 durability | 11 số 9 |
| One Zone-IA availability | 99.5% (thấp nhất) |
| S3 thời gian lưu tối thiểu | Standard 0 · IA 30 ngày · Glacier IR/Flexible 90 ngày · Deep Archive 180 ngày |
| Deep Archive retrieval | 12–48 giờ |
| GuardDuty findings | Giữ 90 ngày |
| Config | Lịch sử thay đổi cấu hình |
| Budgets | 4 loại: Cost, Usage, Reservation, Savings Plans |
| CloudWatch billing metric | Ở us-east-1 |
| Cost Explorer | Dự báo tới 18 tháng; lịch sử mặc định 13 tháng |
| Trusted Advisor | 6 category |
| Storage Gateway | 3 loại: File, Volume, Tape |
| MFA | 3 loại; tối đa 8 thiết bị/user; bắt buộc cho root |
| Direct Connect | ≥ 1 tháng triển khai |
| Global Accelerator | 2 static anycast IP |
| Aurora | Nhanh hơn MySQL tới 5 lần; tới 256 TiB |
| Business Support+ / Enterprise / Unified Operations | $29/tháng, < 30 phút / $5,000/tháng, ≤ 15 phút / theo yêu cầu, 5 phút |
| Gói Developer, Business, Enterprise On-Ramp | Ngừng 01/01/2027 |

### Đáp án BẤT KHẢ THI → loại ngay

- **Auto Scaling scale up/down** (chỉ scale out/in).
- **Trade OPEX for CAPEX** (đảo ngược; đúng là CAPEX → variable expense).
- **Giới hạn quyền root bằng IAM policy** (chỉ SCP với member account).
- **Tạo access key cho IAM role/group/policy** (access key gắn IAM user hoặc root).
- **SCP cấp quyền** hoặc **SCP giới hạn management account**.
- **Organizations/tags/Cost Explorer tạo hóa đơn riêng** (phải tách account).
- **WAF chống L3/L4** hoặc **gắn trực tiếp EC2/Route 53**.
- **Shield Advanced miễn phí cho Enterprise Support** (trả phí cho mọi khách).
- **Inspector theo dõi thay đổi cấu hình** (đó là Config).
- **CloudTrail gom log on-prem** / **billing alarm bằng CloudTrail** (đó là CloudWatch).
- **Route 53 cân bằng tải / tăng tốc transfer**.
- **Global Accelerator cache hoặc host website**.
- **VPC Endpoint nối 2 VPC**; **Direct Connect/VPN nối VPC với VPC**.
- **VPC Peering bắc cầu** (A↔B, B↔C không cho A↔C).
- **Gateway Endpoint cho dịch vụ ngoài S3/DynamoDB**.
- **Security Group có Deny rule** (chỉ NACL có).
- **EBS dùng chung nhiều AZ** (EBS khóa 1 AZ).
- **Cài DB lên S3 / S3 là block storage**.
- **Lambda cài DB, truy cập OS**.
- **Elastic Beanstalk là serverless / không truy cập được OS**.
- **Spot cho workload không được gián đoạn**.
- **RI 6 tháng, 2 năm, 5 năm**; **RI cho S3/CloudFront/IAM**.
- **Mua RI hay Direct Connect trên Marketplace**.
- **Rekognition resize/greyscale ảnh**.
- **CloudFormation chuyển dữ liệu giữa Region**.
- **Auto Scaling tự deploy Shield**.
- Mọi tên trong bảng "Đáp án KHÔNG TỒN TẠI" ở §30.

---

### Nguồn đã đối chiếu (các ghi chú 📌 và ❓)

- AWS Support Plans (gói hiện hành, Developer/Business/On-Ramp ngừng 01/01/2027): https://docs.aws.amazon.com/awssupport/latest/user/aws-support-plans.html
- AWS Unified Operations (phản hồi 5 phút): https://docs.aws.amazon.com/awssupport/latest/user/unified-operations-intro.html
- Trusted Advisor theo gói Support: https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html
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
- WAF resources: https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html
- KMS automatic key rotation: https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-enable.html
- CloudWatch Logs agent deprecated, dùng Unified CloudWatch Agent: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/UsePreviousCloudWatchLogsAgent.html
- Elastic Transcoder end of support (nguồn web): https://aws.amazon.com/blogs/media/support-for-amazon-elastic-transcoder-ending-soon/
- CodeStar discontinuation (nguồn web): https://repost.aws/questions/QUzfvPNaF6T3CVMRta-qcziA/code-star-vs-code-catalyst
- CLF-C02 in-scope / out-of-scope services: https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-in-scope-services.html · https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
