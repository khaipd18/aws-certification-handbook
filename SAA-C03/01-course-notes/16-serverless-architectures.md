# SAA-C03 — Serverless Solution Architectures

> Thuộc **Domain 2: Design Resilient Architectures**, **Domain 3: Design High-Performing Architectures** và **Domain 4: Design Cost-Optimized Architectures**: ghép Lambda, API Gateway, DynamoDB, Cognito, S3, CloudFront thành ứng dụng hoàn chỉnh.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.
> Ghi chú: slide chỉ có tiêu đề cho case **MyBlog.com**, không có nội dung; phần MyBlog dưới đây là `[+]` dựng theo mẫu kiến trúc chuẩn của AWS.

---

## 1. TL;DR

- **MyTodoList (mobile app)**: REST API HTTPS → **API Gateway + Lambda + DynamoDB**; đăng nhập bằng **Cognito user pool**; user truy cập file riêng trong **S3** bằng credential tạm từ **Cognito identity pool**.
- Đọc nhiều, ghi ít → thêm **DAX** trước DynamoDB và/hoặc **API Gateway caching** cho response.
- Mọi thành phần serverless: tự scale, trả theo dùng, không quản server.
- `[+]` **MyBlog (website toàn cầu)**: nội dung tĩnh **S3 + CloudFront (OAC)**; API **API Gateway + Lambda**; dữ liệu **DynamoDB Global Tables** (+ DAX); email chào mừng **DynamoDB Streams → Lambda → SES**; thumbnail **S3 event → Lambda**.
- Mẫu chung: **tách đọc/ghi**, **cache ở nhiều tầng** (CloudFront → API Gateway → DAX), **event-driven** cho tác vụ phụ.

---

## 2. Core Concepts

### MyTodoList — yêu cầu
- **Là gì:** ứng dụng mobile quản lý việc cần làm; cần REST API qua HTTPS, kiến trúc serverless, user tương tác trực tiếp với thư mục riêng trên S3, đăng nhập được.
- **Dùng khi nào:** tải đọc lớn hơn ghi nhiều lần.
- **Vì sao quan trọng:** đây là bài mẫu kinh điển để ghép 5–6 dịch vụ serverless.

### MyTodoList — REST API
- **Là gì:** client gọi **API Gateway** (REST, HTTPS) → **Lambda** → **DynamoDB**; xác thực bằng **Cognito user pool** (API Gateway kiểm tra JWT).
- **Dùng khi nào:** mọi thao tác CRUD todo.
- **Vì sao quan trọng:** không có server, scale theo request; API Gateway lo HTTPS, throttling.

### MyTodoList — truy cập S3 trực tiếp
- **Là gì:** sau khi đăng nhập, app đổi token lấy **credential AWS tạm** từ **Cognito identity pool**; IAM role dùng policy variable để giới hạn vào prefix của chính user.
- **Dùng khi nào:** upload/download file đính kèm mà không qua Lambda (tránh giới hạn payload, giảm chi phí).
- **Vì sao quan trọng:** **không** lưu access key trong app, **không** tạo IAM user cho từng người.

### MyTodoList — tối ưu đọc
- **Là gì:** **DAX** cache item DynamoDB (µs, không sửa truy vấn); **API Gateway caching** cache response theo stage.
- **Dùng khi nào:** đọc lặp lại dữ liệu ít đổi.
- **Vì sao quan trọng:** giảm RCU (tiết kiệm) và độ trễ; ghi vẫn đi thẳng DynamoDB.

### `[+]` MyBlog — nội dung tĩnh toàn cầu
- **Là gì:** HTML/JS/ảnh trên **S3**, phân phối qua **CloudFront**; bucket private, chỉ CloudFront đọc qua **OAC**.
- **Dùng khi nào:** website đọc nhiều toàn cầu, cần độ trễ thấp và chi phí thấp.
- **Vì sao quan trọng:** S3 không public; CloudFront giảm tải và chống DDoS cơ bản.

