# SAA-C03 — Serverless: Lambda, Edge Functions, DynamoDB, API Gateway, Step Functions & Cognito

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 3: Design High-Performing Architectures**: compute không server, NoSQL serverless, API managed, điều phối workflow và xác thực người dùng.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Lambda**: chạy hàm theo sự kiện, tối đa **15 phút**, RAM **128 MB–10,240 MB** (CPU tăng theo RAM), `/tmp` 512 MB–10 GB, mặc định **1,000 concurrent executions** mỗi Region; tính phí theo request + thời gian chạy **làm tròn 1 ms**.
- Payload: đồng bộ 6 MB; ⚠️ bất đồng bộ trước 256 KB → nay **1 MB**. Package zip 50 MB (250 MB giải nén), container image tới 10 GB.
- **Reserved concurrency** giới hạn/đảm bảo một hàm; **provisioned concurrency** khử cold start; **SnapStart** giảm cold start (Java, Python, .NET).
- Edge: **CloudFront Functions** (JS siêu nhẹ, viewer request/response) vs **Lambda@Edge** (Node.js/Python, cả origin request/response).
- Lambda trong VPC: truy cập RDS private → dùng **RDS Proxy** để không cạn kết nối.
- **DynamoDB**: NoSQL key-value, item tối đa **400 KB**, mode **on-demand** / **provisioned** (+ auto scaling); **DAX** (cache µs), **Streams**, **global tables** (active-active), **TTL**, **PITR 1–35 ngày**, export/import S3.
- **API Gateway**: REST/HTTP/WebSocket; endpoint **edge-optimized / regional / private**; bảo mật IAM, **Cognito user pool**, **Lambda authorizer**; integration timeout 29 giây mặc định, REST regional/private nâng được.
- **Step Functions** điều phối workflow (Standard vs Express); **Cognito**: **user pool** (đăng nhập, trả JWT) vs **identity pool** (đổi danh tính lấy credential AWS tạm).

---

## 2. Core Concepts

### AWS Lambda
- **Là gì:** function serverless chạy theo nhu cầu, scale tự động; tích hợp API Gateway, S3, DynamoDB Streams, SQS, SNS, Kinesis, EventBridge, Cognito...
- **Dùng khi nào:** xử lý sự kiện (thumbnail khi upload S3), cron serverless (EventBridge schedule), backend API.
- **Vì sao quan trọng:** giới hạn cứng: 15 phút, 10 GB RAM, env vars 4 KB, 5 layer; việc dài hơn → ECS/Fargate, Batch hoặc Step Functions. Lambda tính phí theo 1 ms.

### Concurrency & cold start
- **Là gì:** mỗi request đồng thời cần một execution environment; mặc định 1,000/Region (xin tăng được); **reserved concurrency** đặt trần cho một hàm và giữ phần đó khỏi hàm khác; **provisioned concurrency** khởi tạo sẵn.
- **Dùng khi nào:** một hàm gây throttle cả account → reserved concurrency; API nhạy độ trễ → provisioned concurrency hoặc **SnapStart**.
- **Vì sao quan trọng:** vượt concurrency: gọi đồng bộ trả **429 ThrottleError**; bất đồng bộ tự retry rồi vào DLQ/destination.

### CloudFront Functions & Lambda@Edge
- **Là gì:** **CloudFront Functions**: JavaScript, < 1 ms, hàng triệu request/giây, chỉ viewer request/response, không truy cập mạng; **Lambda@Edge**: Node.js/Python, tới 5–30 giây, cả viewer và origin events, truy cập mạng/AWS SDK.
- **Dùng khi nào:** chuẩn hóa cache key, rewrite URL, chèn header, kiểm tra JWT đơn giản → Functions; logic nặng, gọi service khác, A/B testing phức tạp → Lambda@Edge.
- **Vì sao quan trọng:** Lambda@Edge viết ở `us-east-1`, CloudFront nhân bản ra các edge.

### Lambda trong VPC & với RDS
- **Là gì:** Lambda gắn VPC tạo **Hyperplane ENI** trong subnet để truy cập tài nguyên private.
- **Dùng khi nào:** gọi RDS private → **RDS Proxy** gom kết nối (Lambda phải ở VPC); cần ra Internet từ Lambda trong VPC → NAT gateway.
- **Vì sao quan trọng:** RDS/Aurora (MySQL, PostgreSQL) gọi được Lambda từ trong DB (vd: trigger khi insert); khác với **RDS Event Notifications** chỉ báo sự kiện về instance (tạo, failover...), không về dữ liệu.

