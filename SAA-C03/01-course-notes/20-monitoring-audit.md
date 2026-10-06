# SAA-C03 — Monitoring & Audit: CloudWatch, EventBridge, CloudTrail, Config

> Thuộc **Domain 1: Design Secure Architectures** và **Domain 2: Design Resilient Architectures**: metric, log, alarm, sự kiện, audit API call và tuân thủ cấu hình.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **CloudWatch Metrics**: metric theo namespace + dimension; EC2 mặc định 5 phút (detailed 1 phút); **RAM và disk không có sẵn** → cần **CloudWatch agent**; custom metric tới độ phân giải 1 giây.
- **CloudWatch Logs**: log group/stream, retention 1 ngày–10 năm hoặc vĩnh viễn; **Logs Insights** truy vấn; **subscription filter** stream real-time sang Kinesis/Firehose/Lambda/OpenSearch; export sang S3 (batch, tới 12 giờ).
- ⚠️ Lỗi thời: **CloudWatch Logs agent** cũ → dùng **unified CloudWatch agent** (log + metric hệ thống, cấu hình qua SSM Parameter Store).
- **Alarm**: OK / INSUFFICIENT_DATA / ALARM; action: EC2 (stop/terminate/reboot/recover), ASG, SNS; **composite alarm** gộp nhiều alarm bằng AND/OR.
- **EventBridge**: lịch (cron) + rule theo event pattern; event bus default/partner/custom; **archive & replay**; **schema registry**; resource-based policy cho cross-account.
- **CloudTrail**: lịch sử API call (Console, SDK, CLI) — **management events** (mặc định), **data events** (S3 object, Lambda invoke; tắt mặc định), **Insights events**; giữ 90 ngày, lâu hơn → trail ghi vào S3 rồi Athena.
- ⚠️ **CloudTrail Lake** đóng với khách mới từ 31/05/2026.
- **AWS Config**: ghi lịch sử cấu hình tài nguyên, đánh giá **rule** compliance (managed/custom), **remediation** bằng SSM Automation; không ngăn hành động, chỉ phát hiện.

---

## 2. Core Concepts

### CloudWatch Metrics
- **Là gì:** metric cho mọi dịch vụ AWS (CPUUtilization, NetworkIn...), thuộc **namespace**, có tối đa 30 **dimension**, có timestamp; dashboard tổng hợp.
- **Dùng khi nào:** **Metric Streams** đẩy metric gần real-time qua Firehose tới S3, Redshift, OpenSearch hoặc bên thứ ba (Datadog, Splunk...).
- **Vì sao quan trọng:** "giám sát RAM, disk của EC2" → cài **unified CloudWatch agent** (metric hệ thống không có mặc định).

### CloudWatch Logs
- **Là gì:** nguồn: SDK, CloudWatch agent, Beanstalk, ECS, Lambda, VPC Flow Logs, API Gateway, CloudTrail, Route 53 DNS query; mã hóa KMS ở cấp log group.
- **Dùng khi nào:** **Logs Insights** tìm, lọc, thống kê bằng ngôn ngữ truy vấn riêng (truy vấn nhiều log group, nhiều account); **Live Tail** xem log real-time.
- **Vì sao quan trọng:** export S3 dùng **CreateExportTask** (không real-time); cần real-time → **subscription filter**; gom log nhiều account/Region bằng subscription → Kinesis/Firehose ở account trung tâm.

### CloudWatch agent
- **Là gì:** agent chạy trên EC2 hoặc on-prem gửi log và metric hệ thống (RAM, disk, process, swap...).
- **Dùng khi nào:** cần quyền IAM phù hợp; cấu hình tập trung lưu trong SSM Parameter Store.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "CloudWatch Logs Agent" (chỉ gửi log) đã deprecated → dùng **unified agent** cho cả log và metric.

### CloudWatch Alarms
- **Là gì:** theo dõi một metric (hoặc biểu thức math); trạng thái OK, INSUFFICIENT_DATA, ALARM; period có thể 10/30 giây cho high-resolution custom metric.
- **Dùng khi nào:** action: **EC2 action** (stop, terminate, reboot, **recover** sang host khác giữ IP/metadata), **ASG scaling**, **SNS** (→ Lambda làm gì cũng được).
- **Vì sao quan trọng:** **composite alarm** giảm nhiễu (vd: CPU cao **và** IOPS cao mới báo); alarm cũng tạo được từ **metric filter** trên Logs.