### `[+]` MyBlog — API & dữ liệu toàn cầu
- **Là gì:** API Gateway + Lambda đọc/ghi **DynamoDB Global Tables** (người dùng mọi Region ghi gần mình); **DAX** cho đọc nóng.
- **Dùng khi nào:** bài viết, bình luận, profile user trên nhiều châu lục.
- **Vì sao quan trọng:** Global Tables active-active, độ trễ thấp cho cả đọc và ghi.

### `[+]` MyBlog — tác vụ event-driven
- **Là gì:** user đăng ký → item mới → **DynamoDB Streams** → **Lambda** → **SES** gửi email chào mừng; ảnh upload lên S3 → **S3 Event** → **Lambda** tạo thumbnail (có thể lưu bucket khác, kèm SQS/SNS nếu cần).
- **Dùng khi nào:** tác vụ phụ không nên làm chậm request chính.
- **Vì sao quan trọng:** tách rời bằng sự kiện giúp hệ thống bền và dễ mở rộng.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Serverless REST API | API Gateway + Lambda + DynamoDB | "HTTPS REST API, no servers" |
| Cognito user pool | Đăng nhập, phát JWT | "authenticate mobile users" |
| Cognito identity pool | Credential AWS tạm cho user | "users access own S3 folder" |
| Policy variable | Biến trong IAM policy (vd: ID user) | "restrict to user's prefix" |
| DAX | Cache item DynamoDB | "read-heavy, microsecond" |
| API Gateway caching | Cache response API | "reduce Lambda invocations" |
| OAC | Chỉ CloudFront đọc S3 | "S3 must not be public" |
| Global Tables | DynamoDB active-active đa Region | "global users write locally" |
| DynamoDB Streams | Luồng thay đổi item | "welcome email after sign-up" |
| Amazon SES | Gửi email | "send transactional email" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Upload qua API Gateway/Lambda | Upload thẳng S3 bằng identity pool | Qua Lambda tốn tiền, giới hạn payload; thẳng S3 rẻ, scale | File lớn → thẳng S3 |
| DAX | API Gateway caching | DAX cache dữ liệu DB; API cache cache response HTTP | Có thể dùng cả hai |
| User pool | Identity pool | Xác thực vs cấp quyền AWS | S3 trực tiếp → identity pool |
| CloudFront + S3 | S3 website public | CloudFront: cache, HTTPS, bucket private; website public: không cache, HTTP | Luôn ưu tiên CloudFront + OAC |
| Global Tables | Read replica (RDS) | Global Tables ghi mọi Region; replica chỉ đọc | Ghi toàn cầu → Global Tables |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "serverless REST API over HTTPS" | API Gateway + Lambda + DynamoDB | |
| "mobile users sign in" | Cognito user pool | |
| "each user reads/writes only own S3 folder" | Cognito identity pool + IAM policy variable | |
| "read-heavy DynamoDB, reduce cost" | DAX | |
| "same API responses requested repeatedly" | API Gateway caching | |
| "global static website, private bucket" | CloudFront + S3 + OAC | |
| "global users, low-latency writes" | DynamoDB Global Tables | |
| "send email after user registers" | DynamoDB Streams → Lambda → SES | |
| "create thumbnails on upload" | S3 event → Lambda | |

---

## 5. Câu hỏi tình huống

**Câu 1.** App mobile cần REST API qua HTTPS, không quản lý server, lưu dữ liệu NoSQL. Kiến trúc nào?
- A. ALB + EC2 + RDS
- B. API Gateway + Lambda + DynamoDB
- C. CloudFront + S3 website
- D. Elastic Beanstalk + Aurora

<details><summary>Đáp án</summary>

**B.**
- A, D sai: có server phải quản lý.
- C sai: chỉ nội dung tĩnh, không xử lý API.
</details>

**Câu 2.** User của app cần upload ảnh vào thư mục riêng `s3://bucket/<user-id>/`, không được thấy thư mục người khác. Cách an toàn nhất?
- A. Nhúng access key IAM vào app
- B. Cognito identity pool cấp credential tạm; IAM role dùng policy variable giới hạn prefix theo user
- C. Bucket public, ẩn đường dẫn
- D. Tạo IAM user cho từng người

