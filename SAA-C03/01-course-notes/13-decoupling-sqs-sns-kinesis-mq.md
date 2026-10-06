# SAA-C03 — Decoupling: SQS, SNS, Kinesis & Amazon MQ

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 3: Design High-Performing Architectures**: tách rời ứng dụng bằng hàng đợi, pub/sub, streaming và message broker.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- 3 mô hình tách rời: **queue** (SQS), **pub/sub** (SNS), **real-time streaming** (Kinesis); ứng dụng cũ dùng giao thức chuẩn (AMQP, MQTT, STOMP, OpenWire) → **Amazon MQ**.
- **SQS Standard**: throughput gần như vô hạn, **at-least-once**, có thể sai thứ tự; retention mặc định 4 ngày, tối đa **14 ngày**.
- ⚠️ Lỗi thời: "SQS message tối đa 256 KB" → nay **1 MiB**; lớn hơn thì dùng SQS Extended Client + S3.
- **SQS FIFO**: đúng thứ tự theo **message group**, **exactly-once processing**; 300 TPS mỗi API/partition (batch 10 → 3,000 msg/s), **high throughput mode** vượt mức này.
- Khái niệm SQS: **visibility timeout** (mặc định 30 giây), **long polling** (1–20 giây), **DLQ**, **delay queue** (tới 15 phút); scale ASG theo **ApproximateNumberOfMessages**.
- **SNS**: một message gửi tới nhiều subscriber (SQS, Lambda, HTTP, email, SMS, Firehose...); **fan-out SNS → nhiều SQS**; **message filtering**; SNS FIFO.
- **Kinesis Data Streams**: shard 1 MB/s (1,000 record/s) vào, 2 MB/s ra; retention tới 365 ngày, replay được; chế độ **on-demand** hoặc **provisioned**. Record mặc định 1 MiB, nay cấu hình tới **10 MiB**.
- **Amazon Data Firehose** (tên cũ Kinesis Data Firehose): nạp gần real-time vào S3, Redshift, OpenSearch, Splunk, HTTP endpoint...; buffer **0–900 giây**; serverless, không replay.

---

## 2. Core Concepts

### SQS Standard Queue
- **Là gì:** hàng đợi managed (từ 2004), không giới hạn số message, độ trễ thấp (< 10 ms), message tới 1 MiB.
- **Dùng khi nào:** đệm giữa web tier và worker tier, hấp thụ tải đột biến, xử lý bất đồng bộ.
- **Vì sao quan trọng:** **at-least-once** → consumer phải idempotent; consumer xóa message bằng DeleteMessage sau khi xử lý.

### Visibility timeout, long polling, DLQ
- **Là gì:** **visibility timeout**: message bị ẩn với consumer khác sau khi được nhận (mặc định 30 giây), xử lý lâu thì gọi **ChangeMessageVisibility**; **long polling**: chờ tới 20 giây khi hàng đợi trống, giảm số API call và độ trễ; **DLQ**: chứa message lỗi quá **maxReceiveCount**.
- **Dùng khi nào:** message xử lý trùng → tăng visibility timeout; tốn tiền vì poll rỗng → long polling; message độc làm kẹt → DLQ.
- **Vì sao quan trọng:** DLQ của FIFO cũng phải là FIFO; dùng **redrive** để đưa message từ DLQ về hàng đợi gốc sau khi sửa lỗi.

### Delay queue & message size
- **Là gì:** delay queue hoãn giao message 0–15 phút (cấp queue hoặc từng message); message tối đa **1 MiB**.
- **Dùng khi nào:** cần thời gian chờ trước khi consumer xử lý.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: giới hạn 256 KB → nay 1 MiB; message lớn hơn → lưu payload ở S3, gửi tham chiếu (Extended Client).

### SQS FIFO
- **Là gì:** First-In-First-Out theo **MessageGroupId**, chống trùng bằng **deduplication ID** hoặc content-based dedup (cửa sổ 5 phút); tên kết thúc `.fifo`.
- **Dùng khi nào:** đơn hàng, giao dịch phải xử lý đúng thứ tự; mỗi group một consumer, nhiều group để song song.
- **Vì sao quan trọng:** 300 TPS mỗi API mỗi partition ở chế độ thường (batch 10 → 3,000 msg/s); bật **high throughput mode** để vượt; tối đa 120,000 message in-flight.

