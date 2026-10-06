# SOA-C03 — Monitoring, Audit & Performance: CloudWatch, CloudTrail, AWS Config

> Thuộc **Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization (22%)** và **Domain 4: Security and Compliance (16%)**: metric, log, alarm, EventBridge, ghi lại API call, theo dõi cấu hình và tự khắc phục.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- Chia ba vai trò:
  - **CloudWatch**: metric, log, alarm (hiệu năng).
  - **CloudTrail**: ai gọi API nào, lúc nào.
  - **AWS Config**: cấu hình tài nguyên thay đổi ra sao, có tuân thủ không.
- Metric EC2 mặc định 5 phút, Detailed 1 phút; **không có memory và disk space**, phải đẩy qua Unified CloudWatch Agent.
- Custom metric (`PutMetricData`): standard 60 giây, high resolution 1/5/10/30 giây. Chỉ nhận timestamp trong khoảng **2 tuần trước đến 2 giờ sau**.
- **Logs Insights** là công cụ truy vấn, không real-time. **Export S3** có thể trễ tới 12 giờ. Muốn real-time → **subscription filter** tới Kinesis Data Streams, Data Firehose, Lambda.
- **Alarm** có ba trạng thái OK / ALARM / INSUFFICIENT_DATA; action tới EC2, Auto Scaling, SNS, Lambda… **Composite alarm** gộp nhiều alarm bằng AND/OR.
- **CloudTrail**:
  - Bật sẵn, event history 90 ngày.
  - **Data event tắt mặc định**.
  - Log file tới S3 trung bình khoảng 5 phút, không real-time.
- **AWS Config** theo Region, chỉ phát hiện (không chặn); tự khắc phục qua **SSM Automation**.
- ⚠️ Lỗi thời:
  - CloudWatch Events → EventBridge.
  - Kinesis Data Firehose → Amazon Data Firehose.
  - CloudWatch Logs Agent → Unified CloudWatch Agent.
  - CloudTrail Lake đóng với khách mới.

---

## 2. Core Concepts

### CloudWatch metrics & custom metrics
- **Là gì:** metric thuộc namespace, có tối đa 30 dimension; mỗi tổ hợp dimension là một custom metric tính phí riêng.
- **Dùng khi nào:** RAM, disk, số user đăng nhập, metric ứng dụng → `PutMetricData` với `StorageResolution`.
- **Vì sao quan trọng:** đồng hồ instance lệch làm metric bị bỏ âm thầm. `[+]` Retention: dưới 60 giây giữ 3 giờ, 60 giây giữ 15 ngày, 5 phút giữ 63 ngày, 1 giờ giữ 455 ngày; dữ liệu tự gộp sang độ phân giải thô hơn.

### Anomaly detection & dashboards
- **Là gì:** anomaly detection học dải giá trị kỳ vọng từ lịch sử; dashboard là **global**, gộp được nhiều account và Region, chia sẻ cho người không có tài khoản AWS.
- **Dùng khi nào:** metric có tính mùa vụ, ngưỡng tĩnh hay báo sai → alarm theo anomaly band.
- **Vì sao quan trọng:** loại trừ được khoảng thời gian bất thường khỏi dữ liệu huấn luyện.

### CloudWatch Logs
- **Là gì:** log group (thường một ứng dụng), log stream (một instance/container/file); retention 1 ngày–10 năm hoặc **không hết hạn (mặc định)**; mã hóa mặc định, dùng được KMS key riêng.
- **Dùng khi nào:** nguồn log: SDK, Unified CloudWatch Agent, Lambda, ECS, VPC Flow Logs, API Gateway, CloudTrail, Route 53 query logs.
- **Vì sao quan trọng:** không đặt retention → chi phí lưu tăng mãi. Metric filter biến log thành metric để đặt alarm.

### Logs Insights, export & subscription
- **Là gì:** Logs Insights là ngôn ngữ truy vấn, chạy trên nhiều log group và nhiều account; `CreateExportTask` đẩy log sang S3 (trễ tới 12 giờ, task timeout sau 24 giờ); subscription filter stream real-time.
- **Dùng khi nào:** điều tra ad-hoc → Insights; lưu trữ theo lô → export; gom log nhiều account về một chỗ, xử lý gần real-time → subscription (cross-account dùng destination policy + IAM role).
- **Vì sao quan trọng:** đề hỏi "real-time" thì loại export S3 và Logs Insights.

### Logs data protection
- **Là gì:** phát hiện và **che dữ liệu nhạy cảm** (email, số thẻ, mật khẩu…) bằng managed data identifiers và identifier tùy chỉnh.
- **Dùng khi nào:** log chứa PII, cần tuân thủ.
- **Vì sao quan trọng:** chỉ principal có `logs:Unmask` xem được giá trị gốc; cảnh báo bằng metric `LogEventsWithFindings`.

