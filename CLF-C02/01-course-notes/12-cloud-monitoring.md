# CLF-C02 — Cloud Monitoring

> Thuộc **Domain 3** (management & governance) và **Domain 2** (audit, logging). Trọng tâm: CloudWatch vs CloudTrail vs EventBridge vs X-Ray vs Health Dashboard.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **CloudWatch Metrics** = số liệu hiệu năng của mọi dịch vụ (CPU, network...); EC2 mặc định **5 phút**, Detailed Monitoring **1 phút**; **không có RAM** mặc định.
- **CloudWatch Alarms** = kích hoạt hành động khi metric vượt ngưỡng: Auto Scaling, EC2 action (stop/terminate/reboot/recover), gửi **SNS**. Billing alarm dùng metric billing ở **us-east-1**.
- **CloudWatch Logs** = gom log từ Beanstalk, ECS, Lambda, CloudTrail, Route 53, EC2/on-prem (cần **CloudWatch agent**).
- **EventBridge** (tên cũ CloudWatch Events) = phản ứng với **event** hoặc chạy theo **lịch (cron)**; có event bus cho AWS, đối tác SaaS và app tự viết.
- **CloudTrail** = **lịch sử API call** trong account (ai làm gì, khi nào, từ đâu); **bật mặc định**; điều tra tài nguyên bị xóa thì xem CloudTrail đầu tiên.
- **X-Ray** = **tracing** request qua ứng dụng phân tán để tìm bottleneck, lỗi, phụ thuộc.
- **AWS Health Dashboard**: **Service History** (tình trạng chung mọi dịch vụ, mọi Region) và **Your Account** (sự kiện AWS **ảnh hưởng tới tài nguyên của bạn**).

---

## 2. Core Concepts

### CloudWatch Metrics
- **Là gì:** metric = biến cần theo dõi (CPUUtilization, NetworkIn...), có timestamp; mọi dịch vụ AWS đều có metric; tạo được **dashboard**.
- **Dùng khi nào:** EC2: CPU, status check, network (**không có RAM**); EBS: disk read/write; S3: BucketSizeBytes, NumberOfObjects; Billing: Total Estimated Charge (chỉ ở **us-east-1**); service limits; **custom metric** do bạn đẩy lên.
- **Vì sao quan trọng:** EC2 mặc định **5 phút**; Detailed Monitoring (tốn phí) **1 phút**.

### CloudWatch Alarms
- **Là gì:** kích hoạt thông báo hoặc hành động cho bất kỳ metric nào; chọn được period, kiểu thống kê (%, max, min...).
- **Dùng khi nào:** action: **Auto Scaling** (tăng/giảm desired count), **EC2 action** (stop, terminate, reboot, recover), **SNS notification**.
- **Vì sao quan trọng:** trạng thái alarm: **OK**, **INSUFFICIENT_DATA**, **ALARM**. Ví dụ quen thuộc: billing alarm.

### CloudWatch Logs
- **Là gì:** thu thập log từ Elastic Beanstalk, ECS (container), Lambda, CloudTrail (theo filter), CloudWatch agent trên EC2 hoặc on-prem, Route 53 (DNS query).
- **Dùng khi nào:** giám sát log real-time; điều chỉnh được thời gian lưu (retention).
- **Vì sao quan trọng:** EC2 **mặc định không gửi log** lên CloudWatch: phải chạy **CloudWatch agent** và cấp đúng IAM permission; agent cũng chạy được on-prem.

### Amazon EventBridge
- **Là gì:** tên cũ **CloudWatch Events**; **Schedule** (cron job) và **Event Pattern** (rule phản ứng khi dịch vụ làm gì đó, vd: root user đăng nhập → SNS email).
- **Dùng khi nào:** nguồn: EC2 start, CodeBuild lỗi, S3 upload, Trusted Advisor finding, CloudTrail API call, lịch; đích: Lambda, Batch, ECS task, SQS, SNS, Kinesis, Step Functions, CodePipeline, CodeBuild, SSM, EC2 action.
- **Vì sao quan trọng:** event bus: **Default** (dịch vụ AWS), **Partner** (SaaS đối tác), **Custom** (app của bạn); có **Schema Registry**, **archive** và **replay** event.

