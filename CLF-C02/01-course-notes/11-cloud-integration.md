# CLF-C02 — Cloud Integration

> Thuộc **Domain 3** (application integration) và nguyên tắc **loose coupling** của Well-Architected. Trọng tâm: SQS vs SNS vs Kinesis vs Amazon MQ.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- Ứng dụng giao tiếp theo 2 kiểu: **synchronous** (app gọi thẳng app) và **asynchronous / event-based** (app → queue → app).
- Gọi đồng bộ dễ sập khi traffic tăng đột biến → **decouple** bằng **SQS** (queue), **SNS** (pub/sub), **Kinesis** (streaming real-time). Các dịch vụ này scale độc lập với app.
- **SQS** = queue: nhiều producer gửi, nhiều consumer **chia nhau** đọc; message bị xóa sau khi xử lý; giữ mặc định **4 ngày**, tối đa **14 ngày**. **FIFO queue** giữ đúng thứ tự.
- **SNS** = pub/sub: một message gửi tới **mọi subscriber** (email, SMS, Lambda, SQS, HTTP...); **không lưu** message.
- **Kinesis** = **streaming dữ liệu lớn real-time**: Data Streams (ingest), **Amazon Data Firehose** (nạp vào S3, Redshift, OpenSearch).
- **Amazon MQ** = message broker managed cho giao thức mở (**MQTT, AMQP, STOMP, OpenWire, WSS**) — **ActiveMQ** và **RabbitMQ**; dùng khi migrate app on-prem không muốn viết lại cho SQS/SNS.

---

## 2. Core Concepts

### Synchronous vs Asynchronous
- **Là gì:** synchronous: dịch vụ A gọi trực tiếp dịch vụ B; asynchronous: A đẩy vào queue/topic, B đọc khi sẵn sàng.
- **Dùng khi nào:** khi tải đột biến (vd: thường mã hóa 10 video, bỗng phải mã hóa 1.000) thì nên decouple.
- **Vì sao quan trọng:** decoupling giúp lỗi hoặc quá tải ở một thành phần không lan sang thành phần khác.

### Amazon SQS — Standard Queue
- **Là gì:** dịch vụ lâu đời nhất của AWS (hơn 10 năm), fully managed (gần như serverless), dùng để **decouple** ứng dụng.
- **Dùng khi nào:** scale từ 1 tới hàng chục nghìn message/giây; giữ mặc định **4 ngày**, tối đa **14 ngày**; số message trong queue không giới hạn; latency thấp (< 10 ms).
- **Vì sao quan trọng:** message **bị xóa sau khi consumer đọc và xử lý**; nhiều consumer chia nhau công việc và scale ngang.

### SQS để decouple giữa các tier
- **Là gì:** web server (ASG) đẩy request vào SQS; video processing (ASG khác) lấy ra xử lý.
- **Dùng khi nào:** hai tier scale độc lập, có thể scale tier xử lý theo độ dài queue.
- **Vì sao quan trọng:** mẫu kiến trúc kinh điển cho câu "decouple" trong đề.

### Amazon SQS — FIFO Queue
- **Là gì:** **First In First Out**: message được consumer xử lý **đúng thứ tự** gửi.
- **Dùng khi nào:** thứ tự là bắt buộc (vd: giao dịch tài chính, lệnh đặt hàng theo thứ tự).
- **Vì sao quan trọng:** từ khóa "order of messages must be preserved" → SQS FIFO.

### Amazon Kinesis
- **Là gì:** dịch vụ managed để thu thập, xử lý, phân tích **dữ liệu streaming real-time** ở mọi quy mô. Exam: "Kinesis = real-time big data streaming".
- **Dùng khi nào:** **Kinesis Data Streams**: ingest latency thấp từ hàng trăm nghìn nguồn (click stream, IoT, metric, log); **Amazon Data Firehose**: nạp stream vào S3, Redshift, OpenSearch...
- **Vì sao quan trọng:** tên cũ "Kinesis Data Firehose" nay là **Amazon Data Firehose**. Chi tiết từng thành phần vượt mức CLF.

### Amazon SNS
- **Là gì:** dịch vụ **pub/sub**: publisher gửi message vào **một SNS topic**, mọi subscriber của topic đều nhận **toàn bộ** message.
- **Dùng khi nào:** gửi một sự kiện tới nhiều nơi (email, fraud service, shipping service qua SQS...); subscriber: SQS, Lambda, Data Firehose, HTTP(S), SMS & mobile push, email.
- **Vì sao quan trọng:** tới 12.500.000 subscription/topic, giới hạn 100.000 topic (theo slide); SNS **không giữ** message.