### Amazon EventBridge
- **Là gì:** bus sự kiện serverless: rule theo **schedule** (cron) hoặc **event pattern** (vd: IAM root sign-in, EC2 state change), gửi tới Lambda, SQS, SNS, Step Functions, ECS task, Kinesis...
- **Dùng khi nào:** **partner event bus** nhận sự kiện SaaS (Zendesk, Datadog); **custom bus** cho ứng dụng riêng; **archive & replay** để chạy lại sự kiện; **schema registry** sinh code cho sự kiện.
- **Vì sao quan trọng:** **resource-based policy** trên event bus để nhận sự kiện từ account khác (gom sự kiện của Organization về một bus trung tâm).

### CloudWatch Insights
- **Là gì:** **Container Insights** (ECS, EKS, Kubernetes trên EC2, Fargate), **Lambda Insights** (CPU, memory, cold start), **Contributor Insights** (top-N: IP gửi nhiều request, URL lỗi nhiều nhất), **Application Insights** (dashboard tự động cho app .NET/SQL Server/Java, gợi ý nguyên nhân sự cố).
- **Dùng khi nào:** tìm "top talkers" trong VPC Flow Logs → Contributor Insights.
- **Vì sao quan trọng:** đề phân biệt các loại Insights theo đối tượng giám sát.

### AWS CloudTrail
- **Là gì:** ghi lại API call trong account; bật mặc định; trail áp cho mọi Region (mặc định) hoặc một Region; gửi tới CloudWatch Logs hoặc S3.
- **Dùng khi nào:** "ai đã xóa tài nguyên?" → CloudTrail; **management events** (thao tác trên tài nguyên, tách read/write); **data events** (S3 GetObject/PutObject, Lambda Invoke; khối lượng lớn, tắt mặc định).
- **Vì sao quan trọng:** **CloudTrail Insights** phát hiện hoạt động bất thường (vượt giới hạn dịch vụ, burst IAM action); lịch sử 90 ngày, lâu hơn → S3 + Athena. ⚠️ **CloudTrail Lake** đóng với khách mới từ 31/05/2026.

### CloudTrail + EventBridge
- **Là gì:** mọi API call ghi trong CloudTrail đều thành sự kiện trong EventBridge.
- **Dùng khi nào:** cảnh báo khi có `DeleteTable`, `AssumeRole`, thay đổi SG inbound → SNS.
- **Vì sao quan trọng:** mẫu "phát hiện và phản ứng gần real-time với API call nguy hiểm".

### AWS Config
- **Là gì:** ghi cấu hình tài nguyên và thay đổi theo thời gian; **Config rules** (AWS managed, hơn 75 loại, hoặc custom bằng Lambda/Guard) đánh giá theo thay đổi hoặc định kỳ; dịch vụ theo Region, gom được về một account (aggregator).
- **Dùng khi nào:** "SG có mở SSH 0.0.0.0/0 không?", "bucket có public không?", "ACM cert sắp hết hạn?".
- **Vì sao quan trọng:** Config **không ngăn** hành động; **remediation** qua **SSM Automation** (vd: tự vô hiệu access key cũ); thông báo qua EventBridge/SNS.

### CloudWatch vs CloudTrail vs Config
- **Là gì:** CloudWatch = hiệu năng (metric, log, alarm, dashboard); CloudTrail = **ai** gọi **API gì** khi nào; Config = **cấu hình** tài nguyên thay đổi ra sao, có tuân thủ không.
- **Dùng khi nào:** với một ELB: CloudWatch theo dõi request/5xx; Config theo dõi rule SG/SSL; CloudTrail ghi ai sửa cấu hình ELB.
- **Vì sao quan trọng:** cặp so sánh xuất hiện thường xuyên nhất của chương.

