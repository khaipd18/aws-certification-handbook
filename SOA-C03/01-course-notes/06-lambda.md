# SOA-C03 — AWS Lambda

> Thuộc **Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization (22%)** và **Domain 3: Deployment, Provisioning, and Automation (22%)**: giới hạn, mô hình invoke và retry, concurrency, xử lý lỗi, Lambda trong VPC, giám sát.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- Lambda chạy **function**, không quản lý server; tự scale; tính tiền theo số request + thời gian chạy (GB-giây, làm tròn 1 ms).
- **Memory 128 MB–10,240 MB** là núm hiệu năng duy nhất: tăng memory thì tăng CPU và network theo tỷ lệ (1,769 MB ≈ 1 vCPU).
- **Timeout mặc định 3 giây, tối đa 15 phút**. Việc dài hơn → ECS/Fargate, Step Functions, EC2.
- Ba kiểu invoke:
  - **Synchronous**: bên gọi tự retry.
  - **Asynchronous**: Lambda retry thêm 2 lần, rồi đưa vào DLQ hoặc Destination.
  - **Event source mapping**: Lambda poll SQS, Kinesis, DynamoDB Streams.
- **Concurrency** mặc định 1,000 mỗi Region (soft quota). Bị throttle → metric `Throttles`, lỗi HTTP 429 `TooManyRequestsException`.
- **Reserved concurrency** giới hạn/đảm bảo phần của một function. **Provisioned concurrency** khởi tạo sẵn để bỏ cold start.
- ⚠️ Lỗi thời:
  - "Async payload 256 KB" → nay **1 MB**; sync vẫn 6 MB.
  - CloudWatch Events → nay là **EventBridge**.

---

## 2. Core Concepts

### Lambda vs EC2 & mô hình giá
- **Là gì:** EC2 là server chạy liên tục; Lambda là function ngắn hạn, giới hạn theo thời gian chạy. Free tier 1 triệu request và 400,000 GB-giây mỗi tháng.
- **Dùng khi nào:** tải theo sự kiện, ngắn, đột biến; tải luôn nóng hoặc chạy lâu thường rẻ hơn trên EC2/Fargate.
- **Vì sao quan trọng:** memory × thời gian nhân nhau, nên chạy nhanh gấp đôi với memory gấp đôi thì chi phí gần như không đổi.

### Runtime & đóng gói
- **Là gì:** managed runtime (Node.js, Python, Java, .NET, Ruby), custom runtime, hoặc container image tối đa 10 GB (phải hiện thực Lambda Runtime API).
- **Dùng khi nào:** zip trực tiếp 50 MB, giải nén (kể cả layer) 250 MB, tối đa 5 layer; biến môi trường tổng 4 KB.
- **Vì sao quan trọng:** Docker image tùy ý, chạy lâu → ECS/Fargate. Runtime có lịch hết hỗ trợ, phải nâng cấp.

### Ba kiểu invoke & retry
- **Là gì:** sync (API Gateway, ALB, Function URL, invoke trực tiếp); async (S3, SNS, EventBridge); event source mapping (SQS, Kinesis, DynamoDB Streams).
- **Dùng khi nào:** async lỗi → Lambda retry 2 lần (chờ khoảng 1 phút rồi 2 phút); throttle hoặc lỗi hệ thống thì retry tối đa 6 giờ.
- **Vì sao quan trọng:** "event bị mất khi function lỗi" → thêm DLQ hoặc Destination. SQS/Kinesis không phải async push mà là poll.

### Concurrency & throttling
- **Là gì:** số execution environment chạy đồng thời; mỗi function scale thêm tối đa 1,000 environment mỗi 10 giây.
- **Dùng khi nào:** **Reserved** để bảo vệ DB downstream hoặc đảm bảo phần cho function quan trọng; đặt **0** để tạm vô hiệu function. **Provisioned** cho đường đi nhạy độ trễ.
- **Vì sao quan trọng:** provisioned concurrency đặt trên version hoặc alias, không đặt trên `$LATEST`, và tính phí cả khi rảnh. Theo dõi `Throttles`, `ConcurrentExecutions`, `UnreservedConcurrentExecutions`.

