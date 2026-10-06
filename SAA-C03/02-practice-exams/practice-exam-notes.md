# AWS SAA — Sổ tay cốt lõi & kiến thức dễ nhầm

> Tổng hợp toàn bộ ý tưởng cốt lõi + kiến thức liên quan/dễ nhầm, theo nhóm dịch vụ.
> **🎯 = ý tưởng cốt lõi cần khắc sâu** · **⚠️ = kiến thức liên quan / dễ nhầm trong đề** · **📌 = cập nhật theo docs AWS hiện hành (10/2026)**.
> Xem **mục 42 (Phụ lục)** để tra nhanh các dịch vụ hay xuất hiện ở đáp án SAI (distractor) và lý do sai.
>
> **Ôn cấp tốc trước khi thi:** §45 (Nguyên tắc chung) → §46 (Bảng so sánh dễ nhầm) → §47 (Từ khóa → đáp án) → §48 (Con số & kết hợp bất khả thi).
>
> **Khi có đề mới:** chèn kiến thức vào đúng mục dịch vụ (§1–§41, §44), cập nhật §42 và §45–§48. Không tạo phần riêng cho từng đề.

---

## 1. AWS Config

🎯 Config **không chỉ tracking** thay đổi. Sức mạnh thật là **Config Rules** — đánh giá tài nguyên có **TUÂN THỦ** không (COMPLIANT / NON_COMPLIANT). Nhớ: *"resource-specific history, audit & compliance"* → **Config**.

🎯 Config Rule chạy theo **lịch định kỳ (periodic)** được → xử lý cả thứ "không phải thay đổi" như certificate sắp hết hạn. *"least scripting/maintenance"* → **Managed Rule**.

⚠️ **Bộ tứ giám sát — hỏi theo "câu đề đặt ra":**

| Câu hỏi đề | Dịch vụ |
| --- | --- |
| Cấu hình có tuân thủ chuẩn không / lịch sử cấu hình | **AWS Config** |
| AI gọi API nào, khi nào (account activity/audit) | **CloudTrail** |
| Hiệu năng tài nguyên (CPU, latency), alarms | **CloudWatch** |
| Phát hiện mối đe dọa / hành vi bất thường | **GuardDuty** |
| Quét lỗ hổng (vulnerability) | **Inspector** |

⚠️ **IaC / template chuẩn tái sử dụng** (vd chuẩn hóa cấu hình RDS cho cả team) → **CloudFormation** (không phải Config/Trusted Advisor/Lambda-scan — những cái này chỉ phát hiện/khuyến nghị, không tạo template tái dùng).

---

## 2. Dịch vụ bảo mật / phát hiện (GuardDuty / Macie / Inspector / Shield)

🎯 Giám sát hoạt động **độc hại** trên S3/tài khoản → **GuardDuty**. Phát hiện dữ liệu **nhạy cảm/PII** trên S3 → **Macie**. Hai vai trò khác nhau, dễ bị hoán đổi trong đáp án.

🎯 Chống **DDoS + có đội phản ứng DDoS + logging audit**, ít đổi kiến trúc → **AWS Shield Advanced**. GuardDuty **không** mitigate DDoS (chỉ phát hiện); Inspector chỉ quét lỗ hổng; CloudFront+WAF đòi đổi kiến trúc.

📌 Đội DDoS Response Team (DRT) nay gọi là **Shield Response Team (SRT)**.

🎯 **EC2 query IP liên quan đào cryptocurrency / hành vi bất thường** → **GuardDuty** phát hiện (vd finding `CryptoCurrency:EC2/BitcoinTool.B`). WAF/Shield/Firewall Manager không phát hiện được hành vi này.

🎯 **Chặn kẻ tấn công gửi quá nhiều request/giây (rate abuse)** → **WAF rate-based rule** (đếm request theo IP, chặn khi vượt ngưỡng). Shield Advanced **không** có rate-based rule; NACL chỉ chặn IP tĩnh; sticky session không liên quan.

🎯 **Quản trị tập trung firewall rule qua nhiều account (Organizations)** → **AWS Firewall Manager**. Cấu hình rule được trên: **WAF, Shield Advanced, VPC Security Group, Network Firewall, Route 53 Resolver DNS Firewall**. KHÔNG áp cho: NACL, Route Table, GuardDuty, Inspector.

⚠️ **Phân biệt nhanh:**

| Dịch vụ | Vai trò |
| --- | --- |
| **GuardDuty** | Phát hiện mối đe dọa (malicious IP, anomaly) từ CloudTrail/VPC Flow/DNS logs |
| **Macie** | ML phát hiện & phân loại PII trên S3. KHÔNG enforce quyền |
| **Inspector** | Quét lỗ hổng (vulnerability) EC2/container/Lambda |
| **Shield** (Standard free / Advanced) | Chống DDoS. Advanced có SRT + báo cáo audit |
| **WAF** | Lọc request L7 (IP, SQLi, XSS) cho CloudFront/ALB/API GW |
| **IAM Access Analyzer** | Phát hiện resource (S3/IAM role/KMS/SNS) **chia sẻ ra ngoài** account/Org (dùng automated reasoning trên policy) |

🎯 **Tìm resource (S3/IAM role/KMS key/SNS) bị chia sẻ ngoài account/Org, đánh giá policy** → **IAM Access Analyzer**. **IAM Access Advisor** khác: chỉ cho biết service nào **được truy cập lần cuối** (để bỏ quyền thừa), KHÔNG phân tích external access. Inspector = lỗ hổng phần mềm (không phân tích policy). Config = theo dõi thay đổi cấu hình (không suy luận external access).

---

## 3. ACM & Certificate

🎯 *"Third-party/imported certificate"* → ACM **không tự gia hạn** (chỉ ACM-issued auto-renew).

🎯 Cảnh báo certificate sắp hết hạn + ít công sức → **AWS Config managed rule** (`acm-certificate-expiration-check`) thắng CloudWatch alarm (một rule quản tất cả cert).

---

## 4. CloudWatch

🎯 Config đánh giá theo **loại tài nguyên** (áp cả nhóm). CloudWatch alarm giám sát **từng metric/tài nguyên** (cấu hình lẻ). Bẫy: **memory/disk KHÔNG phải metric mặc định** của EC2 (cần **CloudWatch Agent**). CloudWatch = hiệu năng, Config = tuân thủ.

### EC2 auto-recovery

🎯 EC2 đơn lẻ tự recover (không ASG) → **CloudWatch alarm + recover action**. 2 ràng buộc: chỉ recover **SYSTEM status check** (lỗi hạ tầng AWS), instance **phải dùng EBS** (instance store không hỗ trợ).

🎯 **Instance bị "freeze/treo" định kỳ, cần tự reboot rẻ nhất** → **CloudWatch alarm + REBOOT action** (dùng cho **Instance status check** failure — lỗi OS/config của bạn). Đừng dùng EventBridge/Lambda gọi API reboot (lãng phí). Phân biệt: **Reboot** = Instance check fail; **Recover** = System check fail.

🎯 Recovered instance **identical**: giữ instance ID, private IP, Elastic IP, metadata, **và giữ public IPv4**. Nhưng **dữ liệu in-memory bị MẤT** (migrate qua reboot). Terminated instance **không recover** được.

⚠️ **System status check** = lỗi hạ tầng AWS → recover được. **Instance status check** = lỗi OS/config của bạn → phải reboot/sửa. EventBridge **không** trigger recover EC2 trực tiếp. Trusted Advisor chỉ **khuyến nghị**.

⚠️ **Metric mịn hơn (≤1 phút)** → bật **Detailed Monitoring** trên EC2 (1 phút thay vì 5 phút mặc định). Ít setup nhất, native. CloudWatch Agent = cần cài + cấu hình (chỉ khi cần memory/disk/custom metric hoặc high-resolution).

⚠️ **Giữ log trước khi ASG terminate instance** (debug về sau) → cài agent đẩy log lên **CloudWatch Logs** liên tục (không mất khi instance chết). Snapshot/SSH/Lambda copy = tốn công, không elastic.

📌 Thu thập log và metric từ EC2/on-prem → **unified CloudWatch agent** (gom cả log lẫn metric).

### CloudWatch Logs vs CloudTrail Logs

🎯 **CloudTrail Logs** = "ai làm gì" (lời gọi API) → audit. **CloudWatch Logs** = "app/hệ thống nói gì" (log nội dung) → vận hành/debug. Kết hợp: CloudTrail → CloudWatch Logs → Metric Filter + Alarm để cảnh báo real-time.

🎯 **Cảnh báo near-real-time khi có nhiều API call bất thường/lỗi (illegal API queries)** → đẩy **CloudTrail logs vào CloudWatch Logs → tạo Metric Filter** (đếm error codes) **→ CloudWatch Alarm → SNS** báo team. CloudTrail **KHÔNG stream trực tiếp vào Kinesis** (chỉ S3 & CloudWatch Logs). Athena+QuickSight = báo cáo, không near-real-time. Trusted Advisor alarm chỉ kích hoạt khi **chạm service limit** (không phát hiện pattern bất thường).

⚠️ **CloudTrail Insights** = tự động phát hiện hoạt động write API bất thường so với baseline của account (bổ trợ cho metric filter thủ công).

---

## 5. EventBridge

🎯 EventBridge = event bus: bắt sự kiện → Rule → Target. Rule 2 kiểu: **pattern nội dung** (event-driven) hoặc **lịch/cron** (scheduled).

🎯 **Decouple + tích hợp SaaS bên thứ ba** → **EventBridge** (dịch vụ event-based DUY NHẤT tích hợp trực tiếp SaaS partners). SNS không tích hợp SaaS; SQS/ELB không hợp.

🎯 Cron job serverless định kỳ (vd chạy Python 5 phút mỗi tuần) → **EventBridge cron + Lambda**. Glue = ETL (đắt hơn); Spot = không đảm bảo chạy đúng giờ; Scheduled Reserved Instance = không serverless.

🎯 **Cron job container CHẠY LÂU (>15 phút, vd 30 phút) + serverless + ít refactor** → **EventBridge Scheduler → ECS Fargate task**. Lambda **không hợp** (timeout tối đa 15 phút + phải refactor script thành function). AWS Batch = nhiều setup/compute env; Step Functions Wait = phức tạp thừa cho cron đơn giản.

⚠️ **EventBridge vs SNS vs SQS:**

| Nhu cầu | Chọn |
| --- | --- |
| Phản ứng sự kiện AWS / lọc nội dung / SaaS / cron | **EventBridge** |
| Fan-out: 1 message → nhiều subscriber | **SNS** |
| Hàng đợi / đệm / decouple / xử lý sau | **SQS** |

---

## 6. Messaging — SQS

🎯 **Standard** = throughput gần vô hạn, at-least-once (có thể trùng), best-effort ordering. **FIFO** = đúng thứ tự + exactly-once (Message Group ID nhóm thứ tự, Deduplication ID chống trùng).

🎯 Giảm chi phí khi **nhận** message → **Long Polling** (chờ có message → giảm empty receive → giảm request tính phí). Short polling trả về ngay kể cả rỗng → đắt hơn.

🎯 Decouple + process nhanh/chậm khác nhau + lưu message tới khi xử lý → **SQS** (poll, đệm). SNS/EventBridge = push (cần consumer sẵn sàng ngay).

🎯 Async + có **retry** built-in + rẻ (kết hợp Spot Instances) → **SQS**. SNS không phải queue.

### Migration Standard → FIFO

🎯 **Không convert được** Standard → FIFO: phải **xóa và tạo lại** (hoặc tạo queue FIFO mới). Tên FIFO phải kết thúc **`.fifo`**. Throughput FIFO: **3,000 msg/s có batching** (batch tối đa 10 msg/op → 300 op/s × 10), 300 msg/s không batching.

📌 Con số 300 op/s và 3,000 msg/s chỉ đúng cho FIFO **không bật high throughput mode**. Bật **high throughput FIFO** thì tùy Region, cao nhất tới **70,000 TPS** (không batch) / **700,000 msg/s** (có batch) ở us-east-1, us-west-2, eu-west-1. Đề SAA vẫn hay ra toán theo con số 300.

🎯 **Toán batch FIFO** (đề hay hỏi): cần xử lý theo thứ tự ở đỉnh ~1,000 msg/s → FIFO **batch 4 msg/operation** = 300 op/s × 4 = **1,200 msg/s** (đủ). Batch 2 chỉ 600 msg/s (thiếu). FIFO không batch = 300 msg/s (thiếu). Standard queue = không đảm bảo thứ tự (loại).

🎯 **Xử lý theo thứ tự ĐỘC LẬP + nhiều consumer song song** (vd mỗi thiết bị/desktop một luồng) → **SQS FIFO + Message Group ID** (mỗi group giữ thứ tự riêng, nhiều consumer đọc song song theo group). FIFO không có Group ID → chỉ 1 consumer. Kinesis: số consumer bị giới hạn bởi **số shard** (ít hơn số producer) → không scale bằng SQS Group ID.

⚠️ **4 khái niệm timing SQS — dễ nhầm:**

| Khái niệm | Chức năng |
| --- | --- |
| **Long/Short Polling** | CÁCH nhận message (long rẻ hơn, chờ tối đa 20s) |
| **Visibility Timeout** | Ẩn message sau khi nhận, tránh xử lý trùng (default 30s, max 12h) |
| **Delay / Message Timer** | Trì hoãn message xuất hiện lần đầu (0–15 phút) |
| **Retention** | Giữ message bao lâu (default 4 ngày, min 1 phút, max 14 ngày) |

DLQ = message xử lý lỗi nhiều lần → chuyển sang để điều tra.

🎯 **Message xử lý FAIL nhiều lần** (vd order tham chiếu story đã bị xóa) → **Dead-Letter Queue (DLQ)** để cô lập/điều tra. KHÔNG dùng long/short polling (chỉ là cách nhận message) hay temporary queue (request-response pattern, tránh tạo queue cho mỗi response) để xử lý lỗi.

🎯 **Ưu tiên xử lý (vd user pro trước user lite)** → tạo **2 queue riêng** (một pro, một lite) + cấu hình consumer **poll queue pro trước**. Long/short polling hay visibility timeout **không** đổi được độ ưu tiên.

🎯 **Request-response high-throughput, tiết kiệm thời gian dev + chi phí** → **SQS Temporary Queues** (Temporary Queue Client, virtual queue, tự dọn dẹp khi không dùng, ít API call). DLQ/FIFO/delay queue không phục vụ mục đích này.

🎯 **Ingest data + xử lý fully serverless, không provision capacity, tự scale** → **SQS + Lambda** (poll theo batch) → DynamoDB. **Kinesis Firehose KHÔNG ghi trực tiếp vào DynamoDB** (chỉ S3/Redshift/OpenSearch/Splunk/HTTP endpoint…). Dùng EC2 để poll = không serverless (loại).

🎯 **Xử lý near-real-time thay cho batch đêm** (vd resize ảnh ngay khi upload, mỗi việc <15 phút) → **S3 Event Notification → SQS → Lambda** (buffer + xử lý bất đồng bộ, pay-per-use, tự scale). Rẻ & đơn giản hơn EC2 Reserved/Fargate/App Runner cho tác vụ ngắn thưa. Lambda hợp vì mỗi việc <15 phút + ít RAM.

🎯 **Buffer chống mất data khi tải tăng vọt (IoT/sensor) + cần replay** → đặt **SQS** trước consumer (đệm, at-least-once, DLQ). **SNS không poll được** (push, dễ mất nếu subscriber quá tải). Firehose→S3 = trễ + cần thêm Lambda.

📌 Kích thước message SQS tối đa **1 MiB**. Lớn hơn → Extended Client Library (lưu payload ở S3).

---

## 7. Messaging — SNS, Amazon MQ, Kinesis

🎯 Giữ **giao thức chuẩn** (RabbitMQ/ActiveMQ/AMQP/JMS/**MQTT**/STOMP) + migrate ít đổi code → **Amazon MQ**. App mới, scale khổng lồ, decouple → SQS/SNS.

🎯 Real-time streaming + retry + replay + nhiều consumer + xây app **xử lý/phân tích tùy biến** → **Kinesis Data Streams**. Firehose = chỉ **load** stream vào data store (S3/Redshift/OpenSearch), không build app xử lý tùy biến.

🎯 **`ProvisionedThroughputExceededException` do gửi record một-một tần suất cao** + giữ chi phí thấp → **batch record (PutRecords)** thay vì PutRecord trong vòng lặp (rẻ nhất, tối ưu shard). Tăng shard = tốn tiền; Exponential Backoff = chỉ hoãn tạm, không giải quyết gốc; giảm retention = mất data.

🎯 **Nhiều consumer đọc song song 1 stream nhưng bị lag** (mặc định 2MB/s/shard **chia sẻ** cho mọi consumer) → bật **Enhanced Fan-Out** (mỗi consumer có ống **2MB/s/shard RIÊNG**, tự scale theo số shard). Không đổi sang SQS/Firehose (mất khả năng multi-consumer streaming).

### Pipeline phân tích streaming real-time (không mất data)

🎯 Ingest → **Kinesis Data Streams** → phân tích real-time → **Kinesis Data Analytics** → xuất → **Kinesis Data Firehose** → lưu **S3**. QuickSight/Athena **không** phân tích streaming real-time (Athena query S3, QuickSight visualize).

📌 **Kinesis Data Analytics** nay là **Amazon Managed Service for Apache Flink**; bản **KDA for SQL** đã ngừng (không tạo mới từ 15/10/2025, xóa app từ 27/01/2026). **Kinesis Data Firehose** nay tên **Amazon Data Firehose**.

### Stream dữ liệu từ S3 → Kinesis (nhanh nhất, ít code)

🎯 Đẩy data hiện có + cập nhật từ **S3 → Kinesis Data Streams** nhanh nhất → dùng **AWS DMS** làm cầu nối (S3 = source, Kinesis = target, hỗ trợ full + CDC, không code). Lambda + S3 event = phải code nhiều.

⚠️ **Bảng chọn messaging/streaming:**

| Dịch vụ | Dùng khi |
| --- | --- |
| **SQS** | Hàng đợi, decouple, đệm, poll, xử lý bất đồng bộ |
| **SNS** | Pub/sub, push, fan-out 1→nhiều |
| **Amazon MQ** | Broker managed cho giao thức chuẩn (AMQP/JMS/MQTT), migration ít đổi code |
| **Kinesis Data Streams** | Streaming real-time, replay, nhiều consumer, xử lý tùy biến |
| **Kinesis Data Firehose** | Load stream vào S3/Redshift/OpenSearch (near real-time, không code) |
| **Kinesis Data Analytics** (nay: Managed Service for Apache Flink) | Query/phân tích stream real-time |
| **EventBridge** | Event-driven, lọc nội dung, cron, tích hợp SaaS |

---

## 8. Database — RDS

### Multi-AZ vs Read Replica — BẪY KINH ĐIỂN

🎯 **Multi-AZ** = AVAILABILITY (HA/failover), **SYNCHRONOUS**, no data loss / 2 node. **Read Replica** = SCALABILITY (scale đọc/report), **ASYNCHRONOUS**. Standby của Multi-AZ **không đọc được** (chỉ failover); Read Replica chỉ đọc.

🎯 **Tăng read throughput / báo cáo nặng làm chậm production** (dù CPU/mem chỉ 50%) → **Read Replica**, trỏ report tool vào replica. Không tăng size instance (không phải nghẽn tài nguyên), không dùng standby Multi-AZ (không đọc được).

🎯 No data loss + 2 node + synchronous → **RDS Multi-AZ**. Read replica luôn **async** (mọi đáp án "read replica synchronously replicate" đều SAI).

🎯 **SQL Server / bất kỳ engine, max availability, ít overhead** → **RDS Multi-AZ** (không tự host trên EC2). **Không có "cross-region Multi-AZ"** — Multi-AZ chỉ trong 1 Region.

⚠️ **Cơ chế failover Multi-AZ:** khi primary chết, RDS **tự động flip CNAME record** trỏ sang standby (được promote thành primary mới). **URL/endpoint KHÔNG đổi**, failover tự động (không cần admin), không cần đổi app. Không có email chờ can thiệp tay. Standby **không đọc được** khi bình thường.

### Giảm replica lag

🎯 Read replica lag lớn khi tải cao + ít đổi code → migrate **RDS MySQL → Aurora MySQL**, thay bằng **Aurora Replicas + Aurora Auto Scaling**. Aurora Replica **chia sẻ storage** với primary → lag ~10ms (vs giây của RDS). ElastiCache/DynamoDB = đổi nhiều code.

### Encryption / engine upgrade / storage / in-transit / auth

🎯 RDS encryption chỉ bật **lúc tạo**. Mã hóa DB chưa mã hóa → snapshot → copy có encrypt → restore → xóa DB gốc. Multi-AZ ≠ mã hóa. Replica không mã hóa được nếu master chưa mã hóa.

🎯 **Engine level upgrade** → primary+standby nâng **đồng thời** → CÓ downtime (dù Multi-AZ). OS/hardware maintenance → tuần tự qua failover → downtime tối thiểu. Hết storage + ít effort → bật **Storage Auto-Scaling**.

⚠️ **App báo "connection timed out" tới RDS** (nhưng bastion query được) → nguyên nhân là **Security Group của RDS chưa cho inbound từ SG của app server** (vấn đề mạng), KHÔNG phải sai username/password (sai credential sẽ báo "access denied", không phải timeout).

🎯 **Mã hóa data-in-transit** tới RDS → bật **SSL/TLS** (không phải IAM auth, không phải SG/NACL — bạn không SSH vào RDS).

🎯 **Short-lived credentials** Lambda/EC2 → RDS (MySQL/PostgreSQL) → **IAM database authentication** (token sống 15 phút) + gắn **IAM role** cho Lambda. Không cần lưu password.

### DR cho RDS (cross-region)

🎯 **DR cho RDS** → (1) **Cross-Region Read Replica** (promote thành standalone khi region chính hỏng) + (2) **Automated backup Multi-AZ có Cross-Region backup** (point-in-time recovery, backup chạy trên standby để giảm I/O primary). Provisioned IOPS = tăng hiệu năng, KHÔNG phải DR. **Database cloning chỉ có ở Aurora** (không có ở RDS).

### DMS Serverless (replication tải biến động)

🎯 Replicate DB (existing + CDC liên tục) với **tải biến động + tự cấp/scale compute, ít quản lý** → **AWS DMS Serverless** (tự provision & scale theo workload, hỗ trợ CDC). Glue = ETL batch (không CDC transactional); DMS trên EC2 + script auto-scale = nhiều overhead; Lambda = không hợp replication stateful khối lượng lớn.

### Migration SQL Server → Aurora PostgreSQL (ít refactor app)

🎯 Migrate SQL Server (app dùng **T-SQL**) → Aurora PostgreSQL, **ít sửa code app** → dùng **Babelfish for Aurora PostgreSQL** (Aurora hiểu T-SQL + SQL Server wire protocol) **+ SCT + DMS** (chuyển schema + data). Aurora **KHÔNG** tự "emulate SQL Server" nếu không bật Babelfish. Glue = ETL (không dịch T-SQL). Aurora Global Database = DR/read cross-region (không giúp compatibility).

### RDS Custom (truy cập host OS)

🎯 Cần **customize host OS + database** (patch đặc biệt, app third-party cần privileged access) + ít maintenance → **RDS Custom for Oracle/SQL Server** (Multi-AZ cho HA). RDS thường **không** cho truy cập host OS. Tự host trên EC2 = nhiều maintenance.

### RDS Proxy (connection pooling)

🎯 Aurora/RDS **timeout khi tải cao do quá nhiều connection** (flash sale) → đặt **RDS Proxy** giữa app và DB (pool + tái dùng connection, giảm áp lực). Kết hợp ASG + SQS để buffer burst.

### Cross-account private access tới RDS (không internet/DX/VPN)

🎯 VPC khác account truy cập RDS **riêng tư** (không internet, không DX, không VPN) → phía sở hữu DB tạo **NLB trước RDS + PrivateLink service**, phía kia tạo **Interface VPC Endpoint**. LƯU Ý: **không** dùng đồng thời Transit Gateway + VPC Peering cho cùng một cặp VPC; NAT Gateway không đặt được trong subnet không có IGW.

### Chia sẻ DB với account khác (auditor cần bản sao)

🎯 Chia sẻ RDS với **auditor account khác + họ cần BẢN SAO riêng** → tạo **encrypted snapshot + share snapshot + cấp quyền dùng KMS key** (thêm account kia vào KMS key policy). KHÔNG truy cập được snapshot trong S3 trực tiếp (AWS quản). Read replica ≠ bản sao độc lập; export ra text file = nhiều việc thừa.

### Managed vs auto-scaling DB (giảm overhead)

🎯 SQL Server nhạy cảm + max availability + ít quản lý → **RDS for SQL Server Multi-AZ + KMS encryption at-rest**. Tự host trên EC2 = nhiều overhead (patch OS/SQL). Timestream = time-series (không phải relational). Cần **customize host OS** thì mới dùng RDS Custom.

🎯 App MySQL cần **auto-scaling compute cho tầng DB + HA + managed + tải biến động** → **Aurora Serverless v2 (MySQL-compatible)** (tự scale capacity theo tải, Multi-AZ). RDS Multi-AZ = HA nhưng **không** auto-scale compute (phải resize tay). ElastiCache/Neptune ≠ thay thế MySQL relational.

⚠️ Replicate read replica cùng Region (kể cả khác AZ) → **FREE**; cross-Region → **CHARGED**. App **query** replica ở AZ khác (≠ replication) → tính phí liên-AZ.

---

## 9. Database — Aurora

🎯 Tách read khỏi write → **Aurora Replica + reader endpoint** (tự load-balance). Aurora **không có "standby"** (từ của RDS); Aurora Replica vừa đọc vừa failover. Restore từ **automated backup** để tạo dev DB không đụng production.

🎯 **Giảm chi phí + tăng read cho REST API/web read-heavy, data ít đổi, chấp nhận stale** → thường là **API Gateway caching** hoặc **ElastiCache** trước Aurora, chứ không thêm Aurora Replica (đắt hơn).

🎯 **Workload I/O-intensive** (nhiều read/write, spiky) → **Aurora I/O-Optimized storage** (giá phẳng, không phí theo I/O, throughput cao). gp2 (burst-bucket, không scale IOPS độc lập), io1/Magnetic không phải storage type chọn trực tiếp cho Aurora.

🎯 **Một bảng global + các bảng khác regional, ít refactor** → dùng **Aurora Global Database** cho bảng global + **Aurora thường** cho bảng regional (2 cluster). **KHÔNG trộn Aurora (SQL) với DynamoDB (NoSQL)** — API hoàn toàn khác nhau → refactor nhiều. Cùng lý do: bảng global ≠ tự động chuyển sang DynamoDB Global Table.

⚠️ **3 endpoint Aurora:**

| Endpoint | Vai trò |
| --- | --- |
| **Writer/Cluster** | Ghi (tự trỏ sang replica được promote khi primary chết) |
| **Reader** | Đọc (load-balanced trên các Aurora Replica) |
| **Instance** | Nhắm 1 node cụ thể |

