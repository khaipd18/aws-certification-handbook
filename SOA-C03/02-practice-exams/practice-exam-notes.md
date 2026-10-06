# AWS SOA-C03 — Sổ tay cốt lõi & kiến thức dễ nhầm

> Tổng hợp từ **520 câu hỏi (8 đề thi thử)**, gộp theo **nhóm dịch vụ**, không tách theo đề.
> **🎯 = ý tưởng cốt lõi cần khắc sâu** · **⚠️ = kiến thức liên quan / dễ nhầm trong đề** · **📌 = cập nhật 10/2026** (đề còn dùng tên/số liệu cũ: khi thi cứ chọn theo logic đề, nhưng biết thực tế đã đổi) · **❓ = đáp án đề đáng ngờ hoặc chưa kiểm chứng được**.
> Xem **§40 (Phụ lục)** để tra nhanh dịch vụ hay xuất hiện ở đáp án SAI và các đáp án **không tồn tại**.
>
> **Ôn cấp tốc trước khi thi:** §41 (Nguyên tắc làm bài) → §42 (Bảng so sánh dễ nhầm) → §43 (Từ khóa → đáp án) → §44 (Con số & kết hợp bất khả thi).
>
> 5 domain SOA-C03 (AWS Certified CloudOps Engineer – Associate): **Monitoring, Logging, Analysis, Remediation & Performance Optimization 22% · Reliability & Business Continuity 22% · Deployment, Provisioning & Automation 22% · Security & Compliance 16% · Networking & Content Delivery 18%**.
>
> **Chủ đề xuất hiện ở nhiều đề nhất (học trước):** RDS Multi-AZ vs Read Replica (§17) · SG vs NACL + ephemeral port + Flow Logs (§35) · Auto Scaling (§14) · ALB (§15) · Systems Manager (§11) · CloudFormation (§12) · Config + remediation (§10) · CloudWatch Agent/metric/alarm (§1–§4) · CloudTrail (§8) · S3 versioning/Object Lock/mã hóa (§23–§24) · EBS gp2/gp3 (§25) · Storage Gateway (§27) · VPC endpoint & Direct Connect VIF (§34, §36) · CloudFront (§37) · KMS (§32) · Trusted Advisor & Cost (§33, §39).
>
> **Khi có đề mới:** chèn kiến thức vào đúng mục dịch vụ (§1–§39), cập nhật §40–§44. Không tạo phần riêng cho từng đề.

---

## 1. CloudWatch Agent

🎯 **EC2 không có log mặc định trong CloudWatch Logs.** Log ghi trên đĩa local chỉ lên được CloudWatch Logs qua **CloudWatch Agent**. Agent **không tự cài** từ Console: cài bằng yum, CLI, **SSM Run Command** hoặc **SSM Distributor**. ASG **không có checkbox cài agent**; muốn instance của ASG có sẵn agent thì **nướng agent vào AMI** (AMI Amazon Linux gốc không có sẵn).

🎯 Quyền: trên **EC2 dùng IAM Role** gắn managed policy **`CloudWatchAgentServerPolicy`**; **IAM User + access key chỉ dùng cho server on-premises**. Security Group không liên quan.

🎯 **Agent chạy, config đúng nhưng không có log** → nguyên nhân số 1 là **IAM role thiếu quyền Logs**: `logs:CreateLogGroup` · `logs:CreateLogStream` · `logs:PutLogEvents` · `logs:DescribeLogStreams`. Thứ tự troubleshoot: **IAM → config file → network/endpoint → trạng thái agent**.

⚠️ Không phải nguyên nhân: retention period (chỉ quyết định khi nào xóa log), thiếu cron gọi `PutLogEvents` (agent tự làm), thiếu Kinesis trung gian.

🎯 **Nhiều file config:** khởi động bằng `-a fetch-config -c file://config1.json`, nối thêm bằng `-a append-config -c file://config2.json`. ⚠️ Các file config trên cùng server **bắt buộc khác TÊN FILE**: `append-config` với file trùng tên sẽ **GHI ĐÈ**, kể cả khi hai file nằm ở path khác nhau.

🎯 **Giám sát disk EBS** (3 bước): IAM role có `CloudWatchAgentServerPolicy` → cài agent (SSM hoặc command line) → alarm trên **`disk_used_percent` > 80%**.

| Metric của agent | Đơn vị |
| --- | --- |
| **`disk_used_percent`** | Phần trăm → dùng cho ngưỡng % |
| `disk_used` / `disk_free` | **Bytes** → không so được với "80%" |
| `mem_used_percent` | Phần trăm RAM |

⚠️ `CloudWatchApplicationInsightsReadOnlyAccess` là policy read-only cho Application Insights, **không phải** quyền cho agent.

🎯 **Plugin của agent:**

| Plugin | Dùng để |
| --- | --- |
| **`procstat`** | Giám sát **từng process** (CPU, memory, process còn sống không) → mẫu "process chết thì tự restart" |
| **StatsD** | Thu custom metric **từ code ứng dụng** (số request, response time) |
| **collectd** | Thu custom metric qua giao thức collectd |

🎯 Mẫu **"process dừng → tự restart"**: `procstat` sinh metric → **CloudWatch Alarm** → **EventBridge** (alarm state change) → **SSM Automation runbook** restart. ⚠️ **EventBridge không nhận trực tiếp giá trị metric**, phải đi qua Alarm.

🎯 **Chuẩn hóa cấu hình agent cho cả fleet** (2 bước): lưu **nội dung file config vào SSM Parameter Store** → tạo **State Manager association** chạy document **`AmazonCloudWatch-ManageAgent`**, chọn Parameter Store làm configuration source, **target theo tag** (vd `OS=Windows`).

⚠️ `AmazonCloudWatch-ManageAgent` đọc config từ **Parameter Store hoặc file local**, **không đọc trực tiếp từ S3**. SSM OpsCenter quản lý OpsItem/sự cố, không lưu file cấu hình.

---

## 2. CloudWatch Metrics

### Metric EC2 & độ phân giải

| Metric | Ý nghĩa | Dùng khi |
| --- | --- | --- |
| `CPUUtilization` | % compute unit đang dùng | Processing power, alarm reboot/stop |
| `NetworkIn` / `NetworkOut` | **Bytes** vào/ra trên mọi NIC | Tổng băng thông, tìm instance tốn bandwidth |
| `DiskReadBytes` / `DiskWriteBytes` | Bytes đọc/ghi **instance store**, gộp cho cả instance | Tốc độ I/O local |
| `CPUCreditUsage` / `CPUCreditBalance` | Credit của họ T | Burst credit |
| **`StatusCheckFailed`** | 0/1, fail **system HOẶC instance** | Alarm chung |
| `StatusCheckFailed_Instance` | OS/network của instance | Reboot |
| `StatusCheckFailed_System` | Hạ tầng AWS | **Recover** |

| Chế độ | Chu kỳ | Ghi chú |
| --- | --- | --- |
| **Basic monitoring** | **5 phút** | Mặc định, miễn phí |
| **Detailed monitoring** | **1 phút** | Có phí; bật lúc launch/running/stopped; không ảnh hưởng metric EBS |
| **High-resolution custom metric** | **1 giây** | Đọc ở period 1/5/10/30 giây; **high-resolution alarm tối thiểu 10 giây** |

🎯 Cần **1 phút** cho metric EC2 có sẵn → **Detailed Monitoring**. Cần **10 giây** → **high-resolution custom metric** (đẩy bằng script/API `put-metric-data`); Detailed Monitoring không đủ nhanh.

🎯 **RAM/memory và disk space KHÔNG phải metric mặc định của EC2** → push custom metric qua CloudWatch Agent hoặc `aws cloudwatch put-metric-data --namespace ... --metric-name ... --dimensions InstanceId=...`.

⚠️ Status check metric mặc định 1 phút, miễn phí. **EC2 status checks không disable/delete được.** Bytes/giây = Sum ÷ 300 (basic) hoặc ÷ 60 (detailed).

⚠️ **Metric bịa:** `NetworkTotalBytes`, `CPUProcessPower`, `CombinedStatusCheckFailed`, `RAMUtilization`, `InstanceConnections` (EFS). `DataTransfer-Out-Bytes` là của Billing/Cost Explorer, không phải EC2.

### EC2 Disk metric vs EBS Volume metric

| Metric | Namespace | Phạm vi |
| --- | --- | --- |
| `DiskReadBytes` / `DiskWriteBytes` / `DiskReadOps` | **`AWS/EC2`** | Gộp cho cả instance (instance store), dimension `InstanceId` |
| **`VolumeReadBytes` / `VolumeWriteBytes` / `VolumeReadOps`** | **`AWS/EBS`** | **Từng EBS volume**, dimension **`VolumeId`** |

🎯 Alarm cho **một EBS volume cụ thể** → dùng `Volume*` với dimension `VolumeId`. Cả hai đều là **default metric, không cần agent** (agent chỉ cần cho disk space/memory).

### Metric ELB

| Metric | Ý nghĩa |
| --- | --- |
| **`ActiveConnectionCount`** (ALB) | Tổng kết nối TCP đồng thời → scale theo số người dùng |
| **`SurgeQueueLength`** (CLB) | Request **đang xếp hàng** vì backend chậm (tối đa 1,024) |
| **`SpilloverCount`** (CLB) | Request **bị từ chối** vì surge queue đầy |
| `BackendConnectionErrors` | Kết nối LB → target thất bại (gồm health check) |
| **`HTTPCode_ELB_4XX_Count`** | 4XX **do chính load balancer sinh** (request lỗi, chưa tới target) |
| `HTTPCode_Target_4XX_Count` | 4XX **do target trả về** |
| `ClientTLSNegotiationErrorCount` | Client không lập được TLS session (lệch cipher/protocol) |
| `NonStickyRequestCount` | Request phải chọn target mới vì không dùng được sticky session |
| **`HealthyHostCount`** | Số target healthy |
| `UnhealthyHostCount` | Số target unhealthy |

🎯 **Alarm khi TẤT CẢ target unhealthy** → **`HealthyHostCount <= 0`** (namespace `AWS/ApplicationELB`). ⚠️ `UnhealthyHostCount >= 1` chỉ nghĩa là **có ít nhất một** target hỏng; `AWS/EC2 StatusCheckFailed` là sức khỏe hạ tầng, instance pass status check vẫn có thể fail health check của ALB.

🎯 `SpilloverCount > 0` → **scale ASG theo `SurgeQueueLength`**. ELB chỉ đẩy metric khi có request đi qua, chu kỳ 60 giây.

⚠️ **Pre-warming ELB** (liên hệ AWS Support) chỉ dành cho flash traffic/load test tăng đột ngột, **không cứu được backend chậm**.

### Metric của các dịch vụ khác

🎯 **Amazon SQS:** dimension **duy nhất là `QueueName`**. Metric: `ApproximateNumberOfMessagesVisible`, `ApproximateAgeOfOldestMessage`, `ApproximateNumberOfMessagesDelayed`, `NumberOfMessagesDeleted`.

🎯 **Amazon S3 request metrics** có metric **5xx** → theo dõi tần suất **`500 Internal Error`**; xử lý 500 bằng **retry + exponential backoff** (SDK có sẵn). ⚠️ **403 = Access Denied** (4xx), không phải lỗi server.

| Metric | Ý nghĩa |
| --- | --- |
| **`ClientConnections`** (EFS) | **Số client/EC2 đang kết nối** file system; dùng statistic **Sum** (chu kỳ > 1 phút thì chia Sum cho số phút) |
| `TotalIOBytes` (EFS) | Throughput, xem Sum theo ngày |
| `BurstCreditBalance` (EFS) | Burst credit còn lại |
| **`AuroraReplicaLagMaximum`** | Lag **lớn nhất** giữa primary và từng instance trong cluster |
| `AuroraReplicaLag` | Lag của **một** Aurora Replica |
| `AuroraBinlogReplicaLag` | Lag của **replica cluster** (Aurora MySQL, binlog, thường cross-Region) |
| **`FreeStorageCapacity`** (FSx/RDS) | Dung lượng trống **tính bằng BYTES** → "dưới 100 GB" là ngưỡng `<= 100000000000`; alarm → **SNS trực tiếp** |
| `FreeLocalStorage` (Aurora) | Temp storage của instance (xem §18) |

---

## 3. CloudWatch Alarms

🎯 **Alarm action hợp lệ:** **SNS topic** · **EC2 action** (stop/terminate/reboot/recover) · **Auto Scaling action** · **SSM OpsItem**. 📌 Thực tế hiện hành còn có: **gọi trực tiếp Lambda function**, tạo **incident trong SSM Incident Manager**, và khởi chạy **CloudWatch investigations**. Đề cũ coi "chỉ 4 loại": đọc kỹ đáp án.

⚠️ **SQS KHÔNG phải target của CloudWatch Alarm.** **SNS không trực tiếp khởi chạy SSM Automation** → phải qua **EventBridge** (bắt alarm state change) hoặc Lambda.

🎯 **Alarm → tạo ticket ở hệ thống ngoài:** CloudWatch Alarm → **SNS topic** → subscribe **HTTP/HTTPS endpoint** của hệ thống ticket. Không cần Lambda trung gian.

🎯 **Hai mẫu EC2 action ra thi nhiều nhất:**

| Bài toán | Cấu hình |
| --- | --- |
| App treo khi CPU cao, cần restart | Alarm `CPUUtilization > 80%` → **Reboot instance action** (cần phản ứng nhanh thì bật Detailed Monitoring) |
| Instance dev bị quên, cần stop để tiết kiệm | Alarm `CPUUtilization < 5%` trong 30 phút → **Stop instance action** |

⚠️ Không cần SSM runbook hay Lambda khi alarm đã có action gốc. CloudTrail chỉ ghi API call (không phản ánh độ bận); `VolumeReadBytes` không phản ánh độ nhàn rỗi; AWS Config theo dõi cấu hình, không theo dõi mức sử dụng.

🎯 **CPUUtilization là METRIC → dùng Alarm**, không phải event → EventBridge.

🎯 **Composite Alarm:** chỉ báo khi **nhiều điều kiện cùng đúng** (vd disk space vượt ngưỡng **VÀ** `DiskReadOps` vượt ngưỡng) → tạo **hai metric alarm** → **composite alarm với rule AND** → composite alarm gửi SNS. ⚠️ Để mỗi alarm tự gửi SNS thì nhận hai thông báo rời và báo cả khi chỉ một điều kiện đúng.

⚠️ Metric disk space cần **CloudWatch Agent**; Detailed Monitoring **không tạo ra** metric disk space.

⚠️ Alarm vẫn đánh giá metric sau khi đã kích hoạt: metric còn vượt ngưỡng thì alarm **ở yên trạng thái ALARM** (bình thường). Muốn khác thì chỉnh threshold.

⚠️ **CloudWatch Billing Alarm** chỉ theo estimated charges **cấp account**, không lọc theo dòng chi phí (lọc theo service/tag → AWS Budgets, §39).

---

## 4. CloudWatch Logs & Dashboards

| Cơ chế | Làm gì | Giới hạn |
| --- | --- | --- |
| **Metric Filter** | Match pattern trong log → sinh **CloudWatch METRIC** (đếm lỗi, trích số như latency) → nền cho alarm, dashboard, percentile | Chỉ pattern matching |
| **Logs Insights** | Truy vấn ad-hoc, tính được percentile trong query | Kết quả chỉ tồn tại lúc query, **không sinh metric bền vững** |
| **Subscription Filter** | Stream log real-time sang Kinesis Data Streams / Firehose / Lambda | Cần dịch vụ khác xử lý |
| **Contributor Insights** | Top contributor (user, IP, resource) | **Không tính percentile** |

🎯 **Theo dõi p90/percentile của một field trong log** → **Metric Filter** trích field thành metric, rồi thống kê p90. Percentile chỉ tính trên **metric**, không tính trực tiếp trên raw log.

🎯 Log **đã nằm trong CloudWatch Logs** (vd VPC Flow Logs của NAT Gateway) và cần **top N** (top 5 IP đích) → **Logs Insights query tại chỗ**. ⚠️ Chuyển log sang S3 rồi dùng Athena cũng được nhưng thêm bước; CloudTrail Insights chỉ phát hiện bất thường trong API call.

🎯 **Mẫu "lỗi trong log → tự khắc phục"**: CloudWatch Agent đẩy log → **Metric Filter** + **Alarm** → **EventBridge rule** bắt alarm state change → **SSM Automation runbook** restart dịch vụ. ⚠️ ALB health check không đọc log local; Logs Insights + scheduled query thêm độ trễ.

🎯 **Dashboard đa Region:** một dashboard hiển thị được resource ở **nhiều Region**, thêm widget theo từng Region bằng region selector. ⚠️ Không có checkbox "global metric" hay widget "multi-region graph".

🎯 **Dashboard tự gồm instance mới của ASG** → **CloudWatch Metrics Explorer**, lọc theo tag **`aws:autoscaling:groupName`** (ASG tự gắn tag này cho mọi instance) → thêm visualization vào dashboard. ⚠️ Lọc theo instance state lấy cả instance ngoài ASG; EventBridge + Lambda cập nhật dashboard là tốn công.

🎯 Dashboard trong CloudFormation: **`AWS::CloudWatch::Dashboard`**, export định nghĩa ra JSON và đặt vào **`DashboardBody`**. ⚠️ Lệnh CLI đúng là `aws cloudwatch put-dashboard` (không có `aws cloudformation put-dashboard`).

---

## 5. Synthetics, RUM, X-Ray, Container Insights & DevOps Guru

🎯 **CloudWatch Synthetics (canary)** = script (Node.js/Puppeteer hoặc Python/Selenium) chạy **theo lịch** (tối thiểu 1 phút), tạo Lambda trong account; đo **availability + latency**, lưu screenshot, HAR file, log.

| Blueprint | Làm gì | Dùng khi |
| --- | --- | --- |
| **Heartbeat monitor** | Tải **toàn bộ trang** như user thật (static + API), lưu screenshot/HAR | Uptime end-to-end, phát hiện lỗi tích hợp frontend↔backend (CORS, đổi endpoint); alarm trên **`SuccessPercent`** |
| API canary | Chỉ test API endpoint | Không bao phủ luồng người dùng |
| Broken link checker | Tìm link hỏng | Không dùng đo uptime |
| GUI workflow / Visual monitoring | Mô phỏng thao tác nhiều bước, so ảnh | Kiểm thử luồng nghiệp vụ |

🎯 **Synthetics vs RUM:** Synthetics **chủ động** (tự truy cập theo lịch) → "phát hiện sự cố **TRƯỚC** khi user báo"; **RUM thụ động** (dữ liệu từ session user thật) → đo trải nghiệm thật, không có dữ liệu khi chưa có user.

⚠️ **ServiceLens** gộp traces + metrics + logs + alarms đã có, không tạo metric từ script.

🎯 **X-Ray** tích hợp API Gateway, Lambda, EC2, ECS, Beanstalk. ⚠️ **ALB chỉ thêm header `X-Amzn-Trace-Id`**, **không gửi data tới X-Ray**, không xuất hiện trên service map.

📌 X-Ray SDK và Daemon vào **maintenance mode từ 25/02/2026**; AWS khuyên instrument bằng **OpenTelemetry** (ADOT) và gửi trace qua CloudWatch agent/OTel collector.

| Dịch vụ | Vai trò |
| --- | --- |
| **Container Insights** | Thu metric container (CPU, memory, network, disk) của EKS/ECS ở cấp cluster, node, pod, service. Chỉ thu và hiển thị |
| **Amazon DevOps Guru** | ML phát hiện bất thường **và đưa KHUYẾN NGHỊ** khắc phục |
| **CloudWatch Anomaly Detection** | Ngưỡng động bằng ML cho alarm, **không đưa khuyến nghị** |

🎯 Đề có chữ **"recommendations"** → **DevOps Guru**. DevOps Guru giám sát **EKS** thì **phải bật Container Insights** trước.

📌 **DevOps Guru không nhận khách mới từ 29/10/2026** và **ngừng hỗ trợ 30/09/2027**; AWS gợi ý chuyển sang tính năng của CloudWatch hoặc Amazon DevOps Agent.

---

## 6. Amazon EventBridge

🎯 Mặc định cho **"event-driven, near real-time, minimal maintenance"**: EC2 phát **instance state-change event** → EventBridge rule → **SNS topic**.

🎯 EventBridge bắt được **CloudWatch alarm state change** → khởi chạy **SSM Automation**, Lambda, SNS. Mọi API call ghi trong CloudTrail cũng thành event (vd SG thay đổi → `AWS-CreateServiceNowIncident`).

🎯 **Scheduled rule** (rate/cron) → vd **tạo EBS snapshot định kỳ** (snapshot incremental nhưng mỗi cái restore được full volume). ⚠️ Lambda **không tự gọi chính nó theo lịch**, luôn cần EventBridge.

🎯 **Gọi HTTPS endpoint bên ngoài** (ServiceNow, Jira) với **JSON format riêng**, ít code nhất → EventBridge rule → target **API Destinations** (EventBridge lo HTTPS call, authentication, retry) + **Input Transformer** để đổi định dạng. ⚠️ SNS **không transform được payload**; Kinesis/Step Functions + Glue là quá phức tạp (Glue là ETL big data).

| Target | Cơ chế cấp quyền |
| --- | --- |
| **Lambda / SNS / SQS / CloudWatch Logs** | **Resource-based policy** |
| **Kinesis Data Streams, Step Functions, ECS task...** | **IAM Role** |

⚠️ Gắn IAM Role cho rule có target Lambda → role bị bỏ qua; Lambda **không hề được gọi** → thiếu **resource-based policy** cho `events.amazonaws.com` với `lambda:InvokeFunction` (không phải thiếu execution role).

⚠️ ❓ Đề nói "EventBridge **không gọi trực tiếp** được `TerminateInstances`, phải qua Lambda/SSM". 📌 Thực tế EventBridge có **built-in target** (tạo qua Console) cho EC2: `RebootInstances`, `StopInstances`, `TerminateInstances`, `CreateSnapshot`. Khi đề có đáp án Lambda/SSM Automation thì vẫn hợp lệ; chỉ đừng coi "built-in target EC2" là đáp án bịa.

⚠️ **AWS Config không phù hợp cho cảnh báo real-time.** Trusted Advisor + EventBridge: TA phát event khi check đổi trạng thái (vd dùng > 80% service limit).

---

## 7. AWS Health

| Dịch vụ | Phạm vi |
| --- | --- |
| **Service health** (trước đây Service Health Dashboard, `status.aws.amazon.com`) | Trạng thái **chung** của mọi dịch vụ/Region, công khai |
| **AWS Health Dashboard – Your account health** (trước đây **Personal Health Dashboard**) | Event ảnh hưởng **resource của bạn**, kèm hướng dẫn khắc phục, báo trước lịch bảo trì |
| **AWS Health Organizational View** | Gộp Health event **của mọi member account** về management/delegated admin account |

📌 "Personal Health Dashboard" và "Service Health Dashboard" nay gộp thành **AWS Health Dashboard** (hai phần: service health và account health). Đề cũ còn dùng tên PHD.

🎯 Mặc định Health event chỉ thuộc **từng account** → EventBridge rule ở payer account **không nhận** event của member → **bật Organizational View** qua AWS Organizations. ⚠️ CloudWatch cross-account observability chỉ gộp metrics/logs/traces, **không gộp Health event**.