### Event source mapping: SQS & stream
- **Là gì:** SQS batch tối đa 10,000 (standard) hoặc 10 (FIFO); batch > 10 cần batch window ≥ 1 giây. Kinesis/DynamoDB Streams có parallelization factor tới 10 mỗi shard.
- **Dùng khi nào:** cần buffer, batch, retry → SQS; stream consumer chậm → xem `IteratorAge`, tăng batch size hoặc parallelization factor.
- **Vì sao quan trọng:** visibility timeout của queue nên ít nhất 6 lần timeout function, nếu không message hiện lại giữa chừng → xử lý trùng.

### EventBridge & S3 event notifications
- **Là gì:** EventBridge chạy theo schedule (cron/rate) hoặc event pattern, invoke Lambda kiểu async. S3 notification: `s3:ObjectCreated`, `s3:ObjectRemoved`, `s3:ObjectRestore`, `s3:Replication`, lọc prefix/suffix.
- **Dùng khi nào:** thay server cron; pipeline thumbnail S3 → Lambda → S3/DynamoDB.
- **Vì sao quan trọng:** S3 notification thường tới trong vài giây, đôi khi lâu hơn; ghi đồng thời lên object không versioning có thể chỉ ra một event → bật versioning nếu cần mỗi lần ghi một event.

### Quyền: execution role & resource-based policy
- **Là gì:** execution role là quyền function được làm (ghi log, đọc S3…); resource-based policy là ai được invoke function.
- **Dùng khi nào:** "S3 không trigger được function" → thiếu invoke permission trong resource policy. "Không có log" → execution role thiếu quyền CloudWatch Logs.
- **Vì sao quan trọng:** secret không để trong biến môi trường (4 KB, hiện trên console nếu không mã hóa) → Secrets Manager / Parameter Store.

### `[+]` Lambda trong VPC
- **Là gì:** gắn function vào subnet để truy cập RDS, ElastiCache qua ENI do Lambda quản lý.
- **Dùng khi nào:** cần tài nguyên private.
- **Vì sao quan trọng:** function trong VPC **không có Internet mặc định** và không có public IP → cần NAT Gateway hoặc VPC endpoint (S3, DynamoDB, Secrets Manager) để tránh timeout và phí NAT.

### `[+]` DLQ vs Destinations
- **Là gì:** DLQ (SQS/SNS) chỉ nhận event **lỗi** sau khi hết retry; Destinations gửi cả **thành công và lỗi** tới SQS, SNS, EventBridge, Lambda, kèm ngữ cảnh invoke.
- **Dùng khi nào:** cần chuyển tiếp cả kết quả thành công → Destinations.
- **Vì sao quan trọng:** Destinations là cách khuyến nghị cho thiết kế mới; DLQ vẫn gặp trong đề và hệ thống cũ.