⚠️ Multi-AZ (trong region) ≠ Multi-Region. RDS thường cũng multi-region được (cross-region read replica). Aurora đặc biệt có **Aurora Global Database** (replicate storage <1s, failover nhanh). Aurora chỉ **MySQL/PostgreSQL**. Aurora Serverless v2 = tải biến động/thưa (không tối ưu cho read-heavy ổn định cao).

📌 Aurora Global Database hỗ trợ **tối đa 10 secondary Region**. Tổng reader của primary cluster + số secondary cluster ≤ 15.

---

## 10. DynamoDB

🎯 NoSQL, **single-digit millisecond latency**, serverless, scale horizontal → **DynamoDB** (không phải RDS/ElastiCache/Neptune).

🎯 **Chống xóa nhầm bảng** (proactive, ít overhead) → bật **Deletion Protection**. Khác PITR (reactive — khôi phục data sau sự cố).

🎯 Xóa/khôi phục **data hỏng** đến thời điểm ngay trước khi hỏng → **Point-in-Time Recovery (PITR)**: backup liên tục per-second, restore bất kỳ giây nào trong **35 ngày**.

📌 PITR nay **chỉnh được recovery period 1–35 ngày** (mặc định 35).

🎯 Traffic **spiky/không đoán trước, có lúc không dùng** → **on-demand capacity mode** (trả theo request, cấp capacity tức thì). Traffic **đoán trước/ổn định** → provisioned (+ auto-scaling).

🎯 **Hot partition** + không muốn refactor → **DynamoDB Accelerator (DAX)** (in-memory cache, cache hotkeys, transparent). ElastiCache cũng cache được nhưng cần refactor.

🎯 **Tự động xóa item sau N ngày** (vd lưu feedback/log 365 ngày rồi hết hạn) → bật **DynamoDB TTL** (đặt timestamp hết hạn trên item, DynamoDB tự xóa, miễn phí). Kết hợp API Gateway → SQS → Lambda → Comprehend → DynamoDB(TTL) cho pipeline phân tích + lưu có hạn.

⚠️ **PITR vs Deletion Protection vs Backup vs Streams vs Global tables:**

| Tính năng | Dùng khi |
| --- | --- |
| **Deletion Protection** | CHẶN xóa bảng (proactive, chống human error) |
| **PITR** | Khôi phục về bất kỳ thời điểm trong recovery period (tối đa 35 ngày, phải bật trước) — data corruption |
| **On-demand backup** | Phải tạo THỦ CÔNG trước → không cứu sự cố bất ngờ |
| **DynamoDB Streams** | Log thay đổi item 24h → cần code rebuild, không phải restore |
| **Global Tables** | Multi-region multi-active, last-writer-wins. 1 bảng logic |
| **DAX** | Cache in-memory, giải quyết hot partition/read latency |

⚠️ Giới hạn item DynamoDB = **400KB** → không lưu file lớn (vd task output 20MB). Item lớn → EFS/S3.

---

## 11. Amazon Neptune (Graph DB)

🎯 Câu hỏi **quan hệ nhiều tầng** ("bạn của bạn", "số like trên video do bạn bè của user A đăng", recommendation, fraud, social graph) → **Amazon Neptune** (graph database). Không phải Aurora/Redshift/OpenSearch.

---

## 12. ElastiCache

🎯 ElastiCache tốt cho: **read-heavy** (leaderboard, gaming, social, session store) và **compute-intensive** (recommendation engine). Không hợp: write-heavy (cache stale nhanh), ETL (dùng Glue/EMR), JOIN phức tạp (dùng RDS/Aurora).

🎯 **Giảm áp lực read lặp lại lên Aurora/RDS** (repeated identical reads) → đặt **ElastiCache for Redis** phía trước, cache kết quả + TTL. Rẻ & nhanh hơn thêm read replica.

🎯 **Geospatial real-time** (vị trí GPS, tìm trong bán kính) high write+read → **ElastiCache for Redis** (GEOADD/GEORADIUS, TTL-based eviction). OpenSearch không tối ưu ghi tần suất cao.

🎯 **DR cho ElastiCache Redis** minimal downtime + minimal data loss → **Multi-AZ với automatic failover**. Read replica hay backup không đủ (data loss cao/không tự động).

⚠️ Cần thêm caching mà "không đổi code" → thường **không** chọn ElastiCache (caching cần sửa app đọc cache trước). Khi đó cân nhắc DAX (DynamoDB) hoặc API Gateway caching.

📌 ElastiCache có 3 engine: **Valkey**, **Redis OSS**, **Memcached**.

---

## 13. Data Transfer Fees

🎯 **4 nguyên tắc:** IN = FREE; OUT (Internet) = CHARGED; càng xa càng đắt (cùng AZ < cross-AZ < cross-Region < Internet); qua trung gian (NAT, Interface Endpoint) = phí riêng.

🎯 AWS tự replicate nội bộ → nghiêng FREE; app chủ động truyền qua AZ → CHARGED. Private IP + cùng AZ → FREE; public IP hoặc cross-AZ → CHARGED. *"Storage đồng bộ cùng Region = free"* là **cách nhớ SAI** (chỉ miễn phần data transfer replication của managed service).

🎯 **Giảm chi phí replication giữa 2 EC2 (vd DB đồng bộ)** → dùng **Private IP** thay vì Public IP/EIP (traffic qua public IP đi ra internet = đắt; private IP giữ trong mạng AWS). EIP không giúp (vẫn qua internet); PrivateLink/EFA không hợp cho case này.

⚠️ **Quà miễn phí:** Gateway Endpoint (S3/DynamoDB), S3 ↔︎ EC2 cùng Region, Data IN, S3 → CloudFront (origin fetch).

⚠️ **Bẫy tưởng free nhưng CHARGED:** cross-AZ (2 chiều), cùng AZ qua public IP, app query replica khác AZ, traffic S3 qua NAT Gateway.

⚠️ **Giảm cost:** S3/DynamoDB → Gateway Endpoint (tránh NAT); egress → CloudFront; EC2 gọi S3 cùng Region; on-prem lớn → Direct Connect.

⚠️ **Direct Connect pricing** = chủ yếu **Data Transfer OUT (DTO)** từ AWS ra on-prem (tính theo GB, DX rẻ hơn internet). Mẹo giảm phí: **đặt tool/compute CÙNG Region với data**, chỉ kéo phần nhỏ (vd trang web 600KB) ra on-prem thay vì kéo cả query 60MB về on-prem.

📌 Mọi **public IPv4** (kể cả Elastic IP đang gắn) nay đều tính phí theo giờ → thêm lý do ưu tiên private IP.

---

## 14. VPC Endpoints

🎯 EC2 private ↔︎ **S3/DynamoDB** cùng Region giảm phí NAT → **VPC Gateway Endpoint** (FREE, route table, không qua NAT/internet). VPC truy cập dịch vụ khác riêng tư (SQS, SNS, Kinesis…) → **Interface Endpoint (PrivateLink)**.

⚠️ **Chỉ S3 và DynamoDB** hỗ trợ **Gateway Endpoint** (target trong route table). DynamoDB nay **cũng** hỗ trợ Interface Endpoint (PrivateLink) — nhưng để route qua route table thì dùng Gateway Endpoint. Ví dụ EC2 private cần cả S3 và DynamoDB → tạo **2 gateway endpoint** + thêm 2 target vào route table.

⚠️ **Gateway vs Interface — nhớ kỹ:**

| Loại | Dịch vụ | Phí | Cơ chế |
| --- | --- | --- | --- |
| **Gateway Endpoint** | CHỈ **S3 & DynamoDB** | MIỄN PHÍ | Route table, không ENI, không SG |
| **Interface Endpoint (PrivateLink)** | Hầu hết dịch vụ khác (SQS, SNS, KMS, Kinesis, API GW…) | Tính phí giờ + GB | Tạo ENI riêng tư, có SG |

---

## 15. S3 — Storage & Features

🎯 Static content + serverless + cost-optimal + chịu tải → **CloudFront + S3** (combo mặc định). Serverless web static+dynamic → S3(static) + Lambda + DynamoDB(dynamic) + CloudFront. Giảm chi phí egress S3 → **CloudFront** (S3→CloudFront free).

⚠️ **Truy cập S3 từ on-prem với low-latency + local cache** → **S3 File Gateway** (NFS/SMB, cache local). **Mountpoint for Amazon S3** = file client Linux mount S3 trực tiếp nhưng **KHÔNG POSIX-compliant** đầy đủ → không thay được file share on-prem.

📌 Mountpoint for Amazon S3 **có local cache** (đĩa instance/EBS, cả shared cache). Loại Mountpoint khi đề cần: POSIX đầy đủ, Windows, hoặc NFS/SMB cho on-prem.

🎯 Upload **file rất lớn** (≥100MB, bắt buộc nếu >5GB) → **Multipart Upload** (song song, resilient, retry từng phần). Không phải Snowball (offline), không phải S3TA/Versioning.

🎯 Upload nhanh nhất file lớn (~2GB) qua khoảng cách xa → **Multipart Upload + S3 Transfer Acceleration** (kết hợp: song song + đường tối ưu qua edge). Multipart một mình nhanh nhưng thêm S3TA còn nhanh hơn.

🎯 **Copy khối lượng lớn S3 cross-Region một lần** (không dùng Snowball) → **`aws s3 sync`** (CLI) hoặc **S3 Batch Replication** (rồi xóa config). Console copy không khả thi cho PB; S3TA không copy giữa bucket.

🎯 Tối ưu chi phí S3: data không truy cập, chỉ giữ tuân thủ → **Glacier Deep Archive**. Phải query → nén + Parquet. Athena tính tiền theo data **SCAN**.

### Chọn storage class theo THỜI GIAN LƯU (min storage duration)

🎯 Data giữ **rất ngắn (24h) + truy cập nhiều** (vd kết quả query trung gian data lake) → **S3 Standard** (không có min duration, không phí retrieval). Standard-IA/One Zone-IA có **min 30 ngày** + phí retrieval → nếu chỉ giữ 24h vẫn bị tính đủ 30 ngày → đắt hơn. Glacier Instant Retrieval min **90 ngày**.

🎯 Data truy cập **2 lần/năm nhưng cần millisecond + có retry/failover** (vd báo cáo audit) → **S3 Standard-IA** (99.9% availability, retry bù được). Glacier Deep Archive **không** millisecond.

⚠️ **Standard-IA vs Glacier Instant Retrieval (GIR):** cả hai đều truy cập tức thì (millisecond). **GIR rẻ hơn ~68% phí LƯU** nhưng **phí retrieval + GET đắt hơn** Standard-IA. → Data còn được truy cập **đôi khi** (vd sau 30 ngày vẫn thỉnh thoảng đọc) → **Standard-IA rẻ hơn tổng thể**. Data gần như không đọc → GIR rẻ hơn. Case "tháng đầu đọc nhiều, sau đó hiếm nhưng vẫn cần tức thì, giữ 4 năm rồi xóa" → **Standard-IA sau 30 ngày + expiration sau 4 năm**.

⚠️ **Storage classes:** Standard / Standard-IA (immediate, ≥3 AZ) / One Zone-IA (rẻ hơn 20%, **1 AZ** — mất AZ là mất) / Intelligent-Tiering / Glacier Instant-Flexible-Deep Archive. S3 Batch Operations = thao tác hàng loạt, không liên quan phân phối content.

### Performance — Prefixes

🎯 Upload/read fail khi vượt **~3,500 PUT/COPY/POST/DELETE hoặc 5,500 GET/HEAD mỗi giây MỖI PREFIX** → tạo nhiều **prefix** trong CÙNG một bucket để parallelize (10 prefix → ~55,000 GET/s). **KHÔNG** tạo bucket riêng mỗi customer/ngày (bucket phải globally-unique, lãng phí). Không có giới hạn số prefix.

🎯 **Chỉ cần đọc phần đầu mỗi object (vd 250 byte metadata) từ nhiều file lớn** → **Byte-Range Fetch** (header `Range` trong GET, chỉ tải phần cần → nhanh, rẻ, song song được). Đọc cả file = tốn/chậm. **S3 Select** = lọc subset bằng SQL (dùng `ScanRange` để chia nhỏ), khác Byte-Range Fetch. Không import trực tiếp S3 → RDS.

📌 **S3 Select** không nhận khách mới. Thay thế: Athena, hoặc lọc phía client.

🎯 **Xem trạng thái cấu hình S3 (vd bucket nào CHƯA bật Versioning) trên nhiều Region/account, ít công sức** → **S3 Storage Lens** (advanced metrics + dashboard per-bucket, lọc versioning status toàn org). CloudTrail chỉ log thay đổi; IAM Access Analyzer/Multi-Region Access Point không xem được cấu hình bucket.

### Versioning & Bảo vệ khỏi xóa nhầm

🎯 Chống **xóa/ghi đè nhầm object** → bật **Versioning** (giữ mọi version, delete chỉ đặt delete marker) **+ MFA Delete** (cần MFA để xóa vĩnh viễn). Event trigger/SNS sau khi xóa = quá muộn; không có "confirm khi xóa" trên console.

### Lifecycle Policy

🎯 Chuyển tầng theo **prefix** khi chỉ một phần data nguội (vd `images/` → Standard-IA sau 45 ngày nhưng `thumbnails/` vẫn nóng) → Lifecycle rule **có prefix**. Sau 180 ngày archive tất cả → Glacier (không cần prefix).

⚠️ **Lifecycle transition = mô hình waterfall (một chiều, xuống dốc).** Các transition **KHÔNG hợp lệ** (hay ra đề): bất kỳ class → **S3 Standard** (không đi ngược lên); Intelligent-Tiering → Standard-IA; One Zone-IA → Standard-IA/Intelligent-Tiering; bất kỳ → Reduced Redundancy. Hợp lệ: Standard → bất kỳ; Standard-IA → Intelligent-Tiering/One Zone-IA; bất kỳ → Glacier/Deep Archive.

### S3 Transfer Acceleration — billing

🎯 **Data transfer IN (từ internet vào S3) = MIỄN PHÍ.** S3TA **chỉ tính phí nếu thực sự tăng tốc**; nếu S3TA không nhanh hơn → **không tính phí S3TA**. Vậy upload mà S3TA không accelerate → tổng phí transfer = 0.

---

## 16. S3 — Encryption & Object Lock

🎯 **SSE-S3** tự động mã hóa **mỗi object bằng key riêng** — không cấu hình, không overhead → thỏa "mỗi file một key". Encryption context **không** tạo key khác nhau (chỉ ràng buộc ngữ cảnh cho SSE-KMS).

🎯 **Mã hóa at-rest + tự động rotate key mỗi 12 tháng, ít overhead** → **KMS customer managed key + automatic rotation** (bật S3 default encryption dùng key này). SSE-S3 rotation không cấu hình/audit được; imported key material **không** auto-rotate; CloudHSM = overhead cao.

📌 KMS automatic rotation mặc định **mỗi năm**, **chỉnh được rotation period** và **rotate on-demand** cho customer managed key. AWS managed key tự rotate hằng năm, không chỉnh được. Ba loại **KMS key**: customer managed key / AWS managed key / AWS owned key.

🎯 **Chi phí KMS request tăng cao do upload/access dày (vẫn muốn giữ SSE-KMS)** → bật **S3 Bucket Keys** (S3 tạo data key ở tầng bucket, giảm gọi KMS tới **99%**). Giữ nguyên bảo mật/audit của SSE-KMS. Chuyển sang SSE-S3 = mất kiểm soát/audit KMS (không thỏa chính sách). VPC endpoint không giảm phí KMS.

🎯 *"Cùng một key nhiều Region"* → **KMS Multi-Region Key**. Không convert single→multi (tạo key mới). Không share KMS key qua Region.

🎯 **Key mã hóa phải do khách quản/lưu ngoài AWS (on-prem) + dùng key có sẵn + để S3 lo phần mã hóa** → **SSE-C** (khách cung cấp key mỗi request, S3 mã hóa/giải mã, không lưu key). Client-side = khách tự mã hóa trước upload (nhiều việc hơn).

⚠️ **Các loại S3 encryption:** SSE-S3 (AWS quản, mỗi object key riêng, đơn giản) / SSE-KMS (kiểm soát key, audit CloudTrail, rotation) / DSSE-KMS (2 lớp) / SSE-C (khách cung cấp key) / Client-side (mã hóa trước upload).

### Object Lock (WORM) & Glacier Vault Lock

🎯 Data phải **immutable + truy cập ngay** cho audit (không xóa/sửa N năm) → **S3 Object Lock — Compliance mode** (WORM, không ai override kể cả root). Data archive lâu dài rẻ + khóa xóa → **S3 Glacier Vault Lock**.

📌 Archive + WORM → **S3 Glacier storage classes trong S3** + **S3 Object Lock**. **Amazon Glacier dạng vault** (dịch vụ standalone, có Vault Lock) **không nhận khách mới**.

⚠️ **Compliance mode** = không ai override được (kể cả root). **Governance mode** = user có quyền đặc biệt CÓ THỂ override → không hợp compliance nghiêm ngặt. Lifecycle policy **không** ngăn xóa (chỉ chuyển tầng/expire).

⚠️ **Object Lock — retention chi tiết:** đặt **explicit** (mỗi object version có **Retain Until Date** riêng, lưu trong metadata) HOẶC **bucket default** (đặt **duration** ngày/năm, KHÔNG phải Retain Until Date). Explicit **override** bucket default. Các version khác nhau của cùng object có thể có retention mode/period khác nhau.

---

## 17. S3 — Access Control

🎯 Cấp quyền S3 cho **user account khác (cross-account)** → **Bucket Policy** (resource-based, principal có thể ở account khác). **User policy** chỉ cho user trong **cùng account**.

🎯 Sở hữu bucket ≠ sở hữu object. Object mặc định thuộc account **upload**. Cross-account ghi → bucket owner có thể không đọc → cross-account IAM role hoặc **Bucket owner enforced** (ACL disabled). Shared bucket nhiều bên/prefix → **S3 Access Points**.

📌 Bucket mới nay **mặc định Bucket owner enforced** (ACL tắt) → bucket owner tự động sở hữu mọi object; vấn đề "owner không đọc được object do account khác upload" chỉ còn ở bucket bật ACL.

🎯 **ListBucket** áp lên BUCKET → ARN **không** `/*`. **GetObject** áp lên OBJECT → ARN **có** `/*`. Thường viết 2 statement riêng.

⚠️ Đọc bucket policy: `IpAddress: aws:SourceIp CIDR` + `NotIpAddress: 1 IP/32` = **cho phép cả dải CIDR TRỪ một IP**. `0.0.0.0/0` = mọi IP (không phải IP "0.0.0.0"). `aws:SourceIp` = IP người **gọi** request.

⚠️ **Truy cập S3 riêng tư (không qua public endpoint/internet)** → **Gateway VPC Endpoint cho S3** (miễn phí, route table). **NAT Gateway vẫn đi public endpoint** (không riêng tư + tốn phí). **S3 Access Point KHÔNG đổi đường mạng** (chỉ quản quyền, vẫn qua internet nếu không có endpoint). Direct Connect = thừa cho traffic EC2↔︎S3 nội bộ.

---

## 18. Storage Types — EBS / EFS / FSx / S3

🎯 Triệu chứng "mỗi lần vào thấy dữ liệu khác nhau" (nhiều instance sau LB) → EBS cục bộ không chia sẻ. Fix: **S3** (object) hoặc **EFS** (file, mount nhiều instance).

- **Block** (EBS, 1 instance) / **File** (EFS-Linux, FSx-Windows, chia sẻ) / **Object** (S3, API).

### Instance Store

🎯 Cần **high random I/O + cost-optimal** cho data **tạm/replicate được** (mỗi instance có bản sao, app tự replicate, mất instance không sao) → **Instance Store** (đĩa gắn vật lý vào host, **giá đã gồm trong instance**, không phí thêm). EBS io1 = tốn thêm; EFS = tài nguyên thừa; S3 = không đạt high random I/O.

⚠️ Instance Store là **ephemeral** — mất khi stop/terminate. Chỉ dùng cho buffer/cache/scratch/data replicate được.

### EFS

🎯 "File storage" + "nhiều EC2 truy cập đồng thời" → **EFS**. **Performance Mode**: General Purpose (latency) vs Max I/O (parallel/big data). **Throughput Mode**: Bursting (theo dung lượng) vs Provisioned (cố định, độc lập dung lượng).

📌 EFS nay có thêm throughput mode **Elastic** (tự scale theo workload, trả theo lượng đọc/ghi — hợp tải khó đoán) và storage class **Archive**.

🎯 Data ít nhưng cần throughput cao (old data bị archive) → **Provisioned Throughput**. Burst thưa + trung bình thấp → **Bursting** + General Purpose + **Standard storage class** (đa AZ). EFS chỉ **Linux/NFS**, không hỗ trợ Windows/SMB.

🎯 **File storage chia sẻ + ít truy cập + rẻ + vẫn immediate** → **EFS Standard-IA** (không phải S3-IA — S3 là object, không mount đồng thời nhiều EC2 kiểu file system). **Big data/parallel/genomics** → **Max I/O** (throughput tổng cao, đổi lại latency metadata cao hơn); **web/CMS latency-sensitive** → **General Purpose** (mặc định).

🎯 **Lowest latency đa AZ**: tạo **mount target ở MỖI AZ**, mount EC2 vào mount target **cùng AZ** (tránh cross-AZ latency + phí). Mount target là ENI ở tầng VPC/AZ, **không** đặt trên EC2.

🎯 **Access control EFS** = **VPC Security Group** (kiểm soát network in/out file system) + **IAM policy** (client nào được mount + quyền gì) + **EFS Access Points** (override POSIX user/group). NACL ở tầng subnet (không phải instance); GuardDuty = phát hiện threat, không access control; không có "IAM policy root credentials".

🎯 **Cross-account access EFS** → **EFS resource policy** cho phép account khác + **VPC peering/shared VPC** + mount qua **EFS access point**. Không cần copy sang S3 (DataSync = tốn phí, trễ). **Cross-region access EFS** → **inter-region VPC peering** (EFS là regional nhưng truy cập được từ region khác qua peering; không phải copy sang region khác).

🎯 **Shared POSIX file system đa AZ cho container (ECS Fargate) + backup cross-Region có RPO đảm bảo (vd ≤8h)** → **EFS Standard + AWS Backup cross-Region backup** (EFS đa AZ mặc định, AWS Backup lên lịch backup cross-Region tự động). **FSx ONTAP** cross-Region cần **NetApp SnapMirror** (thủ công, không RPO managed qua AWS Backup); FSx Lustre = HPC (không đa AZ mount); S3 = không POSIX.

### FSx family — giao thức

🎯 **Bảng vàng giao thức:**

| Yêu cầu | Dịch vụ |
| --- | --- |
| SMB (Windows) | **FSx for Windows** |
| NFS (Linux) | **EFS** |
| Cả SMB **VÀ** NFS đồng thời (multi-protocol) | **FSx for ONTAP** |
| HPC / Lustre client | **FSx for Lustre** |
| Object | **S3** |
| Block | **EBS** |

🎯 Windows file storage + SMB + NTFS + AD integration → **FSx for Windows** (Linux cũng mount SMB qua cifs-utils). **Storage Gateway File Gateway** cũng hỗ trợ SMB/NFS (→S3). EFS/EBS/S3 **không** hỗ trợ SMB.

🎯 Migrate workload trên **Microsoft DFS (Distributed File System)** on-prem → **FSx for Windows** (hỗ trợ DFS Namespaces, gom shares tới hàng trăm PB). Managed AD / SQL Server on AWS / FSx Lustre **không** hỗ trợ DFS.

🎯 **HPC "hot data" (parallel, distributed, tốc độ cao) + "cold data" giữ rẻ trên S3** → **FSx for Lustre** (tích hợp S3: trình bày object S3 như file, ghi ngược về S3). EMR/Glue không đạt tốc độ Lustre.

⚠️ FSx ONTAP = **duy nhất** hỗ trợ cả SMB + NFS đồng thời. OpenZFS = NFS-only (không SMB). Lustre = HPC/Linux (không Windows ACL/SMB). Container persistent shared storage fully managed → **ECS Fargate + EFS**.

### EBS

⚠️ **EBS types:** gp2/gp3 (SSD, đa dụng/boot) < io1/io2 (SSD, ≤64,000 IOPS, cần IOPS cao, vd 25,000). HDD: st1 (throughput: big data/log/Kafka) / sc1 (cold, rẻ nhất). Cần IOPS cao → io1/io2. **Tối ưu cost khi io1 under-utilized + burst thưa → chuyển gp2/gp3** (rẻ hơn, gp3 có baseline 3,000 IOPS).

📌 **gp2 tối đa 16,000 IOPS**; **gp3 tới 80,000 IOPS, 2,000 MiB/s, 64 TiB**, **không burst** (baseline 3,000 IOPS + 125 MiB/s luôn có), rẻ hơn gp2 ~20%/GiB. **io2 Block Express** tới 256,000 IOPS.

⚠️ **Boot volume:** gp2/gp3, io1/io2, Instance Store **dùng làm boot được**. **st1 và sc1 (HDD) KHÔNG làm boot volume được.**

⚠️ **EBS encryption (bật lúc tạo volume, dùng KMS):** mã hóa **data-at-rest + data-in-transit giữa instance↔︎volume + mọi snapshot tạo ra + volume tạo từ snapshot đó** — tất cả tự động mã hóa. HIPAA/compliance OK.

⚠️ **Chống xóa nhầm EBS snapshot (ít công sức)** → **Recycle Bin retention rule** (giữ snapshot đã xóa N ngày, khôi phục được, sau đó tự xóa hẳn). IAM deny delete = chặn cả xóa hợp lệ (phình cost). AWS Backup Vault Lock chỉ áp cho snapshot do **AWS Backup** tạo (không áp snapshot tạo tay/script).

⚠️ **Clone data test từ production nhanh + IOPS cao ngay** → snapshot → **Fast Snapshot Restore (FSR)** → tạo volume mới (full performance tức thì, không cần "hâm nóng"). Cách ly test↔︎prod. **EBS Multi-Attach** (io1/io2, **cùng AZ**) = share 1 volume cho nhiều instance cho **clustered app**, KHÔNG dùng để clone/tách môi trường (rủi ro ghi đè).

⚠️ **So sánh giá lưu trữ (cùng 1GB):** **S3 Standard < EFS < EBS**. Lý do EBS đắt nhất trong ví dụ điển hình: EBS tính theo **dung lượng PROVISIONED** (vd cấp 100GB thì trả 100GB dù chỉ dùng 1GB), còn S3/EFS tính theo dung lượng thực dùng.

---

## 19. Storage Gateway & Data Migration

### Volume Gateway Cached vs Stored

🎯 Chỉ **cache data hay truy cập local** + toàn bộ volume trên S3 (giảm local storage, mở rộng lên cloud, không phí retrieval) → **Volume Gateway Cached**. Toàn bộ volume **local** + async copy lên S3 (backup) → **Volume Gateway Stored**.

### Tape Gateway