🎯 **AWS Health API** = truy cập bằng chương trình chính dữ liệu của Health Dashboard → tích hợp vào công cụ nội bộ. Gọi từ account không đủ gói Support → lỗi **`SubscriptionRequiredException`**.

📌 Đề ghi "cần gói **Business hoặc Enterprise**". Hiện hành: cần **Business Support+, Enterprise Support hoặc Unified Operations** (khách chưa chuyển gói vẫn dùng được với Business, Enterprise On-Ramp, Enterprise). Gói Developer ngừng từ **01/01/2027**.

⚠️ Không chạy diagnostics trực tiếp bên trong Health Dashboard (chỉ gắn Lambda khi event được route). Trusted Advisor notification chỉ là email hằng tuần.

---

## 8. AWS CloudTrail

🎯 **Quy tắc ngón tay cái:** hiệu năng, event, alarm → **CloudWatch** · ai gọi API gì (audit theo **account**) → **CloudTrail** · lịch sử cấu hình, compliance của **resource** → **AWS Config**.

🎯 **Organization trail:** tạo ở **management account**, tự áp cho mọi member account. Member account **thấy** trail nhưng **không sửa/xóa được**, mặc định **không có quyền** đọc file log trong bucket.

🎯 Mặc định CloudTrail chỉ ghi **management event (bucket-level)**; object-level phải **bật S3 Data Events** (có phí). Management event xem được 90 ngày trong Event history.

🎯 File log CloudTrail **mặc định đã mã hóa SSE-S3**; có thể chuyển sang **SSE-KMS**, nhưng ⚠️ **digest file luôn mã hóa SSE-S3**.

🎯 **Log File Integrity Validation:** CloudTrail hash mỗi log file, **mỗi giờ** tạo **digest file** (tham chiếu log file của giờ trước + chữ ký digest trước đó), ký bằng private key (mỗi Region một cặp khóa), thuật toán **SHA-256** (hash) và **SHA-256 with RSA** (ký) → không thể sửa/xóa/giả mạo log mà không bị phát hiện. Digest nằm **cùng bucket nhưng thư mục riêng**.

🎯 **Bảo vệ log CloudTrail ít công nhất** → **Log File Integrity Validation + MFA Delete** trên bucket log. ⚠️ Versioning vẫn cho sửa mà không cảnh báo; CloudWatch alarm khi log đổi tốn công; dùng root user vi phạm best practice.

🎯 **Trail đa Region:** Console mặc định tạo **all-Regions trail** (trail một Region phải dùng CLI). CloudTrail giao log **mọi Region về MỘT bucket S3** (và một log group, một SNS topic); bucket không bắt buộc ở home Region.

🎯 **Sinh IAM policy least-privilege** → bật CloudTrail + **IAM Access Analyzer policy generation** (phân tích tới 90 ngày log). Xem §30.

📌 **CloudTrail Lake** không nhận khách mới từ **31/05/2026** (khách cũ vẫn dùng). Phân tích log dài hạn cho khách mới: trail → S3 → Athena.

---

## 9. Phân tích log truy cập bằng Athena

🎯 **Amazon Athena** = SQL serverless trực tiếp trên S3, trả theo query → công cụ phân tích log mặc định trong đề.

| Cần phân tích | Bật log | Query bằng |
| --- | --- | --- |
| Ai truy cập / bị từ chối **S3 object** | **S3 Server Access Logging** | Athena |
| Lỗi 4xx/5xx **qua CloudFront** | **CloudFront standard (access) logs** | Athena |
| **Latency + IP client** ở tầng LB, hoặc lỗi khi instance đã terminate | **ELB Access Logs** | Athena |
| **Ai gọi API nào** (KMS, ELB, IAM, S3 bucket-level) | **CloudTrail** (trail → S3) | Athena |
| Traffic IP-level trong VPC | **VPC Flow Logs** | Athena / Logs Insights |

🎯 **S3 Server Access Logging:** miễn phí bật, chỉ trả phí lưu log; ghi requester, bucket, thời gian, action, HTTP status, error code. Bucket đích **cùng Region**, không có default retention.

🎯 **ELB Access Logs:** **mặc định TẮT**; chứa thời gian, **IP client**, **latency**, request path, response; tự mã hóa SSE-S3. Instance đã terminate → không dùng EC2Rescue, AWS Support không khôi phục, chỉ ELB log còn dấu vết.

⚠️ CloudFront log giao theo **best-effort** (có thể trễ/thiếu) → không dùng đối soát billing. Website qua CloudFront → S3 access log **không đủ** (mất IP thật, phần lớn request bị cache).

🎯 **Bytes theo TỪNG target của ALB** → bật **ALB Access Logs** → Athena tính `received_bytes + sent_bytes`, **`GROUP BY target:port`**. ⚠️ `client:port` là cổng ngẫu nhiên của user; metric `ProcessedBytes` chỉ là tổng của cả LB; agent trên instance không biết ALB gửi bao nhiêu.

🎯 **Điều tra hàng triệu request lạ trên S3** (vd LIST) → **CloudTrail Data Events** (source IP, identity, thời gian; có phí) hoặc **S3 Server Access Logging** (rẻ, hợp lưu lâu dài). ⚠️ EventBridge, Health Dashboard, Trusted Advisor không cho chi tiết nguồn request.

| Công cụ ELB | Cho biết |
| --- | --- |
| **ELB Access Logs** | Request của user: thời gian, IP client, latency, path, response |
| **CloudTrail** | API call tới ELB (ai tạo/sửa LB) |
| **CloudWatch metrics** | Số liệu thời gian thực (RequestCount, TargetResponseTime...) |
| **Request tracing** | Header `X-Amzn-Trace-Id`, không có dữ liệu latency |

---

## 10. AWS Config — Compliance & Auto-Remediation

🎯 Config ghi **cấu hình resource + quan hệ giữa resource**, đánh giá bằng **Config Rules** (COMPLIANT/NON_COMPLIANT). **Auto Remediation**: gắn **SSM Automation document** vào rule → resource vi phạm **tự được sửa**.

🎯 Mẫu cho mọi câu **"tự phát hiện + tự sửa, ít vận hành nhất"** → **AWS Config managed rule + Automatic Remediation bằng SSM Automation runbook có sẵn** (ưu tiên runbook có sẵn hơn tự viết Lambda).

| Yêu cầu | Managed rule | Remediation |
| --- | --- | --- |
| Mọi S3 bucket phải bật logging | **`s3-bucket-logging-enabled`** | **`AWS-ConfigureS3BucketLogging`** |
| EC2 chỉ chạy từ AMI đã duyệt | **`approved-amis-by-id`** | **`AWS-TerminateEC2Instance`** |
| Resource phải có tag bắt buộc | **`required-tags`** (managed, không cần viết Lambda) | Runbook gắn tag |
| IAM access key phải xoay vòng | **`access-keys-rotated`** (periodic) | Runbook xoay khóa |
| S3 không được public | **`s3-bucket-public-read-prohibited`** · **`s3-bucket-public-write-prohibited`** | Runbook gỡ quyền public |
| SG không mở SSH ra Internet | **`restricted-ssh`** (và `restricted-common-ports`) | Runbook đóng port |
| Không để Elastic IP rời | **`eip-attached`** | **`AWS-ReleaseElasticIP`** |
| S3 phải mã hóa | `s3-bucket-server-side-encryption-enabled` | Runbook bật default encryption |

🎯 **Trigger của rule:** **configuration changes** (khi resource được tạo/sửa/xóa), **periodic** (theo tần suất chọn), hoặc **cả hai** trong **một rule**. Đề "đánh giá khi tạo resource **và** mỗi X giờ" → một rule hai trigger, không tạo 2 rule.

🎯 **Cảnh báo compliance:** Config rule → **EventBridge rule** (event pattern + input transformer) match `NON_COMPLIANT` → **SNS**.

⚠️ Config **chỉ gửi thông báo khi trạng thái compliance THAY ĐỔI**. Resource đang NON_COMPLIANT mà lần sau vẫn NON_COMPLIANT → không có thông báo mới (rule vẫn được đánh giá).

⚠️ EBS volume bị xóa **gián tiếp** qua `TerminateInstances` (`DeleteOnTermination = true`) **không phát `DeleteVolume`** → Config chưa ghi nhận ngay, volume còn hiện trong kết quả tới khi **baseline chạy mỗi 6 giờ** tìm configuration item `ResourceDeleted`. Xóa bằng `DeleteVolume` thì gỡ ngay.

🎯 **"Ai đã thay đổi cấu hình resource?"** → **Config Configuration Timeline** (lịch sử trạng thái, liên kết CloudTrail để biết danh tính).

| | **AWS Config** | **Trusted Advisor** | **CloudTrail** |
| --- | --- | --- | --- |
| Mục đích | Cấu hình + compliance | Best practice (cost, performance, security, fault tolerance, **service limits**) | Log API call |
| Auto-remediation | ✅ | ❌ chỉ khuyến nghị | ❌ |
| Real-time | Sau khi resource đổi (có trễ) | Định kỳ | Gần real-time |
| Theo dõi service limit | ❌ | ✅ | ❌ |

⚠️ Distractor hay gặp: Security Hub (chỉ tổng hợp finding, không có remediation dựng sẵn) · Trusted Advisor (phát hiện, không tự sửa) · bucket policy (không bật được logging) · EventBridge (không có managed rule đánh giá tuân thủ) · launch template (không terminate instance) · CloudWatch (không có metric AMI ID) · Inspector (không tạo custom finding kiểm tra AMI).

---

## 11. AWS Systems Manager (SSM)

| Capability | Làm gì | Không làm gì |
| --- | --- | --- |
| **Patch Manager** | Scan & cài patch OS + ứng dụng; **patch baseline**, nhóm bằng **tag / patch group**, lịch qua **Maintenance Window**, báo cáo compliance. Trên Windows chỉ vá app do Microsoft phát hành | Không rescue, không thu metadata |
| **Inventory** | Thu **metadata** phần mềm/cấu hình của managed node (EC2 + on-prem) → S3 tập trung; có custom inventory (JSON); đa Region, đa account | Không vá lỗi, không đọc dữ liệu ứng dụng |
| **Automation** | Chạy **runbook** (restart hàng loạt, tạo AMI, remediation); là target của EventBridge / Config remediation | Không phải patch management |
| **Run Command** | Chạy lệnh ad-hoc trên nhiều node, target theo tag, có **command history** | — |
| **State Manager** | Gắn document với nhóm node để **bootstrap lúc launch** và **giữ trạng thái mong muốn** theo lịch | — |
| **Distributor** | Đóng gói & phân phối phần mềm (kể cả `.msi`), quản lý version | — |
| **Session Manager** | Shell không cần SSH key, bastion, port 22 (đi outbound HTTPS 443) | — |
| **Fleet Manager** | UI hợp nhất: health, duyệt file, Windows Registry, user OS | Không vá lỗi |
| **Maintenance Windows** | Lên lịch tác vụ có thể gián đoạn (patch, driver, cài phần mềm) | Không phải công cụ backup |
| **Parameter Store** | Lưu config/secret (**SecureString** mã hóa KMS) | Không auto-rotation |

🎯 **Vá OS/app hàng loạt** → Patch Manager. **Thu metadata phần mềm** → Inventory. **Cài phần mềm khi EC2 khởi động** (không sửa AMI/launch template) → **State Manager association**.

🎯 **Patch fleet lớn** (3 thành phần): gom instance vào **Resource Group** (theo tag) → **schedule trong Patch Manager** target resource group → **Maintenance Window** cho mỗi resource group. Không cần Automation runbook để patch hay theo dõi trạng thái patch.

🎯 **Auto-approval delay là thuộc tính của PATCH BASELINE** (không phải patch group, không phải maintenance window). Dev delay 2 ngày, Prod delay 5 ngày, tất cả trong một khung 2 giờ → **2 patch group + 2 patch baseline + 1 maintenance window**. ⚠️ Predefined baseline của AWS không chỉnh được delay.

🎯 **Script khẩn cấp (zero-day) trên hàng trăm EC2 Windows + Linux, có bằng chứng** → **Run Command** với `AWS-RunShellScript` / `AWS-RunPowerShellScript`, **target theo tag OS**, **command history** làm bằng chứng. ⚠️ Session Manager dành cho phiên tương tác; sửa user data chỉ chạy lúc boot.

🎯 **Agent `.msi` của bên thứ ba, tự cập nhật** → tạo **Distributor package** + **State Manager association** chạy **`AWS-ConfigureAWSPackage`**, target theo tag, lịch 1 ngày. ⚠️ `AWS-RunRemoteScript` không quản lý version; Inventory chỉ thu metadata; OpsItem là mục sự cố.

🎯 **Session Manager vào được máy khác nhưng không vào được máy này** (agent đã cài, user có quyền) → instance thiếu **instance profile** có **`AmazonSSMManagedInstanceCore`**. Không cần mở port 22, không cần key pair.

🎯 **Session Manager không đi Internet** → 3 **interface endpoint**: **`ssm`** (control plane, đăng ký node), **`ec2messages`** (agent nhận/xác nhận message), **`ssmmessages`** (kênh dữ liệu phiên). Tùy chọn: `logs` (ghi log phiên vào CloudWatch), `kms` (mã hóa phiên), `s3`. ⚠️ Endpoint `ec2` không cần.

🎯 **Server on-premises, không lưu credential dài hạn** → **Hybrid Activation**: tạo activation → nhận **Activation Code + Activation ID** → cài SSM Agent → đăng ký. Server nhận credential tạm từ IAM role của activation, tự làm mới. ⚠️ Instance profile chỉ cho EC2; IAM user + access key là credential dài hạn.

🎯 **Onboard toàn bộ Organization vào SSM, ít công nhất** → ở management account, **SSM Quick Setup** + **Default Host Management Configuration** → mọi EC2 hiện tại và tương lai thành managed node. ⚠️ StackSet tạo instance profile vẫn phải gắn từng instance; SCP không cấp quyền; Control Tower controls không cấu hình host management.

| Automation document | Công dụng |
| --- | --- |
| **`AWSSupport-ExecuteEC2Rescue`** | Khôi phục EC2 **impaired/unreachable** (lỗi network, RDP, firewall), chỉ cần Instance ID |
| **`AWSSupport-TroubleshootS3PublicRead`** | Chẩn đoán **403 khi đọc object trong bucket public** (bucket policy, ACL, Block Public Access, SCP); chỉ xét object public |
| `AWS-UpdateLinuxAmi` / `AWS-UpdateWindowsAmi` | Tạo golden AMI |
| `AWS-UpdateCloudFormationStackWithApproval` | Update stack có bước duyệt |
| **`AWS-ConfigureS3BucketLogging`** | Bật server access logging (remediation cho `s3-bucket-logging-enabled`) |
| **`AWS-TerminateEC2Instance`** | Terminate instance (remediation cho `approved-amis-by-id`) |
| **`AWS-CreateServiceNowIncident`** | Tạo incident ServiceNow, chỉ cần URL/username/password |
| `AWS-ReleaseElasticIP` | Release EIP rời (remediation cho `eip-attached`) |

🎯 **Parameter Store SecureString:** giá trị mã hóa KMS, lấy giá trị đã giải mã bằng **một** API call (đơn giản hơn tự mã hóa KMS rồi lưu: hai call). Customer managed key → kiểm soát bằng IAM policy + key policy; **audit lần giải mã qua CloudTrail**.

⚠️ **Tên bịa:** "Amazon Patch Manager", "Patch Fleet", "SSM Audit Trail". **Vá guest OS, phần mềm, cấu hình SG là trách nhiệm khách hàng.**

---

## 12. AWS CloudFormation

### Intrinsic functions & cross-stack

| Hàm | Trả về | Ví dụ |
| --- | --- | --- |
| **`!GetAtt`** | **Thuộc tính** của resource | `!GetAtt myRDS.Endpoint.Address`, `!GetAtt myELB.DNSName` |
| `!Ref` | Giá trị parameter, hoặc ID/tên của resource | `!Ref myEC2Instance` → Instance ID |
| `!Sub` | Thay biến trong chuỗi | `!Sub "arn:aws:s3:::${BucketName}/*"` |
| `!FindInMap` | Giá trị trong **Mappings** | `!FindInMap [RegionMap, !Ref "AWS::Region", AMI]` |
| **`!ImportValue`** | Giá trị được **export** bởi stack khác | `!ImportValue NetworkStack-SubnetId` |
| `Fn::Transform` | Gọi macro | — |

🎯 **Dùng output của stack A trong stack B** → stack nguồn khai báo **`Export`** trong **`Outputs`**, stack đích dùng **`Fn::ImportValue`**. ⚠️ `Ref` **không** import được từ stack khác; chỉ import trong **cùng account và Region**; stack đang bị import thì **không xóa được** và không sửa được giá trị export. Field "Expose" không tồn tại.

🎯 **Conditions** (vd prod/test) chỉ dùng được trong **`Resources`** và **`Outputs`**, **không** dùng trong `Parameters`.

### Stack Policy

| Đặc điểm | Chi tiết |
| --- | --- |
| Bản chất | JSON quy định **update action nào được phép trên resource nào** |
| Phạm vi | **Chỉ tác dụng khi UPDATE stack**, **không phải access control** (ai được làm gì → IAM) |
| Số lượng | **1 stack policy / stack** |
| Sau khi set | **Mọi resource mặc định được bảo vệ**; muốn cho update phải có `Allow` tường minh |
| `Principal` | Bắt buộc nhưng **chỉ nhận `*`** |
| Deny vs Allow | **`Deny` luôn thắng `Allow`**; `Allow` + `NotResource: LogicalResourceId/MyDatabase` tương đương cặp `Deny MyDatabase` + `Allow *` |

### Bảo vệ & vòng đời resource

| Cơ chế | Vai trò |
| --- | --- |
| **Termination Protection** (stack) | Chặn xóa stack. Bật mà vẫn xóa → **thất bại im lặng: stack và status giữ nguyên, không báo lỗi rõ** → tắt protection rồi xóa lại. Áp cả cho nested stack (xóa qua root stack) |
| **`DeletionPolicy`** | `Delete` (mặc định) / **`Retain`** / **`Snapshot`** (EBS Volume, RDS DBInstance/DBCluster, ElastiCache, Redshift, Neptune, DocumentDB) |
| **Nested Stacks** | Tách component lặp lại thành template riêng (`AWS::CloudFormation::Stack`) |
| **Parameter constraints** | Kiểm tra giá trị đầu vào, không chống xóa |

🎯 **Giữ EC2 và dữ liệu khi xóa stack** → **`DeletionPolicy: Retain`** (resource ra khỏi quản lý của CloudFormation nhưng vẫn chạy). ⚠️ **`Snapshot` không áp dụng cho EC2 instance.** AWS Backup/DLM chỉ sao lưu, không ngăn CloudFormation terminate.

🎯 **Giữ snapshot EBS khi xóa stack** → `DeletionPolicy: Snapshot` trên `AWS::EC2::Volume`.

| Triệu chứng khi xóa stack | Nguyên nhân |
| --- | --- |
| Không báo lỗi, không thành công, stack không đổi | **Termination Protection đang bật** |
| **`DELETE_FAILED`** | **S3 bucket còn object** · **SG còn bị resource khác tham chiếu** · thiếu IAM permission · lỗi dependency |

🎯 `DELETE_FAILED` vì SG bị tham chiếu → **xóa stack lần nữa và chọn RETAIN resource đó** (`delete-stack --retain-resources <LogicalId>`). ⚠️ Không có timeout cho thao tác DELETE; nested stack được xóa cùng parent; `--disable-rollback` chỉ ảnh hưởng CREATE/UPDATE.

🎯 **Custom resource (Lambda) chạy trước khi EC2 sẵn sàng** → thêm **`DependsOn: <LogicalId EC2>`** (CloudFormation tạo song song khi không có phụ thuộc). ⚠️ `ServiceToken`, `cfn-response`, `Fn::If` không điều khiển thứ tự.

🎯 **Component lặp lại ở nhiều template** → **Nested Stacks**. ⚠️ StackSets là đa account/Region; change set chỉ xem trước; macro là biến đổi template nâng cao.

### Change Sets & Resource Import

🎯 **Execute một change set → CloudFormation xóa mọi change set còn lại** của stack. Change set **không kiểm tra** account limit, IAM permission, resource có hỗ trợ update không; update fail thì stack rollback.

🎯 **Resource Import** đưa resource tạo ngoài CloudFormation vào quản lý (qua change set), cần template mô tả toàn stack (**mỗi resource import phải có `DeletionPolicy`**) + identifier. ⚠️ Drift Detection chỉ so sánh cấu hình thực tế với template.

### StackSets

🎯 **Deploy cùng stack ra nhiều account × Region** (vd IAM role, bật AWS Config cho mọi account) → **StackSets + Organizations**, deploy **stack instances** tới Organization hoặc OU. ⚠️ "Deploy stack policies" là nhầm khái niệm; SCP không triển khai resource.

| Mô hình quyền | Ai tạo IAM role |
| --- | --- |
| **Self-managed** | **Bạn tự tạo** role ở administrator account và target account (dùng khi account không cùng Organization) |
| **Service-managed** | **StackSets tự tạo**; hỗ trợ **automatic deployment** cho account mới vào Organization |

⚠️ Chỉ có hai mô hình (không có "resource-managed"). Cần **trust relationship** giữa administrator và target account. Xóa stack instance với **Retain Stacks** → stack vẫn do CloudFormation quản lý, chỉ nằm ngoài stack set.

🎯 **Giám sát tập trung toàn Organization** → StackSet tạo **CloudWatch alarm ở từng account** cùng trỏ về **một SNS topic** ở account logging (topic cho phép cross-account publish).

### Secrets & IAM trong template

🎯 **Credentials RDS trong template, có auto-rotation** → **`AWS::SecretsManager::Secret`** (dùng **`GenerateSecretString`** để AWS tự sinh mật khẩu) + **`AWS::SecretsManager::RotationSchedule`**, tham chiếu bằng **`{{resolve:secretsmanager:...}}`**. ⚠️ `{{resolve:ssm-secure}}` (Parameter Store SecureString) mã hóa được nhưng **không auto-rotation**; `{{resolve:ssm}}` là plaintext; Parameter + `Ref` là plaintext.

🎯 **Quyền để deploy stack:** quyền CloudFormation (`cloudformation:CreateStack`) **và** quyền trên từng service template tạo ra (vd `s3:CreateBucket`). ⚠️ `CreateStackSet` là cho đa account; `s3:ListBucket`/`PutObject` không phải quyền tạo bucket.

🎯 **Bỏ property `BucketName`** → CloudFormation tự sinh tên duy nhất; resource khác tham chiếu bằng `!Ref` / `!GetAtt`.

⚠️ **Custom-named IAM resource:** IAM là global, tên phải unique trong account. Đặt tên cứng rồi dùng template ở nhiều Region cùng account → các stack **dùng chung một IAM resource**, sửa/xóa ở stack này phá stack kia. → Không đặt tên cứng cho IAM resource.

### Template đa Region

🎯 **Template chạy ở Region này, fail ở Region khác** → **AMI ID là region-specific**, hoặc **dịch vụ chưa có ở Region đích**. ⚠️ IAM là global nên không phải nguyên nhân; thiếu quyền thì fail cả hai Region.