### AWS CloudTrail
- **Là gì:** cung cấp **governance, compliance, audit**: lịch sử event/API call trong account từ Console, SDK, CLI và dịch vụ AWS.
- **Dùng khi nào:** đưa log vào CloudWatch Logs hoặc S3; trail áp dụng **mọi Region** (mặc định) hoặc một Region.
- **Vì sao quan trọng:** **bật mặc định**. Tài nguyên bị xóa → **điều tra CloudTrail trước tiên**. `[+]` Event history trong console giữ **90 ngày** management event; muốn lâu hơn thì tạo trail đẩy ra S3.

### CloudTrail Insights
- **Là gì:** phân tích tự động CloudTrail event để phát hiện hoạt động API **bất thường**.
- **Dùng khi nào:** phát hiện đột biến số lần gọi API, lỗi, giới hạn tài nguyên.
- **Vì sao quan trọng:** từ khóa "unusual API activity" → CloudTrail Insights.

### AWS X-Ray
- **Là gì:** **phân tích trực quan** ứng dụng: trace request qua nhiều service phân tán.
- **Dùng khi nào:** tìm bottleneck hiệu năng, hiểu phụ thuộc giữa microservice, tìm lỗi/exception, kiểm tra SLA thời gian, biết chỗ bị throttle, xác định người dùng bị ảnh hưởng.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: instrument bằng X-Ray SDK/Daemon → từ **25/02/2026** X-Ray SDK và Daemon vào **maintenance mode** (chỉ vá bảo mật); AWS khuyến nghị dùng **OpenTelemetry** (ADOT, CloudWatch agent) để gửi trace về X-Ray. Dịch vụ X-Ray vẫn hoạt động.

### AWS Health Dashboard — Service History
- **Là gì:** hiển thị tình trạng **mọi dịch vụ, mọi Region**, có lịch sử theo ngày và RSS feed.
- **Dùng khi nào:** xem AWS có đang gặp sự cố chung hay không.
- **Vì sao quan trọng:** tên cũ: **AWS Service Health Dashboard**.

### AWS Health Dashboard — Your Account
- **Là gì:** tên cũ **AWS Personal Health Dashboard (PHD)**; cảnh báo và hướng dẫn khắc phục khi sự kiện AWS **ảnh hưởng tới tài nguyên của bạn**.
- **Dùng khi nào:** xem góc nhìn cá nhân hóa, thông báo chủ động về bảo trì theo lịch; tổng hợp được cho cả AWS Organization.
- **Vì sao quan trọng:** dịch vụ **global**. "Outage có ảnh hưởng tới **tài nguyên của tôi** không" → Your Account.