🎯 Chuyển **băng từ vật lý (tape) → cloud, không đổi backup workflow** → **Tape Gateway** (virtual tape → S3 Glacier/Deep Archive, rẻ). Không phải DataSync/Direct Connect.

### DataSync vs Transfer Family vs Snowball vs File Gateway

🎯 Tự động hóa & tăng tốc chuyển **online** lượng lớn data → **S3/EFS/FSx** → **AWS DataSync**. Transfer Family chỉ vào **S3/EFS** (không FSx). File Gateway chỉ **→S3**. Snowball = **offline** (mạng kém/rất lớn). DataSync hỗ trợ NFS/SMB (không dùng cho tape).

🎯 **DataSync từ on-prem NFS → EFS qua Direct Connect (riêng tư)** → dùng **Interface VPC Endpoint (PrivateLink) cho EFS + private VIF**. KHÔNG dùng "VPC peering endpoint" (peering không chuyển data qua DX), KHÔNG dùng S3 Gateway Endpoint (không đi qua DX). Có thể lịch DataSync task định kỳ (vd mỗi 24h).

🎯 **Snowball → long-term archive rẻ nhất** → Snowball **phải target S3 bucket trước**, rồi **lifecycle zero-day (cùng ngày) chuyển sang Glacier Deep Archive**. KHÔNG target trực tiếp Glacier/Deep Archive Vault từ Snowball. Deep Archive rẻ hơn Glacier Flexible.

📌 **Snow Family (Snowball Edge…) không nhận khách mới từ 07/11/2025.** Chuyển dữ liệu lớn: **DataSync** (online), **AWS Data Transfer Terminal** (mang thiết bị tới điểm upload); edge compute: **Outposts**.

### Lift-and-shift server migration (VM → EC2)

🎯 **Rehost (lift-and-shift) VM on-prem → EC2, nhanh, ít refactor** → **AWS Application Migration Service (MGN)**: cài **Replication Agent** trên source VM → replicate liên tục → launch **test instance** kiểm thử → **cutover instance** khi replication đã up-to-date. KHÔNG dùng CloudEndure DR (dành cho disaster recovery, không phải migration), không dùng ASG (chỉ scale, không import VM), không tạo AMI thủ công (downtime, thủ công).

📌 CloudEndure Disaster Recovery nay là **AWS Elastic Disaster Recovery (DRS)**.

### Database migration

🎯 Migrate DB **khác engine** (heterogeneous, vd Oracle→Aurora) → **DMS + SCT** (Schema Conversion Tool). **Basic Schema Copy** KHÔNG migrate secondary index/foreign key/stored procedure → cần SCT cho schema phức tạp. DMS cũng làm cầu S3→Kinesis.

⚠️ DMS Serverless cho replication tải biến động: xem §8.

### SFTP ingest — immediate processing

🎯 SFTP + xử lý **ngay** (không polling) + HA + private → **Transfer Family INTERNAL endpoint (2 AZ) + S3 + managed workflow gọi Lambda** ngay sau upload. Internal > internet-facing (đã có kết nối riêng). Managed workflow > S3 Event Notification. EventBridge Scheduler = polling (sai).

⚠️ **Storage Gateway 3 loại:** File (NFS/SMB→S3), Volume (block/iSCSI, Cached/Stored), Tape (thay tape). **Transfer Family:** SFTP/FTPS/FTP. **AppFlow:** SaaS API (không SFTP).

📌 **FSx File Gateway** không nhận khách mới → muốn file share Windows trên AWS thì dùng thẳng FSx for Windows.

---

## 20. CloudFront, Global Accelerator & Content Delivery

🎯 Giảm chi phí egress S3 / phân phối static → **CloudFront** (S3→CloudFront FREE, cache edge). **Nội dung động thay đổi liên tục + 10x traffic** → **Auto Scaling Group** (CloudFront cache không hợp nội dung động thường xuyên).

🎯 Chặn IP ở CloudFront → **WAF** (không SG/NACL — CloudFront không trong VPC). Khóa S3 chỉ cho CloudFront → **OAI/OAC** + bucket policy.

⚠️ **Custom domain HTTPS trên CloudFront:** certificate ACM **BẮT BUỘC ở us-east-1** (dù bucket/origin ở region khác). **OAC** (mới, thay OAI) hỗ trợ cả **đọc VÀ ghi/upload** vào S3 an toàn, hỗ trợ SSE-KMS, mọi Region. S3 **static website endpoint chỉ GET/HEAD** (không upload PUT/POST được). Signed URL thường dùng cho download.

🎯 **Website ĐỘNG on-prem, backend phải ở lại on-prem, cần giảm latency toàn cầu, gấp** → **CloudFront với custom origin trỏ THẲNG tới server on-prem** (edge cache gần user). S3 chỉ host static (không chạy server-side script). Route 53 geo-proximity trỏ về on-prem **không** giảm latency (server vẫn ở xa). CloudFront custom origin **không** trỏ tới "DNS record trên Route 53".

### CloudFront capabilities

🎯 CloudFront có thể: route **nhiều origin theo content type** (static→S3, dynamic→ALB); **origin group** (primary + secondary) cho **failover HA**; **field-level encryption** (mã hóa field nhạy cảm ở edge, tối đa 10 field). CloudFront **không** route theo price class; geo restriction ≠ HA.

### Global Accelerator

🎯 **Non-HTTP** (UDP gaming / MQTT IoT / VoIP) low-latency → **Global Accelerator** (proxy TCP/UDP, static anycast IP). **Nhiều ALB nhiều Region + giảm số IP firewall/on-prem phải whitelist** → **Global Accelerator** (2 static IP, đăng ký ALB các region). ALB **không gán được Elastic IP** (NLB thì được).

⚠️ **CloudFront vs S3TA vs Global Accelerator:**

| Dịch vụ | Dùng khi |
| --- | --- |
| **CloudFront** | HTTP/HTTPS content (static+dynamic), cache edge, giảm egress |
| **S3 Transfer Acceleration** | Tăng tốc UPLOAD & download object LỚN (>1GB) qua khoảng cách xa |
| **Global Accelerator** | TCP/UDP (non-HTTP), static IP, đa Region, né DNS caching, giảm số IP |

⚠️ WAF bảo vệ CloudFront/API GW/ALB/AppSync/Cognito — **không** gắn S3. **OAC > OAI**. Cả GA & CloudFront tích hợp Shield DDoS.

---

## 21. IAM — Roles & Credentials

🎯 Compute cần quyền → gắn **ROLE**, đừng nhét key. EC2 → IAM Role qua **Instance Profile**; Lambda → execution role; ECS → task role. Mọi đáp án "lưu access key trên máy/S3/local" (mã hóa hay không) đều **SAI**.

🎯 Gán role cho EC2 dùng **Instance Profile**. Service role cho EC2 **tự động** có EC2 là trusted entity → không cần thêm thủ công vào trust policy.

⚠️ Rủi ro ở **sự tồn tại** của long-term credentials, không phải mã hóa. Ngoài AWS cần secret → Secrets Manager / Parameter Store (SecureString). Instance profile chứa **1 role/instance**.

🎯 **Delegate quyền cho user từ account khác** (dev truy cập prod) → tạo **IAM Role** với quyền cần + cho user account kia **assume role** (temporary credentials). KHÔNG tạo IAM user mới & share credential; IAM role và IAM user **không** dùng thay thế cho nhau.

🎯 **Root user best practices:** đặt **password mạnh** + bật **MFA**; **KHÔNG** tạo access key cho root (nếu có thì xóa/rotate); không share credential/không email password; root không dùng cho tác vụ hằng ngày.

🎯 **IAM best practices khác:** bật **MFA cho privileged user**; bật **CloudTrail log mọi IAM action** (audit); dùng **role** cho EC2 (không dùng user credential); **least privilege** (không cấp quyền tối đa cho tiện); không share account giữa nhân viên.

🎯 **Đảm bảo thông báo (billing/security/operations) tới đúng đội, không phụ thuộc cá nhân** → đặt **root email = alias trỏ vào mailbox chung** (đội admin theo dõi) + cấu hình **Alternate Contacts** (Billing / Security / Operations, dùng distribution list của công ty). KHÔNG gán root email cho 1 cá nhân (single point of failure); IAM không route thông báo AWS service theo loại.

🎯 **Cấp quyền cho nhiều user nhanh, ít thay đổi** (vd 50/200 user cần write S3) → tạo **IAM Group**, gắn policy vào group, thêm user vào group. Gán policy từng user = khó scale; sửa bucket policy = giới hạn kích thước (20KB); MFA không cấp quyền.

### AMI

⚠️ **AMI capabilities (hay bị hỏi):** CÓ thể copy across Regions; CÓ thể share với account khác; copy AMI có **encrypted snapshot không thể ra unencrypted** (encrypted→encrypted, unencrypted→encrypted OK). Đổi encryption status khi copy (theo chiều được phép).

⚠️ **Copy AMI sang Region B tạo ra:** 1 AMI + **1 snapshot** (AMI dựa trên snapshot nền → tự tạo snapshot ở Region B) + (nếu launch) 1 instance. Vậy sau khi launch = **1 instance + 1 AMI + 1 snapshot** ở Region B.

---

## 22. IAM — Policy Evaluation, SCP & Boundary

🎯 Chặn IP cho **managed service** (API GW, S3…) → **Resource Policy** (IpAddress/NotIpAddress), không Security Group (chỉ cho tài nguyên trong VPC).

🎯 Chặn cả **ROOT user** member account → chỉ **SCP** (đứng trên account). SCP = trần quyền (∩ IAM), **không tự cấp quyền**. Áp cả root nhưng **2 ngoại lệ**: management account + service-linked role. Cần Organizations.

🎯 Chống self-escalation trong 1 account (không Organizations) → **Permissions Boundary** cho từng user/role. Boundary **chỉ gắn user/role, không group**.

⚠️ **Loại resource-based policy DUY NHẤT mà IAM service hỗ trợ = Trust Policy** (role trust policy — định nghĩa ai được assume role). SCP/Permissions Boundary/ACL không phải resource-based policy của IAM.

⚠️ **Bộ ba trần quyền:** IAM policy (CẤP quyền) / Permissions Boundary (trần, user-role) / SCP (trần, account-OU). Đọc policy: "Deny + StringNotEquals" = phủ định kép → đảo thành khẳng định. **Explicit Deny > mọi Allow**.

⚠️ **Condition keys hay bị nhầm:** `aws:SourceIp` = **IP của người GỌI** API (vd hạn chế theo IP on-prem). `aws:RequestedRegion` = **Region ĐÍCH của API call** (vd chỉ cho tạo resource ở eu-west-1, gọi từ đâu cũng được). `ec2:Region` = region của resource. Ví dụ "Deny khi ec2:Region ≠ us-west-1 + Allow TerminateInstances khi SourceIp trong CIDR" = chỉ terminate được instance ở us-west-1 khi caller ở đúng IP.

⚠️ **SCP effects:** ảnh hưởng mọi user/role **gồm root** của member account; **không** ảnh hưởng service-linked role; nếu IAM cho phép nhưng SCP không cho → **không thực hiện được**.

📌 Organizations nay còn có **RCP (Resource Control Policy)** — trần quyền đặt trên **resource** (S3, KMS, STS, SQS, Secrets Manager…), bổ sung cho SCP (đặt trên principal).

---

## 23. Cross-Account, Organizations & RAM

🎯 Cross-account = phân quyền **cả hai phía**: caller (IAM role) + resource (resource-based policy cho phép role đó). Same-account chỉ cần IAM role.

🎯 Nhiều account Organizations cùng Region, chung mạng, rẻ nhất → **VPC Sharing qua RAM**. Lưu ý: VPC Sharing chia sẻ **subnet(s)**, không chia sẻ cả VPC.

🎯 Deploy cùng template qua **nhiều account + Region** (Organizations) → **CloudFormation StackSets**. Stack = 1 account/1 Region; Template = chỉ file.

⚠️ RAM share (free): Subnet, Transit Gateway, Route 53 Resolver rules, License Manager. RAM **không** deploy template (đó là StackSets).

🎯 **Bắt buộc tag chuẩn hóa + không cho tạo/xóa resource sai tag** (Organizations) → **Tag Policy** (định nghĩa key `dataClassification` + giới hạn value cho phép, gắn OU → chuẩn hóa/audit tag) **KẾT HỢP SCP** (Deny `ec2:RunInstances` nếu thiếu tag key + Deny `ec2:DeleteTags`). Tag Policy một mình **không chặn** tạo resource (chỉ audit); SCP mới enforce. Lambda/Config remediation = reactive (không ngăn từ đầu).

---

## 24. IAM Identity Center / SSO / Directory

🎯 **SSO nhiều account (Organizations) + dùng on-prem AD + ÍT overhead NHẤT (không muốn dựng directory trong cloud)** → **AD Connector** (proxy tới on-prem AD, không host directory) **+ IAM Identity Center + permission sets** gán quyền theo AD group. Đây là phương án ít maintenance nhất. Control Tower + IAM role thủ công/CLI = không scale.

🎯 Nếu cần **chạy directory-aware workload / two-way trust thực sự trong AWS** → **AWS Managed Microsoft AD + trust** với on-prem (nhiều overhead hơn AD Connector nhưng chạy được workload). Không tự tạo user thủ công, không OpenLDAP-EC2, không Cognito (Cognito = auth app web/mobile).

🎯 Chạy **directory-aware workload (SQL Server) + trust + SSO** với on-prem AD → **AWS Managed Microsoft AD**. **AD Connector** chỉ là cầu nối auth (không chạy workload, không trust). **Simple AD** rẻ, ≤5000 user, không trust. **Cloud Directory** = hierarchical app data.

⚠️ Phân biệt: **AD Connector** = chỉ proxy auth, ít overhead nhất, không chạy workload/không trust. **Managed Microsoft AD** = AD thật, chạy được directory-aware workload + trust relationship (overhead cao hơn).

---

## 25. EC2 & Compute

### Tenancy

🎯 Single-tenant hardware rẻ nhất → **Dedicated Instances**. Kiểm soát hardware/BYOL → **Dedicated Hosts**. On-Demand/Spot = shared.

🎯 Tenancy **Launch Template + VPC**: nếu **EITHER** (LT hoặc VPC) đặt **dedicated** → instance là **dedicated**.

### Serverless compute

🎯 Python microservice + scale tự động + overhead thấp nhất + API → **Lambda + API Gateway** (+ provisioned concurrency giảm cold start). Container cần persistent shared storage fully managed → **ECS Fargate + EFS**.

⚠️ **Lambda:** event/API, overhead thấp nhất. **Fargate:** container serverless. **App Runner:** web app chạy dài, tốn khi idle. **EKS+Fargate:** Kubernetes ít overhead. **Spot:** không tin cậy production.

📌 **AWS App Runner không nhận khách mới (từ 30/04/2026)**; AWS khuyên chuyển sang **Amazon ECS Express Mode** (1 lệnh dựng ECS on Fargate + ALB + auto scaling, không phụ phí).

⚠️ **SSH tạm thời, không phát khóa dài hạn, audit + thu hồi ngay:** **EC2 Instance Connect** (inject public key một-lần lúc kết nối, log qua CloudTrail). Instance có **public IP** → connect thẳng qua internet (không cần endpoint). Instance ở **private subnet/không public IP** → **EC2 Instance Connect Endpoint**. Truy cập private không mở SSH port + không cần key → **Session Manager** (cần SSM Agent).

### Placement Groups

🎯 HPC / tightly-coupled node-to-node, **high network throughput + low latency** → **Cluster placement group** (packs instances gần nhau 1 AZ, tới 10 Gbps). **Spread** = tách hardware (giảm correlated failure). **Partition** = phân vùng lớn (Hadoop/Cassandra/Kafka).

### HA + chi phí tối thiểu / Spot

🎯 HA chịu mất 1 AZ + rẻ nhất: rải nhiều AZ hơn (thường 3) → mỗi AZ ít instance → tổng ít hơn. Công thức: (số AZ − 1) × (instance/AZ) ≥ ngưỡng.

🎯 ASG + ELB cross 3 AZ, cần HA + rẻ: **min capacity = 2** (2 AZ khác nhau, ASG bù AZ thứ 3 khi tải) + **Reserved Instances** cho baseline (luôn chạy → RI rẻ nhất). Min=1 không HA; min=3 thừa; Dedicated host không cần.

🎯 Spot: hủy request ≠ terminate instance. Persistent request mở lại sau **interrupted** (AWS thu hồi); nếu **bạn stop** thì chỉ mở lại khi **bạn start**. Spot Fleet duy trì target capacity. Spot + SQS = async job rẻ có retry.

### Chiến lược mua (purchasing) — cost-optimal

🎯 **Baseline luôn chạy (steady-state)** → **Reserved Instances / Savings Plans** (rẻ nhất cho tải liên tục 1-3 năm). **Phần dư tải cao / batch chịu gián đoạn** → **Spot** (rẻ, có thể bị thu hồi). **Phần đệm không đoán trước, cần đảm bảo** → **On-Demand**. Ví dụ: 70 always-on → RI, 30 batch → Spot; hoặc baseline 80 → RI + spike → On-Demand/Spot mix (dùng ASG + launch template).

🎯 **Workload biến số/nhiều kích cỡ, chịu gián đoạn, chạy 1 lần/tháng** → **Spot Fleet** (tự chọn spot pool rẻ nhất `lowestPrice`, duy trì target capacity). RI = hợp tải liên tục (không hợp 1 lần/tháng); Dedicated Host = đắt.

⚠️ **Spot instance** = spare EC2, giảm tới 90% vs On-Demand, AWS thu hồi báo trước **2 phút**. **Spot Fleet** = tập Spot Instances **+ tùy chọn On-Demand** để đạt target capacity (không chỉ Spot). Spot Block (đặt 1–6h) đã **deprecated**.

### Savings Plans (loại nào phủ dịch vụ nào)

🎯 **Giảm cost dài hạn, ít quản lý, phủ rộng nhất, ít plan nhất:**
- **Compute Savings Plan** = phủ **EC2 + Fargate + Lambda** (linh hoạt nhất, mọi family/Region).
- **EC2 Instance Savings Plan** = chỉ EC2 theo family/Region (KHÔNG phủ Fargate/Lambda).
- **SageMaker Savings Plan** = riêng cho SageMaker (Compute plan KHÔNG phủ SageMaker).
→ Kiến trúc EC2+Fargate+Lambda+SageMaker: mua **Compute SP + SageMaker SP** (2 plan là đủ). RI = kém linh hoạt hơn SP, không phủ Fargate/Lambda.

📌 Nay có thêm **Database Savings Plans** (4 loại SP: Compute, EC2 Instance, SageMaker AI, Database).

### Bootstrap / user data

🎯 **Giảm thời gian khởi động lại instance** khi app cần vài phút để sẵn sàng → **EC2 Hibernate** (lưu RAM → EBS root, khi start lại giữ nguyên tiến trình/app đang chạy, giữ instance ID). AMI/user data **không** rút ngắn thời gian chạy app.

⚠️ **EC2 User Data:** chạy với **quyền root**; **mặc định chỉ chạy 1 lần lúc first boot** (có thể cấu hình chạy mỗi lần restart); **không sửa được** khi instance đang running (chỉ xem/khi stopped). Golden AMI + user data = giảm thời gian provision (cài sẵn static, chạy dynamic lúc boot).

---

## 26. Auto Scaling (ASG) & Scaling Policies

🎯 ASG không terminate ngay khi: grace period chưa hết, instance Impaired, health check type = EC2 (bỏ qua ELB). Spot **vẫn** terminate được, custom check **vẫn** terminate, tăng min **không** giữ instance hỏng (launch cái mới).

🎯 **Default termination:** cân bằng AZ trước (chọn AZ nhiều instance nhất) → oldest launch **template** (chỉ khi không ai dùng launch config) → oldest launch **configuration** → gần billing hour → random.

📌 Account tạo từ **01/10/2024** không tạo được **launch configuration** nữa, instance type mới cũng không hỗ trợ → dùng **launch template**.

🎯 **Spike đột ngột** → **Target Tracking** (tự tính số instance giữ metric ở mục tiêu). Simple = chờ cooldown (chậm); Scheduled = theo lịch (không xử lý spike); Step = xấp xỉ theo bậc. Metric SQS: backlog per instance = ApproxNumberOfMessages / số instance.

🎯 **Tải đoán trước (theo giờ/ngày) + spike bất ngờ** → **Predictive Scaling** (ML dự báo, scale trước) **kết hợp Target Tracking** (phản ứng real-time). Scheduled = thủ công theo giờ; Step/Simple = chỉ phản ứng sau.

### Bảo trì instance trong ASG (không bị thay thế)

🎯 Patch một instance mà không bị ASG thay ngay → đưa vào **Standby state** (vẫn thuộc ASG, không nhận traffic, xong exit Standby) HOẶC **suspend process `ReplaceUnhealthy`** (ASG ngừng thay instance unhealthy) rồi patch, xong resume. KHÔNG snapshot→AMI→instance mới (chậm); KHÔNG xóa ASG. `ScheduledActions` không liên quan bảo trì.

### Scheduled scaling — desired vs min/max

🎯 Cần đúng **N instance vào một GIỜ định trước** (vd 10 instance lúc chốt sổ cuối tháng) → **Scheduled Action** đặt **Desired Capacity = N**. Chỉ đặt min/max khi muốn một **khoảng** instance. Target/Simple tracking **không** scale theo thời điểm định sẵn.

### Rebalancing AZ vs thay instance unhealthy (thứ tự KHÁC nhau)

🎯 **AZ mất cân bằng** (vd terminate tay instance ở 1 AZ) → ASG **rebalance**: **LAUNCH instance mới TRƯỚC rồi mới terminate** (không hạ availability). **Instance unhealthy** (ELB health check) → ASG **TERMINATE trước rồi mới launch** thay thế. Hai luồng ngược thứ tự nhau.

---

## 27. Containers (ECS / EKS / Fargate)

🎯 Docker orchestration → **ECS** hoặc **EKS**; serverless (không quản EC2) → **+ Fargate**. Auto scale ECS → dựa **ECS Service's CPU utilization** (target tracking), không phải ALB/target group/alarm CPU.

🎯 **EKS: pod scale (HPA) nhưng số node không đổi khi hết capacity** → deploy **Cluster Autoscaler** (tích hợp ASG, launch node cho pending pods). HPA scale **pod**, Cluster Autoscaler scale **node**. Kubernetes ít overhead nhất → EKS + Fargate.

🎯 **Least-privilege IAM cho từng Pod trong EKS** (Pod A chỉ DynamoDB, Pod B chỉ S3) → **IRSA (IAM Roles for Service Accounts)**: tạo service account riêng mỗi service, map với IAM role scope hẹp (qua OIDC provider). KHÔNG gắn policy vào **EC2 instance profile của node** (mọi Pod trên node inherit hết — vi phạm least privilege). KHÔNG "gắn IAM policy vào Pod qua annotation" (không tồn tại). Kubernetes RBAC chỉ kiểm soát API trong cluster, không kiểm soát quyền AWS service.

📌 Gán IAM role cho Pod: **IRSA** hoặc **EKS Pod Identity** (đơn giản hơn, không cần OIDC provider).

⚠️ **Giá ECS:** **EC2 launch type** → trả **EC2 instances + EBS volumes** (không phí ECS thêm). **Fargate launch type** → trả **vCPU + memory** container yêu cầu (từ lúc pull image tới khi task dừng). Fargate = không quản server/patch.

🎯 **Batch job compute-heavy/file lớn (vd xử lý ảnh), tự scale, không quản server + cần điều phối task/state** → **AWS Batch** (tự cấp & scale compute cho job có queue) + **AWS Step Functions** (orchestrate workflow, state, retry) + **S3** (lưu output lớn). EKS self-managed node = overhead; Lambda = không hợp compute-heavy/file lớn (15 phút, RAM giới hạn); ASG tĩnh + EBS = quản lý thủ công.

---

## 28. VPC Networking Components

### NAT & Internet Gateway

🎯 Private subnet ra internet outbound-only IPv4 + **fully managed** → **NAT Gateway** (trong **public** subnet). HA → **mỗi AZ một NAT Gateway**. IPv6 → **Egress-only IGW**. **IGW làm NAT** cho instance có **public IP** (dịch public↔︎private).

🎯 **NAT Instance** = EC2 tự quản → có SG, làm bastion, port forwarding (không managed). **NAT Gateway** = managed → không SG (dùng NACL), không bastion, không port forwarding, tự scale + HA.

### Security Group vs NACL

🎯 **SG stateful** → chỉ cần mở **inbound**, response tự ra. **NACL stateless** → phải mở **cả inbound VÀ outbound** (+ ephemeral ports 1024-65535). Kết nối fail dù SG+NACL đã mở inbound → thiếu **outbound NACL**.

🎯 EC2 chỉ nhận traffic từ ALB → SG của EC2 tham chiếu **SG của ALB** (không phải CIDR — CIDR không đảm bảo chỉ ALB). Đây là "security group referencing".

⚠️ **SG source hợp lệ:** IP đơn / CIDR / Security Group khác / prefix list. **Không** dùng Internet Gateway ID làm source.

⚠️ **ELB đánh dấu instance unhealthy nhưng IP truy cập được** → (1) SG của EC2 không cho traffic từ **SG của ALB**, (2) **route health check** cấu hình sai. Không liên quan EBS/EIP/runtime.

⚠️ **EIP EC2 không ping được** → (1) route table chưa trỏ **IGW**, (2) SG chưa cho **ICMP** từ source. Source/Dest check không liên quan.

⚠️ **NLB target theo instance ID** → route qua **primary PRIVATE IP** của primary network interface (không public/elastic IP, không instance ID).

---

## 29. VPC Connectivity & Hybrid DNS

### Site-to-Site VPN

🎯 AWS Managed IPSec VPN: **Virtual Private Gateway (VGW) phía AWS** + **Customer Gateway (CGW) phía on-premises**. Đừng đảo ngược.

🎯 **Kết nối MẠNG on-prem ↔︎ AWS có mã hóa (network+session layer), thiết lập NHANH NHẤT, rẻ** → **Site-to-Site VPN** (IPsec, dùng ngay). Direct Connect = **không mã hóa mặc định** (cần thêm MACsec/VPN overlay) + mất ~1 tháng lắp cáp. DataSync/Secrets Manager ≠ dịch vụ kết nối mạng.

⚠️ **Site-to-Site VPN vs Client VPN vs Bastion:** Site-to-Site = **network-to-network** (cả mạng on-prem ↔︎ VPC). **Client VPN** = từng **user/máy cá nhân** kết nối vào VPC (không phải network-to-network). **Bastion host** = truy cập SSH/RDP tới từng server (không tạo kết nối mạng toàn diện). Migrate/archive khối lượng lớn 1 lần (PB) → **Snowball** (không dùng VPN/DX).

🎯 **Nhiều chi nhánh (VPN) + HQ (Direct Connect) muốn gửi/nhận data lẫn nhau qua AWS** → **AWS VPN CloudHub** (mô hình hub-and-spoke, các site VPN + DX giao tiếp với nhau qua VGW, không chỉ với VPC). VPC Peering/Endpoint/Software VPN không nối được các site on-prem với nhau.