🎯 Deploy sang nhiều Region → `aws ec2 copy-image` sang từng Region → **`Mappings`** AMI theo Region + `!FindInMap` (hoặc SSM public parameter). ⚠️ ASG không trải qua nhiều Region.

### Đọc template static website

🎯 `AccessControl: PublicRead` + `WebsiteConfiguration` → bucket host website; `DeletionPolicy: Retain` → bucket **không bị xóa** khi xóa stack; `Outputs` dùng `Fn::GetAtt` lấy **`WebsiteURL`** và **`DomainName`**; `!Join` ghép ARN `arn:aws:s3:::` + `!Ref S3Bucket` + `/*`.

### Debug khi tạo stack

| `OnFailure` | Hành vi khi tạo thất bại |
| --- | --- |
| `ROLLBACK` | Mặc định: rollback & xóa resource đã tạo |
| `DELETE` | Xóa stack |
| **`DO_NOTHING`** | **Giữ nguyên resource** → SSH vào đọc `/var/log/cfn-init-output.log` |

⚠️ Chỉ đặt **`OnFailure` HOẶC `DisableRollback`**, không cả hai. Cài CloudWatch agent hay bật Flow Logs vi phạm yêu cầu "không đổi cách EC2 hoạt động".

| Helper | Vai trò |
| --- | --- |
| **`cfn-init`** | Đọc `AWS::CloudFormation::Init` → cài package, ghi file, bật service |
| **`cfn-signal`** | Gửi SUCCESS/FAILURE về CloudFormation |
| `cfn-get-metadata` / `cfn-hup` | Lấy metadata / daemon phát hiện metadata đổi |
| **`WaitCondition` / `CreationPolicy`** | CloudFormation **chờ** đủ signal hoặc hết timeout |

🎯 **Stack báo CREATE_COMPLETE quá sớm** → thiếu **WaitCondition/CreationPolicy**. **Không nhận được signal** → subnet private **thiếu route ra Internet** (NAT/IGW/VPC endpoint), hoặc cfn-signal chưa chạy kịp trước timeout. `Timeout` tối đa **43,200 giây (12 giờ)**. ⚠️ cfn-signal **không cần IAM role**.

---

## 13. EC2 vận hành & AMI

### Purchase options & tenancy

| Model | Đặc điểm | Use case |
| --- | --- | --- |
| **Spot** | Giảm tới ~90%; bị thu hồi chủ yếu khi EC2 cần lại capacity | Fault-tolerant, resume được (batch, ETL, render) |
| **On-Demand** | Trả theo giây, không cam kết | Ngắn hạn, không được gián đoạn |
| **Reserved Instances** | Cam kết 1/3 năm | Tải ổn định dài hạn |
| **Savings Plans** | Cam kết $/giờ 1/3 năm | Ổn định nhưng cần linh hoạt |
| **On-Demand Capacity Reservation** | Giữ capacity trong **1 AZ**, thời hạn tùy ý, **không giảm giá** | Đảm bảo capacity |
| **Dedicated Instance** | Phần cứng riêng cho **một account** | **Single-tenant rẻ nhất** |
| **Dedicated Host** | Cả server vật lý, thấy socket/core, kiểm soát vị trí đặt | **License theo socket/core/VM** (Windows Server, SQL Server, RHEL...) |

📌 Đề cũ nói Spot bị thu hồi "khi giá vượt max price": nay nguyên nhân chính là **EC2 cần lại capacity**; AWS khuyên không đặt max price.

⚠️ Capacity Reservation: không cam kết 1/3 năm, không giảm giá, không dùng với Dedicated Host, không transfer giữa account (nhưng **share được qua RAM**). 📌 Đề cũ ghi "không dùng với Placement Group": nay tạo được Capacity Reservation trong **cluster placement group** (spread/partition vẫn không).

🎯 **Spot interruption behavior:** chỉ **Stop / Hibernate / Terminate** (mặc định). **Không có "Reboot".** Không bật được termination protection cho Spot.

| Spot allocation strategy | Chọn pool | Dùng khi |
| --- | --- | --- |
| **`capacity-optimized`** | Pool **còn nhiều capacity nhất** → ít bị thu hồi nhất | "pools that have the most availability" |
| `price-capacity-optimized` | Cân bằng giá + capacity (AWS khuyến nghị chung) | Vừa rẻ vừa ổn định |
| `diversified` | Trải đều mọi pool | Giảm rủi ro tổng thể |
| `lowest-price` | Pool rẻ nhất | Chấp nhận gián đoạn cao |

⚠️ Spot Instance Advisor chỉ để **tra cứu** tỷ lệ gián đoạn.

🎯 **Stateless, cần tối thiểu 6 instance, rẻ nhất** → **Spot Fleet với On-Demand base capacity = 6**, phần thêm dùng Spot.

🎯 **EC2 sẽ chuyển dần sang Fargate trong 6 tháng, không được thừa reservation** → **Compute Savings Plans, No Upfront** (phủ EC2 + Fargate + Lambda). EC2 Instance SP và RI chỉ cho EC2.

### Hiệu năng & đổi instance type

🎯 **Họ T (burstable):** 1 CPU credit = 1 vCPU 100% trong 1 phút. **Hết credit → bị throttle** (CPU cao nhưng chậm) → bật **T2/T3 Unlimited** hoặc đổi instance type. Chạy 90% CPU và **chậm dần sau vài phút** → đổi sang **compute-optimized (C5/C6i/C7i)**; Provisioned IOPS chỉ cải thiện đĩa, thêm instance vi phạm "chỉ scale dọc", RI chỉ giảm giá.

| Loại instance | Đổi instance type |
| --- | --- |
| **EBS-backed** | **Stop → đổi type → start**; Instance ID giữ nguyên, chuyển sang phần cứng mới |
| **Instance store-backed** | **Không đổi trực tiếp**: tạo AMI → launch instance mới đúng type → **chuyển Elastic IP** → terminate cái cũ |

⚠️ Stop/start **mất public IPv4 cũ** → giữ dịch vụ không gián đoạn phải dùng **Elastic IP**. Giữ nguyên private IPv4, EIP, IPv6. Instance trong ASG: **suspend scaling processes** khi resize (instance stopped bị coi là unhealthy). Luôn có downtime.

| Lỗi launch | Nghĩa | Xử lý |
| --- | --- | --- |
| **`InsufficientInstanceCapacity`** | AWS hết capacity cho type đó ở AZ đó | **Thử AZ khác** / type khác / đợi / giảm số lượng |
| **`InstanceLimitExceeded`** | Chạm **service quota** của account | **Xin tăng quota** |

🎯 **Enhanced Networking (ENA/SR-IOV):** băng thông cao hơn, PPS cao hơn, latency thấp, miễn phí; dùng khi nghẽn PPS. Băng thông cao giữa instance → **cluster placement group** + ENA. ⚠️ Placement group **không** giúp giảm chi phí.

| Placement group | Cách đặt | Use case |
| --- | --- | --- |
| **Cluster** | Gần nhau trong 1 AZ | HPC, latency thấp nhất |
| **Partition** | Partition không chung phần cứng | Hadoop, Cassandra, Kafka |
| **Spread** | Mỗi instance phần cứng riêng | Vài instance quan trọng |

🎯 **Lambda tắt EC2 ban đêm nhưng bỏ qua instance quan trọng** → gắn **tag** (vd `AutoShutdown=false`), Lambda đọc tag để bỏ qua. Không phải sửa code khi thêm instance.

🎯 **App nhỏ, cập nhật thường xuyên** → **User Data** tải bản mới nhất lúc launch (không nướng vào AMI).

🎯 **AMI hằng tuần + quét lỗ hổng lúc build** → **EC2 Image Builder** (pipeline theo lịch, **tích hợp Amazon Inspector**). ⚠️ Packer + cron là tự dựng; `CreateImage` qua EventBridge chỉ chụp instance đang chạy, không quét.

### AMI

🎯 **AMI ID là duy nhất theo Region** → script compliance cần danh sách AMI ID riêng mỗi Region. **CopyImage** copy trong/giữa Region, có thể mã hóa/mã hóa lại; AMI đích là bản độc lập, ID riêng.

🎯 **Share AMI:** chỉ share được AMI **không mã hóa** hoặc mã hóa bằng **customer managed KMS key** (phải share luôn key). **Không share được AMI mã hóa bằng AWS managed key.** Không cần share snapshot. Muốn dùng ở Region khác → **copy AMI sang Region đó** rồi share. ⚠️ Không để AMI mã hóa ở chế độ public.

🎯 **Share AMI đã mã hóa sang account khác:** tạo **customer managed key** → key policy cấp `kms:DescribeKey`, `kms:ReEncrypt*`, `kms:CreateGrant`, `kms:Decrypt` cho account đích → **copy AMI với key mới** → thêm account ID vào launch permission của bản copy.

🎯 **Deregister AMI đã share:** **không launch mới được ở cả hai account**; instance đã chạy **không bị ảnh hưởng**. AMI đã deregister **không restore được**, chỉ tạo lại từ EBS snapshot còn giữ hoặc từ instance đang chạy; AWS Support không giữ backup AMI.

🎯 **AMI application-consistent** → **không chọn "No reboot"** (mặc định): shutdown → flush buffer → snapshot → reboot. Chọn "No reboot" thì chỉ crash-consistent.

---

## 14. Auto Scaling

| Loại scaling | Dùng khi |
| --- | --- |
| **Scheduled** | Traffic **dự đoán được theo giờ cố định** |
| **Dynamic** (target tracking, step, simple) | Phản ứng theo metric thực tế (vd "scale ngay khi CPU vượt 50%") |
| **Predictive** | ML dự báo pattern phức tạp, scale trước |

⚠️ "Event-based scaling" **không tồn tại**. Target tracking là reactive (trễ) nên không thắng scheduled khi đã biết giờ; tăng min/desired cố định là lãng phí.

🎯 **Simple scaling:** mỗi lần chỉ một hành động, rồi **chờ hết cooldown (mặc định 300 giây)**. Metric vượt ngưỡng lần hai trong cooldown → **bị bỏ qua**. Step scaling/target tracking linh hoạt hơn.

🎯 **Nhiều policy cùng kích hoạt** → ASG **chọn policy tạo thay đổi capacity LỚN NHẤT**, không cộng dồn (áp cả scale-out lẫn scale-in).

🎯 **Scale theo SQS** → **metric math** `ApproximateNumberOfMessagesVisible ÷ số instance InService` + **target tracking** (vd giữ 100 message/instance). ⚠️ `ApproximateAgeOfOldestMessage` không cho biết số message; worker pull từ queue nên `ALBRequestCountPerTarget` sai.

🎯 **ASG không scale out dù CPU 90%** → đã chạm **max capacity** hoặc process **`Launch` bị suspend**. ⚠️ Hết capacity instance type thì có lỗi rõ; warmup chỉ làm instance mới chưa tính vào metric. Metric kích hoạt là **giá trị tổng hợp** của ASG.

### Health check & thay instance

🎯 **Healthy trong ASG nhưng unhealthy trong ALB** → ASG đang dùng **EC2 status check** thay vì **ELB health check**. Đứng sau LB thì bật ELB health check cho ASG.

🎯 Instance unhealthy **không tự khỏi** → bị thay gần như ngay. Khi terminate, **Elastic IP bị gỡ** và **không tự gắn** sang instance mới. `HealthCheckGracePeriod` mặc định 300 giây (Console), 0 (CLI/SDK). `set-instance-health` = Unhealthy → ASG thay instance.

### Suspended processes & bảo vệ instance

| Process bị suspend | Hậu quả |
| --- | --- |
| **Launch** | Không scale out |
| **Terminate** | Không scale in; nếu **AZRebalance vẫn chạy** thì rebalance launch mới mà không terminate cũ → ASG phình **tới 10% trên max** |
| **AZRebalance** | Không tự cân bằng lại AZ |
| **ReplaceUnhealthy** | Không thay instance unhealthy |

🎯 **Đóng băng ASG để troubleshoot** → suspend **`Launch` + `Terminate`**, xong thì resume. ⚠️ Scale-in protection không chặn scale-out; Standby có thể khiến ASG launch instance thay thế; gỡ listener ALB không dừng scaling.

| Cơ chế | Chặn được | Không chặn được |
| --- | --- | --- |
| **`DisableApiTermination`** | Terminate qua Console/CLI/API | Shutdown từ OS · ASG terminate · **không bật được cho Spot** |
| **Scale-in protection** | ASG terminate khi scale in | Terminate thủ công · thay instance unhealthy · Spot interruption |
| **Termination policy** | Chọn **thứ tự** instance bị xóa | Không bảo vệ instance nào |

🎯 Tất cả instance được scale-in protection mà có scale-in event → **desired GIẢM** nhưng **không instance nào bị terminate**.

🎯 **Developer cần SSH vào chẩn đoán trước khi ASG terminate** → bật scale-in protection cho **instance mới launch** (`--new-instances-protected-from-scale-in`), xong thì tắt từng instance.

🎯 **Tác vụ dài (xử lý message SQS tới 12 giờ)** → script **tự bật scale-in protection cho chính instance** khi bắt đầu, tắt khi xong. ⚠️ Termination policy chỉ đổi thứ tự; suspend process áp cả group; "Delete on termination" là thuộc tính EBS.

### Launch template, mixed instances, warm pool

🎯 **N instance On-Demand luôn chạy + Spot khi rẻ, ít công nhất** → **một ASG + Launch Template + mixed instances policy** (On-Demand base capacity + Spot). ⚠️ Hai ASG riêng là nhiều công hơn.

📌 **Launch Configuration** không sửa được sau khi tạo, đã deprecated; account tạo từ **01/10/2024** không tạo được launch configuration mới → dùng **Launch Template**.

🎯 **Boot script lâu làm scale-out chậm, cấm overprovision** → **Warm Pool** giữ instance đã khởi tạo ở trạng thái **Stopped** (hoặc Running/Hibernated); instance Stopped chỉ tốn phí EBS. ⚠️ Tăng min capacity là overprovision; predictive scaling vẫn chờ boot script; instance lớn hơn không rút ngắn boot.

### Tính số instance chịu lỗi AZ

🎯 Giữ tối thiểu **N** instance khi mất 1 AZ → **tổng = N ÷ (số AZ − 1) × số AZ**. N = 12: **3 AZ × 6 = 18** (ít nhất); 2 AZ × 12 = 24 thừa; 3 AZ × 4 = 12 chỉ còn 8.

| Lỗi launch của ASG | Nguyên nhân |
| --- | --- |
| **`Client.InternalError: Client error on launch`** | EBS mã hóa bằng **customer managed KMS key**, **service-linked role của ASG không có quyền dùng key** |
| `The security group <name> does not exist` | SG trong launch config/template đã bị xóa |
| `Invalid device name` | Block device mapping sai tên |
| `Placement groups may not be used with instances of type ...` | Instance type không hợp lệ cho cluster placement group |

---

## 15. Elastic Load Balancing

| Loại | Tầng | Đặc điểm |
| --- | --- | --- |
| **NLB** | L4 | Latency rất thấp, **hàng triệu request/giây**, IP tĩnh/EIP mỗi AZ |
| **ALB** | L7 | Route theo path/host/header; target EC2, IP, container, **Lambda** |
| **GWLB** | L3 | Đưa traffic qua **security appliance** bên thứ ba (firewall, IDS/IPS); không đọc HTTP |
| **CLB** | L4/L7 cơ bản | Legacy; có `SurgeQueueLength`/`SpilloverCount` |

⚠️ "Infrastructure Load Balancer" không tồn tại.

🎯 **NLB và AZ:** mỗi AZ một subnet, ELB tạo node + ENI trong subnet đó; với internet-facing NLB có thể gán **một EIP mỗi subnet lúc tạo**. ⚠️ ❓ Đề nói "không thể tắt AZ của NLB sau khi tạo, chỉ thêm được". 📌 Docs hiện hành cho phép **gỡ subnet (AZ) khỏi NLB** (trừ khi NLB có VPC endpoint association đang hoạt động; gỡ mất tới 3 phút). Khi thi, chọn theo logic đề.

### Cấu trúc ALB

🎯 **Listener** kiểm tra kết nối theo protocol + port (**phải có ít nhất một listener**). **Rule** = priority + action + condition; **default rule không có condition**. **Target group gắn vào listener** qua rule. Target nên **trải nhiều AZ**; **một target đăng ký được nhiều target group**.

🎯 **User bị logout liên tục** → **Sticky Sessions** bật ở **target group**: cookie **duration-based** (ALB tạo **`AWSALB`**) hoặc **application-based** (cookie của app, ALB bọc trong `AWSALBAPP`, đồng bộ timeout với app). Client phải hỗ trợ cookie. ⚠️ Slow start, connection draining, `X-Forwarded-For` không liên quan session.

🎯 **CloudFront trước ALB, user bị logout sớm** → (1) chuyển ALB sang **application-based cookie** (2) **bật cookie forwarding** trong cache behavior của CloudFront (mặc định CloudFront bỏ cookie).

🎯 **Chỉ cho EC2 nhận traffic từ LB** → SG của EC2 đặt **source = SG của LB**. ⚠️ SG rule chỉ nhận CIDR, SG ID, prefix list (không nhận ENI); NACL theo CIDR của LB phải bảo trì thủ công.

🎯 **Custom health check** → đặt **Health Check Path** (vd `/health`) trên target group; app trả 200 = healthy. ⚠️ ALB không đọc trạng thái từ S3/ElastiCache/API Gateway.

🎯 **Trình duyệt cũ lỗi SSL với ALB** → đổi **Security Policy (SSL negotiation)** sang policy có cipher/TLS cũ (vd `ELBSecurityPolicy-TLS-1-0-2015-04`). ⚠️ Không phải vấn đề certificate; không cần ALB thứ hai.

🎯 **Ép HTTPS mà không phá user đang dùng `http://`** → thêm **listener HTTPS 443** (forward tới target group, cert ACM) + đổi default rule của **listener HTTP 80 thành REDIRECT sang 443**. ⚠️ Xóa listener 80 phá user cũ; không gắn được cert vào listener HTTP.

| Header | Cho biết |
| --- | --- |
| **`X-Forwarded-For`** | **IP thật của client** |
| `X-Forwarded-Proto` | HTTP hay HTTPS |
| `X-Forwarded-Port` | Port client dùng |
| `X-Amzn-Trace-Id` | ID để trace request |

### Troubleshoot ELB

🎯 **Client không kết nối được LB:** SG/NACL không cho traffic client trên listener port, hoặc **internet-facing LB đặt nhầm private subnet**.

🎯 Còn ít nhất 1 target healthy → chỉ route tới target healthy. **Toàn bộ target unhealthy → ALB vẫn route tới target unhealthy.**

| Mã | Ý nghĩa |
| --- | --- |
| **200** | Thành công (success code mặc định của health check) |
| **403** | Bị **WAF Web ACL** chặn |
| **500** | Lỗi do LB sinh |
| **502** | Target trả response lỗi/không hợp lệ |
| **503** | Target group **không có target đăng ký** |
| **504** | Target không phản hồi kịp |

---

## 16. Amazon Route 53

| Policy | Dùng khi |
| --- | --- |
| **Simple** | 1 resource |
| **Geolocation** | Theo **vị trí nơi phát sinh DNS query**; giới hạn nội dung theo khu vực. Không có default record → trả "no answer" cho vị trí chưa khai báo |
| **Geoproximity** | Vị trí + **bias** mở rộng/thu hẹp vùng |
| **Latency** | Nhiều Region → Region latency thấp nhất |
| **Failover** | Sang resource dự phòng khi chính unhealthy |
| **Weighted** | Chia theo tỷ lệ (A/B, blue-green) |
| **Multivalue answer** | Nhiều IP kèm health check |

| Record | Công dụng |
| --- | --- |
| **CNAME** | Tên → tên (kể cả **domain bên thứ ba**); **không dùng ở zone apex**; có phí query |
| **Alias** | Tới **tài nguyên AWS** (CloudFront, ELB, S3 website, API Gateway) hoặc record cùng zone; **dùng được ở apex**; **miễn phí query** |
| **A** | Domain → địa chỉ IP |
| **PTR** | Reverse DNS |

🎯 **Trỏ domain của tôi sang domain bên thứ ba** → **CNAME**. **`www` → CloudFront, rẻ nhất** → **Alias** (CloudFront không có IP tĩnh nên không dùng A record).

🎯 **Failover đa Region (mỗi Region ALB + ASG)** → **Alias tới từng ALB + Failover policy + Evaluate Target Health = Yes**. ⚠️ ALB **không route cross-Region** (target group chỉ chứa target cùng Region).

🎯 **Failover cho website ON-PREMISES** (chỉ chuyển khi HTTP 2xx/3xx) → **A record cho mỗi server + Route 53 HTTP health check**. ⚠️ Alias chỉ cho tài nguyên AWS; TCP health check không đọc được status code.

🎯 **Một EC2 cần IP tĩnh + tên miền, ít công nhất** → **Elastic IP + A record**. ALB/ASG là thừa; CNAME không trỏ được tới IP.

🎯 **Route 53 → S3 static website:** **tên bucket phải trùng tên record** (`example.com` → bucket `example.com`); domain + `www` → **hai bucket**, bucket `www` redirect về bucket chính. ⚠️ **S3 website endpoint không hỗ trợ HTTPS**; tên bucket không có wildcard, không có `http://`.

🎯 **VPC mới cần phân giải private hosted zone có sẵn** → **associate** zone với VPC (`AssociateVPCWithHostedZone`). Cross-account (VPC ở A, zone ở B): **B chạy `CreateVPCAssociationAuthorization` trước**, rồi **A chạy `AssociateVPCWithHostedZone`**. ⚠️ Không cần SG/NACL/route cho Resolver; sửa `/etc/resolv.conf` là thủ công.

| Resolver endpoint | Chiều query | Security Group | Dùng khi |
| --- | --- | --- | --- |
| **Inbound** | **On-prem → AWS** | Cho phép **inbound** TCP/UDP 53 từ DNS on-prem | On-prem phân giải private hosted zone / tên EC2 qua DX hoặc VPN |
| **Outbound** | **AWS → on-prem** | Cho phép **outbound** TCP/UDP 53 tới DNS on-prem | VPC phân giải domain nội bộ on-prem (kèm forwarding rule) |

---

## 17. Amazon RDS

| | **Multi-AZ** | **Read Replica** |
| --- | --- | --- |
| Replication | **Đồng bộ** | **Bất đồng bộ** |
| Phạm vi | ≥ 2 AZ trong **1 Region** | Cùng AZ, cross-AZ, **cross-Region** |
| Mục đích | **HA / failover tự động** | **Scale đọc**, tách báo cáo/ETL |
| Mất transaction đã commit? | **Không** | **Có thể** (lag) |
| Failover | Tự động, **DNS endpoint giữ nguyên** | Promote **thủ công**, endpoint **đổi** |
| Phục vụ đọc? | **Không** (standby ẩn) | **Có** |

🎯 Multi-AZ tự failover khi: primary lỗi · AZ outage · đổi instance type · patch OS · **Reboot with failover**. "HA, không mất commit" → Multi-AZ. "ETL ban đêm làm chậm website" → **Read Replica cho ETL**. ⚠️ Backup + PITR bảo vệ dữ liệu, không tạo HA; cross-Region replica có lag → không zero data loss; JVM DNS TTL ≤ 60 giây giúp client kết nối lại nhanh sau failover.