### CloudWatch alarms
- **Là gì:** đánh giá theo period, evaluation periods, datapoints to alarm (M trong N); action: stop / terminate / reboot / recover EC2, Auto Scaling, SNS, Lambda, Systems Manager.
- **Dùng khi nào:** "chỉ báo khi vượt ngưỡng 3 trong 5 điểm" → M of N; kiểm thử đường thông báo → `SetAlarmState`.
- **Vì sao quan trọng:** high-resolution metric cho period 10 hoặc 30 giây. `[+]` **Treat missing data** (missing / notBreaching / breaching / ignore) quyết định alarm có rơi vào INSUFFICIENT_DATA khi metric ngắt quãng.

### Composite alarms
- **Là gì:** alarm theo trạng thái của nhiều alarm khác với AND / OR / NOT.
- **Dùng khi nào:** giảm báo động giả, ví dụ chỉ gọi người trực khi CPU **và** latency cùng cao.
- **Vì sao quan trọng:** composite alarm đánh giá trạng thái alarm, không đánh giá metric thô.

### Synthetics, Container Insights, Internet Monitor, Network Synthetic Monitor
- **Là gì:** **Synthetics canary** (Node.js/Python, headless browser) kiểm tra URL/API theo lịch; **Container Insights** cho ECS/EKS/Fargate; **Internet Monitor** đo ảnh hưởng của Internet tới người dùng; **Network Synthetic Monitor** đo packet loss, latency giữa AWS và on-prem qua Direct Connect/VPN, không cần agent.
- **Dùng khi nào:** phát hiện lỗi trước khách hàng → canary; metric theo task/container → Container Insights (enhanced observability).
- **Vì sao quan trọng:** canary chạy trong VPC cần bật DNS hostnames/resolution và có NAT hoặc endpoint tới CloudWatch, S3. ⚠️ Lỗi thời: tên "Network Monitor" trong slide → docs gọi **Network Synthetic Monitor**.

### Amazon EventBridge
- **Là gì:** rule theo **schedule** hoặc **event pattern**; bus default, partner, custom; archive và replay; Schema Registry; Pipes.
- **Dùng khi nào:** phản ứng sự kiện dịch vụ AWS (EC2 state change, Config non-compliant, Health) → Lambda, SSM Automation, SNS, Step Functions.
- **Vì sao quan trọng:** resource-based policy cho phép gửi event cross-account / cross-Region; event gửi lỗi theo retry policy rồi vào DLQ SQS.

### Giám sát quota
- **Là gì:** Service Quotas tạo CloudWatch alarm khi mức dùng gần chạm quota; Trusted Advisor cũng có check service limits.
- **Dùng khi nào:** cảnh báo trước khi chạm giới hạn (Lambda concurrency, số EIP…).
- **Vì sao quan trọng:** Service Quotas bao phủ rộng hơn check của Trusted Advisor. Quota thực tế của account: kiểm tra Service Quotas console hoặc CLI.

### AWS CloudTrail
- **Là gì:** ghi API call từ console, SDK, CLI, dịch vụ AWS; trail có thể áp mọi Region (mặc định khi tạo trên console) hoặc một Region; gửi tới S3 và/hoặc CloudWatch Logs.
- **Dùng khi nào:** "ai đã xóa tài nguyên" → CloudTrail; lưu lâu hơn 90 ngày → trail tới S3 + Athena.
- **Vì sao quan trọng:** không real-time (log file khoảng mỗi 5 phút). **Organization trail** ghi mọi account, account thành viên không sửa hay xóa được.

### Loại event & tính toàn vẹn của CloudTrail
- **Là gì:** **management event** (bật mặc định, tách read/write); **data event** (S3 object-level, Lambda Invoke… tắt mặc định); **Insights event** phát hiện bất thường về tần suất API.
- **Dùng khi nào:** audit ai đọc object S3 → bật data event; chứng minh log không bị sửa → **log file integrity validation** (digest file mỗi giờ, SHA-256, ký RSA).
- **Vì sao quan trọng:** `[+]` ⚠️ Lỗi thời: CloudTrail Lake **đóng với khách mới từ 31/05/2026**; truy vấn lâu dài dùng trail → S3 → Athena.

### AWS Config
- **Là gì:** ghi configuration item theo thời gian và đánh giá rule (AWS managed hoặc custom Lambda/Guard) khi có thay đổi, theo lịch, hoặc cả hai.
- **Dùng khi nào:** "SSH có mở 0.0.0.0/0?", "bucket có public?", "ALB đã đổi thế nào?".
- **Vì sao quan trọng:** **theo Region**, **không ngăn được hành động**, chỉ phát hiện. Không thấy lịch sử của một loại tài nguyên → recorder không ghi loại đó.