### SQS với ASG & bảo mật
- **Là gì:** CloudWatch metric **ApproximateNumberOfMessages** (queue length) làm alarm/target cho ASG.
- **Dùng khi nào:** scale worker theo backlog; SQS làm buffer ghi vào DB để không mất đơn hàng khi DB quá tải.
- **Vì sao quan trọng:** bảo mật: HTTPS, SSE-SQS/SSE-KMS, IAM policy, **SQS access policy** (cross-account, cho SNS/S3 gửi vào).

### Amazon SNS
- **Là gì:** pub/sub: publisher gửi tới **topic**, mọi subscriber nhận; subscriber: SQS, Lambda, Firehose, HTTP(S), email, SMS, mobile push.
- **Dùng khi nào:** gửi một sự kiện tới nhiều hệ thống; nhận thông báo từ CloudWatch Alarm, ASG, S3, CloudFormation...
- **Vì sao quan trọng:** **message filtering** (JSON filter policy) để subscriber chỉ nhận message liên quan.

### Fan-out SNS + SQS
- **Là gì:** publish một lần vào SNS, nhiều SQS queue subscribe; tách hoàn toàn, không mất dữ liệu, có thể thêm queue sau.
- **Dùng khi nào:** S3 event chỉ cho một đích cho mỗi cặp event type + prefix → dùng SNS fan-out ra nhiều queue; cross-Region delivery.
- **Vì sao quan trọng:** **SNS FIFO + SQS FIFO** để fan-out có thứ tự và chống trùng; SNS → Firehose → S3 để lưu message.

### Kinesis Data Streams
- **Là gì:** stream real-time; dữ liệu chia theo **shard** bằng **partition key**; mỗi shard 1 MB/s hoặc 1,000 record/s vào, 2 MB/s ra (shared) hoặc 2 MB/s mỗi consumer (**enhanced fan-out**).
- **Dùng khi nào:** clickstream, IoT, log real-time, cần nhiều consumer đọc lại (replay); retention 24 giờ tới 365 ngày.
- **Vì sao quan trọng:** dữ liệu đã ghi **không xóa được** (immutable) tới khi hết retention; record mặc định 1 MiB, nay cấu hình tới **10 MiB** (cho record lớn ngắt quãng).

### Kinesis capacity modes
- **Là gì:** **Provisioned**: tự chọn số shard, trả theo shard-giờ; **On-demand**: tự scale theo throughput 30 ngày gần nhất, trả theo stream-giờ và GB.
- **Dùng khi nào:** tải đoán được → provisioned; tải biến động/khó đoán → on-demand.
- **Vì sao quan trọng:** "ProvisionedThroughputExceeded" → thêm shard (split) hoặc partition key phân tán hơn.

### Amazon Data Firehose
- **Là gì:** nạp dữ liệu streaming tới **S3, Redshift (qua S3 COPY), OpenSearch, Splunk, HTTP endpoint**, partner (Datadog, New Relic...), Apache Iceberg tables; biến đổi bằng Lambda; chuyển Parquet/ORC; nén.
- **Dùng khi nào:** "load streaming data into S3/Redshift, gần real-time, không quản lý".
- **Vì sao quan trọng:** ⚠️ Lỗi thời: tên **Kinesis Data Firehose** → **Amazon Data Firehose**; buffer interval nay **0–900 giây** (không còn tối thiểu 60 giây). Không lưu trữ, không replay.

### Amazon MQ
- **Là gì:** broker managed cho **ActiveMQ** và **RabbitMQ**, hỗ trợ MQTT, AMQP, STOMP, OpenWire, WSS.
- **Dùng khi nào:** migrate ứng dụng on-prem đang dùng broker chuẩn mà không sửa code sang SQS/SNS.
- **Vì sao quan trọng:** không scale như SQS/SNS; chạy trên server, HA bằng **active/standby Multi-AZ** với storage chung (EFS cho ActiveMQ).