🎯 **Test nâng version engine với dữ liệu production** → **snapshot → restore ra instance MỚI (cùng version) → upgrade instance mới → test**. ⚠️ Read replica chỉ read-only; template CloudFormation không có dữ liệu; restore thẳng vào version mới là sai thứ tự.

🎯 **Lambda báo "too many connections"** → **RDS Proxy** (pooling, multiplexing; trỏ Lambda sang proxy endpoint). RDS Proxy còn **giảm thời gian gián đoạn khi failover** (AWS công bố tới ~66%) vì giữ pool và tự chuyển sang primary mới. ⚠️ `max_connect_errors` chỉ là số lần lỗi trước khi chặn host; tăng reserved concurrency của Lambda làm tệ hơn; standby Multi-AZ không nhận kết nối.

🎯 **Automated backup bật, retention hợp lệ (1–35 ngày) nhưng không có bản backup** → instance ở trạng thái **`STORAGE_FULL`** (RDS chỉ backup khi `available` và còn dung lượng) → **tăng storage**. ⚠️ Không có "S3 lưu backup bị đầy"; Multi-AZ không bắt buộc cho backup.

🎯 **DB hay hết dung lượng, ít đổi ứng dụng nhất** → **RDS Storage Auto Scaling** (CloudFormation: `MaxAllocatedStorage`). ⚠️ Đổi sang Aurora/DynamoDB phải sửa app; `VACUUM` chỉ thu hồi không gian, không tăng dung lượng cấp phát.

🎯 **Tạo DB mới từ dữ liệu ≤ 24 giờ, ít công nhất** → **restore từ automated snapshot gần nhất** hoặc **tạo read replica rồi promote**. ⚠️ `pg_dump` qua EC2, native backup ra S3, DMS đều nhiều công hơn.

🎯 **Share snapshot RDS ĐÃ MÃ HÓA sang account khác** → tạo snapshot + **sửa key policy KMS** cho account đích (`kms:Decrypt`, `kms:CreateGrant`...) + share snapshot. ⚠️ Tạo key mới cùng alias ở account đích không giải mã được; không có cross-account read replica; KMS key không replicate sang account khác.

🎯 **Read replica MySQL hỏng replication** khi: ghi vào bảng trên replica (kể cả đặt `read_only=0`) · `max_allowed_packet` của replica **nhỏ hơn** source · dùng engine **phi giao dịch (MyISAM)** (replica cần InnoDB) · query non-deterministic (`SLEEP()`, `SYSDATE()`, `UUID()`...).

🎯 **Ép RDS PostgreSQL chỉ nhận SSL** → DB Parameter Group **`rds.force_ssl = 1`**. MySQL/MariaDB: `require_secure_transport = ON` hoặc `REQUIRE SSL` cấp user. ⚠️ SG chỉ lọc nguồn/port; không có nút "Enable SSL"; không cài patch/addon lên RDS.

🎯 **Mã hóa RDS phải bật LÚC TẠO**; không bật được cho instance có sẵn (snapshot → copy có mã hóa → restore). ⚠️ RDS MySQL không có TDE (TDE là của Oracle/SQL Server); không mã hóa trực tiếp EBS của RDS.

| Cơ chế giám sát RDS | Nguồn | Cho biết |
| --- | --- | --- |
| **CloudWatch metrics** | **Hypervisor** | CPU, IOPS, connections ở mức máy ảo |
| **Enhanced Monitoring** | **Agent trong instance** | Metric **OS-level** real-time: free/active memory, swap, process, file system; từng process dùng CPU ra sao. Ghi JSON vào CloudWatch Logs (log group `RDSOSMetrics`), giữ **30 ngày** mặc định |
| **Performance Insights** | DB engine | **DB load**, top SQL, wait event; không có metric OS |
| **Event Subscriptions** | Sự kiện RDS | SNS khi instance, snapshot, parameter group, SG thay đổi |

📌 **Performance Insights** nay nằm trong **CloudWatch Database Insights** (RDS bật Database Insights mặc định ở Advanced mode). Đề cũ vẫn gọi là Performance Insights; vai trò "DB load, top SQL" không đổi.

---

## 18. Amazon Aurora

| Endpoint | Trỏ tới | Dùng khi |
| --- | --- | --- |
| **Cluster (writer)** | Primary hiện tại | Đọc + ghi; tự trỏ sang primary mới sau failover |
| **Reader** | **Load-balance mọi Aurora Replica**; mỗi cluster đúng 1 reader endpoint | Read-only |
| **Custom** | Nhóm instance tự chọn | Tách workload |
| **Instance** | 1 instance | Debug |

🎯 App hard-code endpoint của replica, replica đổi do Auto Scaling → dùng **Reader Endpoint**. ⚠️ ELB **không** load-balance cho RDS/Aurora.

🎯 Cluster single-writer có tối đa **15 Aurora Replica**. Primary và **mọi reader cùng một AZ, AZ đó chết** → phải **tạo thủ công instance mới ở AZ khác**.

🎯 **Aurora Auto Scaling** chỉ **thêm/bớt SỐ LƯỢNG reader**, không đổi instance class, **không scale writer**. Metric: **CPU trung bình của các Aurora Replica** hoặc `DatabaseConnections` trung bình. Đặt target **thấp hơn ngưỡng gây sự cố** (hiệu năng giảm ở 200 connection → target 195). ⚠️ Theo dõi CPU của primary để scale replica là sai.

🎯 **Báo cáo nặng làm nghẽn writer** → thêm **reader instance** và trỏ báo cáo sang reader. ⚠️ ACU là của Aurora Serverless, không áp cho cluster provisioned.

| Khôi phục | Engine | Hành vi |
| --- | --- | --- |
| **PITR** | Aurora MySQL & **PostgreSQL** | **Tạo DB cluster MỚI** ở mốc thời gian, không rollback tại chỗ → tạo instance cho cluster mới → **đổi endpoint ứng dụng** |
| **Backtrack** | **Chỉ Aurora MySQL** | Tua ngược cluster **tại chỗ** |

🎯 **Lỡ `DROP TABLE` trên Aurora PostgreSQL** → **PITR tạo cluster mới**. ⚠️ Không có Backtrack cho PostgreSQL; DMS không dừng được ở mốc lịch sử tùy ý.

🎯 **`FreeLocalStorage` cạn** (file tạm khi sort/group dữ liệu lớn) → **nâng instance class** (temp storage gắn với instance class). ⚠️ Storage auto scaling chỉ cho cluster volume; thêm replica không tăng temp storage; query plan management chỉ điều khiển cách chạy SQL.

🎯 **Aurora Serverless: bật log qua cluster parameter group** (slow query, general, audit log) → tự publish lên CloudWatch Logs; log không bật mặc định. ❓ Nhận định "không có instance cố định nên không xem log trên RDS Console" là của **Aurora Serverless v1**; v2 chạy như DB instance thông thường.

⚠️ ❓ Đề nói "RDS MySQL/MariaDB không có nhiều writer, chỉ Aurora có multi-master". Multi-master của Aurora chỉ có ở Aurora MySQL v1 (MySQL 5.6), bản này đã hết vòng đời 28/02/2023. Khi thi vẫn chọn "RDS không hỗ trợ nhiều writer".

---

## 19. Amazon ElastiCache

| | **Valkey / Redis OSS** | **Memcached** |
| --- | --- | --- |
| Cấu trúc dữ liệu | Phong phú (list, set, sorted set, stream...) | Key-value đơn giản |
| Replication / Multi-AZ / failover | ✅ | ❌ |
| Persistence / snapshot | ✅ | ❌ (bản node-based) |
| Multi-threaded | ❌ | ✅ |

📌 ElastiCache nay có thêm engine **Valkey** (tương thích Redis OSS); **Memcached serverless** hỗ trợ backup/snapshot, chỉ Memcached node-based là không.

🎯 **Redis HA, RTO thấp nhất** → **replication group** (1 primary + replica) trải nhiều AZ + **Multi-AZ với automatic failover** (endpoint giữ nguyên). ⚠️ AOF/RDB chỉ lo durability, không tự failover; Memcached không replication; ALB không đứng trước Redis (TCP).

🎯 **Evictions cao** → **node type lớn hơn** (nhiều RAM). ⚠️ Tăng TTL làm tệ hơn, giảm TTL tăng cache miss.

🎯 **Evict ngẫu nhiên, cần giữ item phổ biến, không tăng chi phí** → đổi eviction policy sang **`allkeys-lfu`**. `allkeys-lru` xóa key lâu chưa dùng; `allkeys-random`/`volatile-random` là tệ nhất; `volatile-*` chỉ xét key có TTL.

🎯 **Đặc tính Redis Multi-AZ:** replication **bất đồng bộ** (failover có thể mất ít dữ liệu); promote replica có **lag nhỏ nhất**; **promote thủ công chỉ khi tắt Multi-AZ/auto failover**; **reboot primary do khách chủ động KHÔNG kích hoạt failover**, primary khởi động lại trống và replica **cũng xóa bản sao** → mất dữ liệu.

🎯 Scale đọc cho RDS → **ElastiCache** hoặc **Read Replica**. ⚠️ DAX chỉ cho DynamoDB; Multi-AZ không scale đọc.

---

## 20. Amazon DynamoDB

🎯 **Chống xóa nhầm bảng + giảm mất dữ liệu** → **Deletion Protection** (chặn mọi `DeleteTable`, kể cả từ Lambda/CLI) + **PITR** (backup liên tục tới từng giây, **1–35 ngày**; bảng bị xóa thì AWS giữ system backup 35 ngày). ⚠️ CloudFormation termination protection chỉ chặn xóa stack; backup hằng ngày có thể mất 24 giờ dữ liệu.

🎯 **Cảnh báo TRƯỚC khi bị throttle** → alarm trên **`ConsumedReadCapacityUnits`** gần provisioned capacity (80–90%) → SNS. ⚠️ Bắt `THROTTLING_EXCEPTION` trong log là đã muộn; auto scaling + EventBridge chỉ báo sau khi scale.

🎯 **DR đa Region** → bật **DynamoDB Streams** + thêm Region vào **Global Table** (multi-active). ⚠️ DAX là cache cùng Region; GSI là pattern truy vấn trong bảng; PITR là backup cùng Region.

🎯 **Truy cập DynamoDB cross-account** → Account B tạo **IAM role** (quyền query) + **trust policy** cho role của EC2 ở Account A → EC2 **`sts:AssumeRole`**. ❓ Đề nói "DynamoDB không có resource-based policy": 📌 DynamoDB nay **đã hỗ trợ resource-based policy** cho bảng và stream; khi đáp án chỉ có AssumeRole là hợp lệ thì vẫn chọn AssumeRole (an toàn nhất). ⚠️ Static access key là rủi ro; `iam:PassRole` không dùng cho mô hình này.

🎯 **Tích hợp DynamoDB với CSDL quan hệ có sẵn** → **DynamoDB Streams + Lambda** (write-through về SQL). ⚠️ DAX chỉ cache cho DynamoDB; Kinesis một chiều; ElastiCache không tích hợp hai bên.

---

## 21. AWS Lambda — vận hành

🎯 **Lambda không có log** (logging bật mặc định, không ghi file local) → execution role **thiếu `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`** → gắn **`AWSLambdaBasicExecutionRole`**.

🎯 **Lambda chậm với tác vụ nặng CPU** → **tăng MEMORY** (CPU tỷ lệ theo memory; **1,769 MB ≈ 1 vCPU**; dải 128–10,240 MB). ⚠️ Không có CPU option/hyperthreading; layer chỉ chia sẻ code.

🎯 **Execution role** = quyền **từ Lambda đi ra**; **resource-based policy** = quyền **đi vào Lambda** (EventBridge cần `events.amazonaws.com` + `lambda:InvokeFunction`).

🎯 **Dừng ngay vòng lặp đệ quy** (S3 PutObject → Lambda → ghi lại chính bucket) → **Reserved Concurrency = 0** (throttle mọi invocation). Phòng ngừa: prefix/suffix filter hoặc ghi sang bucket khác. ⚠️ Xóa object mất dữ liệu; bucket policy deny thì Lambda vẫn chạy rồi lỗi; publish version mới không dừng invocation đang chạy.

🎯 **Lambda không kết nối được DB ở private subnet** → **cấu hình Lambda vào VPC** (subnet + SG), dùng private endpoint của DB. ⚠️ IAM role chỉ lo quyền, không lo đường mạng; PrivateLink là để tới AWS service/SaaS.

⚠️ Lambda không có tùy chọn "Multi-AZ" (tự chạy đa AZ).

---

## 22. Amazon SQS & SNS

🎯 **SQS không có priority** → **hai queue riêng** (cao/thấp), app poll queue ưu tiên cao trước. ⚠️ Scale ASG nhanh hơn chỉ tăng throughput; SNS fan-out không tạo thứ tự ưu tiên.

🎯 **Standard → FIFO: không đổi tại chỗ** → **tạo queue FIFO mới** (tên `.fifo`). Bắt buộc **Message Group ID** (thứ tự trong nhóm); chống trùng bằng **Message Deduplication ID** hoặc **content-based deduplication** (SHA-256 nội dung, hợp khi body vốn duy nhất). ⚠️ `DelaySeconds` chỉ trì hoãn hiển thị.

🎯 **SNS** gửi trực tiếp tới HTTP/HTTPS, email, SMS, Lambda, SQS → mẫu "alarm → hệ thống ngoài" không cần Lambda. ⚠️ SNS không transform payload (dùng EventBridge Input Transformer).

---

## 23. S3 — versioning, lifecycle, replication, storage class, hiệu năng

🎯 **Versioning:** ghi đè tạo version mới; DELETE thường chỉ chèn **delete marker** (object thật không bị xóa). Delete marker có **key + version ID**, **không có dữ liệu, không có ACL**, **GET thường không lấy được** (chỉ thấy qua `GET Bucket versions`).

🎯 **Bucket "trông trống" nhưng báo "bucket is not empty"** → còn **object version + delete marker**. ⚠️ S3 có **strong read-after-write consistency** (không phải "chờ 2 phút"); bị chặn bởi policy thì lỗi là Access Denied.

🎯 **Lifecycle:** **transition** (Standard-IA, One Zone-IA, Intelligent-Tiering, Glacier Instant/Flexible, Deep Archive) và **expiration** (object, noncurrent version, delete marker hết hạn, **AbortIncompleteMultipartUpload**).

| Storage class | Truy cập | Vị trí | Ghi chú |
| --- | --- | --- | --- |
| S3 Standard | ms | Nhiều AZ | Truy cập thường xuyên |
| **S3 Standard-IA** | **ms** | **Nhiều AZ** | Ít truy cập nhưng cần HA + ms |
| S3 One Zone-IA | ms | **1 AZ** | Không đạt "highly available" |
| Glacier Instant Retrieval | ms | Nhiều AZ | Archive cần truy cập tức thì |
| **Glacier Flexible Retrieval** | Expedited 1–5 phút · Standard 3–5 giờ · Bulk 5–12 giờ | Nhiều AZ | Đạt "lấy lại < 5 giờ" |
| Glacier Deep Archive | 12–48 giờ | Nhiều AZ | Rẻ nhất, không đạt "< 5 giờ" |

🎯 **90 ngày đầu ít truy cập nhưng cần HA + ms, sau đó lưu rẻ, lấy lại < 5 giờ** → **Standard-IA → (lifecycle) → Glacier Flexible Retrieval**.

🎯 **Replication (CRR/SRR):** copy bất đồng bộ, **versioning bật ở cả nguồn và đích**. Cross-account: replication rule ở nguồn + bucket policy ở đích. **Replica bị Access Denied ở account đích** → bật **"Change object ownership to destination bucket owner"**. ⚠️ Replication không chống xóa nhầm, không tăng tốc upload; storage class/RTC/prefix không liên quan quyền.

| Nhu cầu replication | Giải pháp |
| --- | --- |
| Cảnh báo **ngay** khi object replicate chậm/thất bại | **S3 Replication Time Control (RTC)**: SLA 99.99% object trong 15 phút, kèm **replication metrics + event** (SQS/SNS/Lambda) |
| **Danh sách** object thất bại, hằng ngày | **S3 Inventory** |

⚠️ CloudWatch metric cho replication chỉ có khi bật RTC (hoặc bật replication metrics).

🎯 **S3 Inventory:** file CSV/ORC/Parquet liệt kê object + metadata, **hằng ngày hoặc hằng tuần**; audit **trạng thái replication và mã hóa** (trường `encryption status`); thay cho `List` API trên bucket lớn. ⚠️ `list-objects` không trả trạng thái mã hóa; Storage Lens là dashboard usage, không phải danh sách object.

🎯 **Incomplete multipart upload:** **xem** dung lượng bằng **S3 Storage Lens** (metric incomplete MPU bytes), **xóa** bằng **lifecycle `AbortIncompleteMultipartUpload`** (`DaysAfterInitiation`). ⚠️ Storage Lens không xóa được gì.

🎯 **Hiệu năng S3:** **3,500 PUT/COPY/POST/DELETE và 5,500 GET/HEAD mỗi giây cho mỗi prefix**. **Upload 500k file lớn, EC2 và bucket cùng Region, throughput thấp** → **chia nhiều prefix + ghi song song**. ⚠️ Transfer Acceleration chỉ có ích khi client ở xa Region; PrivateLink không tăng throughput; tắt encryption luôn sai.

🎯 **Upload nhiều GB từ khắp thế giới** → **Transfer Acceleration** (qua edge location) **+ Multipart Upload** (song song, part lỗi chỉ gửi lại part đó; nên dùng từ 100 MB). Bật Transfer Acceleration thì app phải gọi endpoint **`<bucket>.s3-accelerate.amazonaws.com`**. ⚠️ CloudFront tối ưu download; Global Accelerator không nhận S3.

📌 **S3 Select** (lấy một phần dữ liệu trong một object bằng SQL) **không nhận khách mới** → dùng Athena hoặc lọc phía client.

---

## 24. S3 — bảo mật & mã hóa

### Object Lock

🎯 Object Lock (WORM) **chỉ trên bucket bật versioning**; retention và legal hold áp cho **từng object version**. Đặt explicit trên version bằng **"Retain Until Date"**; bucket default dùng **duration** (ngày/năm). **Explicit ghi đè default**; các version khác nhau có retention khác nhau (PUT version mới 60 ngày không ảnh hưởng version cũ 30 ngày).

| Mode | Ai bypass được? | Dùng khi |
| --- | --- | --- |
| **Governance** | User có quyền đặc biệt | Bảo vệ vừa phải, có ngoại lệ |
| **Compliance** | **Không ai, kể cả root** | WORM pháp lý: "không xóa được trong ít nhất 3 tháng" |

🎯 **Legal Hold** khóa vô thời hạn, không tự hết hạn (gỡ bằng quyền **`s3:PutObjectLegalHold`**) → không hợp với "khóa đúng X tháng".

🎯 **Xóa object đang bị Governance** → cần **cả** quyền **`s3:BypassGovernanceRetention`** **và** header **`x-amz-bypass-governance-retention:true`** trong request. ⚠️ `s3:BypassLegalRetention` và `x-amz-bypass-legal-retention` là **bịa**; Compliance mode không có cách bypass.

🎯 **Giữ báo cáo đúng 5 năm rồi tự xóa** → **Object Lock "Retain Until Date" = ngày tạo + 5 năm** + **lifecycle expire sau 5 năm**.

📌 **Glacier Vault Lock** (WORM ở cấp vault, policy bất biến sau khi lock; quy trình initiate → 24 giờ test → complete) dựa trên **Glacier vault**, nay **không nhận khách mới** → khách mới dùng **S3 Object Lock** (có thể trên class Glacier). Đề cũ chọn Vault Lock cho "WORM, immutable, chứng minh compliance".

| Cơ chế chống sửa/xóa | Bảo vệ | Hạn chế |
| --- | --- | --- |
| Glacier Vault Lock | WORM cấp vault, policy bất biến | Truy xuất chậm; 📌 vault không nhận khách mới |
| **S3 Object Lock** | WORM cấp object version | Cần versioning |
| Versioning + MFA Delete | Khôi phục sau xóa/ghi đè | Vẫn cho ghi đè tạo version mới → không đủ chặt cho "không tamper" |

### MFA Delete

🎯 MFA Delete chỉ yêu cầu MFA cho **xóa vĩnh viễn một version** và **đổi trạng thái versioning** (kể cả suspend). Không cần MFA khi bật versioning lần đầu, upload, liệt kê, đọc. **Chỉ root (bucket owner) bật được, chỉ qua CLI** (header `x-amz-mfa`, HTTPS).

### Mã hóa

| Tùy chọn | Ai giữ key | Ai mã hóa | Audit dùng key? |
| --- | --- | --- | --- |
| **SSE-S3** (`AES256`) | AWS | AWS | ❌ |
| **SSE-KMS** (`aws:kms`) | KMS (customer hoặc AWS managed) | AWS | ✅ biết key dùng khi nào, bởi ai |
| **SSE-C** | **Khách gửi key mỗi request** | S3 | ❌ |
| Client-side | Khách | **Khách tự mã hóa trước khi upload** | — |

🎯 **Key nằm ở app on-prem của khách nhưng muốn S3 mã hóa hộ** → **SSE-C**. **Cần audit trail ai dùng key khi nào** → **SSE-KMS**. Header đúng: `x-amz-server-side-encryption: aws:kms` hoặc `AES256` (giá trị `SSE-KMS`/`SSE-S3` là sai).

📌 **SSE-C:** từ **04/2026** bucket general purpose mới **mặc định chặn SSE-C**; muốn dùng phải bật lại bằng `PutBucketEncryption`. 📌 Từ 01/2023 mọi object mới **tự động mã hóa SSE-S3**.

🎯 **Ép mọi object upload phải mã hóa** → **Default Encryption** trên bucket (SSE-S3 hoặc SSE-KMS). Bucket policy:

| Condition | Ý nghĩa thật |
| --- | --- |
| `Deny` + `"Bool": {"aws:SecureTransport": false}` | **Chặn HTTP** (ép HTTPS in-transit), không liên quan at-rest |
| `Deny` + `"Bool": {"aws:SecureTransport": true}` | Chặn HTTPS (bẫy ngược) |
| `Deny` + `"Null": {"s3:x-amz-server-side-encryption": true}` | Chặn PUT không khai báo header mã hóa |

🎯 **Kiểm tra trạng thái mã hóa của mọi object** → **S3 Inventory** kèm trường encryption status.

🎯 **S3 Bucket Keys** cho SSE-KMS giảm tới **99%** chi phí request KMS.

### Quyền truy cập

🎯 **Chia sẻ file cho khách / truy cập tạm cho người không có AWS account** → **Pre-signed URL** (credentials người tạo + bucket + key + HTTP method + thời hạn; ai có URL thì truy cập được). ⚠️ IAM user cho từng khách, bucket policy theo IP, website + ACL đều sai; mTLS chỉ có ở API Gateway; S3 không đặt mật khẩu cho file.

🎯 **Hàng trăm ứng dụng dùng chung một bucket, mỗi app một quyền** → **S3 Access Points** (mỗi access point có hostname + policy riêng, giới hạn được theo prefix, **VPC-only**, Block Public Access riêng; không tính phí). ⚠️ VPC endpoint, Transfer Acceleration, Batch Operations không dùng để phân quyền.