### `[+]` CloudWatch agent & custom metrics
- **Là gì:** agent cài trên EC2/on-prem để đẩy cả **log** và **metric hệ thống** như **memory**, disk space.
- **Dùng khi nào:** đề hỏi "theo dõi RAM của EC2".
- **Vì sao quan trọng:** RAM không có sẵn trong metric mặc định của EC2, phải dùng agent hoặc custom metric.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| CloudWatch Metric | Số liệu theo thời gian của dịch vụ | "CPU utilization", "monitor performance" |
| Detailed Monitoring | Metric EC2 mỗi 1 phút (tốn phí) | "1-minute granularity" |
| Custom metric | Metric tự đẩy lên CloudWatch | "memory usage of EC2" |
| CloudWatch Dashboard | Bảng hiển thị nhiều metric | "visualize metrics" |
| CloudWatch Alarm | Kích hoạt action khi metric vượt ngưỡng | "notify when CPU > 80%" |
| Billing alarm | Alarm trên metric chi phí (us-east-1) | "alert when bill exceeds $X" |
| CloudWatch Logs | Lưu và giám sát log | "centralize application logs" |
| CloudWatch agent | Agent đẩy log/metric từ EC2 hoặc on-prem | "collect logs from on-prem servers" |
| Amazon EventBridge | Event bus, rule và schedule | "react to events", "cron", "serverless scheduled task" |
| Schema Registry / Archive & Replay | Mô hình hóa schema; lưu và phát lại event | "replay past events" |
| AWS CloudTrail | Lịch sử API call trong account | "who deleted", "audit API calls" |
| Trail | Cấu hình đẩy CloudTrail log ra S3/CloudWatch Logs | "store logs longer than 90 days" |
| CloudTrail Insights | Phát hiện API activity bất thường | "unusual API activity" |
| AWS X-Ray | Tracing ứng dụng phân tán | "trace requests across microservices" |
| Health Dashboard – Service History | Tình trạng chung mọi dịch vụ | "status of all AWS services" |
| Health Dashboard – Your Account | Sự kiện ảnh hưởng tới tài nguyên của bạn | "events affecting my resources" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| CloudWatch | CloudTrail | **Hiệu năng** (metric, log, alarm) vs **ai đã gọi API gì** (audit) | "Who did what" → CloudTrail |
| CloudTrail | AWS Config | Lịch sử **API call** vs lịch sử **cấu hình tài nguyên** và compliance | "Configuration changed over time" → Config (chương 14) |
| CloudWatch Logs | CloudTrail | Log ứng dụng/hệ thống vs log API của account | |
| CloudWatch Alarms | EventBridge | Phản ứng theo **ngưỡng metric** vs phản ứng theo **event** hoặc lịch | "When root user signs in" → EventBridge |
| EventBridge | SNS | Event bus có rule, schedule, nhiều nguồn vs pub/sub đơn giản | "Run every hour" → EventBridge |
| X-Ray | CloudWatch | Trace một request qua nhiều service vs metric/log tổng quát | "Distributed tracing" → X-Ray |
| Service History | Your Account | Tình trạng chung vs ảnh hưởng tới tài nguyên của bạn | "My resources" → Your Account |
| CloudTrail | CloudTrail Insights | Ghi event vs phát hiện bất thường | "Anomalies in API calls" → Insights |
| Basic monitoring | Detailed monitoring | 5 phút miễn phí vs 1 phút tốn phí | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "who terminated the EC2 instance" | AWS CloudTrail | |
| "alert when estimated charges exceed $100" | CloudWatch Billing Alarm (hoặc AWS Budgets) | Metric ở us-east-1 |
| "automatically recover/reboot EC2 on failed status check" | CloudWatch Alarm + EC2 action | |
| "collect logs from EC2 and on-prem servers" | CloudWatch agent → CloudWatch Logs | |
| "monitor EC2 memory" | CloudWatch agent / custom metric | |
| "run Lambda every 4 hours" | EventBridge schedule | |
| "notify when root user signs in" | EventBridge rule → SNS | |
| "trace latency across microservices" | AWS X-Ray | |
| "is AWS having an outage in a Region" | Health Dashboard – Service History | |
| "scheduled maintenance affecting my instances" | Health Dashboard – Your Account | |
| "detect unusual API call volume" | CloudTrail Insights | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một S3 bucket quan trọng bị xóa. Cần biết ai xóa và khi nào. Dùng dịch vụ nào?
- A. Amazon CloudWatch Metrics
- B. AWS CloudTrail
- C. AWS X-Ray
- D. AWS Health Dashboard

<details><summary>Đáp án</summary>

**B.** CloudTrail ghi lại mọi API call, gồm người thực hiện và thời điểm.
- A sai: số liệu hiệu năng.
- C sai: trace request ứng dụng.
- D sai: tình trạng dịch vụ AWS.
</details>

**Câu 2.** Team muốn nhận email khi chi phí ước tính của tháng vượt $500. (Chọn 2) Những cách nào phù hợp?
- A. CloudWatch Billing Alarm + SNS
- B. AWS Budgets
- C. AWS X-Ray
- D. CloudTrail Insights
- E. Amazon Inspector