### DX Virtual Interfaces (VIF)

🎯 **Truy cập S3 (dịch vụ public) từ on-prem QUA Direct Connect (không qua internet)** → **Public VIF** (advertise dải IP public của AWS như S3 qua DX). **Private VIF** = nối tới tài nguyên trong VPC (EC2/RDS, private IP) — KHÔNG dùng cho S3. TGW + DX Gateway cũng không cho truy cập S3 private-IP (S3 là public service). VPN qua internet = vi phạm yêu cầu private.

🎯 Cần **kết nối vừa DEDICATED/low-latency/high-throughput VỪA MÃ HÓA** (chấp nhận thời gian thiết lập) → **Direct Connect + VPN** (DX cho băng thông/độ trễ ổn định + IPSec để mã hóa). DX một mình **không mã hóa**; VPN một mình **không** low-latency/high-throughput; TGW một mình không tạo được kết nối on-prem.

### Maximize VPN throughput

🎯 Tăng throughput VPN qua tải cao → **Transit Gateway + ECMP (equal cost multipath) + nhiều VPN tunnel**. Một tunnel max **1.25 Gbps**; ECMP vượt giới hạn đó. **VGW không hỗ trợ ECMP**. (S3TA/Global Accelerator không tăng throughput VPN.)

### Kết nối VPC & on-prem

🎯 Vài VPC (**< 10, cùng account**) giao tiếp private, rẻ nhất → **VPC Peering** (point-to-point, không transitive, không phí hạ tầng theo giờ). Nhiều VPC + on-prem qua **1 hub, ít overhead** → **Transit Gateway** (không phải Transit VPC tự dựng bằng EC2 VPN, đắt).

🎯 Nhiều VPC (25) + on-prem qua Direct Connect + full-mesh → **Transit Gateway** (attach tất cả, route propagation) + **Transit VIF** (từ DX gắn vào TGW). DX Gateway + VGW **không** cho transitive routing giữa VPC.

🎯 **Multi-region DX + nhiều VPC + full connectivity giữa 2 data center** → **Direct Connect Gateway** (global, gắn VGW nhiều Region). Inter-region VPC peering không transitive; private VIF không gắn foreign-region VPC.

### Route 53 Resolver (hybrid DNS)

🎯 On-prem resolve tên trong **AWS VPC** → **INBOUND endpoint** (on-prem forward query VÀO Resolver). AWS VPC resolve tên **on-prem** → **OUTBOUND endpoint** (Resolver forward query RA on-prem). Nhớ: **Inbound = vào AWS, Outbound = ra on-prem**.

⚠️ **Bảng chọn connectivity:**

| Nhu cầu | Chọn |
| --- | --- |
| Nhiều account cùng Org, chung subnet, rẻ nhất | **VPC Sharing (RAM)** |
| Vài VPC 1-1 (<10, cùng account) | **VPC Peering** |
| Nhiều VPC/on-prem, hub, transitive | **Transit Gateway** |
| Share 1 service riêng tư | **PrivateLink** |
| On-prem↔︎AWS cáp vật lý | **Direct Connect** (VPN = qua internet, IPSec) |

---

## 30. Route 53

🎯 Map domain→domain (bên thứ ba) → **CNAME**. Map domain→AWS resource hoặc **zone apex** → **Alias** (miễn phí, Route 53-only). CNAME **không** dùng ở zone apex; Alias **không** trỏ ngoài AWS.

🎯 **Thay đổi record nhưng user chưa được redirect** → **TTL còn hiệu lực** (recursive resolver cache). Đổi setting đang dùng → nên đặt TTL ngắn (vd 300s) trước. Simple record không có health check.

🎯 Private hosted zone không resolve → check VPC bật **cả hai**: enableDnsHostnames + enableDnsSupport.

### Routing policies

🎯 **Thay đổi động kích thước vùng địa lý** route tới resource → **Geoproximity routing** (bias +/− mở rộng/thu hẹp vùng). **Geolocation** = theo vị trí user (không đổi kích thước vùng).

🎯 **Trang lỗi tĩnh dự phòng khi site chính (ALB) chết, ít thay đổi** → **Failover routing (active-passive)**: primary = ALB (có health check), secondary = **S3 static website** chứa trang lỗi. **KHÔNG có "active-active failover policy"** (active-active dùng weighted/latency + health check; active-passive chỉ dùng failover policy). Latency/Weighted không phải giải pháp dự phòng lỗi.

⚠️ **A**→IPv4; **AAAA**→IPv6; **PTR**→reverse; **MX**→mail; **NS**→name server. Routing: Simple / Weighted (bị DNS caching) / Latency / Failover / Geolocation / Geoproximity / Multivalue. "DNS caching là vấn đề" (blue-green) → Global Accelerator, không Route 53.

---

## 31. Load Balancing (ELB)

🎯 **Cross-zone ON**: chia đều **tất cả** instance (mỗi cái bằng nhau). **OFF**: chia đều theo AZ trước (mỗi node LB 50% nếu 2 AZ) → AZ ít instance gánh nhiều hơn. ALB mặc định ON (free); NLB mặc định OFF.

- Ví dụ (cross-zone OFF, 2 AZ): AZ-A 4 target, AZ-B 6 target → mỗi node nhận 50% → **AZ-A mỗi target 12.5%**, AZ-B mỗi target 8.3%.

🎯 Rate limit + quota **per-client** ở tầng API → **API Gateway usage plans + API keys**. ALB/NLB/GWLB không có rate limiting per-client.

🎯 **Whitelist một public IP tĩnh + HA + scale** → **NLB + ASG** (NLB có static/Elastic IP). ALB/Classic expose **DNS** (không IP tĩnh) → không whitelist IP được.

🎯 **Bastion host HA (SSH = TCP port 22)** → **public NLB (L4/TCP) + ASG các bastion**. ALB chỉ HTTP/HTTPS (L7) → không hợp SSH. Một EIP không HA (chỉ gắn 1 instance); VPC Endpoint không đặt trước EC2.

🎯 **Nhiều HTTPS domain khác nhau trên 1 ALB, ít cấu hình** → **SSL cert + SNI** (nhiều cert trên cùng listener). Wildcard cert chỉ hợp subdomain cùng gốc; SAN cần cùng CA.

🎯 **Content-based routing / định tuyến theo nội dung request** (vd `/orders` → microservice A, `/products` → B) → **ALB** (L7). NLB (L4) **không** làm content-based routing. Kết hợp ASG + multi-AZ cho HA. **Không** dùng ASG/EIP để "phân phối traffic" (ASG chỉ quản instance, EIP chỉ mask lỗi 1 instance).

⚠️ **Các kiểu routing của ALB:** **Path-based** (`/orders` vs `/products`) / **Host-based** (nhiều domain) / **HTTP header-based** / **HTTP method-based** / **Query string-based** / **Source IP CIDR-based**. Định tuyến theo **URL path** → **Path-based**.

⚠️ **Host-based routing** `*.example.com` khớp `test.example.com` **nhưng KHÔNG** khớp `example.com` (không case-sensitive, cần ≥1 dấu `.`).

⚠️ **ALB (L7):** HTTP/HTTPS, path/host routing, WAF, Cognito. **NLB (L4):** TCP/UDP, static IP, ultra-low latency, target private IP. **GWLB (L3/4):** virtual appliance bên thứ 3.

📌 **Classic Load Balancer** là thế hệ trước (previous generation) — gần như luôn là đáp án sai.

---

## 32. API Gateway

🎯 Validate **JWT từ OIDC provider** natively + ít latency/cost + không viết auth code → **API Gateway HTTP API + native JWT authorizer**. REST API + Lambda authorizer = custom code (đắt, latency); WebSocket = không native JWT.

🎯 **Giảm chi phí Aurora read + chấp nhận stale ~24h + read-heavy REST API** → bật **API Gateway caching** (TTL tới 3600s, giảm call tới backend). Không cần Read Replica/ElastiCache.

🎯 **REST API real-time analytics qua HTTP** (nhận data từ mobile, lưu & phân tích) → **API Gateway + Kinesis Data Analytics** (API Gateway integrate trực tiếp Kinesis). Lambda **không** dùng để "lưu & truy vấn data để phân tích"; Athena/QuickSight không tạo REST API.

⚠️ **REST API = stateless** (HTTP, GET/POST/PUT/PATCH/DELETE). **WebSocket API = stateful, full-duplex** hai chiều (real-time). API Gateway hỗ trợ cả hai.

⚠️ Chặn IP cho API Gateway → **Resource Policy** (không SG). Rate limit per-client → usage plans + API keys. HTTP API rẻ/nhanh hơn REST API.

---

## 33. Cognito & Authentication

🎯 Offload authentication ít effort → **Cognito User Pools** tích hợp sẵn vào **ALB** (không code). **User Pools** = authentication (đăng nhập). **Identity Pools** = AWS credentials tạm (authorization). CloudFront auth cần Lambda@Edge (nhiều effort).

---

## 34. Disaster Recovery

🎯 "Least downtime / RTO thấp" → hạ tầng **đã dựng sẵn** (pre-provisioned, ASG sẵn sàng). CloudFormation/Lambda provision **khi** sự cố → loại (tốn thời gian).

🎯 **Scaled-down version FULLY functional luôn chạy** trên cloud → **Warm Standby**. **Pilot Light** = chỉ core tối thiểu chạy. **Multi-Site** = active-active. **Backup & Restore** = rẻ nhất, RTO cao.

🎯 **RTO 5 phút nhưng cài đặt 45 phút, mọi Region** → tạo **AMI** (đã cài sẵn) + **copy AMI qua tất cả Region**, dùng AMI region-specific để recover. AMI bound theo Region → phải copy.

⚠️ **4 chiến lược (downtime giảm / chi phí tăng):** Backup & Restore → Pilot Light → Warm Standby (bản nhỏ luôn chạy) → Multi-Site Active-Active (RTO ~0).

---

## 35. AWS Systems Manager

🎯 Patch OS EC2 tự động, ít overhead, không đụng IAM role sẵn → **SSM Default Host Management Configuration**. EC2 chỉ gắn **1 role**; Hybrid Activation cho on-prem/non-EC2; detach role → rủi ro gián đoạn; cron = overhead cao.

⚠️ SSM gồm: Patch Manager, Session Manager (SSH không cần bastion), Parameter Store, Run Command, Inventory, State Manager, Automation, Maintenance Windows.

🎯 **Patch EC2 đứng sau ALB mà không rớt traffic** → **SSM Automation runbook `AWSEC2-PatchLoadBalancerInstance`** (tự deregister khỏi target group → chờ drain → patch → reboot → register lại) + lên lịch bằng **SSM Maintenance Windows**. Đừng disable network interface (mất kết nối SSM → patch fail); đừng chỉnh target group thủ công (không scale).

---

## 36. AZ / Region

🎯 Tên AZ (us-west-2a) ánh xạ **khác nhau mỗi account**. Xác định **cùng AZ vật lý qua nhiều account** → **AZ ID** (usw2-az2, nhất quán toàn cầu). Không cần AWS Support. VPC span cả region, subnet span 1 AZ nhưng tên bị ánh xạ khác → không đối chiếu được.

---

## 37. Elastic Beanstalk

🎯 **Rút thời gian tạo instance mới < 2 phút** khi cài đặt lâu (45 phút, có static + dynamic) → **Golden AMI** (cài sẵn phần static) + **EC2 user data** (chỉ chạy phần dynamic lúc boot). Không lưu file cài trên S3 (vẫn cần môi trường), user data không dùng để "cài" phần dài.

---

## 38. AWS Lambda — Best Practices

🎯 Lambda mặc định chạy trong **VPC của AWS** → truy cập internet/public AWS API tự do. Khi **VPC-enable** (để gọi resource private như RDS) → cần **route qua NAT Gateway** ở public subnet để ra internet.

🎯 Lambda scale rất nhanh → đặt **CloudWatch Alarm** cho `ConcurrentExecutions`/`Invocations` vượt ngưỡng. Code tái dùng nhiều function → tạo **Lambda Layer** (ZIP libraries, tối đa 5 layer, tổng ≤ 250MB unzipped).

🎯 **SNS → Lambda mất message khi traffic tăng vọt** (vd 100 → 5000 req/s) → nguyên nhân thường là chạm **account concurrency quota của Lambda (mặc định ~1,000/account/region)** → message bị throttle → **liên hệ AWS Support tăng limit**. SNS & Lambda đều **serverless** → KHÔNG "provision thêm server". SNS tự scale (không phải nút thắt).

⚠️ Có thể đóng gói & deploy Lambda dưới dạng **container image**. Đừng over-provision timeout (chạy lâu ngoài ý muốn → tốn tiền). Deploy package lớn → cold-start chậm (nhưng dependencies vẫn gói chung, không tách riêng).

📌 Lambda tính phí theo **1 ms**.

---

## 39. Cost Optimization

🎯 **Idle / low-utilization EC2** → **Cost Optimization Hub** (báo cáo tổng hợp: rightsizing, idle deletion, Savings Plans, RI) + **Compute Optimizer** (gợi ý instance type theo utilization).

🎯 **Enforce trần chi tiêu tự động (vd account dev)** → **AWS Budgets + Budgets Actions** (khi vượt ngưỡng → tự gắn **DenyAll IAM policy** / SCP, chặn tạo thêm resource). Cost Explorer = chỉ xem/phân tích (không enforce). Lambda tự viết = nhiều overhead.

🎯 **Cảnh báo khi COVERAGE của Savings Plans tụt dưới ngưỡng** → **AWS Budgets: Savings Plans coverage budget** (theo dõi % usage được SP phủ, alert qua SNS/email). CloudWatch không có metric coverage sẵn; Compute Optimizer không theo dõi SP coverage.

⚠️ S3 Storage Class Analysis chỉ gợi ý chuyển Standard → Standard-IA (không gợi ý One Zone-IA/Glacier). Trusted Advisor **không** tự renew RI. CloudFormation là **free** (chỉ tính tài nguyên nó tạo).

📌 **CUR** nay nên dùng **CUR 2.0 qua AWS Data Exports**.

---

## 40. Analytics & AI/ML (no-code / SQL-based)

🎯 **Trích xuất entity (vd tên nguyên liệu) từ text, không có ML expert, ít overhead** → **Amazon Comprehend** (custom entity recognition, fully managed) + S3 event → Lambda → DynamoDB. KHÔNG SageMaker (cần ML expert), không Transcribe (audio), không Lookout for Vision (ảnh).

🎯 **Trích xuất text từ PDF/scan/ảnh tài liệu (layout phức tạp)** → **Amazon Textract** → rồi phân tích sentiment/entity bằng **Comprehend**. Rekognition = computer vision (ảnh/video), KHÔNG phân tích tone của text. Athena = query structured, không NLP.

🎯 **Tập trung log/sự kiện bảo mật từ nhiều account/service (+ third-party), chuẩn hóa, ít code** → **Amazon Security Lake** (tự thu thập từ CloudTrail/VPC Flow/GuardDuty/Config, chuẩn hóa **OCSF**, lưu S3). Lake Formation/Glue = data lake business (cần ETL thủ công); Lambda gom CSV = nhiều code.

🎯 **Chuẩn bị/làm sạch dữ liệu no-code + data profiling + lineage + chia sẻ "recipe"** cho analyst → **AWS Glue DataBrew** (drag-drop, serverless, recipe tái dùng, column stats). Glue Studio = low-code cho developer (Spark, không profiling built-in); Athena = cần SQL; AppFlow = SaaS data movement.

🎯 **ETL + kho dữ liệu MPP serverless + analyst train ML bằng SQL** → **AWS Glue (ETL) + Amazon Redshift Serverless (MPP) + Amazon Redshift ML** (CREATE MODEL bằng SQL, gọi SageMaker ngầm). EMR/Redshift provisioned = phải quản cluster; RDS/Aurora = không phải MPP warehouse; SageMaker notebook = cần Python.

⚠️ **Kinesis Data Analytics** (nay: Managed Service for Apache Flink) = query/phân tích stream real-time. Pipeline điển hình: KDS → Flink → Firehose → S3. API Gateway integrate được với Kinesis cho REST API ingest real-time.

🎯 **Real-time SEARCH + analytics + dashboard trên streaming data** → **Kinesis Data Streams (ingest) → Amazon OpenSearch Service (index/full-text search) → QuickSight (dashboard)**. OpenSearch = search/log analytics chuyên dụng (DynamoDB/Redshift không tối ưu full-text search realtime).

🎯 **Phân tích cuộc gọi/audio (sentiment) bằng SQL** → **Amazon Transcribe** (speech→text, có speaker identification 2–10 người) → lưu S3 → **Athena** (SQL). QuickSight = visualize (không phải công cụ query SQL). Alexa/Kinesis không đọc audio file.

📌 **Amazon QuickSight** nay là **Amazon Quick Sight** (thuộc Amazon Quick).

---

## 41. Hybrid & Edge (on-premises data residency)

🎯 **Chạy Kubernetes/EKS-compatible HOÀN TOÀN trên on-prem (data phải ở lại data center) + dùng AWS API/CloudWatch/IAM** → **AWS Outposts rack + Amazon EKS Anywhere** (mang AWS service về data center, data không rời on-prem).

⚠️ **Phân biệt hybrid/edge:**

| Dịch vụ | Bản chất |
| --- | --- |
| **Outposts** | Rack AWS đặt TRONG data center khách (data residency tuyệt đối, EKS/EC2/S3/CloudWatch local) |
| **Local Zones** | Hạ tầng AWS gần metro (do AWS quản, **không** trong data center khách → không thỏa data residency) |
| **Snowball Edge** | Thiết bị offline/edge processing tạm thời (không phải production dài hạn; 📌 không nhận khách mới từ 07/11/2025) |
| **Direct Connect + EKS cloud** | Workload/data vẫn ở CLOUD → không thỏa "data ở lại on-prem" |

---

## 42. Phụ lục — Tra cứu dịch vụ & bẫy distractor

> Những dịch vụ hay xuất hiện ở **đáp án SAI**. Nhớ "nó thực chất làm gì" để loại nhanh khi bị nhồi làm distractor.

### Compute

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **AWS Batch** | Chạy batch/compute-heavy job có queue + dependency, tự scale compute environment | Cron container đơn giản (thừa setup) → EventBridge + Fargate; điều phối workflow → thêm Step Functions; realtime → sai |
| **App Runner** | Chạy web app/container từ source, tự scale (📌 đóng với khách mới từ 30/04/2026 → ECS Express Mode) | Task ngắn/thưa event-driven (tốn khi idle) → dùng Lambda |
| **Lightsail** | Hosting đơn giản web/app nhỏ, giá gộp | Backend enterprise cần orchestration/scale → dùng ECS/EKS |
| **Elastic Beanstalk** | PaaS tự provision EC2+ELB+ASG cho app | Khi cần serverless thuần / kiểm soát thấp |
| **Step Functions** | Orchestrate workflow nhiều bước (state machine) | Cron/1 tác vụ đơn (thừa) → EventBridge + Fargate/Lambda |
| **AWS MGN (Application Migration Service)** | Lift-and-shift VM on-prem → EC2 (agent, cutover) | Đây cho migration; DR → Elastic Disaster Recovery |
| **Outposts / EKS Anywhere** | AWS trong data center on-prem (data residency) | Khi data được phép ở cloud (thừa chi phí) |
| **Snowball Edge Compute** | Xử lý edge/offline tạm thời | Production dài hạn (không hợp) |

### Storage

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Instance Store** | Đĩa vật lý gắn host, ephemeral, IOPS cao, rẻ | Cần data bền vững (mất khi stop) |
| **S3 Glacier Vault** | Kho archive (Vault Lock) — 📌 dịch vụ vault không nhận khách mới | Snowball/UNLOAD không target trực tiếp; phải qua S3 |
| **Volume Gateway (iSCSI)** | Block storage hybrid | Cần file share SMB/NFS → dùng File Gateway/FSx |
| **S3 One Zone-IA** | Rẻ hơn 20% nhưng **1 AZ** | Data quan trọng cần đa AZ (rủi ro mất AZ) |
| **EBS st1/sc1 (HDD)** | Throughput/cold, rẻ | Cần IOPS cao → io1/io2; cần boot → không được |
| **Mountpoint for S3** | File client mount S3 (Linux), có local cache | Không POSIX đầy đủ → không thay file share on-prem |
| **EBS Fast Snapshot Restore** | Volume từ snapshot có full perf tức thì | (đúng cho clone data test cần IOPS ngay) |
| **EBS Multi-Attach (io1/io2)** | 1 volume ↔︎ nhiều instance (cùng AZ, clustered) | Clone/tách môi trường (rủi ro ghi đè) |
| **Recycle Bin (EBS)** | Giữ snapshot/AMI/volume đã xóa N ngày (khôi phục được) | (đúng cho chống xóa nhầm snapshot tạo tay/script) |
| **S3 on Outposts** | Object storage trên Outposts (on-prem) | Không phải bài on-prem → thừa |

### Database & Cache

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Amazon Neptune** | Graph DB (quan hệ nhiều tầng) | Không phải bài graph → dùng RDS/Aurora/DynamoDB |
| **Amazon Timestream** | Time-series DB (telemetry/metrics); 📌 Timestream for LiveAnalytics đóng với khách mới | Workload relational chung → dùng RDS |
| **Amazon Redshift** | Data warehouse MPP (analytics) | OLTP/relational giao dịch → dùng RDS/Aurora |
| **QLDB** | Ledger immutable, cryptographically verifiable — 📌 **ngừng hẳn 31/07/2025** (nguồn web), AWS gợi ý Aurora PostgreSQL | Không cần audit-ledger → RDS |
| **ElastiCache (Valkey/Redis OSS/Memcached)** | Cache in-memory | Write-heavy/ETL/JOIN phức tạp/lưu bền → không hợp |
| **DAX** | Cache in-memory cho **DynamoDB** | Không phải DynamoDB → dùng ElastiCache |
| **RDS Custom** | RDS cho phép truy cập host OS | Không cần customize OS → RDS thường |
| **DMS / DMS Serverless** | Replicate/migrate DB (full+CDC); Serverless tự scale | ETL batch → Glue; realtime CDC → DMS |
| **AWS Data Pipeline** | Orchestrate data workflow (📌 không nhận khách mới → Glue/Step Functions/MWAA) | Cross-Region replicate DynamoDB → Global Tables |

### Analytics & AI/ML

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Athena** | Query SQL trực tiếp trên S3 | Không tạo REST API / không NLP / không near-real-time |
| **OpenSearch Service** | Full-text search + log analytics real-time | (đúng cho full-text search; DynamoDB/Redshift không tối ưu) |
| **QuickSight** (📌 nay Quick Sight) | BI visualization/dashboard | Không phải công cụ phân tích/NLP nguồn |
| **Glue** | ETL serverless (Spark) | Không phải no-code prep → dùng DataBrew |
| **Glue DataBrew** | Chuẩn bị data no-code + profiling + recipe | (là đáp án đúng cho no-code prep) |
| **EMR** | Hadoop/Spark cluster big data | Cần managed/serverless đơn giản (nhiều overhead) |
| **Kinesis Firehose** (📌 nay Amazon Data Firehose) | Load stream → S3/Redshift/OpenSearch/Splunk/HTTP (không code) | Cần xử lý tùy biến/replay → Data Streams; không ghi DynamoDB |
| **Kinesis Data Analytics** (📌 nay Managed Service for Apache Flink) | Query/phân tích stream real-time | Không phải streaming → Athena |
| **Lake Formation** | Xây & bảo mật data lake business | Gom log bảo mật → dùng Security Lake |
| **AppFlow** | Chuyển data SaaS↔︎AWS (API) | Không hỗ trợ SFTP/file lớn tại chỗ → Transfer Family/DataSync |
| **Rekognition** | Computer vision (ảnh/video) | Phân tích TEXT/sentiment → Comprehend |
| **Transcribe** | Speech-to-text (audio) | Text sẵn có (thừa vòng) |
| **Textract** | Trích text từ PDF/scan/ảnh tài liệu | (đúng cho extract document) |
| **Lookout for Vision** | Phát hiện anomaly trong ẢNH — 📌 **ngừng 31/10/2025** (nguồn web) | Xử lý text (sai hoàn toàn) |
| **SageMaker** | Xây/train model ML tùy biến | "Không có ML expert / ít overhead" → Comprehend/Rekognition managed |

### Networking

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Transit Gateway** | Hub nối nhiều VPC/on-prem, transitive | Rẻ nhất chia subnet cùng Org → RAM; vài VPC → Peering |
| **Transit VPC** | Mô hình CŨ (EC2 VPN tự quản), đắt | Gần như luôn SAI khi có Transit Gateway |
| **VPC Peering** | Nối 1-1, không transitive | Nhiều VPC full-mesh (không scale) → TGW |
| **PrivateLink** | Chia sẻ 1 service riêng tư (qua NLB) | Nối toàn bộ mạng nhiều VPC (sai mục đích) |
| **Direct Connect** | Cáp vật lý on-prem↔︎AWS (~1 tháng) | Cần gấp/mã hóa một mình (thêm VPN); overkill nội-Region |
| **CloudFront** | CDN HTTP/HTTPS cache | Non-HTTP (TCP/UDP) → Global Accelerator |
| **Global Accelerator** | Static IP, TCP/UDP, đa Region, né DNS cache | Cache HTTP content → CloudFront |
| **Client VPN** | User/thiết bị lẻ kết nối vào VPC | Kết nối network-to-network → Site-to-Site VPN |

### Security & Identity

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **GuardDuty** | Phát hiện threat (không mitigate) | Chống DDoS chủ động → Shield Advanced |
| **Macie** | Phát hiện PII trên S3 | Access control (không enforce quyền) |
| **Inspector** | Quét lỗ hổng EC2/container/Lambda | Threat detection → GuardDuty |
| **Shield Advanced** | Chống DDoS + SRT | Lọc L7/IP → WAF |
| **WAF** | Lọc L7 (IP/SQLi/XSS) cho CloudFront/ALB/API GW/AppSync/Cognito… | Gắn S3/NLB/Global Accelerator (không hỗ trợ) |
| **Config** | Đánh giá tuân thủ + lịch sử cấu hình | "Ai gọi API" → CloudTrail |
| **CloudTrail** | Log API "ai làm gì" (audit) | Hiệu năng → CloudWatch; không stream Kinesis trực tiếp |
| **Control Tower** | Dựng landing zone multi-account | SSO gán quyền theo group → IAM Identity Center |
| **Directory Service (AD Connector/Managed AD/Simple AD)** | Kết nối/chạy AD | Xem §24 để chọn đúng loại |
| **CloudHSM** | HSM chuyên dụng (kiểm soát tuyệt đối key) | Chỉ cần rotate/audit đơn giản → KMS |
| **Secrets Manager / Parameter Store** | Lưu secret ngoài code (rotate) | Đừng hardcode/nhét credential vào EC2 |
| **IAM Access Analyzer** | Phát hiện resource chia sẻ ra ngoài account/Org; unused access | (đúng cho external access); last-accessed → Access Advisor |
| **IAM Access Advisor** | Service nào được dùng lần cuối (bỏ quyền thừa) | Không phát hiện external sharing |
| **Security Hub** | Tổng hợp/xếp hạng findings bảo mật (📌 Security Hub CSPM: kiểm tra tư thế bảo mật; Security Hub: giải pháp hợp nhất) | Không tự chặn SQLi/XSS (đó là WAF) |
| **Firewall Manager** | Quản trị tập trung rule WAF/Shield/SG/Network Firewall/DNS Firewall qua Org | Không áp NACL/Route Table; không phát hiện/chặn trực tiếp tấn công |
| **S3 Storage Lens** | Dashboard cấu hình/dùng S3 toàn org (vd versioning) | Không trigger event (đó là S3 Event Notification) |