🎯 **Cấm public toàn account** → **S3 Block Public Access ở cấp ACCOUNT** (preventive, áp mọi bucket hiện có và tương lai, ghi đè bucket policy lẫn ACL). ⚠️ EventBridge chỉ phản ứng sau; Inspector không quản cấu hình S3; S3 Object Lambda dùng biến đổi dữ liệu khi đọc.

🎯 **Static website trả 403** → **tắt Block Public Access** + bucket policy cho **`s3:GetObject`** với `Principal: "*"`. ⚠️ `s3:PutObject` là quyền ghi; sửa ACL từng file không phải best practice.

🎯 **Bucket policy read-only cho mọi người:** `Effect` chỉ nhận **Allow/Deny**; action đúng là **`s3:GetObject`** + **`s3:ListBucket`** (⚠️ `s3:ReadObject`, `s3:GetBucket` là bịa); cần **cả hai ARN** `arn:aws:s3:::mybucket` (ListBucket) và `arn:aws:s3:::mybucket/*` (GetObject).

🎯 **Root user bị bucket policy từ chối** nếu policy không đưa root vào `Principal`; root vẫn sửa được bucket policy để thêm `arn:aws:iam::<account>:root`.

🎯 **Object do account khác upload:** theo đề, object **thuộc account upload**, bucket owner không có quyền và không delegate cross-account được; account upload phải cấp quyền cho bucket owner trước. Export DynamoDB từ account A sang bucket ở account B → cấp **`PutObjectAcl`** cho object đã export.

📌 Từ 04/2023 bucket mới mặc định **Object Ownership = Bucket owner enforced** (ACL bị tắt) → **bucket owner tự động sở hữu mọi object**, kể cả do account khác upload. Các kịch bản "object thuộc account upload" chỉ đúng với bucket còn bật ACL.

---

## 25. EBS & Instance Store

| Loại | Nhóm | Boot volume? | Ghi chú |
| --- | --- | --- | --- |
| **gp2 / gp3** | SSD | ✅ | Đa dụng |
| **io1 / io2** | SSD | ✅ | io1: tối đa **50 IOPS/GiB** (100 GiB → 5,000 IOPS); Multi-Attach |
| **st1** | HDD | ❌ | Streaming lớn, đo MiB/s |
| **sc1** | HDD | ❌ | Lạnh, rẻ nhất |
| Instance store | — | ✅ | Ephemeral |

| | **gp2** | **gp3** |
| --- | --- | --- |
| IOPS | **3 IOPS/GiB** (100–16,000), volume nhỏ dựa vào **burst credit** | **Baseline 3,000 IOPS + 125 MiB/s bất kể dung lượng**, provision thêm được |
| Burst | `BurstBalance` = 0 → chỉ còn baseline | Không phụ thuộc burst |
| Giá | Cao hơn | Rẻ hơn ~20%/GB |

🎯 **`BurstBalance` = 0 trên gp2 500 GB, cần 3,000 IOPS, rẻ nhất** → **đổi sang gp3**. Cần **10,000 IOPS rẻ nhất** → **gp3 provision 10,000 IOPS** (io2 đắt, st1/sc1 chỉ vài trăm IOPS). **Chính sách cấm đổi volume type** → **tăng dung lượng gp2** (3 IOPS/GiB). ⚠️ Thêm ENI, placement group, "Transfer Acceleration cho EBS" (chỉ có ở S3) đều sai.

📌 Đề ghi gp3 "provision tới 16,000 IOPS": nay gp3 tối đa **80,000 IOPS / 2,000 MiB/s / 64 TiB**.

🎯 **Best practice:** volume **riêng cho OS và cho data**. **Snapshot chỉ chụp dữ liệu đã ghi xuống volume** (không có cache app/OS) → nhất quán: detach sạch hoặc shutdown (root) rồi snapshot. Volume non-root mặc định **`DeleteOnTermination = false`** → dữ liệu còn sau terminate (nhớ tự xóa để khỏi tốn phí). **Instance store mất dữ liệu khi stop, hibernate, terminate.** EBS encryption **hỗ trợ boot volume**. Snapshot nằm trong S3 nhưng **không truy cập qua S3 API**, chỉ qua EC2 API.

🎯 **Tăng size volume xong phải extend file system** (làm được khi volume ở trạng thái `optimizing`, không cần detach/reboot): Linux `growpart` + `resize2fs` (ext4) / `xfs_growfs` (XFS); Windows Disk Management hoặc PowerShell.

🎯 **Volume kẹt `attaching`** → device name đang bị driver dùng (remap tên, chưa release sau force detach) → **force detach rồi attach với device name khác**; vẫn không được → reboot hoặc stop/start (mất instance store).

🎯 **Volume ở trạng thái `error`** → phần cứng nền hỏng, dữ liệu không khôi phục, AWS **không tính phí**, có thông báo trên Health Dashboard → **khôi phục từ backup (snapshot) sang volume MỚI**. ⚠️ Restart instance không sửa được; DLM không "phục hồi" volume.

🎯 **Volume tạo từ snapshot chậm lần đọc đầu** (lazy loading) → **initialize/pre-warm** (đọc qua mọi block) hoặc **Fast Snapshot Restore**. ⚠️ Read-ahead 1 MiB chỉ cho HDD (st1/sc1); RAID 0 và EBS-optimized instance không giải quyết lazy loading.

🎯 **Fast Snapshot Restore:** volume đạt full performance ngay; **bật ở AZ ĐÍCH** (FSR theo AZ). Snapshot là tài nguyên **cấp Region** (không cần copy giữa AZ, chỉ copy khi sang Region khác). **Provisioned IOPS là thuộc tính của volume**, không phải snapshot.

🎯 **Multi-Attach:** chỉ io1/io2, tối đa **16 instance Nitro cùng 1 AZ**, cần **clustered file system** (GFS2/OCFS2).

---

## 26. EFS & FSx

🎯 **EFS Access Points:** điểm vào riêng cho từng ứng dụng, **ép user identity (POSIX)** và **root directory riêng** → kết hợp IAM policy cho "**thư mục riêng cho từng client, chỉ IAM role tương ứng truy cập**". ⚠️ Mount target chỉ là IP endpoint NFS; gateway endpoint chỉ S3/DynamoDB; EBS Multi-Attach không phải file system.

🎯 **EFS mount helper** (`amazon-efs-utils`): mount trên EC2 Linux, **IAM authorization**, qua access point, từ Linux on-prem, **auto-mount khi reboot** (`/etc/fstab`). ⚠️ **EFS không mount được từ Windows**; không xác thực bằng Cognito.

🎯 **Ép mã hóa at-rest cho EFS:** mã hóa **không bật mặc định khi tạo bằng CLI/SDK/API** (Console tick sẵn) → IAM policy với condition **`elasticfilesystem:Encrypted`** trên `CreateFileSystem`, hoặc **SCP** cho cả Organization. ⚠️ AWS Config chỉ phát hiện, không ngăn.

🎯 **Chia sẻ file giữa EC2 trong ASG** → **tạo EFS trước** → **launch template version mới** có user data mount EFS → cập nhật ASG. ⚠️ Launch template **không tạo được EFS**; Multi-Attach chỉ 1 AZ; cron đồng bộ EBS và file server tự dựng đều kém.

| Dịch vụ | Dùng khi |
| --- | --- |
| **FSx for Windows File Server** | **Windows NTFS + SMB + Active Directory**; truy cập từ Windows và Linux qua SMB; **Multi-AZ** cho nhiều server truy cập đồng thời |
| **FSx for Lustre** | HPC, ML, EDA, media processing trên Linux, dữ liệu vào/ra S3 |
| **EFS** | NFSv4 cho Linux, đa AZ |

🎯 **Nhiều server Windows truy cập file đồng thời, ít vận hành** → **FSx for Windows Multi-AZ** (AWS tự replicate). ⚠️ Hai FSx + tự cấu hình DFS replication là gấp đôi chi phí; EFS không hỗ trợ Windows.

---

## 27. Storage Gateway & chuyển dữ liệu

| Loại | Giao thức | Dữ liệu chính ở đâu | Use case |
| --- | --- | --- | --- |
| **S3 File Gateway** | **NFS / SMB** | S3 + cache local | Thay NFS/SMB on-prem, bucket xuất hiện như mount point |
| **Volume Gateway – Stored** | iSCSI | **Toàn bộ dataset on-prem**, snapshot async lên AWS | App cần block storage + POSIX và **toàn bộ dữ liệu ở on-prem** |
| **Volume Gateway – Cached** | iSCSI | **Dữ liệu chính ở AWS**, cache local | Tiết kiệm dung lượng on-prem |
| **Tape Gateway** | iSCSI VTL | S3 / Glacier | Thay băng từ |

🎯 Storage Gateway chỉ gửi **dữ liệu thay đổi**, có **nén**. In-transit = **SSL/TLS** (không phải IPsec); at-rest mặc định **SSE-S3**, chuyển **SSE-KMS** qua API cho file share, volume, **virtual tape**.

🎯 **Tape Gateway báo "Not Enough Space"** → bật **tape auto-create** (số tape tối thiểu, barcode prefix, tape pool).

| Tình huống | Xử lý |
| --- | --- |
| Cache disk thừa, muốn giảm chi phí | **Không thu nhỏ/xóa được cache disk** của gateway đang chạy → **tạo gateway mới** rồi migrate |
| Giảm upload buffer | Được: **shutdown gateway**, gỡ disk upload buffer, cấp disk nhỏ hơn |
| Gateway VM hỏng | Stored volume: khôi phục từ **EBS snapshot** gần nhất; cached volume: **recovery snapshot**; tape: từ recovery point sang gateway mới; file system corrupt → `fsck` |
| Khôi phục từ hypervisor snapshot / EC2 AMI | **Không hỗ trợ** → kích hoạt gateway mới |
| Volume `IRRECOVERABLE` | Không dùng được; stored volume: tạo volume mới từ chính disk đó (giữ dữ liệu) → xóa snapshot job đang chờ → xóa volume hỏng |
| File Gateway ↔ S3 riêng tư, tiết kiệm | **VPC endpoint cho S3** + tạo File Gateway dùng endpoint đó |

📌 **FSx File Gateway** đóng với khách mới; **Storage Gateway Hardware Appliance** ngừng bán cho khách mới từ 12/05/2025.

⚠️ Loại trừ: Glacier (archive chậm) · EFS (NFS cho EC2, không có backend S3) · EBS (gắn EC2) · S3 thuần (object API).

🎯 **PB-scale, data center thiếu băng thông** (đề cũ) → **Snowball Edge Storage Optimized → S3 → lifecycle → Glacier** (không copy thẳng vào Glacier). 📌 AWS **không nhận khách mới đặt thiết bị Snow Family** → thay bằng **DataSync** (online), **AWS Data Transfer Terminal** (mang ổ tới cơ sở AWS) hoặc partner.

| Tình huống | Giải pháp |
| --- | --- |
| Kết nối riêng dài hạn, băng thông cao | Direct Connect (lắp hơn 1 tháng) |
| Cần ngay, băng thông thấp–trung bình | Site-to-Site VPN |
| Migrate database, nguồn vẫn chạy | AWS DMS |

---

## 28. AWS Backup, DLM & Disaster Recovery

🎯 **AWS Backup:** backup **tập trung** cho EBS, EC2, RDS, Aurora, DynamoDB, EFS, FSx, Storage Gateway...; tích hợp **Organizations** (backup policy đa account), **cross-Region copy**. Mẫu: **tag theo môi trường** → backup plan riêng cho prod/non-prod → gán resource bằng tag.

🎯 **Nhiều mức retention cho EC2 + RDS** (daily 6 ngày, weekly 4 tuần, monthly 11 tháng, yearly 7 năm) → **AWS Backup**, mỗi chu kỳ là một **backup rule**. ⚠️ DLM chỉ EBS snapshot/AMI (không RDS, không EFS); RDS automated backup tối đa 35 ngày; tự viết Lambda nhiều công nhất.

🎯 **Khôi phục vài file trong EFS, rẻ nhất** → **AWS Backup + partial restore** (từng file). ⚠️ EFS thứ hai + DataSync tốn gấp đôi; Glacier truy xuất chậm.

🎯 **Backup EBS sang Region khác và ép mã hóa** → **DLM policy có cross-Region copy + mã hóa bản copy bằng KMS key** (bản copy luôn mã hóa kể cả volume nguồn chưa mã hóa). ⚠️ Không copy được EBS snapshot vào bucket S3 của bạn.

⚠️ ❓ Đề nói "snapshot do DLM tạo gắn tag `aws:dlm:lifecycle-policy-id` nên user có `ec2:*` thường **không xóa được**". Docs xác nhận DLM gắn system tag `aws:dlm:...` cho snapshot, nhưng mình **chưa tìm thấy** mô tả cơ chế chặn xóa này. Khi thi chọn theo đề; thực tế muốn chống xóa snapshot thì dùng **Recycle Bin** hoặc **AWS Backup Vault Lock**.

| Chiến lược DR | Region phụ | RTO / RPO | Chi phí |
| --- | --- | --- | --- |
| Backup & Restore | Chỉ có backup | Giờ | Thấp nhất |
| Pilot Light | Lõi + replication liên tục, chưa chạy full | RPO thấp, RTO có thể > 10 phút | Thấp |
| Warm Standby | Bản thu nhỏ đang chạy | RTO thấp; replication theo lịch thì RPO ≠ 0 | Trung bình |
| **Active-Active (multi-site)** | Chạy production ở cả hai Region, replication real-time | **RTO gần tức thì, RPO = 0** | Cao nhất |

🎯 **RTO < 10 phút và RPO = 0** → **Active-Active + replication real-time + Route 53 health check**. ⚠️ Cross-Region read replica + promote tay là async → không RPO = 0.

🎯 **RTO = RPO = 15 phút, rẻ nhất** (EC2 + ALB + ASG + Aurora) → **Aurora Global Database** (RPO thường < 1 giây) + dựng sẵn **ALB + ASG ở Region DR với min = max = desired = 1**. ⚠️ Export backup Aurora là giờ; full capacity là đắt; dựng bằng CloudFormation lúc failover thì quá 15 phút.

---

## 29. Amazon EKS

🎯 **EKS quá tải do nhiều request** → **Horizontal Pod Autoscaler (HPA)** scale **số pod** theo CPU (built-in Kubernetes). **VPA** chỉ chỉnh CPU/memory request của pod. ⚠️ Tạo cluster thứ hai hay chuyển sang EC2 là overhead lớn.

🎯 **`kubectl` không kết nối được EKS** → thiếu **kubeconfig** → `aws eks update-kubeconfig --region <REGION> --name <CLUSTER>`. ⚠️ kube-proxy là add-on chạy trên node; Fargate profile chọn pod chạy Fargate; `eks-connector.yaml` để đưa cluster bên ngoài vào console.

🎯 **Anomaly + khuyến nghị cho EKS** → **Container Insights + DevOps Guru** (📌 DevOps Guru đóng với khách mới từ 29/10/2026, xem §5).

---

## 30. IAM — policy, đánh giá quyền, MFA

| Loại policy | Gắn vào | Ghi chú |
| --- | --- | --- |
| Identity-based | User / group / role | Không có `Principal` |
| Resource-based | Resource (S3, SQS, SNS, KMS key, Lambda...) | **Bắt buộc `Principal`** |
| **Trust policy** | IAM Role | **Resource-based policy duy nhất của IAM**; quy định ai được assume role |
| **SCP** | Root / OU / account | **Giới hạn quyền tối đa**, áp cả root của member account |
| ACL | S3, WAF, VPC | Cho principal **account khác**, không dùng trong cùng account |
| Permissions boundary | User / role | Trần quyền; quyền thực = giao của policy và boundary |

🎯 **Explicit Deny ở bất kỳ policy nào cũng thắng mọi Allow**; thứ tự gắn policy không quan trọng. Không có Deny → cần ít nhất một Allow. **Cross-account cần CẢ identity-based ở account A LẪN resource-based ở account B**; trong cùng account resource-based là đủ.

🎯 **AssumeRole** → principal **bỏ quyền gốc** khi assume. **Resource-based policy** → **giữ quyền gốc** → hợp cho **copy dữ liệu giữa hai account**.

🎯 **Cross-account bằng role:** role tạo ở account chứa **resource** (vd Production), trusted entity = account Development; muốn chỉ Developers dùng → **Deny tường minh `sts:AssumeRole`** cho group Testers.

🎯 **Bắt MFA cho hành động nhạy cảm** → condition `"Bool": {"aws:MultiFactorAuthPresent": "true"}`. ⚠️ `"Effect": "NotAction"` sai cú pháp (`Effect` chỉ Allow/Deny; `NotAction` cùng cấp với `Action`); `aws:PrincipalServiceName` là cho service-to-service.

🎯 **Đọc policy:** `service:*` = mọi action của service (vd `lambda:*` có `InvokeFunction`); `Describe*`/`List*` **chỉ đọc** (`storagegateway:Describe*` không tạo gateway; `sqs:List*` không xóa queue). Quyền service khác không suy ra quyền IAM (tạo role cần `iam:CreateRole`). `Deny` + `NotAction: ["ec2:*", "s3:GetObject"]` → **chặn mọi thứ trừ** danh sách đó. Nhớ đọc `Condition` (vd `ec2:Region`).

🎯 **`iam:PassRole`:** cần để gán role cho dịch vụ. Đọc theo từng statement: `ec2:*` trên `*` + `iam:PassRole` trên `arn:aws:iam:::role/RDS-*` → **được gán role có tiền tố `RDS-` cho EC2**.

🎯 **Ép MFA cho AWS CLI** → long-term access key không mang ngữ cảnh MFA → `aws sts get-session-token --serial-number <MFA-ARN> --token-code <code>` rồi dùng **credential tạm** đó. ⚠️ Đăng nhập Console có MFA không ảnh hưởng CLI.

🎯 **Mất thiết bị MFA của root** → "Trouble signing in?" → **xác minh email + số điện thoại** → gỡ MFA cũ, đăng ký mới. ⚠️ IAM user (kể cả admin) không reset được MFA của root; resync chỉ khi thiết bị còn nhưng lệch giờ. **Đổi Support plan chỉ root làm được.**

🎯 **Sinh policy least-privilege từ hoạt động thật** → **CloudTrail + IAM Access Analyzer policy generation** (tới 90 ngày). ⚠️ Chưa bật CloudTrail thì không có dữ liệu; Security Hub/EventBridge không sinh policy.

🎯 **IAM Group:** user thuộc được **nhiều group**; group **không lồng nhau**, **không có credential**, **không gán Role**.

🎯 **Best practice:** MFA cho user đặc quyền · CloudTrail ghi mọi hành động IAM · least privilege · **IAM Role cho EC2** (credential tạm qua metadata). ⚠️ Không dùng chung tài khoản, không nhúng access key vào app, không thay IAM bằng kiểm soát theo IP.

---

## 31. Organizations, Control Tower, Identity Center & Service Catalog

🎯 **AWS Organizations:** billing hợp nhất, SCP, chia sẻ resource, miễn phí. OU có đúng 1 parent, account thuộc đúng 1 OU; policy gắn ở node chảy xuống mọi node con. **Nhiều phòng ban dùng chung 1 account** → tách account theo phòng ban, gom vào OU. ⚠️ IAM group/role không tách biệt ở mức account.

🎯 **Chặn EC2 ở một Region, chặn root user, chặn xóa CloudTrail** cho account hiện có và tương lai → **Control Tower + SCP** (Region deny control là preventive). **SCP là cơ chế duy nhất hạn chế được root user của member account.** ⚠️ Config + remediation là detective; permissions boundary không áp cho root; Security Hub chỉ hiển thị finding; IAM policy từng account không tập trung.

📌 Control Tower "guardrails" nay gọi là **controls** (preventive / detective / proactive).

| Identity source cho IAM Identity Center | Khi nào |
| --- | --- |
| **AD Connector** | **AD on-premises, ít vận hành nhất**: proxy chuyển xác thực về AD tại chỗ, không lưu user trên cloud |
| AWS Managed Microsoft AD | Cần AD thật trên AWS, có trust với AD on-prem |
| Simple AD | Samba 4, **không hỗ trợ trust** |
| Built-in directory | Không tích hợp AD, phải đồng bộ thủ công |

🎯 **Federation SAML 2.0 với Identity Center:** trao đổi **metadata của Identity Center** (SP entity ID, ACS URL) cho IdP và **metadata của IdP** (entity ID, chứng chỉ ký X.509) cho Identity Center. ⚠️ Không cần IP tĩnh của IdP; Identity Center tự tạo role khi gán **permission set**.

⚠️ ❓ Đề: "account mới bị Access Denied qua Identity Center → gỡ account rồi thêm lại vào Organization". Đây là đáp án theo đề; docs không mô tả đúng kịch bản này.

🎯 **Amazon Cognito:** **User Pool** = thư mục user (đăng ký, đăng nhập, MFA, social/SAML IdP); **Identity Pool** = đổi token lấy **credential AWS tạm** → app mobile upload thẳng S3. ⚠️ Tạo IAM user cho từng người dùng hoặc nhúng access key vào app là sai.

🎯 **Chỉ được dùng cấu hình EC2 đã duyệt** → **Service Catalog product + Launch Constraint** (Service Catalog dùng IAM role riêng để tạo resource, user **không có quyền EC2 trực tiếp**). ⚠️ IAM cho phép tạo EC2 + phát template thì user vẫn tự chọn cấu hình; launch configuration là của ASG.

🎯 **Chia sẻ portfolio Service Catalog** (account-to-account hoặc qua Organizations) tạo **tham chiếu luôn đồng bộ** với bản gốc. Account nhận: được thêm product vào **local portfolio** của mình và cấp quyền cho end user; **không** thêm product vào portfolio import, không sửa constraint/provisioning artifact, không xóa portfolio gốc. ⚠️ Bản sao hoặc StackSets không đồng bộ với bản gốc.

---

## 32. KMS, Secrets Manager & ACM

### KMS

🎯 **Xóa KMS key:** phải **schedule deletion**, waiting period **7–30 ngày (mặc định 30)**; trong trạng thái `Pending deletion` **cancel được**. Hết hạn → dữ liệu không giải mã được. ⚠️ AWS Support và root không khôi phục được; **không export được** key KMS.

🎯 **Chia sẻ key cross-account:** **key policy** ở account sở hữu cho phép account ngoài + **IAM policy** ở account ngoài ủy quyền cho user/role. ⚠️ AWS owned key không xem/dùng/audit được; service-linked role của KMS không dùng chia sẻ key.

| Loại key | Rotation | Kiểm soát |
| --- | --- | --- |
| **AWS managed key** | **Tự động mỗi năm** | Không sửa key policy, không share cross-account |
| **Customer managed key** | Bật automatic rotation, **rotation period tùy chỉnh (mặc định 365 ngày)** + **on-demand rotation** | Toàn quyền: key policy, bật/tắt, audit, share |

📌 Đề ghi "**AWS managed key xoay mỗi 3 năm**" → **sai so với docs hiện hành: AWS managed key xoay hằng năm**. Đề ghi "customer managed key xoay mỗi 1 năm" → nay **chỉnh được rotation period**.

🎯 **Mã hóa at-rest + xoay khóa ít nhất hằng năm, ít công nhất** → **SSE-KMS với customer managed key + automatic rotation**. Rotation tự động chỉ cho key sinh trong KMS. 📌 Key có **imported key material** không có rotation tự động nhưng **nay rotate on-demand được**. **SSE-C** và **client-side** không có rotation tự động.