### Config remediation, aggregator, conformance pack
- **Là gì:** remediation chạy **SSM Automation document** (có retry); aggregator ở account trung tâm gom kết quả đa account/Region; conformance pack đóng gói rule + remediation.
- **Dùng khi nào:** tự đóng port 22 mở toang, tự bật logging cho bucket; báo cáo tuân thủ cả Organization.
- **Vì sao quan trọng:** NON_COMPLIANT có thể kích hoạt EventBridge → SNS/Lambda. Rule phải tồn tại ở từng account nguồn (triển khai bằng StackSets hoặc conformance pack cấp Organization).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| `PutMetricData` | Đẩy custom metric | "publish application metric" |
| High-resolution metric | 1–30 giây | "sub-minute monitoring" |
| Metric filter | Biến log thành metric | "alarm on ERROR in logs" |
| Subscription filter | Stream log real-time | "real-time log processing" |
| Logs Insights | Truy vấn log | "ad-hoc query across log groups" |
| Composite alarm | Gộp alarm AND/OR | "reduce alarm noise" |
| Synthetics canary | Kiểm tra endpoint theo lịch | "detect issues before customers" |
| EventBridge rule | Schedule hoặc event pattern | "react to EC2 state change" |
| Management event | API control plane | "who terminated the instance" |
| Data event | API mức object/function | "who read S3 objects" |
| Log file integrity validation | Digest chứng minh không sửa | "prove logs were not tampered" |
| Organization trail | Trail cho mọi account | "member accounts cannot disable" |
| Config rule | Đánh giá tuân thủ | "detect non-compliant resources" |
| Config aggregator | Gom tuân thủ đa account | "central compliance view" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CloudWatch | CloudTrail | Hiệu năng, log, alarm; ai gọi API | "Who" → CloudTrail |
| CloudTrail | AWS Config | API call; trạng thái cấu hình và tuân thủ | "Configuration history" → Config |
| AWS Config | IAM / SCP | Phát hiện sau; chặn trước | "Prevent" → không phải Config |
| Export S3 | Subscription filter | Theo lô, trễ tới 12 giờ; real-time | Real-time → subscription |
| Logs Insights | Metric filter + alarm | Truy vấn ad-hoc; cảnh báo liên tục | Alerting → metric filter |
| Alarm thường | Composite alarm | Một metric; trạng thái nhiều alarm | Giảm nhiễu → composite |
| Management event | Data event | Bật mặc định; tắt mặc định | S3 GetObject → data event |
| Event history 90 ngày | Trail tới S3 | Console, 90 ngày; lâu dài, Athena | > 90 ngày → trail |
| Synthetics | Internet Monitor | Script chủ động kiểm tra; quan sát Internet ảnh hưởng người dùng | "Test login flow" → canary |
| Service Quotas alarm | Trusted Advisor | Bao phủ rộng; tập check giới hạn | Gần chạm quota → Service Quotas |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "memory utilization of EC2" | Unified CloudWatch Agent | |
| "custom metric data missing" | Đồng hồ lệch, sai IAM, timestamp ngoài cửa sổ | |
| "alarm when ERROR appears in logs" | Metric filter → alarm → SNS | |
| "stream logs from 50 accounts to one place" | Subscription filter + cross-account destination | |
| "logs available in S3 next day is fine" | `CreateExportTask` | |
| "reduce alarm noise, page only when both fire" | Composite alarm | |
| "alarm flaps in INSUFFICIENT_DATA" | Treat missing data | |
| "test website login every 5 minutes" | Synthetics canary | |
| "who deleted the security group" | CloudTrail | |
| "who downloaded objects from bucket" | CloudTrail data events | |
| "prove CloudTrail logs not modified" | Log file integrity validation | |
| "auto-close SSH open to world" | Config rule + SSM Automation remediation | |
| "compliance across all accounts and Regions" | Config aggregator | |
| "alert before hitting service quota" | Service Quotas CloudWatch alarm | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Cần alarm khi log ứng dụng xuất hiện hơn 10 lần chữ `ERROR` trong 5 phút. Chọn gì?
- A. Logs Insights query
- B. Metric filter trên log group + CloudWatch alarm
- C. Export log sang S3
- D. CloudTrail Insights

<details><summary>Đáp án</summary>

**B.**
- A sai: Insights để truy vấn ad-hoc, không phải cảnh báo liên tục.
- C sai: trễ tới 12 giờ.
- D sai: phát hiện bất thường API call, không phải log ứng dụng.
</details>