### Monitoring / Ops / Cost

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Trusted Advisor** | Khuyến nghị (cost/security/limit) | Không tự recover/không phát hiện anomaly API |
| **Compute Optimizer** | Gợi ý instance type theo utilization | (kết hợp Cost Optimization Hub) |
| **Cost Optimization Hub** | Tổng hợp gợi ý tiết kiệm (idle/rightsize/RI) | (đúng cho tối ưu cost EC2 idle) |
| **CloudFormation / StackSets** | IaC; StackSets = nhiều account+Region | Provision KHI sự cố (chậm) → không hợp DR least-downtime |
| **Systems Manager** | Patch/Session/Parameter/Run Command | Xem §35 |
| **EC2 Instance Connect** | SSH tạm (inject key 1 lần), audit CloudTrail | Private subnet → cần Instance Connect Endpoint |
| **Detailed Monitoring** | Metric EC2 1 phút (thay 5 phút) | (đúng khi cần ≤1 phút, ít setup, không cần Agent) |
| **CloudWatch agent** | Đẩy log/metric EC2 → CloudWatch liên tục | (đúng cho giữ log trước khi ASG terminate instance) |

---

## 43. Nguyên tắc làm bài chung

🎯 "least effort / minimal code / urgent" → tính năng **có sẵn** (auto-scaling, managed rule, native integration), tránh migrate/rewrite/tự dựng.

🎯 "most cost-effective" → cái **rẻ nhất vừa đủ**, đừng chọn cái mạnh/quen nhưng thừa. "select N": hỏi "phát biểu này có **đúng** không" TRƯỚC khi xét khớp triệu chứng.

🎯 Managed/edge service (API Gateway, CloudFront, S3, Lambda) **không có Security Group** → chặn IP bằng **Resource Policy / WAF**.

🎯 **Cách loại đáp án SAI (rất hiệu quả):** phần lớn distractor sai vì (1) **sai loại dịch vụ** cho task (Rekognition cho text, Firehose ghi DynamoDB, WAF gắn S3, NLB content-routing); (2) **thừa overhead** khi đã có managed/native (tự dựng SFTP/RabbitMQ trên EC2, CloudFormation provision lúc sự cố); (3) **phát biểu bịa/ngược** ("read replica synchronous", "SNS có thể poll", "chuyển single→multi-region KMS key"); (4) **vi phạm ràng buộc đề** (dùng internet khi đề cấm, EC2 khi đề yêu cầu serverless). Nhận ra 1 trong 4 lỗi này là loại được ngay.

🎯 **Từ khóa bẫy khóa chặt đáp án:** third-party cert · DNS caching · file storage · SMB/NFS · DFS · AMQP/MQTT · root user · least downtime · cross-account · imported vs created · synchronous · serverless · immediate vs polling · UDP/non-HTTP · graph query · hot partition · deletion protection · proactive vs reactive · min storage duration · data residency/on-prem · T-SQL · no-code/no ML expert · MPP · content-based routing · least operational overhead · lift-and-shift/rehost · rate abuse · public VIF vs private VIF · Compute vs EC2 vs SageMaker Savings Plan · coverage budget · byte-range · request-response.

---

## 44. Bổ sung theo dịch vụ (từ đề mock 01–05)

### API Gateway

🎯 API Gateway là **managed service, KHÔNG nằm trong VPC** → không gắn SG/NACL/subnet. Whitelist IP → **Resource Policy** (Deny mọi IP ngoài danh sách). **Private integration** = gọi backend trong VPC (không phải kiểm soát client IP). **Private REST API** = cho phép gọi từ VPC qua Interface Endpoint, nhưng API vẫn nằm ngoài VPC.

⚠️ **HTTP API vs REST API:** HTTP API **rẻ hơn ~70%**, đủ cho proxy đơn giản (Lambda/HTTP backend). REST API khi cần **API key, usage plan, request validation, resource policy**. Resource policy chỉ có ở **REST API** (HTTP API v2 không hỗ trợ).

⚠️ **API Gateway endpoint type:** **edge-optimized** (user toàn cầu) · **regional** (một Region) · **private** (trong VPC).

### IAM / Identity

🎯 **Nhiều account (Organizations) + AD sẵn có** → **AD Connector + IAM Identity Center + permission sets**. **Một account đơn lẻ + AD sẵn có** → **SAML 2.0 federation + IAM role map theo AD group** (đừng phản xạ "AD = Identity Center"). Đã dùng Identity Center rồi → mở rộng permission set, đừng tạo IAM user/role song song.

🎯 **SSO nhiều account nhưng vẫn quản lý user ở AD on-prem** → **IAM Identity Center + two-way forest trust** (với AWS Managed Microsoft AD).

🎯 **Workload chạy on-premises cần credential tạm thời** → **IAM Roles Anywhere** (dựa trên certificate X.509). Đừng phát access key dài hạn cho server on-prem.

🎯 **Nhiều Lambda cùng truy cập S3 nhạy cảm, least privilege** → **mỗi function một execution role riêng**. KHÔNG liệt kê ARN từng function trong bucket policy (không scale, policy có giới hạn độ dài).

🎯 **Cấp quyền cho developer/nhân sự** → **IAM role (credential tạm)**, không phát **access key** (credential dài hạn). Người ngoài tổ chức → **Identity Center / federation**, tuyệt đối không tạo IAM user local ở từng account.

🎯 **Cross-account access**: **role tạo ở account ĐÍCH**, **trust policy trỏ tới account NGUỒN** (vd truy cập từ Dev sang Prod → role nằm ở Prod).

🎯 **Directory (LDAP) không hỗ trợ SAML** → **custom identity broker + AWS STS**.

🎯 **Bật MFA theo rủi ro (vị trí/IP/thiết bị lạ) cho hàng triệu end-user** → **Cognito User Pool + adaptive authentication**.

⚠️ **IAM Access Analyzer** còn có **unused access analyzer** (rà quyền thừa / permission không dùng trên toàn Organization), ngoài chức năng phát hiện external sharing.

⚠️ **Không thể thêm IAM user của account khác vào IAM group của mình** → cross-account phải dùng **role + AssumeRole**.

⚠️ **Root user đăng ký được tối đa 8 thiết bị MFA** → chống mất thiết bị phải chuẩn bị **trước**. 📌 MFA cho root nay **bắt buộc**.

⚠️ **`AdministratorAccess` = toàn quyền**; **`SystemAdministrator` chỉ là job-function policy hạn chế** (EC2, CloudWatch, Systems Manager… **không** gồm IAM/Organizations/billing). Đọc kỹ tên policy.

### Route 53

⚠️ **Multivalue vs Simple:** **Multivalue** trả nhiều IP (tối đa **8 record**) **CÓ health check**; **Simple** cũng trả nhiều IP nhưng **KHÔNG health check**. **Latency** và **Geolocation** chỉ trả về **1 record**.

🎯 **Bản quyền phân phối / phục vụ nội dung khác nhau theo quốc gia** → **Geolocation** (theo vị trí *user*). **Geoproximity** dựa vị trí *resource* + bias → dùng cân tải giữa Region, KHÔNG dùng cho tuân thủ pháp lý.

⚠️ **Weighted routing chia traffic theo tỉ lệ, KHÔNG xét vị trí địa lý** → không giảm latency. Giảm latency toàn cầu cho **LB sẵn có / non-HTTP** → **Global Accelerator**.

### S3 — tính năng bổ sung

🎯 **Requester Pays**: người tải trả phí request + data transfer → giảm chi phí cho **chủ bucket**. (CRR thì giảm chi phí cho **bên nhận** vì data đã ở gần họ.)

🎯 **Cho phép truy cập object tạm thời mà không mở public** → **Presigned URL** (vd CloudFormation cần đọc template trong bucket private theo yêu cầu cụ thể). Đừng nhầm với VPC endpoint (đó là **đường mạng**, không phải **quyền**).

🎯 **Xử lý/xóa hàng triệu object, serverless, ít effort** → **S3 Inventory + S3 Batch Operations (+ Lambda)**. Đừng tự viết script liệt kê từng object. **Mã hóa lại hàng triệu object đã tồn tại** → cũng là **S3 Inventory + S3 Batch Operations**.

🎯 **Hàng triệu file rất nhỏ (~10 KB)** → **gộp thành object lớn trước khi upload**: chi phí **request** có thể lớn hơn chi phí **storage**; Standard-IA/One Zone-IA còn tính **minimum billable 128 KB/object**.

🎯 **Chi phí S3 tăng dần mà SỐ LƯỢNG và KÍCH THƯỚC object không đổi** → có thứ bị tính tiền mà không nhìn thấy: **incomplete multipart upload** (thêm lifecycle rule xoá), noncurrent version, delete marker. KHÔNG phải vấn đề đường truyền (Transfer Acceleration/CloudFront là bẫy).

🎯 **Che/redact PII ngay lúc bên thứ ba ĐỌC file, không nhân bản dữ liệu** → **S3 Object Lambda Access Point** (biến đổi lúc đọc). Muốn có sẵn bản đã xử lý → S3 Batch Operations.

📌 **S3 Object Lambda** từ 07/11/2025 chỉ còn cho khách đang dùng và một số APN partner. Thay thế: Lambda qua CloudFront/API Gateway/function URL, hoặc xử lý ở client.

🎯 **Gom log từ nhiều account về một bucket, cùng Region, ít overhead + rẻ nhất** → **S3 Same-Region Replication (SRR)** (managed, không tốn compute). Đừng viết **Lambda trigger `s3:ObjectCreated` để copy**. **Lifecycle KHÔNG copy object sang bucket khác.**

🎯 **EC2 truy cập S3 dữ liệu mật qua đường riêng + khoá truy cập** → **VPC Endpoint + bucket policy giới hạn theo endpoint** (`aws:SourceVpce`).

⚠️ **Bảng thời gian lấy về (thuộc lòng):** Standard / Standard-IA / **Glacier Instant Retrieval** = **tức thời (ms)**. **Glacier Flexible**: Expedited **1–5 phút nhưng CHỈ cho archive ≤ 250 MB**, Standard 3–5 giờ, Bulk 5–12 giờ. **Deep Archive**: 12–48 giờ. → Đề nói "phát/lấy trong vài phút" với **file lớn (GB)** thì **mọi tầng Glacier đều loại**, chỉ còn Standard/Standard-IA.

⚠️ **Lifecycle theo tuổi object SAI** khi "một số object lâu ngày vẫn hot, một số mới đã nguội" → dùng **Intelligent-Tiering**.

⚠️ **S3 Storage Class Analysis chỉ ĐƯA DỮ LIỆU** để bạn tự viết lifecycle rule → thêm việc thủ công. Muốn tự động hoàn toàn → **Intelligent-Tiering**.

⚠️ **S3 Storage Lens** = dashboard thống kê/cấu hình, **KHÔNG** trigger event (trigger là **S3 Event Notification**).

⚠️ **Access Point** = quản lý **quyền** theo prefix, **không** thay đổi đường đi của traffic.

⚠️ **CloudFront signed URL** = cấp quyền tạm cho **từng nội dung/từng khách** (thuê phim, tải nhạc). **OAC** chỉ khoá đường S3 ↔︎ CloudFront, **không phân biệt khách hàng**.

### EC2 / EBS bổ sung

🎯 **Không share hardware vật lý + chống lỗi tương quan** → **Spread placement group** (mỗi instance một hardware riêng, tối đa **7 instance/AZ/group**). **Cluster placement group** = gom sát để low-latency (ngược mục tiêu). **Dedicated tenancy** = không share với account khác, nhưng instance **cùng account** vẫn có thể chung hardware.

🎯 **Độ trễ mạng thấp nhất giữa node** → **Cluster placement group + enhanced networking (SR-IOV/ENA)** — phụ thuộc **vị trí vật lý + driver mạng**, KHÔNG phụ thuộc băng thông hay số ENI. **HPC tightly coupled** → thêm **EFA** + **FSx for Lustre (scratch)**.

🎯 **io1 đã chạm trần 64,000 IOPS/volume** → **thêm volume + LVM stripe** (scale ngang). Tăng dung lượng **không** tăng IOPS khi đã chạm trần. (io2 Block Express tới **256,000 IOPS** — nhưng nếu đề không đưa io2 vào đáp án thì stripe là đúng.)

🎯 **Cấm tạo EBS volume chưa mã hoá (preventive)** → bật **EBS encryption by default** trong **EC2 account attributes** — lưu ý đây là setting **theo từng Region**, phải bật ở mỗi Region. **AWS Config chỉ PHÁT HIỆN sau khi đã tạo** (detective) → không thỏa "prevent the creation".

🎯 **Write-heavy làm DB chậm** → **Provisioned IOPS SSD (io1/io2)**. Cache **không cứu được** workload write-heavy (chỉ giúp đọc).

🎯 **Ràng buộc "không được chạy quá MỘT instance"** → không thể HA → mục tiêu chuyển sang **rút ngắn RTO**: **CloudWatch alarm + action `recover`** (giữ nguyên IP, metadata, EBS). RAID/termination protection không phải phục hồi. (Lưu ý thực tế: instance thế hệ mới đã **bật auto recovery mặc định**.)

🎯 **Tính HA đúng cách**: "luôn cần ≥ N instance" phải thoả **SAU KHI mất trọn 1 AZ** → đặt **N instance ở MỖI AZ** (vd cần ≥2 → ASG min = 4, mỗi AZ 2 On-Demand). Stateful production → On-Demand, tránh Spot.

🎯 **Bật/tắt EC2 và RDS theo lịch** → **Instance Scheduler on AWS** (giải pháp dựng sẵn), đừng viết Lambda.

⚠️ **Recycle Bin** khôi phục được **EBS snapshot, EBS-backed AMI và (📌 nay cả) EBS volume** bị xóa nhầm (tính năng built-in, không cần script).

⚠️ **Capacity Reservation** = đảm bảo **có chỗ**, KHÔNG tăng performance. **Dedicated tenancy** = cách ly với account khác, KHÔNG tối ưu network. **Cluster placement group** mới là thứ tối ưu latency node-to-node (HPC).

⚠️ **Memory optimized (R/X series)** cho SAP, SQL Server, in-memory workload.

⚠️ **gp3 rẻ hơn gp2 ~20%** ở cùng hiệu năng, IOPS/throughput **tách rời dung lượng** → gp2 **không phải** lựa chọn rẻ nhất.

⚠️ **EC2 KHÔNG có khái niệm "Multi-AZ deployment"** như RDS.

⚠️ **Rightsize:** CPU ~25% → giảm size; CPU ~60% ổn định → giữ nguyên.

### Load Balancer bổ sung

🎯 **ALB là load balancer DUY NHẤT route được tới Lambda target.** NLB **không** hỗ trợ Lambda. NLB route được tới **IP target on-premises**.

🎯 **Domain phải luôn trỏ đúng instance kể cả khi instance được thay mới** → trỏ vào **ALB + listener rule → target group**, KHÔNG trỏ thẳng IP.

🎯 **EC2 phải nằm private subnet nhưng vẫn phục vụ internet** → **ALB internet-facing ở public subnet**.

🎯 **Web app nội bộ chỉ cho văn phòng truy cập nhưng vẫn cần tải patch** → **private subnet + internal ALB + inbound giới hạn CIDR văn phòng + NAT Gateway (ở public subnet)**.

🎯 **GWLB + GWLB endpoint** = chèn appliance (firewall/IDS/IPS) để inspect traffic giữa account/VPC — không dùng cho traffic app thông thường.

⚠️ **Security Group của ALB:** inbound từ `0.0.0.0/0` ở port công khai; **outbound** tới instance cho **cả app port lẫn health-check port** — vì health check do **ALB gửi đi** (outbound của ALB), không phải inbound.

⚠️ **ALB route được theo host, path, HTTP header, HTTP method VÀ query string**; **NLB (L4) KHÔNG đọc được query string**.

⚠️ **ALB/NLB KHÔNG có target ở Region khác** → phân phối cross-Region phải dùng **Route 53** (multivalue/latency/failover) hoặc **Global Accelerator**.

⚠️ **NLB gắn được Elastic IP tĩnh** (cho khách whitelist IP trên firewall); **ALB không**. **CloudFront KHÔNG cung cấp IP tĩnh** → cần IP tĩnh + TCP/UDP + multi-Region → **Global Accelerator**.

### Auto Scaling bổ sung

🎯 **Sự kiện lặp lại có lịch biết trước (peak dịp lễ hàng năm) + "proactively"** → **Scheduled scaling (recurring action)**. CloudWatch alarm theo CPU là **reactive** — khi alarm kêu thì user đã chậm rồi.

🎯 **Có pattern ngày/tuần + vẫn muốn phản ứng thực tế** → **Predictive scaling + target tracking** (kết hợp).

### RDS / Aurora bổ sung

🎯 **Giảm thời gian failover / timeout kết nối / "too many connections"** → **RDS Proxy** (connection pooling, giữ connection qua failover). Read replica giải quyết **read performance**, KHÔNG giảm failover time. Multi-AZ = **availability**, không phải performance. Nghẽn **kết nối** khác nghẽn **tính toán**.

🎯 **Aurora quá tải lúc khuyến mãi, ít downtime** → **thêm reader instance + RDS Proxy target group** (thêm reader không downtime; đổi engine/dựng cụm mới thì có).

🎯 **EC2 xác thực Aurora/RDS mà KHÔNG dùng static credential** → **IAM database authentication + IAM role gắn vào EC2 (instance profile)**. Máy dùng **role**, người dùng **user**. Parameter Store/Secrets Manager vẫn là "có password" — IAM DB auth mới là "bỏ hẳn credential".

🎯 **Mã hoá IN TRANSIT tới RDS** → tải **root certificate do AWS cung cấp** và dùng trong chuỗi kết nối. Snapshot/restore là để bật mã hoá **at rest** — hai cơ chế **tách biệt**, có cái này không suy ra cái kia.

🎯 **DB dev chỉ dùng trong giờ làm việc / chỉ chạy ~2 giờ mỗi tuần** → **Aurora Serverless** (tự dừng/khởi động lại), đừng viết script bật/tắt.

📌 **Aurora Serverless v2 scale về 0 ACU (auto-pause)** khi không có tải.

🎯 **Migrate RDS → Aurora CÙNG engine, downtime gần bằng 0** → tạo **Aurora read replica từ RDS rồi promote**. **Khác engine** (Oracle → PostgreSQL, MySQL → PostgreSQL) mới cần **SCT/DMS Schema Conversion + DMS có CDC**. **DMS full-load KHÔNG bắt thay đổi phát sinh** → cần **full-load + CDC**.

⚠️ **Con số backup:** automated backup retention tối đa **35 ngày**; **PITR tối đa 35 ngày**; manual snapshot giữ **vô hạn** (phải tự xóa). Cần giữ backup **> 35 ngày** hoặc backup **hàng trăm resource** → **AWS Backup**. Nếu đề chỉ yêu cầu **30 ngày** → retention built-in là đủ (đừng thêm service).

⚠️ **Aurora endpoint:** **Reader endpoint** = phân phối tới **tất cả** replica; **Custom endpoint** = load-balance trên **một nhóm replica cụ thể** (vd 3 replica chạy report); **Instance endpoint** = trỏ 1 instance, không load balance.

⚠️ **Multi-AZ DB cluster ≠ Multi-AZ DB instance:** **cluster** có **2 reader ĐỌC ĐƯỢC** → vừa HA vừa tăng năng lực đọc; **instance** chỉ có standby **không đọc được**.

⚠️ **Aurora Standard vs I/O-Optimized:** Standard tính phí **mỗi I/O** → I/O thấp/vừa. **I/O-Optimized** rẻ hơn khi **traffic lớn/I/O nhiều**. Đây là ví dụ "đổi **mô hình định giá**" thay vì đổi kích thước.

⚠️ **SCT + DMS chỉ động tới DATABASE (schema + data), KHÔNG sửa code ứng dụng.** Muốn app T-SQL của SQL Server chạy gần như nguyên vẹn → **Babelfish for Aurora PostgreSQL**.

⚠️ **RDS Custom for Oracle**: khi cần **OS/privileged access** cho tính năng third-party, hỗ trợ **BYOL**.

⚠️ Aurora chỉ hỗ trợ **MySQL/PostgreSQL** — **không có Aurora for Oracle**. RDS gần như **không bao giờ** đặt trong public subnet. **KHÔNG tồn tại read replica xuyên engine.**

### DynamoDB bổ sung

⚠️ **DynamoDB item tối đa 400 KB** — chạm **giới hạn cứng** thì phải **đổi nơi lưu** (lưu object ở **S3**, DynamoDB giữ **metadata + con trỏ/URL**), KHÔNG phải nén hay chia nhỏ (chỉ trì hoãn).

⚠️ **DynamoDB Standard-IA**: rẻ về **storage** nhưng **read/write đắt hơn** → sai cho workload truy cập liên tục. **On-demand không đặt được RCU/WCU** → muốn **chốt ngân sách/ổn định** thì dùng **Provisioned**.

📌 Docs: Standard-IA rẻ hơn **~60% storage** nhưng read/write đắt hơn **~25%**.

⚠️ **DynamoDB deletion protection = NGĂN xoá bảng (phòng ngừa)**; **PITR = khôi phục SAU khi đã mất** ("quay lại bất kỳ thời điểm nào"). Đề nói "prevent" → chọn phòng ngừa.

### ElastiCache bổ sung

🎯 **Cần low-latency + data sống sót qua restart** (session state, leaderboard) → **Redis** (persistence RDB/AOF + replication + Multi-AZ). **Memcached** KHÔNG persistence, KHÔNG snapshot, KHÔNG replication → mất data khi node fail (nhưng multi-thread, đơn giản).

⚠️ **Lazy loading**: cache miss → query DB → nạp cache. Cache hợp khi **read-heavy, write-ít** và bottleneck ở DB (CPU DB cao). **Đọc lặp lại cùng một dữ liệu đè lên Aurora** → **ElastiCache** (thêm replica chỉ trì hoãn vấn đề).

### Container & serverless bổ sung

🎯 **Container chạy on-premises** → **ECS Anywhere**, launch type **EXTERNAL** (KHÔNG phải Fargate — Fargate chỉ chạy trong AWS).

🎯 **Steady state + burst ngắn trên ECS** → **ECS capacity provider trộn Fargate (steady) + Fargate Spot (burst, rẻ ~70%)**. Đừng chọn một loại cho cả hai phần.

🎯 **Container chạy theo lịch** → **ECS scheduled task** (built-in) hoặc **EventBridge Scheduler → ECS Fargate task** (cron/rate, hàng trăm target).

🎯 **Container fully managed + persistent storage dùng chung** → **ECS/EKS Fargate + EFS** (EBS gắn một node, **không dùng được để chia sẻ giữa các task Fargate**).

🎯 **Phân bổ task đều các AZ** → **ECS task placement strategy `spread` theo AZ**.

🎯 **Monolith 20 phút/report, hay downtime khi vá, muốn cải thiện dần** → tách **microservices trên ECS + service auto scaling** (20 phút > 15 phút nên **loại Lambda**).

🎯 **Hàng nghìn Lambda dùng chung logic/thư viện** → **Lambda Layer**.

🎯 **Lambda cần gọi thẳng EC2 trong private subnet** → phải **gắn Lambda vào chính private subnet đó** (Lambda mặc định nằm trong VPC của service Lambda → không có đường tới private resource).

🎯 **Cold start Java/Python/.NET mà KHÔNG có yêu cầu độ trễ nghiêm ngặt + cost-effective** → **Lambda SnapStart** (Java 11+/Python 3.12+/.NET 8+). Cần **đảm bảo** độ trễ và chịu trả phí → **provisioned concurrency**.

⚠️ **Fargate hỗ trợ Windows container; Lambda container image CHỈ chạy nền Linux** (không có runtime Windows dưới bất kỳ hình thức nào) → thấy chữ **"Windows container"** là loại Lambda ngay, bất kể job dài bao lâu. Quy tắc "dưới 15 phút thì Lambda" có ngoại lệ: **kiểm tra nền tảng trước, kiểm tra thời lượng sau**.

⚠️ **Provisioned concurrency vs Reserved concurrency:** **Provisioned** = loại **cold start**, giữ latency ổn định. **Reserved** = **giới hạn/đảm bảo số bản chạy đồng thời**, KHÔNG liên quan cold start.

⚠️ **Tăng memory Lambda = tăng CPU tương ứng** → code CPU-intensive chạy nhanh hơn và có thể **rẻ hơn** (trả ít ms hơn).

⚠️ **Lambda execution role** = để Lambda **gọi ra** service khác. **Lambda resource policy** = để service khác **gọi vào** Lambda. **Environment variables** mã hoá bằng **KMS key + encryption helpers**.

⚠️ **App2Container** = container hoá app .NET/Java sẵn có. **App Runner** = chạy container/web app, **KHÔNG** container hoá được. Managed node group / Karpenter vẫn là EC2 → vẫn còn overhead (Fargate mới là zero node ops).

⚠️ **EKS Cluster Autoscaler scale SỐ NODE** khi pod pending; **HPA chỉ scale pod**.

### Networking bổ sung

🎯 **Lambda/EC2 private subnet timeout vì NAT bão hoà khi đọc S3** → **S3 Gateway Endpoint** (miễn phí, bỏ NAT khỏi đường đi).

🎯 **"Maximum resiliency" cho Direct Connect tới on-prem** → **2 connection từ MỖI data center → 2 DX location khác nhau → 2 device khác nhau** (loại bỏ SPOF ở cả site, location và thiết bị).

🎯 **Vendor expose một service cho nhiều khách hàng, tránh trùng CIDR** → **PrivateLink + NLB** (phía vendor dựng NLB).

🎯 **Một Direct Connect phục vụ hàng chục VPC** → **Transit Gateway + transit VIF + bật route propagation**. **Direct Connect Gateway chỉ nối on-prem ↔︎ VPC, KHÔNG cho VPC ↔︎ VPC.**

🎯 **Kết nối SaaS (Salesforce, ServiceNow…) với AWS, không qua internet** → **Amazon AppFlow + PrivateLink** (AppFlow có connector sẵn).

⚠️ **DX không chỉ đắt mà còn mất hàng tuần–tháng để provisioning** → tự loại khi đề nói **gấp**. **Site-to-Site VPN** dựng trong vài giờ, IPsec sẵn, hợp traffic nhỏ.