### Secrets Manager

🎯 **Built-in rotation** (Lambda rotation template do AWS cung cấp) cho **RDS, DocumentDB, Redshift** → **xoay mật khẩu master RDS tự động, ít công nhất**. ⚠️ EMR/CloudFormation/ECS chỉ đọc secret; Parameter Store SecureString không auto-rotate; IAM DB authentication không xoay mật khẩu master.

🎯 **Xóa rồi tạo lại secret cùng tên ngay** → CLI `DeleteSecret` với **`ForceDeleteWithoutRecovery`** (không khôi phục được). 📌 Đề ghi recovery window "mặc định 7 ngày": docs ghi **mặc định 30 ngày** (cấu hình 7–30 ngày).

### ACM

🎯 **Cert CloudFront tự gia hạn** → request cert **từ ACM** + **DNS validation**; cert cho CloudFront phải ở **`us-east-1`**; cert cho ALB/CLB phải **cùng Region với load balancer**. ⚠️ Self-signed cert, "automatic certificate rotation" trong CloudFront, Secrets Manager phát cert đều sai.

🎯 **Multi-account:** cert phải **request trong chính account chứa resource** (ALB, API Gateway); hosted zone ở account khác → tạo **CNAME validation record** ở account quản lý DNS. KMS không liên quan.

🎯 **Cảnh báo cert sắp hết hạn** → **EventBridge** rule source **`aws.acm`**, event **ACM Certificate Approaching Expiration** (trường `DaysToExpiry`) → **SNS**. ⚠️ CloudWatch không có sẵn metric `DaysToExpiry`; "SMS identity" không phải target. 📌 ACM gửi event hằng ngày bắt đầu **45 ngày** trước hạn cho cert private/imported và **30 ngày** cho cert public (chỉnh được bằng `PutAccountConfiguration`).

🎯 ACM chỉ cấp cert **SSL/TLS** (không dùng cho mã hóa email); không cấp cho domain của Amazon (`amazonaws.com`, `cloudfront.net`...). Private CA cert dùng cho tài nguyên nội bộ.

🎯 **End-to-end encryption ALB → EC2** (theo đề): cài **cert bên thứ ba lên EC2**, import vào ACM để gắn cho ALB; self-signed chỉ cho test. 📌 Đề nói "**không cài ACM cert lên EC2, không tải được private key**": từ 2025 ACM có **exportable public certificate**, export được cert + chain + private key (mã hóa bằng passphrase) để dùng ở bất kỳ đâu, kể cả EC2. Cert ACM thông thường (không đánh dấu exportable) vẫn không export được.

---

## 33. Dịch vụ bảo mật, Trusted Advisor & Shared Responsibility

| Dịch vụ | Làm gì | Không làm gì |
| --- | --- | --- |
| **Amazon Inspector** | Quét lỗ hổng EC2, ECR image, Lambda; báo cáo chứng minh **không còn lỗ hổng chưa vá** | Không vá, không phân tích log |
| **GuardDuty** | Threat detection từ CloudTrail + VPC Flow Logs + DNS logs | Không quét lỗ hổng |
| **Shield** | Chống DDoS | Không patch, không compliance port |
| **WAF** | Firewall L7: allow/block/count theo IP, header, body, URI, SQLi, XSS | Không phát hiện SG mở port |
| **AWS Artifact** | Tải tài liệu compliance/agreement của AWS | Không enforce, không vá |
| **Firewall Manager** | Quản lý tập trung rule WAF, Shield Advanced, SG, Network Firewall trên nhiều account | Tự nó không tạo rule chặn |
| **Security Hub (CSPM)** | Tổng hợp finding + chạy standard (CIS, FSBP) | Không remediation dựng sẵn |
| **Amazon Detective** | Điều tra finding | Không chặn traffic |

📌 Đề mô tả Inspector với "rules packages, Findings report / Full report" là **Inspector Classic**, đã **tắt hoàn toàn từ 20/05/2026**. Inspector hiện hành quét liên tục, xuất finding report; ý "dùng Inspector để chứng minh không còn lỗ hổng" vẫn đúng.

🎯 **Quét CIS Benchmark cho toàn Organization** → chỉ định **Security Hub administrator account**, **tự động bật cho account mới**, bật standard **CIS AWS Foundations Benchmark**. 📌 Phần standards/controls nay mang tên **Security Hub CSPM**. 🧭 Compliance scanning → Security Hub · Vulnerability → Inspector · Threat → GuardDuty.

| | **Shield Standard** | **Shield Advanced** |
| --- | --- | --- |
| Chi phí | Miễn phí, bật sẵn | Trả phí |
| Bảo vệ | L3/L4 | L3/L4 **và L7** |
| Đội hỗ trợ 24/7 | — | ✅ |
| **DDoS cost protection** | — | ✅ hoàn phí tăng đột biến do DDoS (EC2, ELB, CloudFront, Global Accelerator, Route 53) |

📌 Đội "DDoS Response Team (DRT)" nay là **Shield Response Team (SRT)**. "AWS DDoS OpsTeam" là tên bịa.

🎯 **WAF gắn được:** CloudFront, API Gateway REST, ALB, AppSync. 📌 Nay còn gắn được **Cognito user pool, App Runner, Verified Access, Amplify, AgentCore Gateway**. ⚠️ **Không gắn thẳng EC2, NLB, S3.** Mẫu chuẩn: WAF trên CloudFront + **Firewall Manager** nhân bản cấu hình WAF cho nhiều account.

🎯 **Chặn 1 IP độc hại trước ALB** → **WAF IP set + Web ACL action BLOCK** gắn ALB. ⚠️ SG không có Deny.

🎯 **Trusted Advisor:** "**Security Groups – Specific Ports Unrestricted**" (liệt kê SG mở 22/3389/1433/3306 ra `0.0.0.0/0`, cảnh báo đỏ) · "**Low Utilization EC2 Instances**" · service limits (> 80%) · S3 (open access, logging, versioning). ⚠️ Không sinh báo cáo chi phí, không tự remediate. GuardDuty, IAM Access Analyzer, SCP đều sai mục đích khi đề hỏi rà SG.

| Shared Responsibility | Ai? |
| --- | --- |
| EC2: guest OS (update, patch), phần mềm, **cấu hình Security Group** | **Khách hàng** |
| S3/DynamoDB: hạ tầng, OS, platform | **AWS** |
| Dữ liệu, phân loại tài sản, IAM permission, chọn mã hóa, lifecycle rule | Khách hàng |
| Cung cấp cơ chế mã hóa, hạ tầng key, storage dư thừa đa AZ | AWS |
| Vá hạ tầng | AWS |
| Đào tạo nhân viên AWS / nhân viên khách hàng | AWS / Khách hàng |

---

## 34. VPC cơ bản, gateway, endpoint & địa chỉ IP

🎯 **Subnet nằm trong đúng 1 AZ**; VPC trải nhiều AZ trong 1 Region. Mọi subnet cùng VPC **route được với nhau** nhờ local route. IP nội bộ của subnet luôn private. Private subnet có route tới IGW → thành public subnet. ⚠️ VPC Peering là giữa **2 VPC**, không giữa 2 subnet.

🎯 **AWS giữ 5 IP mỗi subnet** (.0, .1 router, .2 DNS, .3 dự phòng, .255 broadcast) → IP dùng được = 2^(32 − x) − 5: `/28` = 11 · `/27` = 27 · **`/26` = 59** · `/25` = 123 · `/24` = 251. **28 instance → `/26`**.

| Gateway | Protocol | Chiều | Ghi chú |
| --- | --- | --- | --- |
| Internet Gateway | IPv4 + IPv6 | 2 chiều | Route ở public subnet |
| **NAT Gateway** | **IPv4** | Outbound | Private subnet ra Internet |
| **Egress-only IGW** | **IPv6** | Outbound | IPv6 ra Internet |
| Virtual Private Gateway | — | qua VPN | Phía AWS của Site-to-Site VPN |
| Customer Gateway | — | — | Mô tả thiết bị phía bạn |
| Carrier Gateway | IPv4 | — | Chỉ cho subnet trong Wavelength Zone |
| Transit Gateway | — | — | Hub nhiều VPC + on-prem; tự nó không tạo đường low-latency tới data center |

🎯 **NAT Gateway:** gắn 1 EIP, hỗ trợ TCP/UDP/ICMP, **không gắn Security Group** (dùng NACL), tối đa **55,000 kết nối đồng thời tới mỗi đích duy nhất trên mỗi IP**. 📌 Đề ghi "5 Gbps tự scale tới 45 Gbps": nay **5 → 100 Gbps**. 📌 Đề ghi "phải đặt ở public subnet": đúng với NAT **zonal**; nay có **Regional NAT gateway** (không cần public subnet, tự trải đa AZ, tới 32 IP/AZ).

🎯 **Hàng trăm EC2 ở 3 AZ gọi API bên thứ ba, đối tác cần danh sách IP tĩnh** → **NAT Gateway mỗi AZ** làm default route của private subnet → chỉ cần **3 Elastic IP**. ⚠️ 1 EIP chỉ gắn 1 instance; NLB dành cho inbound.

| | NAT Gateway | NAT Instance |
| --- | --- | --- |
| Quản lý | AWS managed, HA trong AZ | Tự quản EC2 |
| Security Group | ❌ | ✅ |
| Làm bastion | ❌ | ✅ |
| Source/dest check | N/A | Phải tắt |

| | **Gateway Endpoint** | **Interface Endpoint (PrivateLink)** |
| --- | --- | --- |
| Dịch vụ | **Chỉ S3 và DynamoDB** | Hầu hết dịch vụ (SSM, KMS, ECR...) |
| Chi phí | **Miễn phí** | Phí theo giờ + GB |
| Gắn vào | **Route table** | Subnet (ENI + SG) |
| Từ on-prem | ❌ | ✅ (qua DX/VPN) |

🎯 **Truy cập 50 bucket S3 riêng tư, không tốn thêm phí** → **một Gateway Endpoint** cho S3 của Region (thêm vào route table), không cần endpoint riêng mỗi bucket. **Gọi AWS API không qua Internet** → Interface Endpoint.

🎯 **Private subnet không patch được bằng SSM** → **NAT Gateway ở public subnet + route `0.0.0.0/0` trong route table của private subnet** (an toàn hơn: interface endpoint cho SSM).

🎯 **Địa chỉ IP của EC2:** public IPv4 lấy từ pool của Amazon, **không tự gán/gỡ được**; **mất khi stop, hibernate, terminate** (start lại nhận IP mới). Gán EIP → public IP cũ bị thu hồi; không tồn tại song song. Instance không nhận public IP mới nếu có nhiều ENI hoặc secondary private IP đang gắn EIP. Default VPC cấp public IP mặc định; VPC không mặc định phụ thuộc **auto-assign** của subnet (mặc định tắt). EC2 và VPC **luôn dùng IPv4**. IPv6 là public toàn cầu, **không gán EIP**, không tự ra Internet (cần egress-only IGW).

🎯 **ENI** attach vào **instance** (không "attach vào subnet"). **Elastic IP** là cách giữ dịch vụ không gián đoạn khi migrate instance.

📌 Đề ghi "1 EIP gắn instance đang chạy là miễn phí": từ **02/2024 AWS tính phí mọi public IPv4**, kể cả EIP đang gắn.

---

## 35. Security Group, NACL & VPC Flow Logs

| | **Security Group** | **Network ACL** |
| --- | --- | --- |
| Cấp | Instance / ENI | Subnet |
| Stateful? | ✅ (return tự cho phép; outbound mặc định mở) | ❌ (mở cả hai chiều) |
| Rule | **Chỉ Allow** | Allow + Deny |
| Đánh giá | Mọi rule cùng lúc | Theo số rule nhỏ → lớn, khớp đầu tiên dừng |
| Mặc định | Outbound mở | **Default NACL allow hết · custom NACL deny hết** |

🎯 SG và NACL là **hai lớp độc lập**, không cha-con. **Mọi subnet luôn gắn đúng 1 NACL** (không khai báo thì là default NACL). SG tham chiếu SG khác → mọi instance thuộc SG đó được phép. `/32` = 1 IP, `/0` = tất cả.

🎯 **Cho resource ở VPC2 truy cập instance ở VPC1 (đã peering)** → chỉ sửa **inbound SG**; default NACL đã allow hết. Cross-account qua peering → source ghi **`<accountId>/sg-xxxx`** (⚠️ không dùng `pcx-...`, không mở cả CIDR VPC peer).

🎯 **Chặn một IP độc hại ở tầng mạng** (vd GuardDuty báo EC2 gửi traffic tới IP lạ) → **NACL rule outbound DENY**. ⚠️ SG không có Deny và không gán cho VPC; Flow Logs chỉ giám sát.

| Nguồn gọi | Dải ephemeral port |
| --- | --- |
| **ELB, NAT Gateway, Lambda** | **1024–65535** |
| Linux (gồm Amazon Linux) | 32768–61000 |
| Windows Server 2008 trở lên | 49152–65535 |
| Windows tới Server 2003 | 1025–5000 |

🎯 **Website qua ELB, đã mở inbound 80 mà không vào được** → thêm **NACL outbound 1024–65535**. SG không cần (stateful).

🎯 **Thêm subnet web thứ 3 vào ASG, một số user mất kết nối DB** (DB subnet dùng custom NACL chỉ allow 2 subnet cũ) → NACL của **DB subnet** thêm **inbound 3306 từ subnet mới** + **outbound ephemeral tới subnet mới**. ⚠️ Outbound 3306 là sai chiều port.

🎯 **EC2 private subnet `curl` fail** (route NAT đúng) → kiểm tra **outbound SG** đúng port (`http` = 80, `https` = 443) tới `0.0.0.0/0`. Không cần inbound (stateful).

🎯 **"Cannot connect to the database"** do: **SG inbound** của DB đổi · **NACL inbound** đổi · **NACL outbound** đổi. ⚠️ Không phải SG outbound (stateful), private IP đổi (app dùng DNS endpoint), vừa tạo read replica.

### VPC Flow Logs

🎯 Đọc log: cột 4 = srcaddr, cột 5 = dstaddr. **Inbound ACCEPT + outbound (return) REJECT** → **NACL** (stateless) chặn chiều về → sửa **outbound NACL** (ephemeral). Dòng có **REJECT** gần như chắc chắn do NACL (SG không có Deny).

🎯 **Không thấy log trong CloudWatch Logs** → IAM role của flow log thiếu quyền: `logs:CreateLogGroup` · `logs:CreateLogStream` · `logs:PutLogEvents` · `logs:DescribeLogGroups` · `logs:DescribeLogStreams`. Trạng thái **`Access error`**: role thiếu quyền, thiếu trust relationship, hoặc trust không khai báo flow logs service. ⚠️ `logs:CreateExportTask` không bắt buộc; peering/IPv6 không ảnh hưởng.

🎯 **Sau khi tạo không sửa được flow log** (IAM role, record format) → **xóa và tạo lại**.

🎯 **Log ít hơn mong đợi** là bình thường: flow log chỉ ghi traffic qua ENI trong VPC, **không ghi traffic on-premises**, loại trừ traffic tới Amazon DNS, Windows license activation, metadata `169.254.169.254`, DHCP, ARP, địa chỉ dành riêng của router.

🎯 **RDP qua VPN vào EC2 private bị timeout** (SG và firewall on-prem đã mở) → **bật Flow Log cho ENI của EC2** để xem ACCEPT/REJECT. ⚠️ Log ứng dụng, log VPN (chỉ UP/DOWN), EC2 Instance Connect không giải quyết.

⚠️ Source/Destination Check chỉ tắt khi instance làm router/NAT/firewall; không ghi đè NACL.

---

## 36. Kết nối lai: VPN, Direct Connect, Client VPN

| Giải pháp | Đặc điểm | Chọn khi |
| --- | --- | --- |
| **Site-to-Site VPN** | IPsec qua Internet, dựng nhanh | Cần ngay, băng thông thấp–trung bình |
| **Direct Connect** | Đường vật lý riêng, VLAN 802.1q; **≥ 1 tháng**, tốn kém; **không mã hóa mặc định** | Low latency + throughput ổn định dài hạn |
| **Direct Connect + VPN** | DX có IPsec end-to-end | "**dedicated + encrypted + low latency + high throughput**" |

| VIF | Truy cập |
| --- | --- |
| **Public VIF** | **Dịch vụ AWS public (S3, DynamoDB...)**, mọi Region qua một kết nối |
| Private VIF | Tài nguyên trong VPC |
| Transit VIF | Transit Gateway |

🎯 **On-prem tải S3 ở 3 Region, ổn định, rẻ nhất** → **một Direct Connect + một Public VIF**. ⚠️ Private VIF không truy cập S3.

🎯 **VPN tunnel UP nhưng không ping được** → bật **Route Propagation cho VGW** trên route table của subnet. ⚠️ SG/NACL không tạo route; DHCP options set chỉ DNS/NTP.

🎯 **Subnet mới ở AZ mới mất kết nối on-prem** → route table của subnet mới **thiếu route tới VGW**. VGW là tài nguyên cấp VPC, đã hoạt động mọi AZ → không cần VPN mới, không cần ticket Support.

🎯 **Client VPN chỉ cho traffic tới VPC đi qua tunnel** → bật **split-tunnel** (full-tunnel là mặc định, đưa cả traffic Internet qua VPN). ⚠️ NAT, DNS server, private certificate không điều khiển luồng traffic client.

🎯 **Traffic giữa 2 EC2 qua public IP:** cùng Region → ở trong mạng AWS; khác Region có inter-Region VPC peering → có; khác Region không peering → không đảm bảo.

🎯 **VPC Peering troubleshoot:** **route table cả hai VPC** (phổ biến nhất) → SG → NACL → DNS resolution. Không transitive; được cross-account, cross-Region.

---

## 37. Amazon CloudFront

🎯 **Xóa file khỏi edge cache ngay** → **Invalidation** (vài giây tới mọi edge; **không hủy được sau khi submit**) hoặc **versioned file name**. Cache mặc định 24 giờ. ⚠️ Default root object, cache/origin request policy không xóa cache.

🎯 **Cache hit ratio thấp** → **tăng Default TTL & Maximum TTL** + **Origin Shield** ở Region gần origin. ⚠️ Origin group là HA; compression giảm kích thước; Route 53 weighted chia nhiều distribution làm tệ hơn.

🎯 **TTL khác nhau cho từng trang** → origin trả header **`Cache-Control: max-age=<giây>`** (bị kẹp trong khoảng Min–Max TTL). ⚠️ Đặt header ở viewer response là quá muộn; invalidation không phải đặt TTL; `no-cache` gần như tắt cache.

🎯 **Đã tạo CloudFront mà origin vẫn nhận toàn bộ traffic** → **DNS vẫn trỏ thẳng ALB**, hoặc **Default/Min/Max TTL = 0**. ⚠️ OAI chỉ cho S3; SG chặn CloudFront thì sinh 5xx chứ không phải "traffic vẫn tới ALB".

🎯 **Ép HTTPS + tên miền riêng** → **Viewer Protocol Policy = Redirect HTTP to HTTPS** cho mọi cache behavior + **Alternate Domain Name** + **custom SSL cert** (ACM ở `us-east-1`). Alternate domain cần **đăng ký domain** + **cert từ CA tin cậy** phủ domain đó (SNI là cách phục vụ, không phải bước bắt buộc).

🎯 **Chỉ subscriber mới xem được** → **Signed URL** (một file) hoặc **Signed Cookie** (nhiều file, không đổi URL). ⚠️ Ép HTTPS tới origin hay chọn cipher không giới hạn người xem.

| Origin | Bảo vệ | HTTPS CloudFront → origin |
| --- | --- | --- |
| S3 REST endpoint | **OAC** (đề cũ: OAI) + bucket policy | Theo protocol viewer dùng |
| **S3 website endpoint** | Là **custom origin** → **không dùng OAI/OAC**, dùng **custom header** | **Không dùng được HTTPS** (website endpoint không hỗ trợ HTTPS) |

📌 **OAI** là cơ chế legacy → AWS khuyên **OAC** (hỗ trợ SSE-KMS, mọi Region). 📌 CloudFront nay có **VPC origins** (ALB/NLB/EC2 trong private subnet). ⚠️ SG và NACL không dùng được với CloudFront.

🎯 **Cookie:** mặc định CloudFront **bỏ qua cookie** (và gỡ `Set-Cookie` khi không forward) → user bị bắt login lại dù ALB có sticky session → **forward cookie** (all hoặc whitelist) trong cache behavior. ⚠️ Không có "sticky session trên CloudFront".

| Mã | Ý nghĩa |
| --- | --- |
| **404** | **Do origin sinh**, CloudFront chỉ trả lại (CloudFront không tự sinh 404; cache miss không phải lỗi) |
| **400** "authorization header is malformed; the region ... is wrong" | Bucket origin bị xóa ở Region A và tạo lại cùng tên ở Region B → cập nhật distribution |
| **502** | CloudFront không kết nối được origin |
| **504** | Origin không phản hồi kịp |

🎯 **File lớn phân phối từ EFS + ALB, tối ưu chi phí** → **S3 + CloudFront** (S3 rẻ hơn EFS nhiều; data transfer out qua CloudFront rẻ hơn đi thẳng từ S3).

---

## 38. AWS Global Accelerator

🎯 **2 anycast IP tĩnh**, đi trên backbone AWS từ edge gần user; endpoint: **ALB, NLB, EC2, Elastic IP** (⚠️ **không S3**). Failover bằng health check ở tầng dịch vụ, **trong vài giây, không phụ thuộc DNS TTL**.

🎯 **Khách chỉ whitelist tối đa 2 IP tĩnh, app sau ALB** → **Global Accelerator trước ALB**. ⚠️ **ALB không gắn được Elastic IP** (chỉ NLB).

🎯 **Đa Region, latency thấp nhất + failover nhanh nhất** → **Global Accelerator** (endpoint group mỗi Region). ⚠️ Route 53 latency/failover phụ thuộc DNS TTL nên chậm hơn; geolocation không đảm bảo latency thấp nhất.

---

## 39. Quản lý chi phí

| Giải pháp | Lọc theo dimension? | Ngưỡng %? | Effort |
| --- | --- | --- | --- |
| **AWS Budgets (cost budget)** | ✅ service, usage type, Region, tag | ✅ (vd 75%) | Thấp nhất |
| CloudWatch Billing Alarm | ❌ chỉ estimated charges cấp account | Số tuyệt đối | Sai khi cần lọc |
| Cost Anomaly Detection | Bất thường (ML) | ❌ | Sai ngữ nghĩa |
| Lambda + Cost Explorer API + SES | Tự code | Tự code | Cao nhất |

🎯 **Alert khi chi phí đạt X% ngưỡng, lọc theo loại** → **AWS Budgets**. ⚠️ Budgets **không cho biết chi phí từng bucket S3**.

🎯 **Báo cáo chi phí theo phòng ban / theo từng S3 bucket** → gắn tag → **ACTIVATE Cost Allocation Tags** trong Billing console (tag không tự xuất hiện) → báo cáo theo tag trong **Cost Explorer**. Hai loại tag: AWS-generated (`aws:createdBy`) và user-defined. IAM cần quyền Billing console + `s3:GetBucketTagging`/`s3:PutBucketTagging`. ⚠️ Organizations gộp billing giữa account, không chia trong một account; Pricing Calculator chỉ ước tính; Trusted Advisor không sinh báo cáo chi phí.