<details><summary>Đáp án</summary>

**A, B.** Billing alarm trên metric chi phí, hoặc AWS Budgets (mạnh hơn, chương 16).
- C, D, E sai: không theo dõi chi phí.
</details>

**Câu 3.** Admin muốn theo dõi mức sử dụng **RAM** của EC2. Cách đúng?
- A. Bật Detailed Monitoring
- B. Cài CloudWatch agent để đẩy memory metric
- C. Dùng CloudTrail
- D. Dùng AWS Health Dashboard

<details><summary>Đáp án</summary>

**B.** RAM không có trong metric mặc định; cần agent hoặc custom metric.
- A sai: chỉ tăng tần suất metric có sẵn.
- C sai: audit API.
- D sai: tình trạng dịch vụ.
</details>

**Câu 4.** Cần chạy một Lambda dọn dẹp tài nguyên mỗi đêm lúc 2 giờ sáng, không dùng server. Chọn gì?
- A. Amazon EventBridge scheduled rule
- B. CloudWatch Alarm
- C. AWS CloudTrail
- D. Amazon SQS

<details><summary>Đáp án</summary>

**A.** EventBridge có schedule dạng cron để kích hoạt Lambda.
- B sai: kích hoạt theo ngưỡng metric.
- C sai: ghi log API.
- D sai: queue.
</details>

**Câu 5.** Ứng dụng microservices chậm và team cần biết service nào gây độ trễ trong chuỗi xử lý request. Chọn gì?
- A. AWS X-Ray
- B. AWS CloudTrail
- C. AWS Config
- D. Amazon Inspector

<details><summary>Đáp án</summary>

**A.** X-Ray trace request qua các service, chỉ ra bottleneck.
- B sai: audit API.
- C sai: lịch sử cấu hình.
- D sai: quét lỗ hổng.
</details>

**Câu 6.** Công ty muốn biết sự cố AWS đang diễn ra có ảnh hưởng tới EC2 của **họ** không và nhận hướng dẫn khắc phục. Chọn gì?
- A. AWS Health Dashboard – Service History
- B. AWS Health Dashboard – Your Account
- C. Amazon CloudWatch Logs
- D. AWS Trusted Advisor

<details><summary>Đáp án</summary>

**B.** "Your Account" cá nhân hóa theo tài nguyên của bạn, kèm hướng dẫn.
- A sai: chỉ tình trạng chung.
- C sai: log.
- D sai: khuyến nghị best practice.
</details>

**Câu 7.** (Chọn 2) CloudWatch Alarm có thể thực hiện action nào?
- A. Gửi thông báo tới SNS topic
- B. Thay đổi desired capacity của Auto Scaling group
- C. Tạo IAM user mới
- D. Mã hóa S3 bucket
- E. Đổi Region của EC2

<details><summary>Đáp án</summary>

**A, B.** (Ngoài ra còn EC2 action: stop, terminate, reboot, recover.)
- C, D, E sai: không phải action của alarm.
</details>

**Câu 8.** Phát biểu nào về CloudTrail là **đúng**?
- A. Phải bật thủ công mới có lịch sử API
- B. Bật mặc định, ghi lại API call từ Console, SDK, CLI và dịch vụ AWS
- C. Chỉ ghi lại API call từ Console
- D. Đo CPU của EC2

<details><summary>Đáp án</summary>

**B.**
- A sai: CloudTrail bật mặc định (event history); trail để lưu lâu thì tự tạo.
- C sai: ghi từ mọi nguồn.
- D sai: đó là CloudWatch.
</details>

---

### Nguồn đã đối chiếu
- X-Ray SDK and Daemon support timeline (maintenance mode từ 25/02/2026): https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-daemon-timeline.html
- Migrating from X-Ray instrumentation to OpenTelemetry: https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-migration.html
- CloudTrail event history (90 ngày): https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html
- Amazon EventBridge: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html
- CloudWatch agent: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html
- AWS Health Dashboard: https://docs.aws.amazon.com/health/latest/ug/what-is-aws-health.html