### `[+]` Versions, aliases & quan sát
- **Là gì:** version bất biến; `$LATEST` thay đổi được; alias trỏ tới version, hỗ trợ chia trọng số giữa hai version.
- **Dùng khi nào:** canary/linear deployment; truy vết downstream chậm → X-Ray; metric hệ thống → Lambda Insights.
- **Vì sao quan trọng:** metric chính `Invocations`, `Duration`, `Errors`, `Throttles`, `IteratorAge`, `DeadLetterErrors`. Log group mặc định không hết hạn → đặt retention.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Timeout | Mặc định 3 giây, tối đa 900 giây | "job runs 40 minutes" |
| Memory | 128 MB–10 GB, kéo theo CPU | "CPU-bound, slow function" |
| `/tmp` | 512 MB mặc định, tới 10,240 MB | "scratch space" |
| Reserved concurrency | Giới hạn/đảm bảo concurrency | "protect downstream DB" |
| Provisioned concurrency | Environment khởi tạo sẵn | "cold start latency" |
| `Throttles` | Invoke bị từ chối | "HTTP 429" |
| Async invoke | Lambda xếp hàng, retry 2 lần | "S3/SNS/EventBridge trigger" |
| Event source mapping | Lambda poll nguồn | "SQS, Kinesis, DynamoDB Streams" |
| `IteratorAge` | Tuổi record cũ nhất trong batch | "stream consumer falling behind" |
| DLQ | Hàng đợi event lỗi | "failed events lost" |
| Destinations | Định tuyến kết quả thành công/lỗi | "route success and failure" |
| Alias | Con trỏ tới version, chia trọng số | "canary 10%" |
| Resource-based policy | Ai được invoke | "S3 cannot invoke function" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Lambda | ECS/Fargate | Tối đa 15 phút, runtime quản lý; container tùy ý, không giới hạn thời gian | > 15 phút → Fargate |
| `Errors` | `Throttles` | Code lỗi; bị từ chối trước khi chạy | 429 → Throttles |
| Reserved | Provisioned | Chia/giới hạn pool; khởi tạo sẵn | Cold start → Provisioned |
| Tăng memory | Tăng timeout | Chạy nhanh hơn; cho chạy lâu hơn | CPU-bound → memory |
| Sync | Async | Bên gọi retry; Lambda retry 2 lần | Event mất → DLQ/Destination |
| DLQ | Destinations | Chỉ lỗi; cả thành công và lỗi | "Success too" → Destinations |
| Event source mapping | Async trigger | Lambda poll; dịch vụ đẩy | SQS → poll |
| `IteratorAge` | `Duration` | Backlog stream; thời gian một lần chạy | Consumer chậm → IteratorAge |
| Execution role | Resource-based policy | Function làm gì; ai gọi function | "Cannot trigger" → resource policy |
| VPC endpoint | NAT Gateway | Tới dịch vụ AWS riêng tư; ra Internet | Gọi DynamoDB rẻ → endpoint |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "runs longer than 15 minutes" | Không phải Lambda | Fargate, Step Functions |
| "slow and CPU-bound" | Tăng memory | |
| "first request after idle is slow" | Provisioned concurrency | |
| "too many DB connections from Lambda" | Reserved concurrency (+ RDS Proxy) | |
| "HTTP 429 / TooManyRequestsException" | Throttling → tăng quota / chỉnh reserved | |
| "disable function without deleting" | Reserved concurrency = 0 | |
| "events lost after failures" | DLQ hoặc Destination | |
| "SQS messages processed twice" | Visibility timeout ≥ 6× timeout function | |
| "Kinesis consumer lagging" | `IteratorAge`, tăng parallelization factor | |
| "Lambda in VPC times out calling public API" | NAT Gateway | |
| "canary 10% to new version" | Alias weighted routing | |
| "which downstream call is slow" | X-Ray | |
| "payload larger than 6 MB" | Truyền con trỏ S3 | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Function xử lý ảnh chạy chậm, CPU luôn 100%, memory đặt 512 MB. Cách cải thiện đơn giản nhất?
- A. Tăng timeout lên 15 phút
- B. Tăng memory
- C. Bật provisioned concurrency
- D. Chuyển sang arm64

<details><summary>Đáp án</summary>

**B.** Tăng memory thì CPU tăng theo tỷ lệ.
- A sai: chỉ cho chạy lâu hơn, không nhanh hơn.
- C sai: xử lý cold start, không tăng CPU.
- D sai: có thể rẻ hơn nhưng không giải quyết thiếu CPU.
</details>

**Câu 2.** API Gateway gọi Lambda; request đầu tiên sau khi rảnh có p99 latency rất cao. Chọn gì?
- A. Reserved concurrency
- B. Provisioned concurrency trên alias
- C. Tăng timeout
- D. Thêm DLQ

<details><summary>Đáp án</summary>

**B.**
- A sai: chỉ chia pool, không khởi tạo sẵn.
- C sai: không giảm độ trễ khởi tạo.
- D sai: chỉ dành cho async lỗi.
</details>