🎯 **Báo cáo chi phí chi tiết tới từng giờ, toàn Organization** → **Cost and Usage Report (CUR) → S3 → Athena → Quick Sight**. ⚠️ Cost Explorer không có custom dashboard hourly như CUR; CloudWatch không lấy dữ liệu từ Cost Explorer. 📌 CUR nay là **CUR 2.0 qua Data Exports**; QuickSight nay là **Amazon Quick Sight**.

🎯 **Right-size EBS theo IOPS/throughput, Lambda theo memory, EC2/ASG, ECS Fargate, ít công nhất** → **AWS Compute Optimizer** (ML trên metric CloudWatch, có ước tính tiết kiệm; bật trong Organizations để gom mọi account). ⚠️ Xem graph thủ công, chạy `fio`, EBS-optimized instance đều không phải right-sizing.

🎯 **Cảnh báo khi dùng tới X% service quota** → **Service Quotas console tạo CloudWatch alarm** (vd ≥ 70%) → SNS. Muốn **tự xin tăng quota** → alarm → **Lambda** gọi **`RequestServiceQuotaIncrease`**. ⚠️ Alarm `CPUUtilization` sai chỉ số; Config và Health Dashboard không có metric quota.

🎯 **Dọn EC2 dùng thấp** → Trusted Advisor check **Low Utilization EC2 Instances** → status change sang **SNS** → **Lambda** stop/terminate. ⚠️ GuardDuty là threat detection; Health Dashboard không có metric sử dụng.

---

## 40. Phụ lục — Tra cứu dịch vụ & bẫy distractor

> Những dịch vụ/đáp án hay xuất hiện ở **đáp án SAI**. Nhớ "nó thực chất làm gì" để loại nhanh.

### Đáp án KHÔNG TỒN TẠI (loại ngay)

| Đáp án nhiễu | Thực tế |
| --- | --- |
| Metric `NetworkTotalBytes`, `CPUProcessPower`, `CombinedStatusCheckFailed`, `RAMUtilization` (mặc định EC2), `InstanceConnections` (EFS) | Không tồn tại |
| "Amazon Patch Manager", "Patch Fleet", "SSM Audit Trail", "EC2 Billing Report", "IAM usage report", "AWS Data Migration", "AWS DDoS OpsTeam" | Đúng là SSM Patch Manager, CloudTrail, AWS DMS, Shield Advanced + SRT |
| "Infrastructure Load Balancer", "static file caching trên ALB", "dedicated placement group", "Event-based scaling" | Không tồn tại |
| CloudFormation field "Expose", StackSets "resource-managed permissions", "rollback trigger DO_NOTHING", `aws cloudformation put-dashboard` | Đúng là `Export`; self-/service-managed; `OnFailure=DO_NOTHING`; `aws cloudwatch put-dashboard` |
| "Enable SSL connections" trên RDS Console, cài patch/addon lên RDS, RDS MySQL có TDE, bật mã hóa RDS sau khi tạo | Dùng parameter group; TDE chỉ Oracle/SQL Server; mã hóa phải lúc tạo |
| "automatic certificate rotation" trong CloudFront, mTLS trên CloudFront, sticky session trên CloudFront, SG gắn vào CloudFront | Không tồn tại |
| `s3:ReadObject`, `s3:GetBucket`, `Effect: "read-only"`, `"Effect": "NotAction"` | Đúng: `s3:GetObject`, `s3:ListBucket`; Effect chỉ Allow/Deny |
| Header `x-amz-server-side-encryption: SSE-KMS`/`SSE-S3`, `x-amz-bypass-legal-retention`, quyền `s3:BypassLegalRetention` | Đúng: `aws:kms`/`AES256`; Legal Hold gỡ bằng `s3:PutObjectLegalHold` |
| "IPv6 NAT gateway", "Transfer Acceleration cho EBS", Global Accelerator/NLB nhận S3 làm target, gán EIP cho ALB | Không tồn tại |
| Security Group có rule Deny, SG gán cho VPC, ALB target group chứa instance ở Region khác, ASG trải nhiều Region | Không tồn tại |
| SQS là target của CloudWatch Alarm, EventBridge nhận trực tiếp giá trị metric, SNS transform payload, "SMS identity" làm target EventBridge | Không tồn tại |
| CloudWatch có sẵn metric `DaysToExpiry` cho ACM, CloudWatch có metric AMI ID, Inspector tạo custom finding kiểm tra AMI, Trusted Advisor tự remediate | Không tồn tại |
| Launch template tự tạo EFS, instance profile gắn server on-prem, Lambda có "Multi-AZ"/"CPU launch options" | Không tồn tại |
| Đổi SQS Standard → FIFO tại chỗ, cross-account RDS read replica, KMS key replicate sang account khác, export KMS key | Không tồn tại |
| Spot interruption "Reboot", `DeletionPolicy: Snapshot` cho EC2, CloudFormation timeout cho DELETE | Không tồn tại |
| Thu nhỏ cache disk Storage Gateway, khôi phục gateway từ hypervisor snapshot/AMI, mount EFS từ Windows, xác thực EFS bằng Cognito, Backtrack cho Aurora PostgreSQL | Không tồn tại |
| Đặt EIP và public IP thường cùng lúc trên một instance, AMI mã hóa để public, AWS managed key sửa key policy, Alias trỏ server on-prem | Không tồn tại |

### Monitoring & Ops

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **CloudTrail Insights** | Bất thường trong **API call** | Đề cần phân tích traffic hoặc top-N IP |
| **Contributor Insights** | Top-N contributor | Đề cần percentile |
| **Logs Insights** | Query ad-hoc | Đề cần metric bền vững cho alarm (→ metric filter) |
| **RUM** | Dữ liệu từ user thật (passive) | "Phát hiện trước khi user báo" (→ Synthetics) |
| **CloudWatch Anomaly Detection** | Ngưỡng động ML | Đề cần "recommendations" (→ DevOps Guru) |
| **AWS Config** | Cấu hình + compliance | Cảnh báo real-time, đo mức sử dụng resource |
| **Trusted Advisor** | Khuyến nghị best practice | Tự sửa (→ Config remediation), báo cáo chi phí theo tag |
| **Health Dashboard** | Sự cố/bảo trì của AWS | Metric quota hay metric sử dụng instance |
| **SSM OpsCenter** | Quản lý OpsItem | Lưu file cấu hình |
| **SSM Inventory** | Thu metadata | Cài/vá phần mềm |
| **Fleet Manager** | UI quản lý node | Vá lỗi |
| **Data Lifecycle Manager** | Snapshot/AMI EBS theo lịch | RDS, EFS, nhiều mức retention đa dịch vụ (→ AWS Backup) |

### Deployment & Compute

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **StackSets** | Deploy đa account/Region | Mô-đun hóa template (→ Nested Stacks) |
| **Change set** | Xem trước thay đổi | Triển khai đa account, gỡ `DELETE_FAILED` |
| **Drift detection** | So cấu hình thực tế với template | Import resource, xóa stack |
| **Stack policy** | Chặn update resource | Access control (→ IAM), chống xóa stack (→ termination protection) |
| **Termination policy (ASG)** | Thứ tự instance bị xóa | Bảo vệ instance (→ scale-in protection) |
| **Standby state** | Tạm tách 1 instance khỏi service | Đóng băng ASG (→ suspend Launch + Terminate) |
| **Predictive scaling** | ML dự báo pattern | Giờ cố định đã biết (→ scheduled), boot chậm (→ warm pool) |
| **Spot Instance Advisor** | Tra cứu tỷ lệ gián đoạn | Cấu hình allocation strategy |
| **Packer + cron** | Tự dựng pipeline AMI | Có EC2 Image Builder |

### Storage & Database

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **S3 Storage Lens** | Dashboard usage/activity | Danh sách object (→ Inventory), xóa MPU dở (→ lifecycle) |
| **S3 Select** | SQL lấy một phần một object (📌 không nhận khách mới) | Audit trạng thái object |
| **S3 Object Lambda** | Biến đổi dữ liệu khi đọc (📌 chỉ khách cũ) | Access control |
| **S3 RTC** | SLA replication 15 phút + event | Sửa Access Denied replication |
| **Transfer Acceleration** | Upload xa qua edge | Client cùng Region |
| **Glacier / Deep Archive** | Archive giờ–ngày | "Recover rapidly" |
| **One Zone-IA** | 1 AZ | "Highly available" |
| **EBS Multi-Attach** | io1/io2, 16 instance, 1 AZ | File share đa AZ cho ASG (→ EFS) |
| **EFS** | NFS Linux | Windows/SMB (→ FSx for Windows) |
| **DAX** | Cache cho DynamoDB | Cache cho RDS |
| **Read Replica** | Scale đọc, async | HA/failover không mất dữ liệu (→ Multi-AZ), test upgrade |
| **Multi-AZ standby** | Failover | Phục vụ đọc |
| **AOF/RDB (Redis)** | Durability | Failover tự động |
| **DMS** | Migrate/replicate | Khôi phục mốc lịch sử |

### Security & Network

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **GuardDuty** | Threat detection | Rà SG mở port, compliance benchmark, chặn traffic |
| **Inspector** | Vulnerability scan | CIS benchmark cho account, cấu hình S3 |
| **IAM Access Analyzer** | Phân tích resource/IAM policy, sinh policy | Rà network rule |
| **SCP** | Giới hạn quyền tối đa (preventive) | Cấp quyền, triển khai resource, tìm cấu hình đã tồn tại |
| **Permissions boundary** | Trần quyền user/role | Hạn chế root user |
| **Amazon Detective** | Điều tra finding | Chặn IP |
| **AWS RAM** | Chia sẻ resource | Nhân bản định nghĩa, chặn IP |
| **Parameter Store SecureString** | Lưu secret mã hóa | Auto-rotation |
| **IAM DB authentication** | Token IAM cho app | Xoay mật khẩu master |
| **NLB** | L4, IP tĩnh | Routing theo URL, gắn WAF |
| **GWLB** | L3 cho security appliance | Routing HTTP |
| **Transit Gateway** | Hub VPC + on-prem | Thay Direct Connect, gắn private hosted zone |
| **VPC Peering** | Nối 2 VPC | Nối on-prem, transitive |
| **PrivateLink / interface endpoint** | Tới AWS service/SaaS | Đưa Lambda vào VPC, tăng throughput S3 |
| **Origin Shield** | Giảm tải origin | Mã hóa, xác thực user |

---

## 41. Nguyên tắc làm bài chung

🎯 **"Least operational overhead / ít công nhất"** → tính năng **managed có sẵn**: Config managed rule + runbook có sẵn, Storage Auto Scaling, RDS Proxy, Quick Setup, Compute Optimizer, Service Quotas alarm. Đáp án "tự viết Lambda/script/cron" gần như luôn thua.

🎯 **Metric → Alarm; event → EventBridge.** CPU, disk, quota là metric. Trạng thái EC2, API call, alarm state change, Config compliance là event.

🎯 **Preventive vs detective:** "không cho phép / chặn" → SCP, Block Public Access, Object Lock, IAM Deny, launch constraint. "Phát hiện / báo cáo" → Config, Trusted Advisor, Security Hub, GuardDuty, Inspector.

🎯 **Đọc đơn vị của metric:** `disk_used_percent` (%) vs `disk_used` (bytes); `FreeStorageCapacity` (bytes); `NetworkIn` (bytes); Sum vs Average.

🎯 **Stateful vs stateless:** SG stateful → chỉ mở một chiều; NACL stateless → luôn kiểm tra cả chiều về (ephemeral port).

🎯 **Thứ tự troubleshoot:** quyền IAM → cấu hình → mạng/route → trạng thái dịch vụ. Lỗi "không có log nào" gần như luôn là **IAM thiếu quyền `logs:*`**.

🎯 **Phân biệt quyền đi ra và đi vào:** execution role / instance profile (dịch vụ gọi ra) vs resource-based policy (ai được gọi vào).

🎯 **Cẩn thận đáp án thay đổi kiến trúc** (đổi engine DB, đổi sang DynamoDB, thêm cluster mới) khi đề yêu cầu "ít thay đổi ứng dụng nhất".

🎯 **Đề cũ vs hiện tại:** gặp tên cũ (PHD, OAI, Vault Lock, Snowball, DRT, Launch Configuration, Inspector Classic) thì chọn theo logic đề, nhưng biết thực tế đã đổi (📌 trong từng mục).

---

## 42. Bảng so sánh dễ nhầm

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| Detailed Monitoring vs high-res custom metric | 1 phút metric có sẵn vs 1 giây metric tự đẩy | "1 phút" vs "10 giây" |
| Metric Filter vs Logs Insights vs Contributor Insights | Sinh metric bền vững vs query ad-hoc vs top-N | percentile/alarm vs điều tra vs top contributor |
| Synthetics vs RUM | Proactive vs passive | "trước khi user báo" → Synthetics |
| DevOps Guru vs Anomaly Detection | Có khuyến nghị vs chỉ ngưỡng động | "recommendations" → DevOps Guru |
| CloudWatch vs CloudTrail vs Config | Hiệu năng vs API call vs cấu hình | "ai gọi" → CloudTrail; "compliance" → Config |
| Enhanced Monitoring vs CloudWatch vs Performance Insights | OS-level (agent) vs hypervisor vs DB load | "memory, swap, process" → Enhanced Monitoring |
| Health Dashboard (account) vs service health vs Organizational View | Resource của tôi vs chung vs gộp member | "member account" → Organizational View |
| Patch Manager vs Inventory vs Automation vs Run Command vs State Manager | Vá vs metadata vs runbook vs lệnh ad-hoc vs trạng thái mong muốn | đọc động từ trong đề |
| Distributor vs Run Command | Gói có version, tự cập nhật vs lệnh một lần | ".msi tự cập nhật" → Distributor |
| Stack Policy vs Termination Protection vs DeletionPolicy | Chặn update vs chặn xóa stack vs giữ resource khi xóa | "khóa resource khi update" → stack policy |
| Nested Stacks vs StackSets | Tái sử dụng component vs đa account/Region | |
| `Export`/`ImportValue` vs `Ref` vs `GetAtt` | Cross-stack vs ID/parameter vs thuộc tính | "dùng output stack khác" → ImportValue |
| `Retain` vs `Snapshot` | Giữ nguyên resource vs giữ snapshot (không cho EC2) | "giữ EC2" → Retain |
| Multi-AZ vs Read Replica | Sync HA vs async scale đọc | "không mất commit" vs "báo cáo chậm" |
| PITR vs Backtrack | Cluster mới vs tua tại chỗ (chỉ Aurora MySQL) | Aurora PostgreSQL → PITR |
| Redis vs Memcached | Replication/failover/persistence vs multi-thread đơn giản | "HA" → Redis/Valkey |
| `allkeys-lfu` vs `allkeys-lru` vs random | Giữ item phổ biến vs giữ item mới dùng vs ngẫu nhiên | "popular items, không tăng chi phí" → lfu |
| Scheduled vs dynamic vs predictive scaling | Giờ cố định vs theo metric vs ML | "giờ đã biết" → scheduled |
| Scale-in protection vs suspend process vs termination policy | Bảo vệ instance vs dừng hoạt động cả group vs thứ tự | |
| EC2 health check vs ELB health check | Hạ tầng vs ứng dụng | "healthy ASG nhưng unhealthy ALB" → bật ELB |
| Warm Pool vs tăng min capacity | Instance stopped sẵn vs chạy thừa | "không overprovision" → Warm Pool |
| HPA vs VPA | Số pod vs tài nguyên pod | "traffic tăng" → HPA |
| ALB vs NLB vs GWLB | L7 routing vs L4 hiệu năng/IP tĩnh vs L3 appliance | |
| Duration-based vs application-based cookie | ALB tạo cookie vs dùng cookie của app | "app tự có session cookie" → application-based |
| Alias vs CNAME vs A | AWS resource, free, apex vs domain khác, có phí vs IP | "on-prem server" → A; "domain bên thứ ba" → CNAME |
| Inbound vs outbound Resolver endpoint | On-prem → AWS vs AWS → on-prem | |
| Gateway vs Interface endpoint | S3/DynamoDB, free, route table vs ENI, có phí | "không thêm chi phí" → Gateway |
| Public VIF vs Private VIF | Dịch vụ public (S3) vs VPC | "S3 qua DX" → Public VIF |
| Split-tunnel vs full-tunnel | Chỉ traffic VPC vs mọi traffic | |
| Signed URL vs Signed Cookie vs Pre-signed URL | CloudFront 1 file vs CloudFront nhiều file vs S3 trực tiếp | |
| Global Accelerator vs Route 53 failover | Failover vài giây, IP tĩnh vs phụ thuộc TTL | "nhanh nhất / 2 static IP" → GA |
| SSE-S3 vs SSE-KMS vs SSE-C vs client-side | Đơn giản vs audit vs khách giữ key S3 mã hóa vs khách tự mã hóa | |
| Governance vs Compliance (Object Lock) | Bypass được vs không ai bypass | "kể cả root" → Compliance |
| S3 RTC vs S3 Inventory | Event real-time vs danh sách định kỳ | |
| Storage Lens vs Inventory vs lifecycle | Dashboard vs danh sách object vs hành động xóa/chuyển | |
| File vs Stored vs Cached vs Tape Gateway | NFS/SMB vs block toàn bộ on-prem vs block chính ở AWS vs băng từ | |
| gp2 vs gp3 vs io2 vs st1/sc1 | Burst theo dung lượng vs baseline 3,000 vs IOPS cao vs HDD | "BurstBalance = 0" → gp3 |
| AWS Backup vs DLM | Đa dịch vụ, đa account vs chỉ EBS/AMI | "RDS + EC2 nhiều retention" → Backup |
| Dedicated Host vs Dedicated Instance | License theo socket/core vs single-tenant rẻ nhất | |
| `InsufficientInstanceCapacity` vs `InstanceLimitExceeded` | Hết capacity AZ vs chạm quota | đổi AZ vs tăng quota |
| `capacity-optimized` vs `diversified` | Pool nhiều capacity nhất vs trải đều | "most availability" → capacity-optimized |
| Compute SP vs EC2 Instance SP vs RI | EC2 + Fargate + Lambda vs chỉ EC2 family vs chỉ EC2 | "chuyển sang Fargate" → Compute SP |
| Budgets vs Billing Alarm vs Cost Anomaly Detection | Lọc + % vs tổng account vs bất thường | |
| Cost Explorer vs CUR | Xem tổng quan vs chi tiết hourly + Athena | "từng giờ" → CUR |
| Security Hub vs Inspector vs GuardDuty | Compliance vs lỗ hổng vs mối đe dọa | |
| Shield Standard vs Advanced | Miễn phí L3/4 vs đội 24/7 + hoàn phí | |
| AD Connector vs Managed AD vs Simple AD | Proxy AD on-prem vs AD thật có trust vs Samba không trust | |
| Cognito User Pool vs Identity Pool | Đăng nhập vs credential AWS tạm | |

---

## 43. Từ khóa → đáp án