### `[+]` X-Ray chuyển sang OpenTelemetry
- **Là gì:** X-Ray SDK và Daemon vào **maintenance mode từ 25/02/2026** (chỉ sửa lỗi bảo mật); AWS khuyến nghị instrument bằng **OpenTelemetry** (ADOT) và gửi trace qua CloudWatch agent/OTel collector.
- **Dùng khi nào:** distributed tracing cho microservice: dịch vụ X-Ray vẫn dùng, đổi cách instrument.
- **Vì sao quan trọng:** đáp án "X-Ray để trace request qua nhiều service" vẫn đúng ở mức khái niệm.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Namespace / Dimension | Nhóm metric / thuộc tính metric | "custom metrics" |
| Detailed monitoring | Metric EC2 mỗi 1 phút | "faster ASG reaction" |
| Unified CloudWatch agent | Gửi log + RAM/disk | "memory utilization" |
| Logs Insights | Truy vấn log | "search logs with queries" |
| Subscription filter | Stream log real-time | "real-time log processing" |
| Metric filter | Đếm mẫu trong log thành metric | "alarm on ERROR count" |
| Composite alarm | Gộp alarm bằng AND/OR | "reduce alarm noise" |
| EC2 recover action | Chuyển instance sang host mới | "status check failed, keep IP" |
| EventBridge schedule | Cron serverless | "run job every hour" |
| Archive & replay | Lưu và phát lại sự kiện | "reprocess past events" |
| Contributor Insights | Top-N contributor | "top IP addresses" |
| Management events | Thao tác trên tài nguyên | "who terminated the instance" |
| Data events | Thao tác trên dữ liệu (S3 object) | "who read this object" |
| CloudTrail Insights | Phát hiện API bất thường | "unusual API activity" |
| Config rule | Đánh giá compliance cấu hình | "track configuration changes" |
| Config remediation | Tự sửa qua SSM Automation | "auto-fix non-compliant" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CloudWatch | CloudTrail | Hiệu năng vs lịch sử API | "Who did it" → CloudTrail |
| CloudTrail | Config | API call vs trạng thái cấu hình theo thời gian | "Compliance over time" → Config |
| Config | SCP/IAM | Config phát hiện; SCP/IAM ngăn chặn | "Prevent" → SCP |
| Subscription filter | Export to S3 | Real-time vs batch (tới 12 giờ) | Real-time → subscription |
| Metric filter | Logs Insights | Metric filter tạo metric liên tục; Insights truy vấn ad-hoc | Alarm trên log → metric filter |
| Management events | Data events | Management mặc định bật; data events tắt mặc định, khối lượng lớn | S3 object-level → data events |
| EventBridge | CloudWatch Alarm | EventBridge phản ứng theo sự kiện; Alarm theo ngưỡng metric | "When API X called" → EventBridge |
| Composite alarm | Alarm thường | Composite gộp trạng thái nhiều alarm | Giảm báo động giả → composite |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "monitor EC2 memory and disk" | Unified CloudWatch agent | |
| "stream logs real-time to OpenSearch/Kinesis" | Subscription filter | |
| "centralize logs from many accounts" | Subscription → Kinesis/Firehose ở account trung tâm | |
| "alarm when 'ERROR' appears > 10 times" | Metric filter + alarm | |
| "auto-recover EC2 on hardware failure, keep IP" | Alarm EC2 recover action | |
| "run Lambda every 15 minutes" | EventBridge schedule | |
| "notify when root user signs in" | EventBridge rule → SNS | |
| "who deleted the S3 bucket" | CloudTrail | |
| "who downloaded a specific S3 object" | CloudTrail data events | |
| "detect unusual API activity" | CloudTrail Insights | |
| "check if SGs allow unrestricted SSH, auto-remediate" | Config rule + SSM Automation | |
| "configuration history of a resource" | AWS Config | |
| "top IPs hitting the app" | Contributor Insights | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Team muốn ASG scale theo mức RAM sử dụng của EC2. Bước đầu tiên?
- A. Bật detailed monitoring
- B. Cài unified CloudWatch agent để đẩy metric RAM
- C. Bật CloudTrail
- D. Dùng AWS Config

<details><summary>Đáp án</summary>

**B.** RAM không có sẵn trong metric EC2.
- A sai: chỉ tăng tần suất metric có sẵn.
- C sai: ghi API call.
- D sai: theo dõi cấu hình.
</details>