### Amazon DynamoDB
- **Là gì:** NoSQL managed, Multi-AZ, scale hàng triệu request/giây, độ trễ mili giây một chữ số; bảng có **primary key** (partition key hoặc partition + sort key), item tối đa 400 KB, schema linh hoạt.
- **Dùng khi nào:** session store, shopping cart, gaming, IoT, metadata; truy vấn theo key đã biết.
- **Vì sao quan trọng:** table class **Standard** và **Standard-IA** (rẻ hơn ~60% lưu trữ, đọc/ghi đắt hơn ~25%) cho bảng ít truy cập.

### DynamoDB capacity modes
- **Là gì:** **Provisioned**: đặt RCU/WCU, có auto scaling, trả theo giờ; **On-demand**: tự scale, trả theo request.
- **Dùng khi nào:** tải ổn định, dự đoán được → provisioned; tải khó đoán, đột biến, bảng mới → on-demand.
- **Vì sao quan trọng:** on-demand đắt hơn mỗi request nhưng không phải lập kế hoạch capacity.

### DAX, Streams, Global Tables, TTL
- **Là gì:** **DAX**: cache in-memory cho DynamoDB, độ trễ micro giây, không sửa logic truy vấn; **Streams**: thay đổi item theo thứ tự, giữ 24 giờ, kích hoạt Lambda; **Global Tables**: active-active đa Region, ghi ở Region nào cũng được; **TTL**: tự xóa item hết hạn.
- **Dùng khi nào:** đọc nóng cần µs → DAX; phản ứng thay đổi (gửi email chào mừng) → Streams + Lambda (hoặc Kinesis Data Streams); người dùng toàn cầu → Global Tables; xóa session cũ → TTL.
- **Vì sao quan trọng:** DAX cho object/item; ElastiCache cho kết quả tính toán tổng hợp. Global tables version **2019.11.21 (Current)**; version cũ 2017.11.29 là Legacy.

### DynamoDB backup & tích hợp S3
- **Là gì:** **PITR** tùy chỉnh **1–35 ngày**, restore ra bảng mới; **on-demand backup** giữ đến khi xóa (hoặc qua AWS Backup, copy cross-Region); **export to S3** (cần PITR, không tốn RCU) và **import from S3** (CSV, DynamoDB JSON, ION).
- **Dùng khi nào:** phân tích bằng Athena trên dữ liệu export; khôi phục sau ghi nhầm.
- **Vì sao quan trọng:** export không ảnh hưởng read capacity của bảng.

### Amazon API Gateway
- **Là gì:** API managed: REST, HTTP, WebSocket; versioning, stage, throttling, API key, usage plan, caching, request validation, Swagger/OpenAPI import.
- **Dùng khi nào:** tích hợp Lambda (serverless REST), HTTP backend, hoặc gọi thẳng AWS service (vd: ghi vào Kinesis/SQS không cần Lambda).
- **Vì sao quan trọng:** integration timeout mặc định 29 giây; REST **Regional/private** nâng được cao hơn (có thể phải giảm throttle quota). Payload tối đa 10 MB.

### API Gateway endpoint types & security
- **Là gì:** **Edge-optimized** (mặc định, qua CloudFront edge, gateway vẫn ở một Region), **Regional** (client cùng Region, tự gắn CloudFront được), **Private** (chỉ truy cập trong VPC qua interface endpoint + resource policy).
- **Dùng khi nào:** xác thực: **IAM** (SigV4, cho dịch vụ nội bộ), **Cognito user pool** (user ứng dụng), **Lambda authorizer** (token tùy chỉnh như OAuth bên thứ ba).
- **Vì sao quan trọng:** custom domain HTTPS dùng ACM; edge-optimized cần cert ở **us-east-1**, regional cần cert cùng Region với API.

### AWS Step Functions
- **Là gì:** workflow trực quan dạng state machine; tích hợp Lambda, ECS, Batch, DynamoDB, SNS, SQS, API Gateway, EventBridge, SageMaker...; có bước **chờ con người duyệt**.
- **Dùng khi nào:** điều phối nhiều bước có rẽ nhánh, retry, song song; workflow dài hơn 15 phút.
- **Vì sao quan trọng:** **Standard** (tới 1 năm, exactly-once) vs **Express** (tới 5 phút, throughput cao, at-least-once).

### Amazon Cognito
- **Là gì:** **User pools**: thư mục user cho web/mobile, đăng nhập username/password, MFA, social login (Google, Facebook), SAML/OIDC, trả **JWT**; **Identity pools**: cấp credential AWS tạm (qua IAM role) cho user đã xác thực hoặc khách.
- **Dùng khi nào:** user pool tích hợp API Gateway và ALB để xác thực; identity pool để user mobile ghi trực tiếp vào S3/DynamoDB với quyền giới hạn theo user (policy variable).
- **Vì sao quan trọng:** đề "hàng trăm nghìn user ứng dụng, không tạo IAM user" → Cognito.