### Amazon MQ
- **Là gì:** message broker managed cho **ActiveMQ** và **RabbitMQ**, dùng giao thức mở (MQTT, AMQP, STOMP, OpenWire, WSS).
- **Dùng khi nào:** migrate app on-prem đang dùng giao thức mở lên cloud **mà không phải viết lại** cho SQS/SNS (vốn dùng giao thức độc quyền của AWS).
- **Vì sao quan trọng:** Amazon MQ **không scale bằng** SQS/SNS, chạy trên server (có thể Multi-AZ với failover); có cả tính năng queue (~SQS) và topic (~SNS).

### `[+]` SNS + SQS fan-out
- **Là gì:** publish một lần vào SNS topic, nhiều SQS queue subscribe để mỗi service xử lý độc lập.
- **Dùng khi nào:** một sự kiện "đơn hàng mới" cần đi tới shipping, fraud, email cùng lúc và không mất message.
- **Vì sao quan trọng:** kết hợp "gửi cho nhiều bên" của SNS với "giữ message" của SQS.

### `[+]` Amazon SES (Simple Email Service)
- **Là gì:** dịch vụ gửi **email** quy mô lớn (transactional, marketing) và nhận email.
- **Dùng khi nào:** app cần gửi email xác nhận đơn hàng, đặt lại mật khẩu, bản tin.
- **Vì sao quan trọng:** SES nằm trong danh sách in-scope CLF-C02 (Business Applications). "Send bulk emails from application" → SES.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Decoupling | Tách thành phần để scale và lỗi độc lập | "loosely coupled", "decouple" |
| Synchronous | Gọi trực tiếp, chờ phản hồi | "direct call between services" |
| Asynchronous | Gửi qua queue/topic, xử lý sau | "event-based", "queue" |
| Amazon SQS | Queue managed, consumer chia nhau đọc | "buffer requests", "decouple tiers" |
| Message retention | Thời gian SQS giữ message (mặc định 4, tối đa 14 ngày) | "store messages up to 14 days" |
| SQS FIFO | Queue giữ đúng thứ tự | "preserve order" |
| Amazon SNS | Pub/sub, gửi tới mọi subscriber | "notify multiple subscribers", "fan out" |
| SNS topic | Kênh để publisher gửi message | "publish once, deliver to many" |
| Amazon Kinesis | Streaming dữ liệu lớn real-time | "real-time analytics", "clickstream", "IoT telemetry" |
| Kinesis Data Streams | Ingest stream latency thấp | "ingest real-time data" |
| Amazon Data Firehose | Nạp stream vào S3/Redshift/OpenSearch | "load streaming data into S3" |
| Amazon MQ | Broker managed ActiveMQ/RabbitMQ | "MQTT, AMQP", "migrate without rewriting" |
| Amazon SES | Gửi/nhận email quy mô lớn | "send marketing/transactional email" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| SQS | SNS | Queue: consumer **chia nhau**, giữ message tới 14 ngày vs pub/sub: **mọi** subscriber nhận, không lưu | "Many receivers get same message" → SNS |
| SQS Standard | SQS FIFO | Thứ tự không đảm bảo vs đúng thứ tự | "Order matters" → FIFO |
| SQS/SNS | Kinesis | Messaging decouple vs streaming dữ liệu lớn real-time | "Real-time big data streaming" → Kinesis |
| SQS/SNS | Amazon MQ | Giao thức độc quyền AWS, scale cực lớn vs giao thức mở, migrate từ on-prem | "MQTT/AMQP existing app" → Amazon MQ |
| Kinesis Data Streams | Amazon Data Firehose | Ingest & xử lý vs nạp vào đích (S3, Redshift) | "Load into S3" → Firehose |
| SNS | SES | Thông báo đa kênh (SMS, push, email, Lambda) vs dịch vụ chuyên email | "Bulk marketing email" → SES |
| SNS | EventBridge | Pub/sub đơn giản vs event bus có rule lọc, schedule, nguồn SaaS | Chi tiết EventBridge ở chương 12 |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "decouple web tier from processing tier" | Amazon SQS | |
| "messages must be processed in order" | SQS FIFO | |
| "keep messages up to 14 days" | SQS | |
| "send one message to many subscribers" | Amazon SNS | |
| "email, SMS and Lambda notification from one event" | SNS | |
| "real-time clickstream / IoT streaming analytics" | Amazon Kinesis | |
| "load streaming data into S3/Redshift" | Amazon Data Firehose | |
| "migrate on-prem app using AMQP/MQTT without code changes" | Amazon MQ | |
| "send transactional emails from app" | Amazon SES | |
| "process spikes of traffic without losing requests" | SQS (buffer) | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Website nhận đơn hàng tăng đột biến khiến hệ thống xử lý phía sau quá tải. Cách tách hai thành phần để không mất đơn?
- A. Amazon SNS
- B. Amazon SQS
- C. Amazon Kinesis Data Firehose
- D. Route 53