<details><summary>Đáp án</summary>

**B.**
- A sai: lộ credential, mọi user dùng chung quyền.
- C sai: ai cũng đọc được.
- D sai: không scale, sai mô hình cho app user.
</details>

**Câu 3.** (Chọn 2) API todo có lượng đọc gấp 100 lần ghi; DynamoDB tốn RCU và Lambda bị gọi quá nhiều. Cải thiện gì?
- A. DAX trước DynamoDB
- B. Bật caching ở API Gateway stage
- C. Đổi sang DynamoDB provisioned không auto scaling
- D. Dùng RDS read replica
- E. Tăng RAM Lambda

<details><summary>Đáp án</summary>

**A, B.** Cache ở tầng DB và tầng API.
- C sai: không giảm số lần đọc.
- D sai: đổi DB không cần thiết.
- E sai: không giảm số lần gọi.
</details>

**Câu 4.** Blog toàn cầu cần phục vụ ảnh và HTML với độ trễ thấp, bucket không public. Chọn gì?
- A. S3 static website public
- B. CloudFront + S3 + OAC
- C. Global Accelerator + S3
- D. EC2 Nginx ở mỗi Region

<details><summary>Đáp án</summary>

**B.**
- A sai: bucket public, không cache toàn cầu.
- C sai: Global Accelerator không nhận S3 làm endpoint.
- D sai: tốn vận hành.
</details>

**Câu 5.** Blog có người dùng ở Mỹ, châu Âu, châu Á cùng viết bình luận; muốn độ trễ ghi thấp ở mọi nơi. Chọn gì?
- A. DynamoDB Global Tables
- B. RDS với read replica cross-Region
- C. DynamoDB một Region + DAX
- D. ElastiCache Global Datastore

<details><summary>Đáp án</summary>

**A.** Ghi ở Region gần nhất, replicate active-active.
- B sai: replica chỉ đọc, ghi vẫn về một Region.
- C sai: DAX tăng tốc đọc, ghi vẫn một Region.
- D sai: cache, không phải DB chính.
</details>

**Câu 6.** Sau khi user đăng ký (item mới trong DynamoDB), cần gửi email chào mừng mà không làm chậm API đăng ký. Chọn gì?
- A. Lambda đăng ký gửi email đồng bộ
- B. DynamoDB Streams → Lambda → Amazon SES
- C. CloudWatch alarm
- D. Route 53 health check

<details><summary>Đáp án</summary>

**B.** Bất đồng bộ, tách rời.
- A sai: làm chậm và tăng lỗi cho request chính.
- C, D sai: không xử lý sự kiện dữ liệu.
</details>

**Câu 7.** Ảnh upload vào S3 cần tạo thumbnail tự động. Chọn gì?
- A. S3 Event Notification → Lambda
- B. EC2 cron quét bucket
- C. CloudFront Functions
- D. Glacier

<details><summary>Đáp án</summary>

**A.**
- B sai: không serverless, trễ.
- C sai: không chạy khi upload, không xử lý ảnh nặng.
- D sai: lưu trữ archive.
</details>

**Câu 8.** Team muốn giảm tối đa vận hành cho app MyTodoList. Thành phần nào **không** cần thiết?
- A. API Gateway
- B. Lambda
- C. EC2 bastion host
- D. Cognito

<details><summary>Đáp án</summary>

**C.** Kiến trúc serverless không có EC2 để SSH.
- A, B, D sai: đều là thành phần cốt lõi.
</details>

---

### Nguồn đã đối chiếu
- Serverless Applications Lens (Well-Architected): https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html
- Cognito identity pools (credential tạm, IAM role): https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-identity.html
- IAM policy variables: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html
- API Gateway caching: https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-caching.html
- DynamoDB Accelerator (DAX): https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html
- DynamoDB Streams & Lambda: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.Lambda.html
- CloudFront OAC: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html