### `[+]` Lambda Managed Instances & durable functions
- **Là gì:** tùy chọn mới: **Lambda Managed Instances** chạy hàm trên EC2 do Lambda quản (invocation bất đồng bộ/event source mapping tới 90 phút); **durable functions** cho workflow dài có checkpoint.
- **Dùng khi nào:** cần phần cứng đặc thù hoặc thời gian chạy dài hơn 15 phút mà vẫn muốn mô hình Lambda.
- **Vì sao quan trọng:** đề SAA hiện vẫn dùng giới hạn chuẩn 15 phút; đây là thông tin cập nhật để không bất ngờ.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Lambda | Function serverless theo sự kiện | "event-driven, pay per ms" |
| Reserved concurrency | Trần/đảm bảo concurrency cho một hàm | "one function throttles others" |
| Provisioned concurrency | Môi trường khởi tạo sẵn | "eliminate cold starts" |
| SnapStart | Snapshot môi trường đã init | "Java cold start" |
| CloudFront Functions | JS siêu nhẹ ở viewer | "header manipulation at scale" |
| Lambda@Edge | Lambda ở edge, cả origin event | "complex logic at edge" |
| DynamoDB on-demand | Trả theo request, tự scale | "unpredictable workload" |
| DAX | Cache µs cho DynamoDB | "microsecond read latency" |
| DynamoDB Streams | Luồng thay đổi item | "react to item changes" |
| Global Tables | DynamoDB active-active đa Region | "multi-Region writes" |
| TTL | Tự xóa item hết hạn | "expire sessions automatically" |
| API Gateway Regional | API cho client cùng Region | "custom CloudFront in front" |
| Private API | API chỉ trong VPC | "internal API only" |
| Lambda authorizer | Xác thực token tùy chỉnh | "third-party OAuth tokens" |
| Step Functions | Điều phối workflow | "orchestrate Lambda, human approval" |
| Cognito user pool | Đăng nhập, JWT | "sign-up/sign-in for app users" |
| Cognito identity pool | Credential AWS tạm cho user | "mobile users write to S3 directly" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Reserved concurrency | Provisioned concurrency | Reserved: giới hạn/giữ chỗ; Provisioned: khởi tạo sẵn chống cold start | Cold start → provisioned |
| CloudFront Functions | Lambda@Edge | Functions: viewer, < 1 ms, rẻ; Lambda@Edge: cả origin, mạnh hơn | Đơn giản, khối lượng cực lớn → Functions |
| DAX | ElastiCache | DAX riêng DynamoDB, không sửa truy vấn; ElastiCache tổng quát | DynamoDB đọc nóng → DAX |
| DynamoDB Streams | Kinesis Data Streams (từ DynamoDB) | Streams giữ 24 giờ; Kinesis giữ tới 1 năm, nhiều consumer hơn | Retention dài → Kinesis |
| Provisioned | On-demand | Provisioned rẻ khi ổn định; on-demand cho tải khó đoán | |
| Edge-optimized | Regional | Edge qua CloudFront cho client toàn cầu; Regional cho client cùng Region | |
| User pool | Identity pool | User pool xác thực (ai là ai); Identity pool cấp quyền AWS | "Access AWS resources" → identity pool |
| Step Functions Standard | Express | Standard dài (1 năm), exactly-once; Express ngắn, khối lượng lớn | IoT ingest → Express |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "generate thumbnail when image uploaded" | S3 event → Lambda | |
| "serverless cron job" | EventBridge schedule → Lambda | |
| "job runs 30 minutes" | Không Lambda → ECS Fargate / Batch / Step Functions | |
| "Lambda exhausts RDS connections" | RDS Proxy | |
| "one function consumes all concurrency" | Reserved concurrency | |
| "rewrite URLs at edge, millions of requests" | CloudFront Functions | |
| "unpredictable traffic, NoSQL, pay per request" | DynamoDB on-demand | |
| "microsecond read latency for DynamoDB" | DAX | |
| "multi-Region active-active NoSQL" | DynamoDB Global Tables | |
| "delete expired sessions automatically" | DynamoDB TTL | |
| "analyze DynamoDB data with Athena without RCU" | Export to S3 | Cần PITR |
| "API only reachable inside VPC" | API Gateway private endpoint | |
| "authenticate with third-party OAuth" | Lambda authorizer | |
| "mobile app sign-in with Google/Facebook" | Cognito user pool | |
| "app users upload directly to their S3 prefix" | Cognito identity pool | |
| "orchestrate steps with human approval" | Step Functions | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một hàm Lambda lỗi vòng lặp chiếm hết concurrency, làm các hàm khác bị throttle. Phòng ngừa thế nào?
- A. Tăng RAM cho hàm
- B. Đặt reserved concurrency cho hàm đó
- C. Bật provisioned concurrency cho mọi hàm
- D. Đổi sang Lambda@Edge

<details><summary>Đáp án</summary>