**Câu 3.** Lambda làm cạn connection của RDS khi tải đột biến. (Chọn 2) hai biện pháp phù hợp?
- A. Đặt reserved concurrency để giới hạn số environment
- B. Dùng RDS Proxy
- C. Tăng memory
- D. Bật provisioned concurrency rất lớn
- E. Tăng timeout

<details><summary>Đáp án</summary>

**A, B.** Giới hạn số function chạy đồng thời và gộp connection.
- C, E sai: không giảm số kết nối.
- D sai: thêm environment còn mở thêm connection.
</details>

**Câu 4.** S3 trigger Lambda; khi function lỗi, một số event biến mất. Làm sao giữ lại event lỗi để xử lý sau?
- A. Tăng memory
- B. Cấu hình on-failure Destination hoặc DLQ
- C. Chuyển S3 trigger sang sync
- D. Tăng reserved concurrency

<details><summary>Đáp án</summary>

**B.** S3 invoke async, hết 2 lần retry thì event đi vào DLQ/Destination.
- A sai: không giữ event.
- C sai: S3 notification là async, không đổi được.
- D sai: không lưu event lỗi.
</details>

**Câu 5.** Lambda đọc SQS, timeout function 60 giây. Message thường bị xử lý hai lần. Nguyên nhân khả dĩ nhất?
- A. Batch size quá nhỏ
- B. Visibility timeout của queue ngắn hơn khuyến nghị (ít nhất 6 lần timeout function)
- C. Thiếu provisioned concurrency
- D. Thiếu X-Ray

<details><summary>Đáp án</summary>

**B.** Message hiện lại trước khi function xử lý xong.
- A sai: batch nhỏ không gây trùng.
- C, D sai: không liên quan visibility.
</details>

**Câu 6.** Sau khi gắn Lambda vào private subnet để đọc RDS, function bị timeout khi gọi API bên thứ ba trên Internet. Sửa thế nào?
- A. Gán public IP cho Lambda
- B. Route private subnet qua NAT Gateway
- C. Tăng timeout
- D. Thêm interface endpoint cho Lambda

<details><summary>Đáp án</summary>

**B.**
- A sai: Lambda trong VPC không có public IP.
- C sai: không có đường ra thì chờ lâu hơn vẫn lỗi.
- D sai: endpoint dành cho dịch vụ AWS, không ra Internet.
</details>

**Câu 7.** (Chọn 2) Consumer Kinesis bằng Lambda chậm dần. Hai việc nên làm?
- A. Theo dõi `IteratorAge`
- B. Tăng parallelization factor (tối đa 10 mỗi shard)
- C. Thêm DLQ cho async
- D. Giảm memory
- E. Chuyển sang S3 notification

<details><summary>Đáp án</summary>

**A, B.**
- C sai: stream là event source mapping, dùng on-failure destination riêng.
- D sai: làm chậm thêm.
- E sai: không phù hợp dữ liệu stream.
</details>

**Câu 8.** Cần tạm dừng một function đang gây lỗi mà không xóa nó. Cách nhanh nhất?
- A. Đặt reserved concurrency = 0
- B. Xóa execution role
- C. Đặt timeout = 1 giây
- D. Xóa alias

<details><summary>Đáp án</summary>

**A.** Mọi invoke bị throttle.
- B sai: function vẫn chạy và lỗi.
- C sai: vẫn chạy.
- D sai: vẫn gọi được qua version hoặc `$LATEST`.
</details>

---

### Nguồn đã đối chiếu
- Lambda quotas (memory, timeout, payload sync 6 MB / async 1 MB, streaming 200 MB, scaling rate): https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
- Async error handling (retry 2 lần, tối đa 6 giờ): https://docs.aws.amazon.com/lambda/latest/dg/invocation-async-error-handling.html
- SQS event source (batch 10,000 / 10, batch window): https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-configure.html
- Lambda memory (1,769 MB ≈ 1 vCPU): https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html
- Reserved concurrency: https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html
- Provisioned concurrency: https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html
- Lambda trong VPC: https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html