⚠️ **VPC Peering tham chiếu Security Group của VPC bên kia CHỈ được khi CÙNG Region.** Peering **khác Region** → bắt buộc dùng **CIDR/IP** trong rule. (Rule phải đặt ở **inbound của SG database**, không phải ở SG application.)

⚠️ VPC Peering: nối được cả **khác Region và khác account**; miễn phí phần thiết lập, chỉ trả **data transfer** → rẻ nhất khi chỉ có 2–3 VPC (vd Lambda account A đọc EFS account B). **EFS mount được qua VPC peering** từ VPC/account khác.

⚠️ **NAT Gateway công cộng đặt ở PUBLIC subnet** (cho private subnet ra internet). **Private NAT Gateway** dùng cho VPC↔︎VPC/on-prem, **không ra internet**.

⚠️ **Outposts vs Local Zones:** **Outposts** = chạy dịch vụ AWS (kể cả EKS, EMR) **ngay trong data center khách hàng** → dữ liệu không rời khỏi chỗ (vd **Hadoop/Spark phải ở on-prem → EMR trên Outposts**). **Local Zones** vẫn là hạ tầng **của AWS** → không thỏa yêu cầu "data phải ở lại data center".

### CloudFront / edge bổ sung

⚠️ **Cache nhưng không phục vụ phiên bản trước sau deploy** → 2 chiến lược **thay thế nhau**: (a) **TTL ngắn** (nội dung đổi liên tục), hoặc (b) **`Cache-Control: max-age` dài + CloudFront invalidation** khi deploy. **`Cache-Control: private`** = cấm shared cache → ngược mục tiêu. **TTL = 0** = tắt cache.

⚠️ **ACM public certificate** = **miễn phí**, dùng cho domain công khai, CloudFront bắt buộc lấy cert ở **us-east-1**. **ACM Private CA** (📌 nay tên **AWS Private CA**) ≈ **$400/tháng** và về bản chất không dùng cho custom domain công khai → sai cả về giá lẫn kỹ thuật.

🎯 **Auth/authorization độ trễ thấp, toàn cầu** → **Lambda@Edge** (xử lý logic tại edge) + Cognito + CloudFront.

### KMS / ACM / Secrets bổ sung

⚠️ **AWS managed key vs Customer managed key:** AWS managed → **tự rotate hàng năm**, không tốn phí key hằng tháng, CloudTrail log được → hợp "rotate + track + rẻ". Customer managed → tốn **$1/tháng/key**, kiểm soát policy/rotation chi tiết → chọn khi cần **key riêng cho từng khách hàng**.

🎯 **Resource mã hóa bằng customer managed key cần HAI lớp quyền:** (1) quyền lên resource (vd `sns:Publish`) trong **execution role/IAM policy** của caller, **VÀ** (2) quyền dùng key trong **KMS key policy** — key policy phải cho phép **cả service principal** (vd `sns.amazonaws.com`) **lẫn principal gọi**.

🎯 **Cách ly dữ liệu giữa nhiều khách hàng, chặn cả nhân viên nội bộ** → **mỗi khách MỘT KMS key + key policy deny mọi principal trừ role của khách**. **Bucket policy KHÔNG kiểm soát được quyền giải mã KMS** (quyền giải mã nằm ở **key policy**).

🎯 **"Encrypt BEFORE sending to AWS" / "third party không được thấy plaintext"** → **Client-side encryption (CSE-KMS)**.

⚠️ **ACM KHÔNG phát hành cert ký bởi third-party CA** → phải tạo cert bên ngoài rồi **import vào ACM**. (Đáp án "create a certificate in ACM signed by the third-party CA" = bất khả thi, loại ngay.)

⚠️ **ACM + DNS validation** = **tự động gia hạn** cert; **email validation phải duyệt tay**.

⚠️ **SSE-S3** không sinh CloudTrail event cho việc dùng key và **không công bố chu kỳ rotate** → trượt yêu cầu "track key usage" hoặc "rotate theo chu kỳ **xác định**/hàng năm". Yêu cầu đó → **KMS customer managed key + automatic rotation**. **CSE-KMS** = mã hóa **trước khi rời client** → đúng cho "encrypt before storing" / "encrypt where processed".

⚠️ **AWS AppConfig** = cấu hình ứng dụng; **Secrets Manager** = credential có rotate. Dùng cặp này khi đề hỏi cả hai.

### Bảo mật & Governance bổ sung

🎯 **DDoS + "proactive engagement" của đội phản ứng DDoS** → **AWS Shield Advanced** (~$3,000/tháng). **WAF rate-based rule** hạn chế DDoS L7 nhưng **không có** proactive engagement. Nguyên tắc: **có Shield trong đáp án → chọn Shield; không có mới dùng WAF**.

🎯 **Vào OS của EC2 private, không SSH key, không bastion, không mở port 22** → **Systems Manager Session Manager** (cần policy **`AmazonSSMManagedInstanceCore`** trên instance profile; **miễn phí**). Thứ tự ưu tiên: Session Manager > bastion host > SSH mở port 22.

🎯 **Tạo account tự động kèm guardrails** → **AWS Control Tower** (Account Factory). **AWS Organizations** chỉ là khung quản lý account/SCP, **không** tự deploy account kèm guardrail.

🎯 **Chặn CloudFormation deploy resource vi phạm TRƯỚC khi tạo** → **AWS Control Tower proactive controls** (triển khai ở **management account**). **Detective controls / AWS Config** chỉ phát hiện **sau**.

🎯 **Chỉ cần "notify khi vi phạm"** (vd SG mở SSH 0.0.0.0/0) → **AWS Config managed rule `restricted-ssh` + SNS** (detective là đủ, không cần chặn).

🎯 **Cấm launch instance size lớn trên MỌI account, ít overhead nhất** → **SCP** (1 lần cấu hình ở Organizations). IAM policy phải sửa trong **từng** account → sai.

🎯 **Three-tier least privilege**: mỗi tầng chỉ nhận **inbound từ Security Group của tầng liền trước**; tầng cuối (DB) chỉ **nhận**, không chủ động gửi ngược.

🎯 **Vá OS hàng loạt + báo cáo lỗ hổng định kỳ** → **SSM Patch Manager** (vá + inventory, Windows & Linux) **+ Amazon Inspector** (quét lỗ hổng).

🎯 **Phát hiện hành vi đáng ngờ rồi TỰ ĐỘNG cập nhật WAF rule** → **GuardDuty → EventBridge → Lambda**.

🎯 **Phát hiện đăng nhập bất thường/thất bại vào Aurora trên nhiều account** → **GuardDuty RDS Protection** (bật theo Organization), đừng tự đẩy log ra rồi phân tích.

🎯 **Macie phát hiện PII rồi báo security team** → **Macie + EventBridge (lọc `SensitiveData`) + SNS**. ⚠️ **Macie là service theo Region** → phải **bật ở từng Region** cần quét PII.

🎯 **Quét CVE cho container image dùng bởi ECS** → **ECR basic scan on push**.

🎯 **Chia sẻ AMI đã mã hoá cho OU khác** → cần **CẢ HAI**: **launch permission cho ARN của OU** **VÀ** **KMS key policy cho phép giải mã**. Least privilege: cấp cho **OU cụ thể**, không cấp cho toàn bộ Organization root.

🎯 **Row-level / cell-level security trên data lake** → **Lake Formation data filters** (built-in).

⚠️ **SCP** = cấm action (preventive). **Tag policy** = enforce tag key/value, **không deny action được**. **Config** = phát hiện vi phạm **sau khi** xảy ra (detective).

⚠️ **SCP gắn vào OU chứa account cần chặn** (hoặc gắn trực tiếp từng account). Gắn ở **root** sẽ chặn cả production (thường sai); gắn ở **management account** thì **không có tác dụng**.

⚠️ **WAF gắn được:** ALB, CloudFront, API Gateway, AppSync, Cognito user pool. **KHÔNG gắn trực tiếp** vào Global Accelerator, NLB, S3. **WAF** = SQLi/XSS/rate-based (L7). **Shield** = DDoS (L3/L4).

📌 Danh sách WAF hiện hành còn có thêm: **App Runner service, Verified Access instance, Amplify, Bedrock AgentCore Gateway** — vẫn **không** có NLB/Global Accelerator/S3.

⚠️ **Security group CHỈ có allow rule, KHÔNG có deny rule** → chặn IP độc hại phải dùng **WAF IP match condition** hoặc NACL. Chặn IP phải làm ở **tầng nhìn thấy IP thật của client** (sau LB thì IP nguồn đã bị che).

⚠️ **Trusted Advisor phải chạy ở MANAGEMENT ACCOUNT** để có góc nhìn hợp nhất toàn org. Các check hữu ích: **RDS Reserved Instance Optimization** (khi utilization **cao**), **Idle DB Instances** (khi utilization **thấp**) — đọc kỹ đề nói "high" hay "idle".

⚠️ **Amazon Managed Grafana đặt trong VPC + RDS dùng private endpoint** = không lộ ra internet.

### Storage file & hybrid bổ sung

🎯 **On-prem đã có SMB/NFS share, cần đẩy file lên S3, KHÔNG sửa code, app on-prem vẫn cần low-latency** → **Storage Gateway (File Gateway)**. Đừng chọn FSx/EFS (đó là khi workload đã ở trên AWS). Cần thêm **giảm chi phí lưu trữ + scale** → **S3 File Gateway + S3 Lifecycle** (EFS đắt hơn S3 đáng kể).

🎯 **SFTP + xác thực bằng AD on-prem, ít vận hành nhất** → **AWS Transfer Family + Directory Service (AD Connector)**. Khi hai đáp án cùng hướng, đáp án **nêu rõ cơ chế** (AD Connector) là đáp án đúng. **Nhận file qua SFTP mà không sửa app đọc file** → **Transfer Family + EFS**.

🎯 **Replicate EFS sang Region khác bằng managed service** → **AWS Backup backup plan có cross-Region copy** (hỗ trợ EC2, EBS, RDS, DynamoDB, EFS trong **một plan**). Ghi chú: AWS còn có **EFS Replication** native (RPO ~15 phút) — nếu xuất hiện trong đáp án thì nó cũng đúng.

🎯 **HPC cần độ trễ dưới mili-giây, dữ liệu từ Snowball** → đưa về **S3 trước**, rồi **FSx for Lustre liên kết S3**.

🎯 **Sao lưu định kỳ lượng nhỏ từ NFS on-prem lên S3, rẻ nhất** → **DataSync agent**.

🎯 **Khảo sát server on-prem để lập kế hoạch migrate** → **Application Discovery Service + Migration Hub** (ADS có 2 chế độ: **Agentless Collector** cho VMware và **Discovery Agent** cài trên máy; Migration Hub cần chọn **home Region**).

📌 **Migration Hub và Application Discovery Service đóng với khách mới từ 07/11/2025** → AWS chuyển sang **AWS Transform** cho discovery/assessment.

⚠️ **FSx for ONTAP**: NFS + SMB **đồng thời** + **auto tiering** giữa tầng nóng/dung lượng. **EFS chỉ NFS · FSx Windows chỉ SMB · FSx OpenZFS chỉ NFS.**

⚠️ **Amazon DLM chỉ quản lý EBS snapshot/AMI, KHÔNG hỗ trợ RDS** → backup nhiều loại resource + cross-Region → **AWS Backup**.

### Messaging & tích hợp bổ sung

🎯 **Giữ nguyên code messaging hiện có dùng AMQP/MQTT/STOMP/JMS** → **Amazon MQ**. SQS/SNS **không nói AMQP** (phải viết lại theo API AWS).

🎯 **Một message tới nhiều Lambda, có LỌC, ít overhead nhất** → **EventBridge rule (event pattern)** — lọc đặt ở **tầng định tuyến**, đừng để mỗi consumer tự lọc rồi bỏ phần lớn.

⚠️ **SNS gửi email trực tiếp được** (email subscription). **SQS KHÔNG gửi email** — chỉ là hàng đợi, cần thêm bộ gửi mail.

### Analytics & AI/ML bổ sung

🎯 **File trong S3 chưa có schema, muốn query SQL** → **Glue crawler** tự sinh schema/table → **Athena**. Dashboard chi phí từ **Cost and Usage Report** → **Athena + QuickSight**.

🎯 **Phát hiện nội dung không phù hợp trong ẢNH, "must not involve training"** → **Rekognition Image Moderation API** (model có sẵn). **Rekognition Video** cho video; **SageMaker** đòi train → loại.

⚠️ **Kinesis Data Firehose là NEAR real-time** (buffer mặc định ~60–300 giây tùy đích) → đề đòi **real-time thật** thì chọn **Data Streams**. **Kinesis Video Streams** chỉ cho video. **Managed Service for Apache Flink** là nơi **xử lý** stream, không phải nơi **ingest**.

📌 Firehose cho đặt **buffering interval = 0 giây** (giao trong vài giây) cho nhiều đích. Cần replay, nhiều consumer, xử lý tùy biến → **Data Streams**.

⚠️ **Glue** có **security configuration** hỗ trợ CSE-KMS. **EMR** chạy trên EC2 → overhead cao hơn Glue.

### Quan sát & vận hành bổ sung

🎯 **Xem log/metric của nhiều account ở một chỗ** → **CloudWatch cross-account observability** (monitoring account + source account, triển khai bằng CloudFormation template). KHÔNG thể gắn policy của account A vào IAM user của account B.

🎯 **Quan sát microservice trên EKS** → **Container Insights (metric hạ tầng) + X-Ray (trace)**.

📌 X-Ray SDK/Daemon vào **maintenance mode từ 25/02/2026** → hướng mới là **OpenTelemetry (ADOT)** gửi trace về X-Ray/CloudWatch.

🎯 **Phân tích traffic bất thường qua ALB** → bật **ALB access log → S3 → Athena**. **CloudTrail ghi API call, KHÔNG ghi traffic ứng dụng hay đăng nhập database.** Tìm `AccessDenied`/`UnauthorizedOperation` → **Athena query trên CloudTrail**.

🎯 **Start/stop resource theo lịch, chọn target bằng tag, không viết code** → **Systems Manager State Manager** (hoặc **Instance Scheduler on AWS**). EventBridge + Lambda cũng làm được nhưng phải viết code → overhead cao hơn.

🎯 **Self-service cho người không rành AWS** → **AWS Service Catalog** (catalog sản phẩm đã duyệt, có governance). CloudFormation chỉ là **nguyên liệu/template thô** — Service Catalog mới là **cửa hàng**.

⚠️ **AWS Budgets có ngưỡng coverage cho Savings Plans**; **Savings Plans alert subscription chỉ báo hết hạn / hàng đợi mua**, không báo coverage.

⚠️ **Bộ công cụ chi phí:** **Cost Explorer** = xem/phân tích + report (dùng với **cost allocation tags** để bóc chi phí theo application, miễn phí). **CUR** = dữ liệu thô chi tiết. **Budgets** = đặt ngưỡng + cảnh báo + Budgets Actions. **Compute Optimizer** = rightsize EC2/ASG/EBS/Lambda bằng ML. **Trusted Advisor** = khuyến nghị tổng quát.

⚠️ **Mức chiết khấu RI/Savings Plans:** 3 năm All Upfront > 3 năm Partial > 1 năm All Upfront > 1 năm No Upfront > On-Demand. Chỉ cam kết khi đề nói **"long-running / steady state / 24-7"**.

---

## 45. Nguyên tắc chung — KHÔNG thuộc riêng service nào

> Đây là phần "ăn điểm" nhanh nhất: rất nhiều câu sai không phải vì thiếu kiến thức service, mà vì bỏ qua các nguyên tắc dưới đây.

### a. Chi phí (cost optimization)

- Traffic **ổn định, dự đoán được** → **provisioned capacity + auto scaling** rẻ hơn on-demand.
- Traffic **spiky, không đoán được** → **on-demand / serverless**, chấp nhận đơn giá cao hơn.
- Reserved / Savings Plans **chỉ** chọn khi đề nói workload chạy **liên tục 1–3 năm**; càng dài hạn + trả trước càng rẻ. Đừng cam kết dài hạn cho **dev/test chạy đứt quãng** (dev dùng On-Demand + tắt khi không dùng).
- Không có Savings Plan nào **phủ chéo họ service**: compute (Compute SP), ML (SageMaker SP) là hai plan riêng.
- Spot rẻ 70–90% nhưng **chỉ hợp lệ** khi đề nói **stateless / fault-tolerant / retry được**. Thấy "cannot be disrupted" / production DB → **loại Spot ngay**.
- Phần chạy **ổn định** dùng capacity thường; phần **burst ngắn** dùng Spot — **chia đôi**, đừng chọn một loại cho cả hai.
- Workload chạy **"thỉnh thoảng / vài phút mỗi lần"** → serverless pay-per-use **luôn thắng** cluster chạy 24/7. Đáp án "chạy instance liên tục cả tuần rồi mới bật job" **luôn thua** đáp án chỉ trả tiền lúc chạy.
- Tài nguyên chỉ dùng **trong giờ làm việc** → chọn thứ **tự động dừng** (Aurora Serverless, Fargate), đừng chọn thứ phải viết script bật/tắt.
- Có phiên bản **miễn phí** thì chọn nó: **Gateway Endpoint > Interface Endpoint > NAT Gateway**. Tính năng miễn phí thắng khi cùng đáp ứng: **Gateway Endpoint, VPC Peering, Session Manager, ACM public cert, SnapStart, SRR**.
- **NAT Gateway tính CẢ phí giờ LẪN phí mỗi GB** → traffic tới AWS service phải đi **VPC Endpoint**.
- Giữ traffic **cùng AZ / cùng Region** để cắt data transfer fee (cross-AZ ≈ $0.01/GB mỗi chiều).
- **Storage class rẻ hơn không tự động đúng**: kiểm tra ràng buộc **retrieval time** và **số AZ** trước, giá chỉ xét **sau khi** ràng buộc được thoả. **IA đắt hơn Standard nếu vẫn truy cập nhiều**.
- Hàng triệu object nhỏ → chi phí **request** có thể lớn hơn **storage** → gộp file.
- **Chi phí tăng mà khối lượng dữ liệu không đổi** → tìm phần bị tính tiền mà không thấy: **incomplete multipart upload, noncurrent version, snapshot mồ côi**.
- Chi phí có thể **chuyển sang bên khác** (Requester Pays) chứ không nhất thiết phải giảm tuyệt đối.
- Có thể đổi **mô hình định giá** để rẻ hơn (Aurora I/O-Optimized, on-demand ↔︎ provisioned) chứ không chỉ đổi kích thước.
- Cùng hiệu năng thì chọn **thế hệ mới hơn** (gp3 thay gp2).
- **Nhân bản dữ liệu ở hai nơi** (hai file system, hai cluster, copy S3 → FSx) là dấu hiệu đáp án **sai về chi phí**.
- **Luôn ước lượng thời gian truyền** = (dung lượng × 8) ÷ băng thông; vượt deadline → chuyển sang giải pháp vật lý.
- **Direct Connect mất hàng tuần–tháng provisioning** → tự loại khi đề nói "gấp / quickly".
- Cross-Region chỉ chọn khi đề **yêu cầu**; nếu không, same-Region luôn rẻ hơn.
- Downsize resource đang dùng **< ~30%** capacity; giữ nguyên resource đang ổn định **~60%**.
- Hai giải pháp **cùng đúng về kỹ thuật** thì **mệnh đề chi phí quyết định**.

### b. High availability & Disaster recovery

- "Multi-AZ", "resilient", "single point of failure", "highly available trong 1 Region" → **Multi-AZ trong 1 Region là đủ** (ASG multi-AZ, EFS, RDS Multi-AZ).
- "Globally", "Region failure", "disaster recovery", có nêu DR Region → **multi-Region** (Global Tables, CRR, Aurora Global DB).
- Bất kỳ đáp án nào có **"single Availability Zone"**, **"một instance"**, **"số instance cố định"** đều **sai** khi đề đòi HA. Mẫu chuẩn: **ASG multi-AZ + load balancer + health check**.
- Một **EC2 đơn lẻ** đóng vai trò server chung (NFS server, DB) luôn là đáp án **sai** khi yêu cầu HA.
- **"Luôn cần ≥ N instance"** phải thoả **SAU KHI mất trọn một AZ** → đặt **N instance ở mỗi AZ**.
- Ràng buộc **"chỉ được chạy một instance"** → không thể HA → chuyển mục tiêu sang **rút ngắn thời gian phục hồi (auto recovery)**.
- **Multi-AZ giải quyết availability, KHÔNG giải quyết performance.**
- RTO/RPO tính bằng **giờ/ngày** → snapshot + copy cross-Region là đủ và rẻ nhất.
- RTO/RPO tính bằng **giây/phút** → replication liên tục **có managed failover** (Aurora Global Database, Multi-AZ, PITR).
- **"LEAST downtime" cho DR** → **pre-deploy** hạ tầng ở Region dự phòng (warm standby), KHÔNG dựng bằng CloudFormation lúc sự cố. Dữ liệu phải được **replicate liên tục TRƯỚC sự cố**.
- Replica phải **promote thủ công** → RTO luôn tệ hơn giải pháp có **managed failover**. **Failover tự động** (DNS failover, managed failover) thắng cơ chế phải chờ alarm rồi gọi Lambda.
- Phân phối traffic cross-Region **bắt buộc có health check**, nếu không vẫn gửi user vào Region đã chết.
- "Maximum resiliency" = loại **mọi** SPOF: site, location, thiết bị — thiếu một tầng là chưa đạt.
- **Backup/snapshot chỉ phục vụ DR**, không giảm failover time. Giảm failover time = giảm thời gian tái lập connection (**proxy/connection pooling**), không phải thêm replica.
- **Read replica và standby KHÔNG phải backup** — "quay lại **bất kỳ thời điểm nào**" → cơ chế **liên tục (PITR)**, không phải snapshot theo lịch.
- **Mất data do thao tác nhầm ≠ mất data do hạ tầng**: cái đầu cần **Recycle Bin / Object Lock / Versioning / deletion protection**, không phải Multi-AZ. **Phòng ngừa** (deletion protection, MFA, Object Lock) khác **phục hồi** (PITR, backup): đề nói **"prevent"** → chọn phòng ngừa.
- Chống **mất quyền truy cập** (mất thiết bị MFA, xoá nhầm) phải chuẩn bị **TRƯỚC**, không xử lý được sau khi mất.
- Thêm **node đọc** là cách mở rộng **không downtime**; đổi engine hay dựng cụm mới thì **có** downtime.
- Đặt **queue** giữa hai tầng để không mất request khi tầng sau chết hoặc đang scale.

### c. Bảo mật & quyền

- Ưu tiên **credential tạm thời** (IAM role, federation, Identity Pool, Roles Anywhere, STS) hơn access key dài hạn.
- **Một identity = một role riêng**; chia sẻ credential hoặc dùng chung role = vi phạm least privilege.
- Cấp quyền cho service (Lambda, EC2) qua **execution role / instance profile**, KHÔNG qua liệt kê ARN trong resource policy.
- Bên thứ ba/vendor → **cross-account role + AssumeRole**, KHÔNG tạo IAM user, KHÔNG phát access key.
- Resource/chia sẻ tài nguyên **đã mã hóa** luôn cần **HAI lớp quyền**: quyền lên resource + quyền dùng KMS key (key policy phải cho cả service principal lẫn caller). **Quyền giải mã nằm ở KEY POLICY**, không nằm ở policy của tài nguyên.
- Cách ly nhiều khách hàng / "encrypted **separately per customer**" → **mỗi khách một key**. Muốn chặn cả **người trong tổ chức** thì phải **deny ở tầng key/policy**.
- **Không bao giờ xác thực bằng IP** của user cuối (IP đổi, không scale). IP chỉ dùng để *lọc* thêm, không phải để *nhận dạng*. Chặn IP phải làm ở **tầng nhìn thấy IP thật của client**.
- Least privilege: chặn bằng **điều kiện động** (`s3:ResourceAccount`, tag) thay vì liệt kê từng resource — policy có giới hạn độ dài và phải sửa liên tục.
- Least privilege nhiều tầng: mỗi tầng chỉ nhận inbound từ tầng **liền trước**; tầng cuối chỉ nhận, không gửi ngược.
- **Security Group là stateful** → xác định chiều theo **bên khởi tạo kết nối**; phản hồi tự động được phép.
- Tận dụng **identity store sẵn có**, đừng tạo identity store thứ hai.
- Phân biệt **authentication** (ai là ai → Cognito/Identity Center) với **authorization/governance** (được làm gì → IAM/SCP).
- **Encryption at rest ≠ in transit** — hai cơ chế **tách biệt**, có cái này không suy ra cái kia. At-rest = bật encryption của **chính dịch vụ lưu trữ** (KMS/SSE/CSE); in-transit = TLS/HTTPS/VPN. VPN có IPsec sẵn, **Direct Connect thì không**.
- **"Encrypt before storing" / "key phải nằm ngoài AWS"** → **client-side encryption**; **"key do mình kiểm soát nhưng vẫn ở AWS"** → KMS customer managed key.
- Yêu cầu **"rotate theo chu kỳ xác định"** → chọn cơ chế có chu kỳ **được công bố** (KMS automatic rotation), không chọn cơ chế nội bộ không tài liệu hoá (SSE-S3).
- **Managed service (S3, API Gateway, DynamoDB, CloudFront) không nằm trong VPC** → không áp SG/NACL/subnet lên chúng.
- **Traffic không được đi qua internet** → **VPC Endpoint / PrivateLink**, KHÔNG phải NAT Gateway.
- SG **không tạo được đường đi**, chỉ **lọc** traffic đã tới được (RDS private subnet mở SG vẫn không reachable từ internet — phải có VPN/DX).
- Phân biệt **"đường đi mạng"** (endpoint, route) và **"quyền truy cập"** (policy, presigned URL) — đề hỏi cái nào thì trả lời cái đó.
- Resource chỉ được truy cập từ **một mạng cụ thể** → **internal endpoint + giới hạn CIDR**, KHÔNG phải mở public rồi lọc. Giữ tài nguyên ở **private subnet** và mở đường vào bằng **một lớp public riêng** (ALB).
- **Preventive** (SCP, Control Tower proactive control, account attribute, Budgets Action, bucket policy) ≠ **detective** (Config, GuardDuty, Trusted Advisor, Inspector). Đề nói **"prevent / must not / block deployment"** → preventive; **"notify khi vi phạm"** → detective là đủ.
- Không mở public rồi đóng lại sau; luôn dùng cơ chế **cấp quyền tạm thời** (presigned URL, STS).
- **Đọc kỹ tên policy có sẵn của AWS**: "System Administrator" **không** đồng nghĩa "toàn quyền".

### d. Hiệu năng & decoupling