**B.** Đặt trần concurrency cho hàm đó.
- A sai: không giới hạn số bản chạy song song.
- C sai: tốn chi phí, không chặn hàm gây hại.
- D sai: không liên quan.
</details>

**Câu 2.** Game có bảng điểm người chơi trên DynamoDB, đọc rất nóng, cần độ trễ micro giây mà không sửa logic truy vấn. Chọn gì?
- A. ElastiCache Redis
- B. DynamoDB Accelerator (DAX)
- C. Global Tables
- D. Tăng WCU

<details><summary>Đáp án</summary>

**B.**
- A sai: phải sửa code đáng kể.
- C sai: đa Region, không cải thiện độ trễ đọc lên µs.
- D sai: WCU là ghi.
</details>

**Câu 3.** Khi user mới đăng ký (ghi vào DynamoDB), hệ thống cần tự gửi email chào mừng. Kiến trúc serverless?
- A. DynamoDB Streams → Lambda → SES
- B. EC2 poll bảng mỗi phút
- C. CloudWatch Logs
- D. DAX

<details><summary>Đáp án</summary>

**A.**
- B sai: không serverless, trễ và tốn kém.
- C sai: không phải luồng thay đổi dữ liệu.
- D sai: cache.
</details>

**Câu 4.** (Chọn 2) Ứng dụng mobile cần cho user đăng nhập bằng Google và upload ảnh trực tiếp vào thư mục riêng trong S3. Chọn gì?
- A. Cognito user pool với Google federation
- B. Cognito identity pool cấp credential tạm với policy theo user
- C. Tạo IAM user cho mỗi người dùng
- D. API Gateway private
- E. Lưu access key trong app

<details><summary>Đáp án</summary>

**A, B.**
- C sai: không scale, sai mô hình.
- D sai: private API chỉ trong VPC.
- E sai: lộ credential.
</details>

**Câu 5.** API nội bộ chỉ được gọi từ ứng dụng trong VPC, không đi ra Internet. Chọn gì?
- A. Edge-optimized API
- B. Regional API với WAF
- C. Private API + interface VPC endpoint + resource policy
- D. CloudFront

<details><summary>Đáp án</summary>

**C.**
- A, B sai: endpoint public.
- D sai: public CDN.
</details>

**Câu 6.** Ứng dụng serverless cần chạy chuỗi 5 bước với retry, rẽ nhánh và một bước chờ quản lý phê duyệt (có thể vài ngày). Chọn gì?
- A. Một Lambda gọi lần lượt các Lambda khác
- B. Step Functions Standard workflow
- C. Step Functions Express workflow
- D. SQS FIFO

<details><summary>Đáp án</summary>

**B.** Standard chạy tới 1 năm, hỗ trợ chờ phê duyệt.
- A sai: bị giới hạn 15 phút và khó quản lý lỗi.
- C sai: Express tối đa 5 phút.
- D sai: hàng đợi, không điều phối logic.
</details>

**Câu 7.** Website toàn cầu cần chèn header bảo mật vào mọi response với hàng triệu request/giây, chi phí thấp nhất. Chọn gì?
- A. Lambda@Edge origin response
- B. CloudFront Functions viewer response
- C. API Gateway
- D. ALB rule

<details><summary>Đáp án</summary>

**B.** Nhẹ, rẻ, đúng việc đơn giản ở viewer.
- A sai: đắt và nặng hơn mức cần.
- C, D sai: không chạy ở edge.
</details>

**Câu 8.** Ứng dụng NoSQL có traffic không đoán được, có ngày gần như không có request. Muốn trả tiền đúng mức dùng. Chọn gì?
- A. DynamoDB provisioned không auto scaling
- B. DynamoDB on-demand
- C. RDS Multi-AZ
- D. DynamoDB Standard-IA với provisioned capacity cao

<details><summary>Đáp án</summary>

**B.**
- A sai: trả cho capacity không dùng hoặc bị throttle.
- C sai: không phải NoSQL, trả theo giờ.
- D sai: Standard-IA giảm phí lưu trữ, không giải quyết tải biến động.
</details>

---

### Nguồn đã đối chiếu
- Lambda quotas (15 phút, 10,240 MB, payload 6 MB/1 MB, concurrency 1,000): https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
- Lambda reserved & provisioned concurrency: https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html
- CloudFront Functions vs Lambda@Edge: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-choosing.html
- DynamoDB table classes (Standard-IA): https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/CostOptimization_TableClass.html
- DynamoDB PITR (1–35 ngày): https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_PointInTimeRecoverySpecification.html
- DynamoDB global tables versions: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/V2globaltables_versions.html
- API Gateway quotas (integration timeout có thể > 29 giây): https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-execution-service-limits-table.html
- Step Functions Standard vs Express: https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html
- Amazon Cognito user pools & identity pools: https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html