**Câu 2.** (Chọn 2) Đội bảo mật cần gom log từ 40 account về một account trung tâm gần real-time để phân tích. Chọn gì?
- A. Subscription filter ở các account nguồn
- B. CloudWatch Logs destination cross-account (Kinesis Data Streams / Firehose) ở account trung tâm
- C. `CreateExportTask` mỗi đêm
- D. Logs Insights
- E. CloudWatch dashboard

<details><summary>Đáp án</summary>

**A, B.**
- C sai: theo lô, không gần real-time.
- D sai: chỉ truy vấn.
- E sai: chỉ hiển thị.
</details>

**Câu 3.** Một security group bị xóa lúc nửa đêm. Cần biết ai xóa. Xem ở đâu đầu tiên?
- A. CloudWatch Logs
- B. CloudTrail event history
- C. AWS Config rule
- D. VPC Flow Logs

<details><summary>Đáp án</summary>

**B.** `DeleteSecurityGroup` là management event, có trong event history 90 ngày.
- A sai: không chứa API call nếu chưa gửi trail tới đó.
- C sai: Config cho biết cấu hình thay đổi, nhưng "ai gọi" là việc của CloudTrail.
- D sai: ghi metadata traffic.
</details>

**Câu 4.** Kiểm toán viên yêu cầu biết ai đã đọc object trong một bucket S3 nhạy cảm. Cần bật gì?
- A. CloudTrail management events
- B. CloudTrail data events cho bucket đó
- C. Config rule `s3-bucket-public-read-prohibited`
- D. CloudWatch metric `NumberOfObjects`

<details><summary>Đáp án</summary>

**B.** `GetObject` là data event, tắt mặc định.
- A sai: không ghi thao tác object.
- C sai: kiểm tra cấu hình public.
- D sai: không cho biết ai đọc.
</details>

**Câu 5.** Cần tự động gỡ rule SSH 0.0.0.0/0 khỏi mọi security group ngay khi xuất hiện. Chọn gì?
- A. AWS Config rule `restricted-ssh` + remediation bằng SSM Automation
- B. CloudTrail Insights
- C. IAM policy
- D. CloudWatch dashboard

<details><summary>Đáp án</summary>

**A.**
- B sai: phát hiện bất thường tần suất API.
- C sai: IAM không sửa cấu hình đã tồn tại.
- D sai: chỉ hiển thị.
</details>

**Câu 6.** Alarm cho Lambda hiếm khi chạy thường nằm ở INSUFFICIENT_DATA. Muốn coi không có dữ liệu là bình thường. Chỉnh gì?
- A. Tăng period lên 1 ngày
- B. Đặt treat missing data = `notBreaching`
- C. Chuyển sang composite alarm
- D. Bật Detailed Monitoring

<details><summary>Đáp án</summary>

**B.**
- A sai: vẫn có thể thiếu dữ liệu, lại phản ứng chậm.
- C sai: không đổi cách xử lý dữ liệu thiếu.
- D sai: áp cho EC2.
</details>

**Câu 7.** (Chọn 2) Cần chứng minh log CloudTrail không bị sửa và đảm bảo mọi account trong Organization đều ghi log mà account thành viên không tắt được. Chọn gì?
- A. Log file integrity validation
- B. Organization trail
- C. CloudWatch composite alarm
- D. Config aggregator
- E. Logs Insights

<details><summary>Đáp án</summary>

**A, B.**
- C sai: không liên quan tính toàn vẹn log.
- D sai: gom tuân thủ Config.
- E sai: chỉ truy vấn.
</details>

**Câu 8.** Cần cảnh báo khi số Elastic IP sắp chạm quota của account. Chọn gì?
- A. Service Quotas + CloudWatch alarm theo mức dùng
- B. CloudTrail
- C. Config rule
- D. Synthetics canary

<details><summary>Đáp án</summary>

**A.**
- B sai: ghi API, không so với quota.
- C sai: đánh giá cấu hình từng tài nguyên.
- D sai: kiểm tra endpoint.
</details>

---

### Nguồn đã đối chiếu
- CloudWatch concepts (metric retention 3 giờ / 15 ngày / 63 ngày / 455 ngày): https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html
- Alarm actions: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/alarm-actions.html
- Export logs to S3 (trễ tới 12 giờ, timeout 24 giờ): https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/S3Export.html
- Network Synthetic Monitor: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/what-is-network-monitor.html
- How CloudTrail works (log file khoảng 5 phút): https://docs.aws.amazon.com/awscloudtrail/latest/userguide/how-cloudtrail-works.html
- CloudTrail Lake availability change: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake-service-availability-change.html
- EventBridge targets & IAM roles: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-events-iam-roles.html
- AWS Config remediation: https://docs.aws.amazon.com/config/latest/developerguide/remediation.html