**Câu 2.** Security cần biết ai đã terminate một EC2 production hôm qua. Dùng gì?
- A. CloudWatch Metrics
- B. CloudTrail
- C. AWS Config
- D. VPC Flow Logs

<details><summary>Đáp án</summary>

**B.**
- A sai: không có danh tính người gọi.
- C sai: biết cấu hình thay đổi, nhưng "ai gọi API" là CloudTrail.
- D sai: chỉ có luồng mạng.
</details>

**Câu 3.** (Chọn 2) Cần phát hiện SG mở port 22 ra 0.0.0.0/0 và tự động gỡ rule đó. Chọn gì?
- A. AWS Config managed rule
- B. Remediation bằng SSM Automation
- C. CloudWatch Logs Insights
- D. CloudTrail Insights
- E. Trusted Advisor

<details><summary>Đáp án</summary>

**A, B.**
- C sai: truy vấn log, không đánh giá cấu hình.
- D sai: phát hiện API bất thường về tần suất.
- E sai: có kiểm tra nhưng không tự remediation.
</details>

**Câu 4.** Ứng dụng cần xử lý log real-time và đưa vào OpenSearch. Chọn gì?
- A. Export log group sang S3 hằng ngày
- B. CloudWatch Logs subscription filter → Firehose → OpenSearch
- C. CloudTrail
- D. Config aggregator

<details><summary>Đáp án</summary>

**B.**
- A sai: không real-time.
- C, D sai: không xử lý log ứng dụng.
</details>

**Câu 5.** Muốn chạy một Lambda dọn dẹp mỗi ngày lúc 2 giờ sáng. Chọn gì?
- A. EventBridge schedule (cron)
- B. CloudWatch alarm
- C. SQS delay queue
- D. EC2 cron

<details><summary>Đáp án</summary>

**A.**
- B sai: kích hoạt theo ngưỡng metric.
- C sai: delay tối đa 15 phút.
- D sai: phải duy trì server.
</details>

**Câu 6.** Alarm CPU cao bắn liên tục khi chạy backup, dù hệ thống vẫn ổn. Muốn chỉ báo khi CPU cao **và** độ trễ cao. Chọn gì?
- A. Composite alarm
- B. Tăng period lên 1 ngày
- C. Xóa alarm
- D. Contributor Insights

<details><summary>Đáp án</summary>

**A.**
- B sai: mất khả năng phát hiện sớm.
- C sai: mất giám sát.
- D sai: phân tích top-N.
</details>

**Câu 7.** Kiểm toán yêu cầu ghi lại ai đọc từng object trong một bucket nhạy cảm. Chọn gì?
- A. CloudTrail management events
- B. CloudTrail data events cho bucket đó
- C. AWS Config
- D. CloudWatch Metrics S3

<details><summary>Đáp án</summary>

**B.** GetObject là data event, tắt mặc định.
- A sai: không ghi thao tác object.
- C sai: không ghi truy cập dữ liệu.
- D sai: chỉ số liệu tổng hợp.
</details>

**Câu 8.** EC2 bị lỗi system status check do phần cứng host; muốn tự khôi phục mà giữ private IP, EIP, metadata. Chọn gì?
- A. CloudWatch alarm với EC2 recover action
- B. ASG terminate và tạo instance mới
- C. Config remediation
- D. Reboot thủ công

<details><summary>Đáp án</summary>

**A.** Recover chuyển instance sang host mới, giữ IP và metadata.
- B sai: instance mới có IP khác.
- C sai: không xử lý lỗi phần cứng.
- D sai: thủ công và có thể không khắc phục lỗi host.
</details>

---

### Nguồn đã đối chiếu
- Previous CloudWatch Logs agent (deprecated, dùng unified agent): https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/UsePreviousCloudWatchLogsAgent.html
- CloudWatch Logs subscription filters: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Subscriptions.html
- Composite alarms: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Create_Composite_Alarm.html
- EventBridge archive & replay: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-archive.html
- CloudTrail data events: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html
- CloudTrail Lake availability change (31/05/2026): https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake-service-availability-change.html
- AWS Config remediation: https://docs.aws.amazon.com/config/latest/developerguide/remediation.html
- X-Ray SDK & Daemon support timeline (maintenance từ 25/02/2026): https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-daemon-timeline.html