<details><summary>Đáp án</summary>

**B.** SQS làm buffer, giữ message tới khi consumer xử lý xong.
- A sai: SNS không lưu message.
- C sai: nạp dữ liệu streaming vào đích.
- D sai: DNS.
</details>

**Câu 2.** Một sự kiện "đơn hàng mới" phải gửi đồng thời tới dịch vụ giao hàng, dịch vụ chống gian lận và email cho khách. Chọn gì?
- A. Amazon SNS
- B. SQS FIFO
- C. Amazon MQ
- D. AWS Glue

<details><summary>Đáp án</summary>

**A.** SNS pub/sub gửi message tới mọi subscriber.
- B sai: consumer chia nhau message, không phải mỗi bên nhận một bản.
- C sai: dùng cho migrate giao thức mở.
- D sai: ETL.
</details>

**Câu 3.** Hệ thống giao dịch yêu cầu message được xử lý **đúng thứ tự** gửi. Chọn gì?
- A. SQS Standard
- B. SQS FIFO
- C. Amazon SNS
- D. Amazon Kinesis Data Firehose

<details><summary>Đáp án</summary>

**B.** FIFO giữ thứ tự.
- A sai: không đảm bảo thứ tự.
- C sai: pub/sub.
- D sai: nạp dữ liệu vào đích.
</details>

**Câu 4.** Công ty thu thập clickstream từ hàng triệu người dùng và cần phân tích **real-time**. Chọn gì?
- A. Amazon Kinesis
- B. Amazon SQS
- C. Amazon MQ
- D. Amazon SES

<details><summary>Đáp án</summary>

**A.** Kinesis dành cho streaming dữ liệu lớn real-time.
- B sai: queue decouple, không phải phân tích stream.
- C sai: message broker.
- D sai: email.
</details>

**Câu 5.** Ứng dụng on-prem dùng RabbitMQ với giao thức AMQP. Công ty muốn chuyển lên AWS mà không sửa code. Chọn gì?
- A. Amazon SQS
- B. Amazon SNS
- C. Amazon MQ
- D. Amazon Kinesis

<details><summary>Đáp án</summary>

**C.** Amazon MQ là broker managed cho ActiveMQ/RabbitMQ, hỗ trợ giao thức mở.
- A, B sai: giao thức độc quyền AWS, phải viết lại app.
- D sai: streaming.
</details>

**Câu 6.** (Chọn 2) Phát biểu nào về Amazon SQS là **đúng**?
- A. Message được giữ tối đa 14 ngày
- B. Mọi consumer đều nhận cùng một message
- C. Message bị xóa sau khi consumer xử lý
- D. SQS chỉ hỗ trợ giao thức AMQP
- E. SQS là dịch vụ pub/sub

<details><summary>Đáp án</summary>

**A, C.**
- B sai: consumer chia nhau message; mỗi bên nhận cùng message là đặc điểm của SNS.
- D sai: AMQP là của Amazon MQ.
- E sai: pub/sub là SNS.
</details>

**Câu 7.** Dữ liệu log streaming cần được tự động nạp vào S3 để phân tích sau với ít công sức nhất. Chọn gì?
- A. Amazon Data Firehose
- B. Amazon SNS
- C. Amazon SQS FIFO
- D. Amazon MQ

<details><summary>Đáp án</summary>

**A.** Data Firehose nạp dữ liệu streaming vào S3, Redshift, OpenSearch.
- B, C, D sai: dịch vụ messaging, không phải delivery stream vào storage.
</details>

**Câu 8.** (Chọn 2) Lợi ích của việc decouple ứng dụng bằng SQS/SNS là gì?
- A. Các thành phần scale độc lập
- B. Lỗi ở một thành phần ít ảnh hưởng tới thành phần khác
- C. Loại bỏ hoàn toàn nhu cầu xử lý lỗi
- D. Bắt buộc mọi thành phần chạy trên cùng EC2
- E. Tăng mức độ phụ thuộc giữa các service

<details><summary>Đáp án</summary>

**A, B.**
- C sai: vẫn cần xử lý lỗi.
- D sai: ngược lại, các thành phần tách biệt.
- E sai: decouple làm giảm phụ thuộc.
</details>

---

### Nguồn đã đối chiếu
- Amazon SQS message retention & visibility timeout: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html
- Amazon SQS FIFO queues: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fifo-queues.html
- Amazon SNS: https://docs.aws.amazon.com/sns/latest/dg/welcome.html
- Amazon Data Firehose: https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html
- Amazon MQ: https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/welcome.html
- CLF-C02 in-scope services (Amazon SES, Kinesis): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-in-scope-services.html