### `[+]` SQS fair queues
- **Là gì:** với standard queue, đặt **MessageGroupId** để SQS giảm hiện tượng một tenant "ồn ào" làm chậm tenant khác.
- **Dùng khi nào:** hệ multi-tenant chia chung một queue.
- **Vì sao quan trọng:** khác FIFO: không đảm bảo thứ tự, chỉ cân bằng độ trễ giữa các group.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| SQS Standard | Queue gần như vô hạn, at-least-once | "decouple", "buffer spikes" |
| SQS FIFO | Đúng thứ tự, exactly-once processing | "order must be preserved" |
| Visibility timeout | Ẩn message khi đang xử lý | "message processed twice" |
| Long polling | Chờ tới 20 giây khi queue trống | "reduce empty receives, cost" |
| Dead-letter queue | Chứa message lỗi nhiều lần | "poison messages" |
| Delay queue | Hoãn giao message tới 15 phút | "postpone delivery" |
| Message group ID | Nhóm thứ tự trong FIFO | "order per customer" |
| SNS topic | Pub/sub tới nhiều subscriber | "notify multiple systems" |
| Fan-out | SNS → nhiều SQS | "same message to many queues" |
| Message filtering | Subscriber chỉ nhận message khớp | "only shipping events" |
| Shard | Đơn vị throughput của Kinesis | "1 MB/s in, 2 MB/s out" |
| Enhanced fan-out | 2 MB/s riêng mỗi consumer/shard | "many consumers, low latency" |
| Data Firehose | Nạp stream vào S3/Redshift/OpenSearch | "near real-time load, serverless" |
| Amazon MQ | ActiveMQ/RabbitMQ managed | "MQTT, AMQP, migrate on-prem broker" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| SQS | SNS | SQS: consumer kéo, message lưu tới 14 ngày; SNS: đẩy tới subscriber, không lưu | "Many receivers" → SNS |
| SQS | Kinesis Data Streams | SQS: xóa sau khi xử lý, một consumer/message; Kinesis: replay, nhiều consumer, thứ tự theo shard | Replay/real-time analytics → Kinesis |
| Standard | FIFO | Standard: vô hạn, có thể trùng/sai thứ tự; FIFO: thứ tự, giới hạn TPS | "Order" → FIFO |
| Data Streams | Data Firehose | Streams: lưu, replay, consumer tự viết; Firehose: nạp vào đích, không lưu | "Load into S3" → Firehose |
| Visibility timeout | Delay queue | Visibility ẩn **sau** khi nhận; delay ẩn **trước** khi nhận lần đầu | |
| SNS/SQS | Amazon MQ | SNS/SQS cloud-native, scale vô hạn; MQ giao thức chuẩn, migrate | "Existing AMQP app" → MQ |
| Provisioned shards | On-demand | Provisioned tự quản shard; On-demand tự scale | Khó đoán → on-demand |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "decouple tiers, absorb traffic spikes" | SQS Standard | |
| "process orders in exact order, no duplicates" | SQS FIFO | |
| "message processed more than once" | Tăng visibility timeout | |
| "reduce cost of empty ReceiveMessage" | Long polling | |
| "messages failing repeatedly block processing" | Dead-letter queue | |
| "one event to multiple services" | SNS fan-out → SQS | |
| "subscriber only receives certain events" | SNS message filtering | |
| "real-time clickstream, multiple consumers, replay" | Kinesis Data Streams | |
| "load streaming data into S3/Redshift, minimal ops" | Amazon Data Firehose | |
| "migrate app using MQTT/AMQP without code change" | Amazon MQ | |
| "scale workers based on queue backlog" | ASG theo ApproximateNumberOfMessages | |
| "message larger than 1 MiB" | Payload trên S3 + Extended Client | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng đặt hàng bị mất đơn khi DB quá tải lúc cao điểm. Cách tách rời đơn giản nhất?
- A. Ghi đơn vào SQS, worker đọc và ghi DB
- B. Tăng instance DB
- C. Dùng SNS gửi email
- D. Dùng CloudFront

<details><summary>Đáp án</summary>

**A.** SQS làm buffer, không mất message khi DB chậm.
- B sai: chỉ dời điểm nghẽn.
- C sai: không lưu đơn hàng.
- D sai: CDN không đệm ghi.
</details>

**Câu 2.** Giao dịch ngân hàng của mỗi khách phải xử lý đúng thứ tự, nhưng khách khác nhau xử lý song song được. Chọn gì?
- A. SQS Standard
- B. SQS FIFO với MessageGroupId = customer ID
- C. SNS Standard
- D. Kinesis Firehose

<details><summary>Đáp án</summary>