- **Xác định đúng tầng bị nghẽn trước** khi chọn giải pháp — scale nhầm tầng làm tệ hơn: **đọc → cache/replica · ghi → IOPS/storage performance · spike → queue · kết nối → proxy · cold start → provisioned concurrency/SnapStart**.
- Thêm **cache** khi read-heavy, write-ít, bottleneck ở DB (CPU DB cao). **Đọc lặp lại cùng một dữ liệu → cache**; thêm replica chỉ **trì hoãn** vấn đề. **Cache KHÔNG cứu được workload write-heavy.**
- Giảm latency mà **giữ nguyên database engine** → thêm **lớp cache chuyên dụng** (DAX/ElastiCache), không đổi database.
- Thêm **queue** khi có spike, sợ mất data, hoặc hai tầng xử lý ở tốc độ khác nhau.
- **Đọc nặng làm chậm DB chính** → tách sang **read replica** trước khi nghĩ tới tăng size.
- **Nghẽn kết nối ("too many connections") ≠ nghẽn tính toán** → dùng **proxy gom kết nối**.
- **Scale theo đúng metric của bottleneck**: queue depth (`ApproximateNumberOfMessages`) cho consumer, KHÔNG phải CPU.
- **Scale ngang (ASG, thêm node/volume) thay vì scale dọc** khi cần liên tục & tự động. **Chạm trần cứng** của một resource (io1 64k IOPS, 400 KB/item) → cộng thêm resource (stripe) hoặc **đổi nơi lưu** (S3 + con trỏ), không phóng to/nén.
- Auto scaling phải **chờ CloudWatch alarm** → **quá chậm với spike đột ngột** → dùng on-demand capacity / serverless.
- **Retry khi hệ thống đang quá tải làm tình hình TỆ HƠN** — phải giảm tải hoặc đệm, không phải thử lại nhiều hơn.
- **Throttling (usage plan)** bảo vệ backend nhưng **từ chối request hợp lệ** → không phải cách "scale".
- Latency toàn cầu: nội dung **HTTP (static + dynamic)** → **CloudFront**; **upload** → **S3 Transfer Acceleration**; **TCP/UDP / non-HTTP** → **Global Accelerator**. Giảm độ trễ đăng nhập/kiểm tra quyền → **đẩy logic ra edge (Lambda@Edge)**.
- **CloudFront vẫn tăng tốc dynamic content** (giữ kết nối qua backbone AWS) — đừng loại CloudFront chỉ vì "dynamic".
- Độ trễ giữa các node phụ thuộc **vị trí vật lý + driver mạng**, KHÔNG phụ thuộc băng thông hay số ENI.
- **Pub/sub (1 message → nhiều subscriber) = SNS**; **queue (1 message → 1 consumer) = SQS**. **"Exactly once"** chỉ có một đáp án đúng: **FIFO queue có deduplication**. **Lọc sự kiện đặt ở tầng định tuyến**, đừng để mỗi consumer tự lọc.
- **Đồng bộ (synchronous) giữa hai tầng = coupling** = không scale độc lập được.
- Nhiều instance cần cùng một tập file **luôn thay đổi** → **shared file system**, không phải sync định kỳ / snapshot. Khác **giao thức** → **một file system đa giao thức (FSx ONTAP)**.
- Truy vấn quan hệ **nhiều tầng** → **graph database (Neptune)**, không phải JOIN.
- Cần granularity **< 5 phút** cho metric → bật **detailed monitoring**, không dựng pipeline log.
- **Cold start là vấn đề khởi tạo runtime**, không sửa được bằng tăng timeout.

### e. Vận hành (operational overhead)

- **"LEAST operational overhead/effort"** → loại mọi đáp án tự quản EC2, tự viết Lambda/script, tự cleanup.
- **Thứ tự ưu tiên:** tính năng **built-in** > **managed service** > **serverless tự ghép** > **code tự viết** > **EC2 tự quản**.
- Có **tính năng chuyên dụng đặt tên đúng bài toán** (cross-account observability, RDS Protection, Instance Scheduler, SnapStart, deletion protection, Budgets coverage) thì **nó thắng mọi cách ghép thủ công**. Bất kỳ đáp án nào bắt **tự viết script/Lambda** cho việc AWS **đã có tính năng sẵn** → sai.
- **Sửa tại chỗ** (modify instance, bật feature) rẻ và ít effort hơn **tạo mới + migrate data**. **Dùng lại hạ tầng sẵn có** (VPC hiện tại, bucket hiện tại) thắng việc dựng mới rồi di chuyển.
- Nếu đề nói công ty **đã dùng service X** → đáp án đúng thường là **mở rộng X**, không thay bằng Y.
- **Giữ nguyên technology stack khi migrate** là cách giảm refactor nhiều nhất: **Kubernetes → EKS · AMQP → Amazon MQ · MySQL → Aurora MySQL**. "Fewest changes" = không đổi **giao thức**, không đổi **engine**, không đổi **orchestration**.
- **"Lift-and-shift" = giữ nguyên kiến trúc** → đừng thêm Auto Scaling, file system chuyên dụng hay tầng mới khi đề nói "không đổi kiến trúc".
- Kiến trúc phải **scale với con số tương lai** đề nêu (5 VPC hôm nay → 100 VPC sau này) → chọn mô hình **hub** (TGW), không chọn kết nối từng cặp (peering O(n²)).
- **Giải pháp nhiều component hơn mà cùng đáp ứng yêu cầu → gần như luôn sai.** Đáp án đúng thường là **cấu hình MỘT thứ**; đáp án sai thường là **ghép ba bốn dịch vụ**.
- Đáp án mô tả **cơ chế cụ thể** (nêu rõ AD Connector, nêu rõ role, nêu rõ endpoint) **thắng** đáp án chung chung cùng hướng.
- Đáp án cần **"self-service cho người không rành AWS"** → phải có **lớp catalog** (Service Catalog), không chỉ template.
- Cảnh giác đáp án **tự mâu thuẫn** (disable rồi lại cấu hình; single-Region rồi lại nói global table).
- Cảnh giác đáp án **ghép service sai chức năng** → **loại ngay, không cần cân nhắc chi phí** (xem §48).
- Khi hai đáp án **chỉ khác nhau một chi tiết**, chi tiết đó **chính là** câu trả lời.
- Câu **"Choose two"**: kiểm tra xem hai đáp án là **hai mảnh của một kiến trúc** hay **hai biến thể của cùng một nguyên tắc** (vd ECS on Fargate **và** EKS on Fargate).

### f. Chiến thuật đọc đề

- Gạch chân **từ khóa ràng buộc**: "LEAST operational overhead", "MOST cost-effective", "cannot be disrupted", "immediately", "cannot be recreated", "globally", "without traversing the internet".
- Một câu thường có **2 ràng buộc**: một về chức năng, một về chi phí/vận hành → đáp án đúng phải thỏa **cả hai**.
- Đề **không nhắc** Organizations/multi-account → đừng tự thêm Organizations vào giải pháp.
- Đề **không nhắc** on-premises → loại Storage Gateway, DX, MGN.
- Trả lời câu hỏi **được hỏi**, không phải câu hỏi bạn *tưởng* được hỏi (routing ≠ blocking; đường mạng ≠ quyền; availability ≠ performance).

---

## 46. Bảng so sánh dịch vụ dễ nhầm (tổng hợp)

> Gộp mọi cặp so sánh từ các đề mock 01–05. Cột cuối là **dấu hiệu trong đề** để chọn đúng.

### Networking & Content Delivery

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| Gateway Endpoint vs Interface Endpoint | Gateway: chỉ S3/DynamoDB, route table, miễn phí, không ENI/SG. Interface: ENI + SG, tính phí, hầu hết service | "S3/DynamoDB + rẻ nhất" → Gateway; service khác / truy cập từ on-prem qua DX → Interface |
| NAT Gateway vs NAT Instance | Gateway: managed, tự scale, không SG. Instance: EC2 tự quản, có SG, làm bastion/port forwarding | "fully managed, HA" → NAT Gateway |
| Security Group vs NACL | SG: stateful, chỉ allow, gắn ENI. NACL: stateless, allow + deny, gắn subnet, cần ephemeral ports | "chặn 1 IP" → NACL/WAF; "chỉ cho phép từ ALB" → SG referencing |
| VPC Peering vs Transit Gateway vs VPC Sharing | Peering: 1-1, không transitive, rẻ. TGW: hub, transitive, có phí giờ. Sharing (RAM): chia subnet cùng Org | Vài VPC → Peering; nhiều VPC + on-prem → TGW; nhiều account dùng chung mạng → VPC Sharing |
| PrivateLink vs VPC Peering | PrivateLink: expose **1 service** qua NLB, không lo trùng CIDR. Peering: nối **toàn mạng**, CIDR không được trùng | "vendor / nhiều khách, trùng CIDR" → PrivateLink |
| Site-to-Site VPN vs Direct Connect | VPN: qua internet, IPsec, dựng trong giờ. DX: cáp riêng, ổn định, không mã hóa mặc định, mất tuần–tháng | "gấp + mã hóa" → VPN; "ổn định, băng thông lớn" → DX; cả hai → DX + VPN |
| Site-to-Site VPN vs Client VPN | S2S: mạng ↔︎ mạng. Client VPN: từng user/thiết bị | "nhân viên làm việc từ xa" → Client VPN |
| Public VIF vs Private VIF vs Transit VIF | Public: tới public service (S3) qua DX. Private: tới VPC qua VGW/DX Gateway. Transit: tới TGW | "S3 qua DX" → Public VIF; "nhiều VPC qua TGW" → Transit VIF |
| DX Gateway vs Transit Gateway | DX Gateway: on-prem ↔︎ VPC nhiều Region, **không** VPC ↔︎ VPC. TGW: transitive giữa VPC | "VPC nói chuyện với nhau" → TGW |
| CloudFront vs Global Accelerator vs S3TA | CloudFront: cache HTTP. GA: static anycast IP, TCP/UDP, không cache. S3TA: upload/download S3 xa | HTTP content → CloudFront; UDP/static IP → GA; upload S3 xa → S3TA |
| ALB vs NLB vs GWLB | ALB: L7, path/host/header/query routing, Lambda target, WAF. NLB: L4 TCP/UDP, static IP/EIP, latency cực thấp. GWLB: chèn appliance | "path routing / Lambda" → ALB; "static IP / UDP / whitelist" → NLB; "firewall bên thứ ba" → GWLB |
| Geolocation vs Geoproximity | Geolocation: theo vị trí **user** (pháp lý, bản quyền). Geoproximity: theo vị trí **resource** + bias | "theo quốc gia" → Geolocation; "mở rộng/thu hẹp vùng" → Geoproximity |
| Simple vs Multivalue | Cả hai trả nhiều IP; chỉ Multivalue có health check (tối đa 8) | "nhiều IP + bỏ IP chết" → Multivalue |
| Failover vs Weighted vs Latency | Failover: active-passive. Weighted: chia tỉ lệ (blue/green). Latency: Region có độ trễ thấp nhất | "trang lỗi dự phòng" → Failover |
| Alias vs CNAME | Alias: zone apex, trỏ AWS resource, miễn phí. CNAME: không ở apex, trỏ domain bất kỳ | "zone apex / ALB" → Alias |
| Resolver Inbound vs Outbound endpoint | Inbound: on-prem hỏi tên trong VPC. Outbound: VPC hỏi tên on-prem | Hướng của câu hỏi DNS quyết định |
| OAC vs OAI vs Signed URL | OAC/OAI: chỉ CloudFront được đọc S3 (OAC mới, hỗ trợ ghi + SSE-KMS). Signed URL: quyền tạm cho **từng khách** | "chặn truy cập thẳng S3" → OAC; "khách thuê phim" → Signed URL |

### Storage

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| EBS vs EFS vs FSx vs S3 | EBS: block, 1 instance (1 AZ). EFS: NFS, nhiều instance, đa AZ. FSx: SMB/Lustre/ONTAP/OpenZFS. S3: object qua API | "file system dùng chung Linux" → EFS; "Windows SMB" → FSx Windows; "object" → S3 |
| Instance Store vs EBS | Instance Store: ephemeral, gắn host, IOPS rất cao, đã gồm trong giá. EBS: bền, snapshot được | "data tạm, replicate được, IOPS cao, rẻ" → Instance Store |
| gp3 vs io2 vs st1 vs sc1 | gp3: SSD đa dụng (3,000 IOPS baseline). io2: IOPS cao, latency thấp. st1: HDD throughput. sc1: HDD lạnh | Boot → không st1/sc1; "IOPS cao, write-heavy DB" → io2; "log/big data tuần tự" → st1 |
| EFS General Purpose vs Max I/O | GP: latency thấp (web/CMS). Max I/O: throughput tổng cao, latency metadata cao | "hàng nghìn client song song" → Max I/O |
| EFS Bursting vs Provisioned (vs Elastic) | Bursting: theo dung lượng. Provisioned: cố định độc lập dung lượng. Elastic: tự scale theo dùng | "data nhỏ nhưng cần throughput cao" → Provisioned |
| FSx Windows vs ONTAP vs Lustre vs OpenZFS | Windows: SMB + AD + DFS. ONTAP: SMB + NFS đồng thời. Lustre: HPC + liên kết S3. OpenZFS: NFS | "cả SMB và NFS" → ONTAP; "HPC + S3" → Lustre |
| Standard-IA vs One Zone-IA vs Glacier IR vs Intelligent-Tiering | IA: ≥3 AZ, min 30 ngày. One Zone-IA: 1 AZ, rẻ hơn 20%. GIR: ms, min 90 ngày, lưu rẻ hơn, đọc đắt hơn. Int-Tiering: tự chuyển tầng | "pattern không biết trước" → Int-Tiering; "tái tạo được" → One Zone-IA |
| Glacier Flexible vs Deep Archive | Flexible: phút–giờ (Expedited ≤250 MB). Deep Archive: 12–48h, rẻ nhất | "lấy trong 12h, rẻ nhất" → Deep Archive |
| Volume Gateway Cached vs Stored | Cached: data chính trên S3, cache nóng local. Stored: data chính local, backup lên S3 | "giảm storage local" → Cached; "low-latency toàn bộ dataset" → Stored |
| File Gateway vs FSx vs DataSync | File Gateway: on-prem cache + S3. FSx: workload chạy trên AWS. DataSync: di chuyển data | "app on-prem vẫn dùng share" → File Gateway; "chuyển data một lần/định kỳ" → DataSync |
| Object Lock Compliance vs Governance | Compliance: không ai override (kể cả root). Governance: user có quyền đặc biệt override được | "regulatory, không ai xóa được" → Compliance |
| Versioning + MFA Delete vs Object Lock | Versioning: giữ version, khôi phục khi xóa nhầm. Object Lock: WORM, cấm xóa/ghi đè trong thời hạn | "chống xóa nhầm" → Versioning; "immutable N năm" → Object Lock |
| Multipart vs S3TA vs Byte-range fetch | Multipart: chia nhỏ upload song song. S3TA: đường edge cho khoảng cách xa. Byte-range: đọc một phần object | "đọc 250 byte đầu" → Byte-range |
| SRR/CRR vs Lifecycle vs Batch Replication | Replication: copy object mới sang bucket khác. Lifecycle: chuyển tầng/xóa. Batch Replication: copy object **đã có** | "copy sang bucket khác" → không bao giờ là Lifecycle |
| Recycle Bin vs Backup Vault Lock | Recycle Bin: khôi phục snapshot/AMI/volume xóa nhầm. Vault Lock: khóa recovery point do AWS Backup tạo | "snapshot tạo bằng script" → Recycle Bin |
| Fast Snapshot Restore vs Multi-Attach | FSR: volume từ snapshot đủ hiệu năng ngay. Multi-Attach: 1 volume io1/io2 gắn nhiều instance cùng AZ | "clone test env" → FSR |

### Compute & Container

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| Lambda vs Fargate vs EC2 | Lambda: ≤15 phút, Linux, pay-per-ms. Fargate: container serverless, chạy lâu, có Windows. EC2: toàn quyền | ">15 phút / Windows container" → Fargate; "event ngắn" → Lambda |
| ECS vs EKS | ECS: orchestrator của AWS, đơn giản. EKS: Kubernetes | "đang dùng Kubernetes" → EKS |
| Provisioned vs Reserved concurrency vs SnapStart | Provisioned: hết cold start, có phí. Reserved: giới hạn/đảm bảo số bản chạy. SnapStart: giảm cold start miễn phí | "latency đảm bảo" → Provisioned; "cost-effective" → SnapStart |
| Cluster vs Spread vs Partition placement | Cluster: sát nhau, latency thấp. Spread: mỗi instance 1 hardware, ≤7/AZ. Partition: nhóm rack cho Hadoop/Kafka | HPC → Cluster; "không chung hardware" → Spread |
| Dedicated Instances vs Dedicated Hosts | Instances: hardware riêng, rẻ hơn. Hosts: thấy socket/core, BYOL | "license theo socket/core" → Dedicated Hosts |
| RI vs Savings Plans vs Spot vs On-Demand vs Capacity Reservation | RI/SP: cam kết 1–3 năm. Spot: rẻ nhất, bị thu hồi. On-Demand: linh hoạt. Capacity Reservation: giữ chỗ, không giảm giá | Baseline → RI/SP; batch chịu gián đoạn → Spot; "đảm bảo có capacity" → Capacity Reservation |
| Compute SP vs EC2 Instance SP vs SageMaker SP | Compute: EC2 + Fargate + Lambda. EC2 Instance: 1 family/Region. SageMaker: riêng | "EC2 + Lambda + Fargate" → Compute SP |
| HPA vs Cluster Autoscaler | HPA: scale pod. Cluster Autoscaler: scale node | "pod pending" → Cluster Autoscaler |
| ECS Anywhere vs Fargate vs Outposts | ECS Anywhere: container trên server khách (EXTERNAL). Fargate: chỉ trong AWS. Outposts: rack AWS tại chỗ | "container on-prem, ít đầu tư" → ECS Anywhere |
| Reboot action vs Recover action | Reboot: instance status check fail. Recover: system status check fail | Lỗi hạ tầng AWS → Recover |
| Target tracking vs Step vs Scheduled vs Predictive | Target: giữ metric. Step: theo bậc. Scheduled: giờ cố định. Predictive: ML dự báo | Spike → Target; "lễ hằng năm" → Scheduled; pattern + spike → Predictive + Target |
| Standby vs suspend ReplaceUnhealthy | Standby: tách instance khỏi traffic để bảo trì. Suspend: ngừng ASG thay instance unhealthy | Cả hai đều đúng cho "patch không bị thay" |
| Hibernate vs Golden AMI vs User data | Hibernate: giữ RAM, khởi động lại nhanh. Golden AMI: cài sẵn. User data: chạy lúc boot | "app cần vài phút warm-up" → Hibernate; "cài 45 phút" → Golden AMI |
| IRSA vs node instance profile | IRSA: role riêng cho từng Pod. Instance profile: mọi Pod trên node dùng chung | "least privilege từng Pod" → IRSA |
| AWS Batch vs Step Functions | Batch: chạy job compute + queue. Step Functions: điều phối nhiều bước, retry | Cần cả hai → Batch + Step Functions |

### Database & Cache

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| Multi-AZ instance vs Read Replica vs Multi-AZ DB cluster | Multi-AZ instance: sync, standby không đọc. Read Replica: async, đọc được. Multi-AZ cluster: 2 reader đọc được | HA → Multi-AZ; scale đọc → Replica; cả hai → Multi-AZ DB cluster |
| RDS vs Aurora vs Aurora Serverless v2 | RDS: nhiều engine. Aurora: MySQL/PostgreSQL, storage chia sẻ, 15 replica. Serverless v2: tự scale, về 0 ACU | "tải biến động / chạy thưa" → Serverless v2 |
| Aurora Replica vs RDS Read Replica | Aurora: chung storage, lag ~ms, tự failover. RDS: async, lag giây | "replica lag cao" → Aurora |
| Aurora Global Database vs cross-Region read replica | Global DB: storage replicate <1s, promote <1 phút. Cross-Region replica: promote thủ công | "RPO/RTO giây–phút đa Region" → Global DB |
| RDS Proxy vs Read Replica | Proxy: gom connection, giảm failover time. Replica: tăng đọc | "too many connections" → Proxy |
| DynamoDB on-demand vs provisioned | On-demand: spiky, không đoán trước. Provisioned (+ auto scaling): ổn định, rẻ hơn, chốt được ngân sách | "traffic không đoán trước" → on-demand |
| DAX vs ElastiCache | DAX: chỉ DynamoDB, không đổi code. ElastiCache: mọi nguồn, cần sửa code | DynamoDB hot key → DAX |
| PITR vs Deletion Protection vs on-demand backup | PITR: khôi phục thời điểm (1–35 ngày). Deletion Protection: chặn xóa bảng. On-demand backup: thủ công | "prevent" → Deletion Protection |
| Global Tables vs Streams | Global Tables: multi-Region multi-active. Streams: log thay đổi 24h | "đa Region ghi đồng thời" → Global Tables |
| Redis vs Memcached | Redis/Valkey: persistence, replication, Multi-AZ, sort/geo. Memcached: multi-thread, không persistence | "sống qua restart / HA" → Redis |
| Neptune vs Redshift vs Timestream vs DynamoDB | Neptune: graph. Redshift: OLAP warehouse. Timestream: time-series. DynamoDB: key-value | "bạn của bạn" → Neptune; "BI warehouse" → Redshift |
| RDS Custom vs RDS vs EC2 | RDS Custom: managed + truy cập OS. RDS: không vào OS. EC2: tự quản toàn bộ | "cần quyền OS + ít quản lý" → RDS Custom |
| Aurora Standard vs I/O-Optimized | Standard: trả theo I/O. I/O-Optimized: giá phẳng | "I/O nhiều" → I/O-Optimized |
| IAM DB auth vs Secrets Manager | IAM DB auth: token 15 phút, không password. Secrets Manager: lưu + rotate password | "không dùng credential tĩnh" → IAM DB auth |

### Messaging & Integration

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| SQS Standard vs FIFO | Standard: throughput gần vô hạn, at-least-once. FIFO: thứ tự + exactly-once, có giới hạn TPS | "thứ tự / không trùng" → FIFO |
| SQS vs SNS vs EventBridge | SQS: queue, poll. SNS: push fan-out. EventBridge: rule lọc nội dung, SaaS, cron | "đệm" → SQS; "1 → nhiều" → SNS; "lọc + SaaS" → EventBridge |
| Kinesis Data Streams vs Firehose vs Managed Flink | KDS: ingest real-time, replay, nhiều consumer. Firehose: load vào đích, near real-time. Flink: xử lý/phân tích | "replay / xử lý tùy biến" → KDS; "đổ vào S3/Redshift không code" → Firehose |
| Amazon MQ vs SQS/SNS | MQ: giao thức chuẩn (AMQP/MQTT/JMS). SQS/SNS: API AWS | "migrate không sửa code messaging" → MQ |
| Long polling vs Visibility timeout vs Delay vs Retention | Long polling: cách nhận. Visibility: ẩn sau khi nhận. Delay: trì hoãn lần đầu. Retention: giữ bao lâu | "xử lý trùng" → Visibility; "giảm empty receive" → Long polling |
| DLQ vs Temporary queue | DLQ: cô lập message lỗi. Temporary queue: request-response | "message fail nhiều lần" → DLQ |
| KDS shared throughput vs Enhanced Fan-Out | Shared: 2 MB/s/shard chia cho mọi consumer. EFO: 2 MB/s/shard **mỗi** consumer | "nhiều consumer bị lag" → EFO |
| AppFlow vs Transfer Family vs DataSync | AppFlow: SaaS ↔︎ AWS qua API. Transfer Family: SFTP/FTPS/FTP. DataSync: NFS/SMB/object → AWS | "Salesforce" → AppFlow; "SFTP" → Transfer Family |
| Step Functions vs EventBridge Scheduler | Step Functions: workflow nhiều bước. Scheduler: chạy theo lịch | "cron đơn giản" → Scheduler |

### Security, Identity & Governance

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| GuardDuty vs Inspector vs Macie vs Security Hub | GuardDuty: threat từ log. Inspector: lỗ hổng phần mềm. Macie: PII trên S3. Security Hub: tổng hợp findings | "crypto mining / IP độc hại" → GuardDuty; "CVE" → Inspector; "PII" → Macie |
| Shield Standard vs Shield Advanced vs WAF | Standard: free, L3/L4. Advanced: SRT, bảo vệ chi phí, báo cáo. WAF: L7 rule | "DDoS + đội hỗ trợ" → Shield Advanced; "SQLi/XSS/rate" → WAF |
| WAF vs Firewall Manager vs Network Firewall | WAF: L7 tại edge/ALB. Firewall Manager: quản rule tập trung toàn Org. Network Firewall: firewall VPC stateful | "nhiều account" → Firewall Manager |
| IAM Access Analyzer vs Access Advisor | Analyzer: external access, unused access. Advisor: service last accessed | "chia sẻ ra ngoài Org" → Analyzer |
| SCP vs Permissions Boundary vs IAM policy vs Tag policy | SCP: trần account/OU (cả root). Boundary: trần user/role. IAM policy: cấp quyền. Tag policy: chuẩn hóa tag, không deny | "chặn cả root" → SCP; "chống tự leo quyền, 1 account" → Boundary |
| Config vs CloudTrail vs CloudWatch | Config: cấu hình + tuân thủ. CloudTrail: ai gọi API. CloudWatch: metric/log/alarm | Đọc câu hỏi đề đặt ra (§1) |
| Control Tower vs Organizations | Control Tower: landing zone + controls + Account Factory. Organizations: khung account/SCP | "tạo account kèm guardrail" → Control Tower |
| AD Connector vs Managed Microsoft AD vs Simple AD | AD Connector: proxy về AD on-prem. Managed AD: AD thật, trust, workload. Simple AD: Samba, nhỏ, không trust | "ít overhead, dùng AD sẵn" → AD Connector; "SQL Server + trust" → Managed AD |
| Identity Center vs SAML federation vs Cognito | Identity Center: SSO nhiều account. SAML: 1 account. Cognito: end-user app web/mobile | Workforce nhiều account → Identity Center; khách hàng app → Cognito |
| Cognito User Pools vs Identity Pools | User Pools: đăng nhập (authentication). Identity Pools: đổi sang AWS credential tạm | "app gọi thẳng S3" → Identity Pools |
| SSE-S3 vs SSE-KMS vs SSE-C vs CSE | SSE-S3: đơn giản. SSE-KMS: audit + rotation + key policy. SSE-C: khách đưa key. CSE: mã hóa trước khi gửi | "audit key usage" → SSE-KMS; "key ngoài AWS, S3 mã hóa" → SSE-C |
| AWS managed key vs customer managed key vs CloudHSM | AWS managed: tự rotate năm, free. Customer managed: $1/tháng, chỉnh policy/rotation. CloudHSM: HSM riêng, toàn quyền | "key riêng mỗi khách" → customer managed; "FIPS, single-tenant HSM" → CloudHSM |
| Secrets Manager vs Parameter Store | Secrets Manager: rotation tự động (RDS…), có phí. Parameter Store: cấu hình + SecureString, rẻ | "rotate password DB" → Secrets Manager |
| Session Manager vs EC2 Instance Connect vs Bastion | Session Manager: không mở port, không key. Instance Connect: SSH key tạm. Bastion: EC2 trung gian | "không port 22, không bastion" → Session Manager |
| IAM Roles Anywhere vs access key | Roles Anywhere: credential tạm cho server on-prem bằng X.509. Access key: dài hạn | "server on-prem gọi AWS" → Roles Anywhere |
| Preventive vs Detective | Preventive: SCP, proactive control, EBS encryption by default. Detective: Config, GuardDuty | "prevent / must not" → Preventive |