| Từ khóa trong đề | Đáp án |
| --- | --- |
| "event-driven, immediate, minimal maintenance" | EventBridge → SNS |
| "app treo CPU 100%, cần tự restart" | CloudWatch Alarm (detailed monitoring) → EC2 reboot action |
| "instance dev để quên, tự stop tiết kiệm" | Alarm CPU < 5% kéo dài → stop action |
| "predictable / consistent daily traffic spike" | Scheduled Scaling |
| "resource nào **của tôi** bị ảnh hưởng bởi sự cố AWS" | AWS Health Dashboard (account health; tên cũ Personal Health Dashboard) |
| "Health events của member account không về payer" | AWS Health Organizational View |
| "metric OS-level của RDS (memory, process, swap)" | RDS Enhanced Monitoring |
| "theo dõi p90/percentile của field trong log" | Metric Filter → CloudWatch metric |
| "theo dõi metric mỗi 10 giây" | High-resolution custom metric |
| "alert chỉ khi CẢ HAI metric vượt ngưỡng" | Composite Alarm (AND) |
| "alarm khi TẤT CẢ target ALB unhealthy" | `HealthyHostCount <= 0` |
| "phát hiện lỗi website TRƯỚC khi user báo" | CloudWatch Synthetics heartbeat canary |
| "chạy script định kỳ monitor endpoint/API" | CloudWatch Synthetics canary |
| "giám sát 1 process Linux, tự restart khi chết" | Agent `procstat` → Alarm → EventBridge → SSM Automation |
| "phát hiện anomaly VÀ khuyến nghị cho EKS" | Container Insights + DevOps Guru (📌 đóng khách mới 29/10/2026) |
| "agent chạy nhưng không có log" | IAM role thiếu `logs:*` |
| "chuẩn hóa cấu hình CloudWatch Agent cho fleet" | Parameter Store + State Manager + `AmazonCloudWatch-ManageAgent` |
| "dashboard tự gồm instance mới của ASG" | Metrics Explorer lọc tag `aws:autoscaling:groupName` |
| "top N IP trong flow log đã ở CloudWatch Logs" | CloudWatch Logs Insights |
| "phân tích log truy cập S3/CloudFront/ELB" | Bật access logs → Athena |
| "bytes theo từng target ALB" | ALB access logs + Athena `GROUP BY target:port` |
| "ai gọi API nào (KMS, ELB, IAM...)" | CloudTrail |
| "ai đã thay đổi cấu hình resource" | Config configuration timeline (+ CloudTrail) |
| "log CloudTrail không được sửa/xóa mà không bị phát hiện" | Log File Integrity Validation (+ MFA Delete) |
| "check compliance + auto remediation" | Config managed rule + SSM Automation remediation |
| "đánh giá khi tạo resource và mỗi X giờ" | Một Config rule với cả hai trigger |
| "gọi HTTPS endpoint ngoài từ event, ít code" | EventBridge API Destinations + Input Transformer |
| "tự tạo incident ServiceNow khi SG đổi" | EventBridge (CloudTrail API call) → `AWS-CreateServiceNowIncident` |
| "vá OS/app hàng loạt" | SSM Patch Manager |
| "delay patch khác nhau dev/prod, 1 khung giờ" | 2 patch baseline + 2 patch group + 1 maintenance window |
| "thu thập metadata phần mềm/cấu hình" | SSM Inventory |
| "cài phần mềm khi EC2 khởi động" | SSM State Manager association |
| "phân phối .msi tự cập nhật" | SSM Distributor + `AWS-ConfigureAWSPackage` |
| "script khẩn cấp trên hàng trăm EC2 + báo cáo" | SSM Run Command (target tag, command history) |
| "recover EC2 impaired/unreachable" | `AWSSupport-ExecuteEC2Rescue` |
| "debug 403 trên S3 bucket public" | `AWSSupport-TroubleshootS3PublicRead` |
| "Session Manager vào máy này không được" | Thiếu `AmazonSSMManagedInstanceCore` |
| "Session Manager không đi Internet" | Endpoint `ssm` · `ssmmessages` · `ec2messages` |
| "onboard cả Organization vào SSM" | Quick Setup – Default Host Management Configuration |
| "quản lý server on-prem bằng SSM, không credential dài hạn" | SSM Hybrid Activation |
| "deploy IAM role cho nhiều account" | CloudFormation StackSets + Organizations |
| "dùng output stack A trong stack B" | `Export` + `Fn::ImportValue` |
| "component lặp lại nhiều template" | Nested Stacks |
| "khóa resource critical khi update stack" | Stack Policy |
| "giữ EC2 debug khi stack fail" | `OnFailure=DO_NOTHING` |
| "stack báo xong quá sớm" | Thiếu WaitCondition/CreationPolicy |
| "custom resource chạy trước EC2" | `DependsOn` |
| "giữ snapshot EBS khi xóa stack" | `DeletionPolicy: Snapshot` |
| "giữ EC2 và dữ liệu khi xóa stack" | `DeletionPolicy: Retain` |
| "xóa stack không báo lỗi cũng không xóa" | Termination Protection đang bật |
| "DELETE_FAILED vì SG bị tham chiếu" | Xóa lại stack, Retain resource đó |
| "credentials RDS trong template, auto-rotation" | `AWS::SecretsManager::Secret` + `{{resolve:secretsmanager}}` |
| "template fail ở Region khác" | AMI region-specific / service chưa có |
| "test nâng version DB engine với dữ liệu production" | Snapshot → restore instance mới → upgrade |
| "Lambda báo too many connections tới RDS" | RDS Proxy |
| "RDS hay hết dung lượng, ít đổi app" | RDS Storage Auto Scaling |
| "automated backup không chạy" | Instance `STORAGE_FULL` |
| "share snapshot RDS đã mã hóa" | Sửa key policy KMS + share snapshot |
| "ép RDS PostgreSQL chỉ nhận SSL" | `rds.force_ssl=1` |
| "app hard-code endpoint Aurora replica" | Reader endpoint |
| "khôi phục bảng bị DROP trên Aurora PostgreSQL" | PITR → cluster mới |
| "Aurora `FreeLocalStorage` cạn" | Nâng instance class |
| "Redis HA RTO thấp nhất" | Replication group Multi-AZ + auto failover |
| "Redis evict ngẫu nhiên, giữ item phổ biến, không tăng chi phí" | `allkeys-lfu` |
| "chống xóa nhầm bảng DynamoDB + giảm mất dữ liệu" | Deletion Protection + PITR |
| "cảnh báo TRƯỚC khi DynamoDB throttle" | Alarm `ConsumedReadCapacityUnits` gần provisioned |
| "Lambda không kết nối được DB private subnet" | Cấu hình Lambda vào VPC |
| "Lambda chạy nhưng không có log" | `AWSLambdaBasicExecutionRole` |
| "Lambda chậm với tác vụ CPU" | Tăng memory |
| "dừng ngay vòng lặp đệ quy Lambda" | Reserved concurrency = 0 |
| "SQS cần ưu tiên message" | Hai queue riêng |
| "chuyển SQS Standard → FIFO" | Tạo queue FIFO mới + Message Group ID |
| "scale ASG theo số message SQS" | Metric math message/instance + target tracking |
| "healthy trong ASG nhưng unhealthy trong ALB" | ASG đang dùng EC2 status check |
| "đóng băng ASG để troubleshoot" | Suspend Launch + Terminate |
| "giảm thời gian scale-out do boot script lâu" | ASG Warm Pool |
| "On-Demand base + Spot, ít công" | 1 ASG + launch template + mixed instances |
| "giữ N instance khi mất 1 AZ" | N ÷ (AZ − 1) × AZ |
| "user bị logout liên tục sau ALB" | Sticky Sessions |
| "lấy IP thật của client sau LB" | `X-Forwarded-For` |
| "hàng triệu request/giây, latency thấp" | Network Load Balancer |
| "`SpilloverCount > 0`" | Scale theo `SurgeQueueLength` |
| "trình duyệt cũ lỗi SSL với ALB" | Đổi Security Policy của ALB |
| "ép HTTPS không phá user cũ" | Listener 80 redirect → 443 |
| "chỉ cho EC2 nhận traffic từ LB" | SG EC2 tham chiếu SG của LB |
| "chỉ phục vụ user Úc" | Route 53 Geolocation |
| "trỏ domain của tôi sang domain bên thứ ba" | CNAME |
| "`www` → CloudFront, rẻ nhất" | Alias |
| "failover đa Region bằng DNS" | Alias + Failover + Evaluate Target Health |
| "failover cho server on-prem" | A record + HTTP health check |
| "on-prem query private hosted zone" | Route 53 Resolver INBOUND endpoint |
| "VPC account A dùng private hosted zone của B" | B `CreateVPCAssociationAuthorization` → A associate |
| "cần 2 static IP trước ALB" | Global Accelerator |
| "failover đa Region NHANH NHẤT" | Global Accelerator |
| "T2 bị throttle CPU" | T2/T3 Unlimited hoặc đổi instance type |
| "license theo socket/core" | Dedicated Host |
| "single-tenant rẻ nhất" | Dedicated Instance |
| "Spot nhiều type, ưu tiên capacity" | `capacity-optimized` |
| "stateless, tối thiểu N instance, rẻ" | Spot Fleet với On-Demand base = N |
| "EC2 → Fargate trong 6 tháng" | Compute Savings Plans, No Upfront |
| "AMI hằng tuần + quét lỗ hổng" | EC2 Image Builder |
| "`InsufficientInstanceCapacity`" | Thử AZ khác |
| "`InstanceLimitExceeded`" | Xin tăng quota |
| "immediately full performance after restore" | Fast Snapshot Restore ở AZ đích |
| "EBS mới từ snapshot chậm lần đọc đầu" | Initialize/pre-warm hoặc FSR |
| "`BurstBalance` = 0 trên gp2, cần 3,000 IOPS, rẻ nhất" | Đổi sang gp3 |
| "cấm đổi volume type, cần thêm IOPS gp2" | Tăng dung lượng |
| "block storage + POSIX + toàn bộ data ở on-prem" | Storage Gateway Stored Volume |
| "thay NFS/SMB on-prem, dùng S3, cache local" | S3 File Gateway |
| "Tape Gateway Not Enough Space" | Tape auto-create |
| "thư mục riêng cho từng client trên NFS theo IAM role" | EFS Access Points |
| "Windows NTFS + SMB + AD" | FSx for Windows File Server |
| "nhiều server Windows truy cập file đồng thời" | FSx for Windows Multi-AZ |
| "khôi phục vài file EFS, rẻ nhất" | AWS Backup partial restore |
| "backup tập trung, đa account, cross-Region" | AWS Backup + Organizations + tag |
| "EC2 + RDS nhiều mức retention" | AWS Backup (nhiều backup rule) |
| "RTO < 10 phút và RPO = 0" | Active-Active đa Region + Route 53 health check |
| "RTO = RPO = 15 phút, rẻ nhất" | Aurora Global Database + ASG min=max=desired=1 |
| "chia sẻ file cho khách / truy cập tạm" | S3 Pre-signed URL |
| "chỉ subscriber xem nội dung CloudFront" | Signed URL / Signed Cookie |
| "WORM, immutable, chứng minh compliance" | S3 Object Lock Compliance (đề cũ: Glacier Vault Lock) |
| "chặn xóa object 3 tháng, kể cả root" | Object Lock Compliance mode |
| "xóa object bị Governance mode" | `s3:BypassGovernanceRetention` + header bypass |
| "ép mọi object upload phải mã hóa" | S3 Default Encryption |
| "khóa do khách tự giữ, S3 mã hóa hộ" | SSE-C (📌 bucket mới chặn mặc định từ 04/2026) |
| "cần audit trail ai dùng key khi nào" | SSE-KMS |
| "xoay khóa mã hóa hằng năm, ít công" | SSE-KMS + customer managed key + automatic rotation |
| "hàng trăm app dùng chung bucket, mỗi app một quyền" | S3 Access Points |
| "cấm public S3 toàn account" | Block Public Access cấp account |
| "static website 403" | Tắt Block Public Access + bucket policy `s3:GetObject` |
| "replication cross-account Access Denied" | Change object ownership to destination |
| "danh sách object replication thất bại hằng ngày" | S3 Inventory (event real-time → S3 RTC) |
| "kiểm tra trạng thái mã hóa mọi object" | S3 Inventory |
| "dọn incomplete multipart upload + xem dung lượng" | Lifecycle `AbortIncompleteMultipartUpload` + Storage Lens |
| "upload 500k file vào S3 cùng Region, throughput thấp" | Nhiều prefix + ghi song song |
| "upload nhiều GB từ toàn cầu" | Transfer Acceleration + Multipart Upload |
| "PB-scale, thiếu băng thông" | Snowball Edge → S3 → Glacier (📌 khách mới: Data Transfer Terminal/DataSync) |
| "90 ngày ms + HA, archive, lấy lại < 5 giờ" | Standard-IA → Glacier Flexible Retrieval |
| "user mobile upload lên S3" | Cognito Identity Pool |
| "ép MFA cho hành động nhạy cảm" | Condition `aws:MultiFactorAuthPresent` |
| "ép MFA cho AWS CLI" | `aws sts get-session-token` |
| "mất thiết bị MFA của root" | Xác minh email + số điện thoại |
| "sinh IAM policy least-privilege" | CloudTrail + IAM Access Analyzer policy generation |
| "chặn Region, chặn root, chặn xóa CloudTrail" | Control Tower + SCP |
| "tách môi trường theo phòng ban" | Organizations + OU |
| "Identity Center + AD on-prem, ít vận hành" | AD Connector |
| "chỉ dùng cấu hình EC2 đã phê duyệt" | Service Catalog + Launch Constraint |
| "xoay mật khẩu master RDS tự động" | Secrets Manager rotation |
| "tạo lại secret cùng tên ngay" | `ForceDeleteWithoutRecovery` |
| "khôi phục KMS key đã lên lịch xóa" | Cancel key deletion (trong 7–30 ngày) |
| "cert CloudFront tự gia hạn" | ACM + DNS validation, `us-east-1` |
| "cảnh báo cert sắp hết hạn" | EventBridge `aws.acm` (`DaysToExpiry`) → SNS |
| "chứng minh không còn lỗ hổng chưa vá" | Amazon Inspector |
| "quét CIS Benchmark toàn Organization" | Security Hub (CSPM) + auto-enable |
| "chặn 1 IP độc hại trước ALB" | WAF IP set + Web ACL BLOCK |
| "chặn 1 IP ở tầng mạng" | NACL DENY |
| "DDoS: đội 24/7 + hoàn phí" | Shield Advanced (SRT) |
| "liệt kê SG mở 22/3389 ra Internet" | Trusted Advisor – Security Groups Specific Ports Unrestricted |
| "IPv6 outbound only" | Egress-only Internet Gateway |
| "private subnet ra Internet (IPv4)" | NAT Gateway + route `0.0.0.0/0` |
| "IP tĩnh cho outbound của hàng trăm EC2" | NAT Gateway mỗi AZ |
| "gọi AWS API không qua Internet" | Interface VPC Endpoint |
| "truy cập S3 riêng tư, không thêm phí" | Gateway VPC Endpoint |
| "PPS cao / nghẽn NIC" | Enhanced Networking (ENA) |
| "dedicated + encrypted + low latency" | Direct Connect + VPN |
| "on-prem tải S3 ổn định, rẻ nhất" | 1 Direct Connect + Public VIF |
| "VPN tunnel UP nhưng không thông" | Route Propagation cho VGW |
| "subnet mới mất kết nối on-prem" | Thêm route tới VGW |
| "chỉ traffic tới VPC đi qua Client VPN" | Split-tunnel |
| "flow log: inbound ACCEPT, outbound REJECT" | Sửa outbound NACL (ephemeral) |
| "flow log không có log nào" | IAM role thiếu `logs:*` |
| "website qua ELB không vào được, đã mở 80" | NACL outbound 1024–65535 |
| "TTL khác nhau cho từng trang CloudFront" | Origin trả `Cache-Control: max-age` |
| "xóa file khỏi CloudFront ngay" | Invalidation |
| "cache hit ratio thấp" | Tăng TTL + Origin Shield |
| "CloudFront + login lại liên tục" | Forward cookie |
| "alert khi chi phí đạt X% ngưỡng, lọc theo loại" | AWS Budgets |
| "chi phí theo phòng ban / từng bucket" | Cost Allocation Tags (activate) + Cost Explorer |
| "báo cáo chi phí tới từng giờ" | CUR + Athena + Quick Sight |
| "right-size EBS/Lambda, ít công" | AWS Compute Optimizer |
| "cảnh báo khi dùng tới X% service quota" | Service Quotas → CloudWatch alarm → SNS |
| "tìm EC2 dùng thấp để dọn" | Trusted Advisor Low Utilization EC2 → SNS → Lambda |
| "kubectl không kết nối EKS" | `aws eks update-kubeconfig` |
| "EKS quá tải do nhiều request" | Horizontal Pod Autoscaler |

---

## 44. Con số cần thuộc & các kết hợp BẤT KHẢ THI

### Con số phải nhớ chính xác

| Hạng mục | Con số |
| --- | --- |
| Basic / detailed monitoring EC2 | 5 phút / 1 phút |
| High-resolution metric / alarm | 1 giây / tối thiểu 10 giây |
| ELB metric | Chu kỳ 60 giây |
| CLB surge queue | Tối đa 1,024 request |
| CloudTrail Event history | 90 ngày |
| CloudTrail digest file | Mỗi giờ |
| Config baseline cho resource bị xóa gián tiếp | 6 giờ |
| RDS Enhanced Monitoring log | Giữ 30 ngày (mặc định) |
| RDS automated backup | 1–35 ngày |
| Aurora replica | Tối đa 15 |
| Aurora Auto Scaling target ví dụ | 195 nếu sự cố ở 200 connection |
| RDS Proxy giảm downtime failover | Tới ~66% (AWS công bố) |
| DynamoDB PITR | 1–35 ngày |
| Lambda memory | 128–10,240 MB; 1,769 MB ≈ 1 vCPU |
| Lambda timeout | 15 phút |
| ASG cooldown mặc định | 300 giây |
| HealthCheckGracePeriod | 300 giây (Console), 0 (CLI/SDK) |
| ASG phình khi suspend Terminate + AZRebalance | Tới 10% trên max |
| WaitCondition timeout tối đa | 43,200 giây (12 giờ) |
| KMS key deletion waiting period | 7–30 ngày, mặc định 30 |
| Secrets Manager recovery window | 7–30 ngày, **mặc định 30** (📌 đề ghi 7) |
| KMS rotation | AWS managed: hằng năm; customer managed: mặc định 365 ngày, chỉnh được |
| ACM Approaching Expiration event | Từ 45 ngày (private/imported), 30 ngày (public) |
| S3 request rate mỗi prefix | 3,500 PUT/COPY/POST/DELETE · 5,500 GET/HEAD /giây |
| S3 RTC | 99.99% object trong 15 phút |
| Multipart upload nên dùng | Từ 100 MB |
| Glacier Flexible Retrieval | Expedited 1–5 phút · Standard 3–5 giờ · Bulk 5–12 giờ |
| Glacier Deep Archive | 12–48 giờ |
| gp2 | 3 IOPS/GiB, 100–16,000 IOPS |
| gp3 | Baseline 3,000 IOPS + 125 MiB/s; tối đa 80,000 IOPS (📌 đề ghi 16,000) |
| io1 | Tối đa 50 IOPS/GiB |
| EBS Multi-Attach | io1/io2, 16 instance Nitro, 1 AZ |
| Subnet reserved IP | 5 mỗi subnet; `/27` = 27, `/26` = 59 |
| NAT Gateway | 5 → 100 Gbps (📌 đề ghi 45); 55,000 kết nối/đích/IP |
| Ephemeral port ELB/NAT/Lambda | 1024–65535 |
| Ephemeral port Linux | 32768–61000 |
| Ephemeral port Windows 2008+ | 49152–65535 |
| Global Accelerator | 2 anycast IP tĩnh |
| CloudFront default TTL | 24 giờ |
| Spot giảm giá | Tới ~90% |

### Kết hợp BẤT KHẢ THI → loại ngay

- **SQS làm target của CloudWatch Alarm.**
- **SNS trực tiếp khởi chạy SSM Automation** (phải qua EventBridge/Lambda).
- **EventBridge nhận giá trị metric** để so ngưỡng (phải qua Alarm).
- **Security Group có rule Deny**; **SG gắn vào CloudFront/S3/DynamoDB**.
- **WAF gắn EC2, NLB, S3**.
- **Global Accelerator/NLB có S3 làm endpoint**; **ALB gắn Elastic IP**.
- **ALB route sang Region khác**; **ASG trải nhiều Region**.
- **CNAME ở zone apex**; **Alias trỏ server on-premises**; **A record trỏ CloudFront**.
- **Gateway Endpoint cho dịch vụ ngoài S3/DynamoDB**; **gateway endpoint truy cập từ on-prem**.
- **Private VIF truy cập S3 public endpoint**.
- **DLM backup RDS/EFS**; **RDS automated backup giữ 7 năm**.
- **Backtrack trên Aurora PostgreSQL**; **PITR rollback tại chỗ**.
- **Cross-account RDS read replica**; **KMS key share sang account khác bằng cách tạo key cùng alias**.
- **Share AMI mã hóa bằng AWS managed key**; **AMI mã hóa để public**.
- **`DeletionPolicy: Snapshot` cho EC2**; **condition dùng trong `Parameters`**; **`Ref` lấy giá trị stack khác**.
- **Spot interruption = Reboot**; **`DisableApiTermination` cho Spot**.
- **Đổi SQS Standard thành FIFO tại chỗ**.
- **Thu nhỏ cache disk Storage Gateway**; **khôi phục gateway từ AMI/hypervisor snapshot**.
- **Mount EFS từ Windows**; **launch template tạo EFS**.
- **Instance profile gắn server on-premises**.
- **S3 website endpoint dùng HTTPS tới origin / dùng OAI**.
- **Object Lock Compliance bị bypass, kể cả root**.
- **MFA Delete bật từ Console hoặc bởi IAM user**.
- **ACM cert cho ALB ở Region khác với ALB**; **ACM cert cho CloudFront ngoài `us-east-1`**.
- **Permissions boundary hạn chế root user**; **SCP cấp quyền**.
- **Instance store-backed đổi instance type tại chỗ**.

---

### Nguồn đã đối chiếu (các ghi chú 📌 và ❓)

- SOA-C03 exam guide (5 domain và tỷ trọng): https://docs.aws.amazon.com/aws-certification/latest/sysops-administrator-associate-03/sysops-administrator-associate-03.html
- CloudWatch alarm actions (Lambda, Incident Manager, investigations): https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/alarm-actions.html
- Invoke a Lambda function from an alarm: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/alarms-and-actions-Lambda.html
- EventBridge built-in targets cho EC2 (Reboot/Stop/Terminate/CreateSnapshot): https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-events-iam-roles.html
- AWS Health API (gói Support yêu cầu, `SubscriptionRequiredException`): https://docs.aws.amazon.com/health/latest/ug/health-api.html
- AWS Health Dashboard (account health / service health): https://docs.aws.amazon.com/health/latest/ug/aws-health-concepts-and-terms.html
- AWS Support plans (Business Support+, Unified Operations; Developer ngừng 01/01/2027): https://docs.aws.amazon.com/awssupport/latest/user/aws-support-plans.html
- Amazon DevOps Guru end of support (đóng khách mới 29/10/2026, ngừng 30/09/2027): https://docs.aws.amazon.com/devops-guru/latest/userguide/devops-guru-end-of-support.html
- Services in full shutdown (Inspector Classic 20/05/2026): https://docs.aws.amazon.com/general/latest/gr/full_shutdown_services.html
- X-Ray SDK & Daemon support timeline: https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-daemon-timeline.html
- CloudTrail Lake availability change (31/05/2026): https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake-service-availability-change.html
- RDS Database Insights (thay vai trò Performance Insights): https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.Overview.html
- Aurora MySQL version 1 end of life (28/02/2023): https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.MySQL56.EOL.html
- ElastiCache backups (Valkey, Redis OSS, Memcached serverless): https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/backups.html
- DynamoDB resource-based policies: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/access-control-resource-based.html
- DynamoDB PITR 1–35 ngày: https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_PointInTimeRecoverySpecification.html
- Lambda quotas (memory, 1,769 MB ≈ 1 vCPU): https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
- SQS FIFO quotas: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-fifo.html
- S3 Object Ownership mặc định Bucket owner enforced: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-ownership-new-bucket.html
- Default SSE-C setting for new buckets (04/2026): https://docs.aws.amazon.com/AmazonS3/latest/userguide/default-s3-c-encryption-setting-faq.html
- S3 Select không nhận khách mới: https://docs.aws.amazon.com/AmazonS3/latest/API/API_SelectParameters.html
- S3 Object Lambda availability change: https://docs.aws.amazon.com/AmazonS3/latest/userguide/amazons3-ol-change.html
- Glacier vault không nhận khách mới: https://docs.aws.amazon.com/amazonglacier/latest/dev/creating-vaults-console.html
- EBS volume types (gp3 80,000 IOPS): https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- DLM system tags: https://docs.aws.amazon.com/ebs/latest/userguide/dlm-elements.html
- Snowball Edge availability change: https://docs.aws.amazon.com/snowball/latest/developer-guide/snowball-edge-availability-change.html
- Storage Gateway Hardware Appliance end of availability: https://docs.aws.amazon.com/filegateway/latest/files3/hardware-appliance.html
- FSx File Gateway không nhận khách mới: https://docs.aws.amazon.com/filegateway/latest/filefsxw/use-fsxw-gateway.html
- Launch configurations (account từ 01/10/2024): https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-configurations.html
- Capacity Reservations trong placement group (chỉ cluster): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/cr-cpg.html
- Spot Instance interruptions: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html
- Network Load Balancer availability zones (gỡ subnet): https://docs.aws.amazon.com/elasticloadbalancing/latest/network/availability-zones.html
- NAT gateway basics (5 → 100 Gbps): https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-basics.html
- NAT gateway 55,000 kết nối mỗi đích: https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-working-with.html
- Regional NAT gateways: https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateways-regional.html
- Elastic IP pricing (mọi public IPv4 tính phí): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html
- Systems Manager endpoints (`ssm`, `ssmmessages`, `ec2messages`): https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-create-vpc.html
- KMS automatic rotation (AWS managed key hằng năm, rotation period): https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-enable.html
- KMS on-demand rotation (gồm imported key material): https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-on-demand.html
- Secrets Manager DeleteSecret (mặc định 30 ngày): https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_DeleteSecret.html
- ACM EventBridge events (45/30 ngày): https://docs.aws.amazon.com/acm/latest/userguide/supported-events.html
- ACM exportable public certificates: https://docs.aws.amazon.com/acm/latest/userguide/acm-exportable-certificates.html
- WAF resources: https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html
- Security Hub CSPM — CIS AWS Foundations Benchmark: https://docs.aws.amazon.com/securityhub/latest/userguide/cis-aws-foundations-benchmark.html
- CloudFront OAC (OAI legacy): https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html
- CloudFront VPC origins: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-vpc-origins.html
- Control Tower controls: https://docs.aws.amazon.com/controltower/latest/controlreference/controls.html
- Shield Response Team: https://docs.aws.amazon.com/waf/latest/developerguide/ddos-srt-support.html
- Amazon Quick Sight: https://docs.aws.amazon.com/quick/latest/userguide/quick-bi.html