**B.** Thứ tự trong group, song song giữa các group.
- A sai: không đảm bảo thứ tự.
- C sai: không đảm bảo thứ tự.
- D sai: không xử lý theo thứ tự cho consumer.
</details>

**Câu 3.** Worker mất 2 phút xử lý mỗi message, nhưng message được worker khác xử lý trùng. Nguyên nhân?
- A. Thiếu DLQ
- B. Visibility timeout (30 giây) ngắn hơn thời gian xử lý
- C. Chưa bật long polling
- D. Message quá lớn

<details><summary>Đáp án</summary>

**B.** Hết timeout, message hiện lại cho consumer khác.
- A sai: DLQ xử lý message lỗi nhiều lần.
- C sai: long polling chỉ giảm poll rỗng.
- D sai: không gây xử lý trùng.
</details>

**Câu 4.** (Chọn 2) Một đơn hàng mới cần gửi cho service thanh toán, kho và email, mỗi service xử lý độc lập và không mất message nếu service tạm ngừng. Chọn gì?
- A. SNS topic
- B. Mỗi service có một SQS queue subscribe vào topic
- C. Một SQS queue chung cho 3 service
- D. Kinesis Firehose
- E. Amazon MQ

<details><summary>Đáp án</summary>

**A, B.** Fan-out SNS → SQS.
- C sai: mỗi message chỉ một consumer xử lý.
- D sai: nạp dữ liệu vào đích lưu trữ.
- E sai: không cần thiết cho ứng dụng cloud-native.
</details>

**Câu 5.** Cần thu clickstream real-time, cho 3 ứng dụng cùng đọc, và đọc lại dữ liệu 7 ngày trước khi sửa bug. Chọn gì?
- A. SQS Standard
- B. Kinesis Data Streams (retention 7 ngày)
- C. SNS
- D. Amazon Data Firehose

<details><summary>Đáp án</summary>

**B.** Nhiều consumer, replay theo retention.
- A sai: message bị xóa sau khi xử lý.
- C sai: không lưu, không replay.
- D sai: không replay, chỉ nạp vào đích.
</details>

**Câu 6.** Cần nạp log streaming vào S3 dạng Parquet, gần real-time, không muốn viết consumer. Chọn gì?
- A. Kinesis Data Streams + EC2 consumer
- B. Amazon Data Firehose với chuyển đổi định dạng
- C. SQS
- D. Amazon MQ

<details><summary>Đáp án</summary>

**B.**
- A sai: phải tự viết và vận hành consumer.
- C, D sai: không nạp/chuyển đổi vào S3.
</details>

**Câu 7.** Ứng dụng on-premises dùng ActiveMQ với giao thức AMQP; muốn chuyển lên AWS ít sửa code nhất. Chọn gì?
- A. SQS
- B. SNS
- C. Amazon MQ
- D. Kinesis

<details><summary>Đáp án</summary>

**C.**
- A, B, D sai: giao thức riêng của AWS, phải sửa code.
</details>

**Câu 8.** Một số message SQS liên tục lỗi do dữ liệu hỏng, làm consumer tốn tài nguyên. Xử lý thế nào?
- A. Cấu hình DLQ với maxReceiveCount phù hợp
- B. Tăng retention lên 14 ngày
- C. Đổi sang FIFO
- D. Tắt visibility timeout

<details><summary>Đáp án</summary>

**A.** Message lỗi quá số lần sẽ vào DLQ để phân tích, sau đó redrive.
- B sai: giữ message lỗi lâu hơn.
- C sai: không giải quyết message hỏng.
- D sai: gây xử lý trùng.
</details>

---

### Nguồn đã đối chiếu
- Amazon SQS message quotas (1 MiB, retention 4–14 ngày, FIFO 300 TPS mỗi partition): https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html
- Amazon SQS FIFO queue quotas (delay 15 phút, long polling 20 giây, 120,000 in-flight): https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-fifo.html
- SQS fair queues: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fair-queues.html
- SNS message filtering: https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html
- Kinesis Data Streams large records (tới 10 MiB): https://docs.aws.amazon.com/streams/latest/dev/large-records.html
- Kinesis Data Streams capacity modes: https://docs.aws.amazon.com/streams/latest/dev/how-do-i-size-a-stream.html
- Amazon Data Firehose buffering hints (0–900 giây): https://docs.aws.amazon.com/firehose/latest/dev/create-configure-backup.html
- Amazon MQ: https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/welcome.html