### Monitoring & Cost

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| Cost Explorer vs Budgets vs CUR | Cost Explorer: xem/phân tích. Budgets: ngưỡng + cảnh báo + Actions. CUR: dữ liệu thô | "chặn khi vượt" → Budgets Actions |
| Cost Optimization Hub vs Compute Optimizer vs Trusted Advisor | Hub: gom mọi gợi ý tiết kiệm. Optimizer: rightsize bằng ML. Trusted Advisor: khuyến nghị tổng quát + service limits | "rightsize EC2/Lambda" → Compute Optimizer |
| CloudWatch Logs vs CloudTrail | Logs: nội dung log app. CloudTrail: API call | "ai xóa bucket" → CloudTrail |
| Detailed Monitoring vs CloudWatch agent | Detailed: metric có sẵn 1 phút. Agent: memory/disk/custom/log | "memory utilization" → agent |
| CloudTrail Insights vs Metric filter | Insights: tự phát hiện bất thường. Metric filter: bạn tự định nghĩa pattern | "đếm error code" → Metric filter |
| Budgets SP coverage vs SP alert subscription | Coverage budget: báo khi % phủ tụt. Alert subscription: hết hạn / hàng đợi mua | "coverage dưới 80%" → Budgets |
| Instance Scheduler vs State Manager vs EventBridge + Lambda | Instance Scheduler: dựng sẵn bật/tắt EC2+RDS. State Manager: áp cấu hình theo tag. EventBridge + Lambda: phải viết code | "bật/tắt theo lịch, không code" → Instance Scheduler |
| Container Insights vs X-Ray | Container Insights: metric hạ tầng container. X-Ray: trace request | "tìm service chậm trong chuỗi gọi" → X-Ray |

### Migration

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| DMS vs SCT vs Babelfish | DMS: chuyển data (full + CDC). SCT: chuyển schema khác engine. Babelfish: Aurora PostgreSQL hiểu T-SQL | "app T-SQL không sửa code" → Babelfish |
| DMS vs DMS Serverless vs Glue | DMS: replication instance tự chọn size. DMS Serverless: tự scale. Glue: ETL batch | "tải biến động, CDC" → DMS Serverless |
| Read replica + promote vs DMS | Cùng engine → replica rồi promote. Khác engine → DMS + schema conversion | "RDS MySQL → Aurora MySQL" → replica + promote |
| MGN vs Elastic Disaster Recovery | MGN: migrate (rehost). DRS: DR liên tục | "lift-and-shift" → MGN |
| DataSync vs Snowball vs Transfer Family vs File Gateway | DataSync: online, lịch định kỳ. Snowball: offline (📌 đóng với khách mới). Transfer Family: SFTP. File Gateway: on-prem dùng share lâu dài | "mạng kém, PB" → offline; "đồng bộ định kỳ" → DataSync |
| ADS + Migration Hub vs MGN | ADS/Migration Hub: khảo sát, lập kế hoạch (📌 nay AWS Transform). MGN: thực hiện chuyển | "inventory server on-prem" → ADS |

---

## 47. Từ khóa → đáp án (keyword mapping nhanh)

| Từ khóa trong đề | Đáp án |
| --- | --- |
| compliance / COMPLIANT / lịch sử cấu hình | AWS Config (managed rule) |
| who made the API call / audit API | CloudTrail |
| cert sắp hết hạn, ít công sức | Config rule `acm-certificate-expiration-check` |
| third-party / imported certificate | Không auto-renew → import lại vào ACM |
| memory / disk utilization EC2 | CloudWatch agent |
| metric 1 phút | Detailed Monitoring |
| system status check fail | CloudWatch alarm + recover |
| instance status check fail / treo định kỳ | CloudWatch alarm + reboot |
| near real-time alert lỗi API | CloudTrail → CloudWatch Logs → metric filter → alarm → SNS |
| SaaS partner event | EventBridge |
| cron serverless < 15 phút | EventBridge + Lambda |
| cron container > 15 phút | EventBridge Scheduler → ECS Fargate task |
| decouple / buffer / spike / retry | SQS |
| giảm empty receive / giảm chi phí nhận | SQS long polling |
| thứ tự + không trùng | SQS FIFO (+ deduplication) |
| thứ tự riêng mỗi thiết bị + song song | FIFO + Message Group ID |
| message lỗi nhiều lần | DLQ |
| request-response | SQS temporary queues |
| fan-out 1 → nhiều | SNS (+ SQS) |
| AMQP / MQTT / JMS / STOMP / RabbitMQ / ActiveMQ | Amazon MQ |
| replay / nhiều consumer / real-time stream | Kinesis Data Streams |
| nhiều consumer lag | Enhanced Fan-Out |
| ProvisionedThroughputExceededException | Batch record (PutRecords) |
| load stream vào S3/Redshift không code | Firehose |
| HA, synchronous, no data loss | RDS Multi-AZ |
| report nặng làm chậm DB | Read Replica |
| replica lag cao | Aurora Replicas |
| too many connections / failover nhanh | RDS Proxy |
| T-SQL, ít sửa code | Babelfish for Aurora PostgreSQL |
| cần quyền OS cho DB | RDS Custom |
| không dùng password DB | IAM database authentication |
| mã hóa DB đang chạy chưa mã hóa | Snapshot → copy encrypt → restore |
| DB tải biến động / dùng thưa | Aurora Serverless v2 |
| I/O nhiều, spiky | Aurora I/O-Optimized |
| RPO/RTO giây–phút đa Region | Aurora Global Database / DynamoDB Global Tables |
| single-digit ms, NoSQL, serverless | DynamoDB |
| hot partition, không refactor | DAX |
| chống xóa bảng | DynamoDB Deletion Protection |
| quay lại bất kỳ thời điểm | PITR |
| tự xóa item sau N ngày | DynamoDB TTL |
| item > 400 KB | S3 + con trỏ trong DynamoDB |
| bạn của bạn / social graph / fraud | Neptune |
| read lặp lại / session / leaderboard / geospatial | ElastiCache (Redis) |
| sống qua restart, replication | Redis (không Memcached) |
| S3/DynamoDB riêng tư, rẻ | Gateway Endpoint |
| service khác riêng tư | Interface Endpoint (PrivateLink) |
| expose service cho khách, trùng CIDR | PrivateLink + NLB |
| file lớn > 100 MB / > 5 GB | Multipart Upload |
| upload xa | S3 Transfer Acceleration (+ multipart) |
| đọc phần đầu object | Byte-range fetch |
| > 3,500 PUT / 5,500 GET mỗi giây | Thêm prefix |
| xóa/ghi đè nhầm object | Versioning + MFA Delete |
| immutable, WORM, kể cả root | Object Lock Compliance |
| pattern truy cập không biết trước | S3 Intelligent-Tiering |
| tái tạo được, rẻ | S3 One Zone-IA |
| giữ < 30 ngày, truy cập nhiều | S3 Standard |
| archive rẻ nhất, chờ 12–48h | Glacier Deep Archive |
| chi phí KMS cao, giữ SSE-KMS | S3 Bucket Keys |
| key ngoài AWS, S3 mã hóa | SSE-C |
| encrypt before sending | Client-side encryption |
| cùng key nhiều Region | KMS multi-Region key |
| người tải trả phí | Requester Pays |
| truy cập tạm không public | Presigned URL |
| hàng triệu object | S3 Inventory + Batch Operations |
| gom log nhiều account cùng Region | S3 SRR |
| chi phí tăng mà số object không đổi | Lifecycle xóa incomplete multipart upload |
| bucket chưa bật versioning toàn org | S3 Storage Lens |
| shared file Linux, nhiều EC2 | EFS |
| SMB / NTFS / Windows / DFS | FSx for Windows |
| SMB + NFS đồng thời | FSx for ONTAP |
| HPC + S3 | FSx for Lustre |
| IOPS > 16,000 / write-heavy | io1/io2 |
| data tạm, IOPS cao, rẻ | Instance Store |
| clone test env, full IOPS ngay | Fast Snapshot Restore |
| khôi phục snapshot xóa nhầm | Recycle Bin |
| prevent EBS chưa mã hóa | EBS encryption by default (từng Region) |
| on-prem share → S3, không sửa app | S3 File Gateway |
| băng từ / tape | Tape Gateway |
| chuyển online NFS/SMB → S3/EFS/FSx | DataSync |
| SFTP / FTPS | Transfer Family |
| SaaS (Salesforce) → AWS | AppFlow |
| lift-and-shift VM | Application Migration Service (MGN) |
| khác engine DB | SCT + DMS (+ CDC) |
| replication tải biến động | DMS Serverless |
| static content, giảm egress | CloudFront + S3 (OAC) |
| UDP / TCP / static IP / né DNS cache | Global Accelerator |
| HTTPS custom domain CloudFront | Cert ACM ở us-east-1 |
| nội dung khác nhau theo quốc gia | Route 53 Geolocation |
| mở rộng/thu hẹp vùng | Route 53 Geoproximity |
| trang lỗi dự phòng | Failover routing + S3 static website |
| zone apex | Alias record |
| on-prem resolve tên VPC | Resolver Inbound endpoint |
| VPC resolve tên on-prem | Resolver Outbound endpoint |
| path / host / query routing, Lambda target | ALB |
| static IP / EIP / UDP / whitelist IP | NLB |
| firewall/IDS bên thứ ba | GWLB |
| nhiều HTTPS domain 1 ALB | SNI |
| rate limit per-client | API Gateway usage plans + API keys |
| JWT/OIDC native | API Gateway HTTP API + JWT authorizer |
| chặn IP API Gateway | Resource policy |
| giảm đọc backend, chấp nhận stale | API Gateway caching |
| login ít effort trước ALB | Cognito User Pools + ALB |
| MFA theo rủi ro end-user | Cognito adaptive authentication |
| app gọi AWS bằng credential tạm | Cognito Identity Pools |
| nhiều VPC + on-prem, transitive | Transit Gateway |
| vài VPC | VPC Peering |
| nhiều account chung mạng | VPC Sharing (RAM) |
| tăng throughput VPN | TGW + ECMP |
| nhiều chi nhánh VPN nói chuyện nhau | VPN CloudHub |
| S3 qua Direct Connect | Public VIF |
| gấp + mã hóa | Site-to-Site VPN |
| ổn định + mã hóa | Direct Connect + VPN |
| maximum resiliency DX | 2 connection × 2 location × 2 device |
| IPv6 outbound only | Egress-only IGW |
| chặn root member account | SCP |
| chống tự leo quyền (1 account) | Permissions Boundary |
| chuẩn hóa tag + chặn sai tag | Tag Policy + SCP |
| template nhiều account + Region | CloudFormation StackSets |
| tạo account kèm guardrail | Control Tower Account Factory |
| chặn deploy vi phạm trước khi tạo | Control Tower proactive controls |
| SSO nhiều account + AD on-prem, ít overhead | IAM Identity Center + AD Connector |
| directory-aware workload + trust | AWS Managed Microsoft AD |
| server on-prem cần credential tạm | IAM Roles Anywhere |
| least privilege từng Pod EKS | IRSA (hoặc EKS Pod Identity) |
| chia sẻ ra ngoài account/Org | IAM Access Analyzer |
| quyền không dùng | Access Analyzer unused access / Access Advisor |
| crypto mining / IP độc hại | GuardDuty |
| PII trên S3 | Macie |
| CVE / lỗ hổng | Inspector (ECR scan on push) |
| DDoS + đội hỗ trợ | Shield Advanced |
| SQLi / XSS / rate abuse | WAF (rate-based rule) |
| firewall rule nhiều account | Firewall Manager |
| gom log bảo mật, OCSF | Security Lake |
| không port 22, không bastion | Session Manager |
| SSH tạm, audit | EC2 Instance Connect (Endpoint nếu private) |
| patch tự động | SSM Patch Manager (Default Host Management Configuration) |
| patch sau ALB không rớt traffic | SSM Automation `AWSEC2-PatchLoadBalancerInstance` |
| cùng AZ vật lý qua nhiều account | AZ ID |
| HPC, latency thấp giữa node | Cluster placement group + ENA/EFA |
| không chung hardware | Spread placement group |
| license theo core/socket | Dedicated Hosts |
| baseline 24/7 | RI / Savings Plans |
| EC2 + Fargate + Lambda | Compute Savings Plan |
| batch chịu gián đoạn | Spot (Spot Fleet) |
| spike đột ngột | Target tracking |
| giờ cố định | Scheduled action (desired capacity) |
| pattern + spike | Predictive + Target tracking |
| patch không bị ASG thay | Standby / suspend ReplaceUnhealthy |
| khởi động lại nhanh, giữ RAM | EC2 Hibernate |
| cài lâu, cần boot nhanh | Golden AMI + user data |
| pod pending | Cluster Autoscaler |
| container on-prem | ECS Anywhere (EXTERNAL) |
| Windows container serverless | Fargate |
| cold start, cost-effective | Lambda SnapStart |
| cold start, latency đảm bảo | Provisioned concurrency |
| SNS → Lambda mất message khi spike | Tăng Lambda concurrency quota |
| code dùng chung nhiều Lambda | Lambda Layer |
| vượt ngân sách → chặn | Budgets Actions |
| SP coverage tụt | Budgets SP coverage budget |
| EC2 idle / rightsizing | Cost Optimization Hub + Compute Optimizer |
| bật/tắt EC2/RDS theo lịch | Instance Scheduler on AWS |
| self-service cho người không rành AWS | Service Catalog |
| log/metric nhiều account một chỗ | CloudWatch cross-account observability |
| entity từ text, không ML expert | Comprehend |
| text từ PDF/scan | Textract |
| audio → text → SQL | Transcribe → S3 → Athena |
| ảnh không phù hợp, không train | Rekognition moderation |
| no-code data prep | Glue DataBrew |
| SQL trên S3, chưa có schema | Glue crawler + Athena |
| MPP warehouse + ML bằng SQL | Redshift Serverless + Redshift ML |
| full-text search real-time | OpenSearch Service |
| row/cell-level security data lake | Lake Formation data filters |
| data phải ở lại data center | Outposts (+ EKS Anywhere / EMR on Outposts) |
| least downtime DR | Warm Standby / Multi-Site (pre-provisioned) |
| bản nhỏ đầy đủ luôn chạy | Warm Standby |
| chỉ core tối thiểu chạy | Pilot Light |

---

## 48. Con số cần thuộc & các kết hợp BẤT KHẢ THI

### Con số phải nhớ chính xác

| Hạng mục | Con số |
| --- | --- |
| SQS visibility timeout | Mặc định 30s, tối đa 12h |
| SQS delay / message timer | 0–15 phút |
| SQS retention | Mặc định 4 ngày, 1 phút – 14 ngày |
| SQS long polling | Chờ tối đa 20s |
| SQS message size | Tối đa 1 MiB; batch tối đa 10 message |
| SQS FIFO (không high throughput) | 300 op/s mỗi API action; 3,000 msg/s khi batch 10 |
| Lambda | Timeout 15 phút; concurrency mặc định 1,000/Region; 5 layer, 250 MB unzipped; tính phí theo 1 ms |
| DynamoDB | Item 400 KB; PITR 1–35 ngày (mặc định 35); Streams giữ 24h |
| RDS | Automated backup tối đa 35 ngày; IAM DB auth token 15 phút |
| Aurora | Tối đa 15 replica; Global Database tối đa 10 secondary Region; replica lag ~10 ms; storage tới 256 TiB (tùy version) |
| EBS gp2 | Tối đa 16,000 IOPS (burst) |
| EBS gp3 | Baseline 3,000 IOPS + 125 MiB/s; tối đa 80,000 IOPS, 2,000 MiB/s, 64 TiB |
| EBS io1 / io2 Block Express | 64,000 / 256,000 IOPS |
| S3 request rate | 3,500 PUT/COPY/POST/DELETE + 5,500 GET/HEAD mỗi giây mỗi prefix |
| S3 multipart | Nên dùng > 100 MB, bắt buộc > 5 GB |
| S3 min storage duration | Standard-IA/One Zone-IA 30 ngày (min 128 KB/object); Glacier IR & Flexible 90 ngày; Deep Archive 180 ngày |
| Glacier retrieval | Flexible: Expedited 1–5 phút (≤250 MB), Standard 3–5h, Bulk 5–12h. Deep Archive: 12–48h |
| S3 Bucket Keys | Giảm tới 99% request KMS |
| One Zone-IA | Rẻ hơn Standard-IA ~20% |
| Kinesis Data Streams | Mỗi shard ghi 1 MB/s, đọc 2 MB/s (chia sẻ); EFO 2 MB/s/shard mỗi consumer |
| Firehose buffering interval | 0–900 giây (mặc định thường 300s với S3) |
| Spot | Báo trước 2 phút; giảm tới 90% |
| Spread placement group | 7 instance/AZ/group |
| VPN tunnel | 1.25 Gbps mỗi tunnel |
| Route 53 multivalue | Tối đa 8 record healthy |
| CloudFront field-level encryption | Tối đa 10 field |
| API Gateway cache TTL | Tối đa 3,600 giây |
| CloudWatch EC2 metric | 5 phút mặc định, 1 phút với Detailed Monitoring |
| Bucket policy | Tối đa 20 KB |
| Root user MFA | Tối đa 8 thiết bị |
| KMS customer managed key | $1/tháng/key; rotation mặc định 1 năm (chỉnh được) |
| Shield Advanced | ~$3,000/tháng |
| AWS Private CA | ~$400/tháng |
| Cross-AZ data transfer | ~$0.01/GB mỗi chiều |
| Direct Connect | Mất hàng tuần – tháng để provisioning |

### Kết hợp BẤT KHẢ THI → loại ngay, không cần cân nhắc chi phí

- **NLB → Lambda target** (chỉ ALB route tới Lambda được).
- **NLB nhận S3 làm target**.
- **NLB đọc được query string / routing theo path-host-header** (đó là ALB).
- **ALB nhận traffic UDP** (đó là NLB).
- **ALB/NLB có target ở Region khác** (ELB là regional; cross-Region → Route 53 / Global Accelerator).
- **WAF gắn vào Global Accelerator / NLB / S3** (WAF gắn: CloudFront, ALB, API Gateway, AppSync, Cognito, App Runner, Verified Access, Amplify).
- **Security Group / NACL cho API Gateway, S3, DynamoDB, CloudFront** (managed service, ngoài VPC).
- **Security group có deny rule** (chỉ có allow; deny dùng NACL/WAF).
- **ACM tạo cert được ký bởi third-party CA** (chỉ import được).
- **ACM private certificate dùng cho custom domain công khai của CloudFront**.
- **Fargate chạy on-premises** (đó là ECS Anywhere, launch type EXTERNAL).
- **Lambda chạy được container Windows** (chỉ nền Linux).
- **Lambda mặc định gọi được EC2 trong private subnet** (phải gắn Lambda vào chính subnet đó).
- **Tăng timeout sửa được cold start** (cold start là vấn đề khởi tạo runtime).
- **Multi-AZ standby của RDS instance dùng để đọc** (chỉ để failover).
- **Read replica synchronous** (luôn async).
- **Read replica xuyên engine** (MySQL → PostgreSQL phải dùng DMS + Schema Conversion).
- **"Cross-Region Multi-AZ"** cho RDS (Multi-AZ chỉ trong 1 Region).
- **Aurora for Oracle / Aurora for SQL Server** (chỉ MySQL & PostgreSQL).
- **Database cloning trên RDS thường** (chỉ Aurora).
- **SCT/DMS sửa được code ứng dụng** (chỉ chuyển schema + dữ liệu; muốn app chạy tiếp → Babelfish).
- **DMS full-load bắt được thay đổi phát sinh** (phải bật CDC).
- **Memcached có snapshot/replication** (hoàn toàn không có).
- **Cache giải quyết được workload write-heavy**.
- **Nén/chia nhỏ để nhét item > 400 KB vào DynamoDB** (phải chuyển sang S3 + con trỏ).
- **Instance Store gắn vào nhiều EC2** hoặc giữ data sau stop/terminate.
- **EBS st1/sc1 làm boot volume**; **Multi-Attach trên gp2/gp3/HDD** (chỉ io1/io2, cùng AZ).
- **EC2 có "Multi-AZ deployment"** như RDS (không tồn tại khái niệm này).
- **Mount S3 làm volume** cho ứng dụng cần file system POSIX thật.
- **Snowball / Redshift UNLOAD ghi thẳng vào Glacier Vault** (phải qua S3 rồi lifecycle).
- **Lifecycle rule copy object sang bucket khác** (chỉ chuyển tầng/xoá; copy phải dùng Replication).
- **Lifecycle chuyển ngược lên S3 Standard** (waterfall một chiều).
- **CloudFront upload qua S3 static website endpoint** (endpoint đó chỉ GET/HEAD).
- **S3 default encryption dùng SSE-C** (chỉ hỗ trợ SSE-S3, SSE-KMS, DSSE-KMS).
- **Convert SQS Standard → FIFO** (phải tạo queue mới, tên kết thúc `.fifo`).
- **Kinesis Firehose ghi thẳng vào DynamoDB** (đích: S3/Redshift/OpenSearch/Splunk/Iceberg/HTTP…).
- **CloudTrail stream trực tiếp vào Kinesis** (đích: S3 & CloudWatch Logs).
- **SNS có thể poll** (SNS là push; muốn poll thì dùng SQS).
- **SQS gửi email** (SNS mới gửi email được).
- **Chuyển KMS single-Region key thành multi-Region key** (phải tạo key mới).
- **Bucket policy kiểm soát được quyền giải mã KMS** (nằm ở key policy).
- **Gateway Endpoint cho dịch vụ ngoài S3/DynamoDB**.
- **VPC Endpoint dùng để tới on-premises** (endpoint chỉ tới AWS service).
- **VPC Peering transitive** (A↔︎B, A↔︎C không cho B↔︎C).
- **Tham chiếu Security Group qua VPC Peering KHÁC Region** (chỉ cùng Region).
- **Direct Connect Gateway cho phép VPC ↔︎ VPC** (chỉ on-prem ↔︎ VPC).
- **NAT Gateway đặt trong private subnet** để ra internet (phải ở public subnet).
- **ALB gán Elastic IP** (chỉ NLB).
- **VGW hỗ trợ ECMP** (phải dùng Transit Gateway).
- **Local Zones giữ dữ liệu trong data center của khách hàng** (vẫn là hạ tầng AWS → dùng Outposts).
- **Tag policy deny được action** (chỉ SCP deny được).
- **SCP gắn ở management account có tác dụng** (không có tác dụng với chính management account).
- **Permissions Boundary gắn vào IAM group** (chỉ user/role).
- **Thêm IAM user của account khác vào IAM group của mình**.
- **Gắn IAM policy của account A vào IAM user của account B**.
- **AWS Organizations tự authenticate người dùng** (đó là IAM Identity Center/Cognito).
- **AWS Config / Trusted Advisor / Inspector "ngăn chặn" được vi phạm** (chúng là **detective**, chỉ phát hiện sau).
- **Compute Savings Plan phủ SageMaker** (không phủ — phải mua SageMaker SP riêng).
- **DataSync chạy query / Config làm backup / State Manager orchestrate backup**.
- **Amazon DLM backup được RDS** (chỉ EBS snapshot/AMI).

---

*Tổng hợp từ 11 bộ đề luyện thi (≈715 câu), gồm kiến thức của cả đáp án đúng, đáp án sai và các dịch vụ ngoài lề được nhắc tới, cộng các nguyên tắc chung và bảng so sánh dễ nhầm. Chúc ôn thi SAA hiệu quả.*

---

### Nguồn đã đối chiếu (các ghi chú 📌)

- EBS gp3 (80,000 IOPS, 2,000 MiB/s, 64 TiB, rẻ hơn gp2 20%): https://docs.aws.amazon.com/ebs/latest/userguide/general-purpose.html
- SQS message quotas (1 MiB, FIFO 300 TPS / 3,000 msg/s, high throughput FIFO): https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html
- SQS FIFO quotas (delay 15 phút, long polling 20s, `.fifo`): https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-fifo.html
- Firehose buffering hints (zero buffering, 0–900s): https://docs.aws.amazon.com/firehose/latest/dev/create-configure-backup.html
- Kinesis Data Analytics for SQL discontinuation: https://docs.aws.amazon.com/kinesisanalytics/latest/dev/discontinuation.html
- DynamoDB PITR recovery period 1–35 ngày: https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_PointInTimeRecoverySpecification.html
- DynamoDB Standard-IA (~60% storage rẻ hơn, ~25% read/write đắt hơn): https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/CostOptimization_TableClass.html
- Aurora Global Database (tối đa 10 secondary Region): https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.configuration.requirements.html
- Aurora Serverless v2 auto-pause (0 ACU): https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html
- Mountpoint for Amazon S3 caching: https://docs.aws.amazon.com/AmazonS3/latest/userguide/mountpoint-usage.html
- S3 Select không nhận khách mới: https://docs.aws.amazon.com/AmazonS3/latest/API/API_SelectParameters.html
- S3 Object Lambda availability change: https://docs.aws.amazon.com/AmazonS3/latest/userguide/amazons3-ol-change.html
- Amazon Glacier (vault) không nhận khách mới: https://docs.aws.amazon.com/amazonglacier/latest/dev/creating-vaults-console.html
- AWS Data Pipeline không nhận khách mới: https://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/dp-limits.html
- App Runner availability change → ECS Express Mode: https://docs.aws.amazon.com/apprunner/latest/dg/apprunner-availability-change.html
- WAF resources: https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html
- KMS automatic rotation (rotation period tùy chỉnh, AWS managed key rotate hằng năm): https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-enable.html
- KMS on-demand rotation: https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-on-demand.html
- Recycle Bin (EBS volumes, snapshots, AMIs): https://docs.aws.amazon.com/ebs/latest/userguide/recycle-bin.html
- EFS throughput modes (Elastic): https://docs.aws.amazon.com/efs/latest/ug/performance.html
- Launch configuration (account từ 01/10/2024 không tạo được): https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-configurations.html
- Resource control policies (RCP): https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html
- EKS Pod Identity: https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html
- CloudWatch Logs agent deprecated, dùng unified CloudWatch agent: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/UsePreviousCloudWatchLogsAgent.html
- QLDB ngừng 31/07/2025, Lookout for Vision ngừng 31/10/2025 (nguồn web, docs không còn trang): https://www.infoq.com/news/2024/07/aws-kill-qldb/ · https://aws.amazon.com/blogs/machine-learning/exploring-alternatives-and-seamlessly-migrating-data-from-amazon-lookout-for-vision/
- Các mốc khác (Snow Family, Migration Hub/ADS, Shield SRT, Quick Sight, Elastic Disaster Recovery, Security Hub CSPM, Savings Plans 4 loại, X-Ray SDK maintenance, ElastiCache Valkey, public IPv4) đã đối chiếu khi viết CLF-C02, xem nguồn ở `CLF-C02/02-practice-exams/practice-exam-notes.md`.
