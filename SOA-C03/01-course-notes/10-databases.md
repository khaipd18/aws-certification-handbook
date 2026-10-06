# SOA-C03 — Databases: RDS, Aurora, RDS Proxy & ElastiCache

> Thuộc **Domain 2: Reliability and Business Continuity (22%)** và **Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization (22%)**: Multi-AZ, read replica, backup/snapshot, giám sát hiệu năng DB, Aurora, RDS Proxy, ElastiCache.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **RDS** là SQL managed: tự provision, vá OS, backup liên tục có **point-in-time restore**, read replica, Multi-AZ. **Không SSH** vào host.
- **Read replica** để scale đọc (async, tối đa 15, promote được). **Multi-AZ** để sẵn sàng cao (sync, một DNS name, tự failover, standby không phục vụ đọc).
- **Backup tự động**: retention 0–35 ngày (0 = tắt). **Snapshot thủ công**: không hết hạn, share/copy được. Restore luôn tạo **DB mới**.
- **Aurora**: 6 bản sao trên 3 AZ (ghi cần 4/6, đọc 3/6), tự lớn tới 256 TiB, 15 replica, failover dưới 30 giây.
  - **Backtrack**: tua lại tại chỗ tới 72 giờ, chỉ Aurora MySQL.
  - **Cloning**: copy-on-write.
  - **Global Database**: tối đa 10 Region phụ.
- **RDS Proxy** gộp connection, giảm thời gian failover, bắt buộc truy cập trong VPC, dùng IAM auth + Secrets Manager.
- **ElastiCache**:
  - **Valkey / Redis OSS**: replication, Multi-AZ, backup.
  - **Memcached**: sharding, đa luồng, không replication.
- ⚠️ Lỗi thời: Performance Insights → nay là **CloudWatch Database Insights**.

---

## 2. Core Concepts

### RDS & storage auto scaling
- **Là gì:** DB quan hệ managed (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Db2, Aurora) trên EBS; storage auto scaling tự tăng dung lượng tới **Maximum Storage Threshold**.
- **Dùng khi nào:** tải khó đoán, tránh hết dung lượng gây sự cố.
- **Vì sao quan trọng:** auto scaling chạy khi dung lượng trống ≤ 10%, kéo dài ít nhất 5 phút, lần sửa trước đã xong storage optimization và có dưới 4 lần sửa storage trong 24 giờ. ⚠️ Lỗi thời: điều kiện "chờ 6 giờ từ lần sửa trước" trong slide.

### Read Replicas
- **Là gì:** tối đa 15 replica async, cùng AZ, khác AZ hoặc khác Region; đọc eventually consistent; promote thành DB độc lập.
- **Dùng khi nào:** đẩy báo cáo, analytics (SELECT) khỏi DB chính.
- **Vì sao quan trọng:** ứng dụng phải đổi connection string. Replication cùng Region miễn phí, khác Region tính phí truyền dữ liệu.

### Multi-AZ
- **Là gì:** **Multi-AZ instance**: standby sync ở AZ khác, không đọc được. **Multi-AZ DB cluster**: 3 instance trên 3 AZ, 2 standby **đọc được**.
- **Dùng khi nào:** HA và DR. Chuyển Single-AZ → Multi-AZ là thao tác modify không downtime.
- **Vì sao quan trọng:** failover khi instance lỗi, vá OS, mất AZ, đổi instance type, hoặc **reboot with failover**. Snapshot trên Multi-AZ lấy từ standby, không ảnh hưởng primary.

### Backup, snapshot & chia sẻ
- **Là gì:** backup tự động chạy trong backup window, PITR; snapshot thủ công giữ đến khi xóa, nên tạo **final snapshot** khi xóa DB.
- **Dùng khi nào:** share cho account khác → chỉ snapshot thủ công (snapshot tự động phải copy trước); snapshot mã hóa phải dùng customer managed key và share key.
- **Vì sao quan trọng:** không mã hóa DB có sẵn tại chỗ: snapshot → copy bật mã hóa → restore DB mới. Read replica theo trạng thái mã hóa của nguồn.

### Events, logs & metrics
- **Là gì:** RDS event (instance, snapshot, parameter group, security group) gửi qua event subscription → SNS, và tới EventBridge. Log (error, slow query, general, audit) publish sang CloudWatch Logs.
- **Dùng khi nào:** metric filter trên log → alarm → SNS; phản ứng tự động với failover hoặc thiếu dung lượng qua EventBridge.
- **Vì sao quan trọng:** metric CloudWatch thường (`DatabaseConnections`, `FreeStorageSpace`, `ReadLatency`, `DiskQueueDepth`, `SwapUsage`) lấy từ hypervisor; **Enhanced Monitoring** chạy agent trên DB, cho metric OS theo **từng process/thread**.

### Database Insights (trước là Performance Insights)
- **Là gì:** trực quan hóa **DB load** (`DBLoad` = số session đang hoạt động), cắt theo wait, câu SQL, host, user.
- **Dùng khi nào:** cần biết **câu query nào** hoặc **wait nào** gây chậm, không chỉ biết DB bận.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: Performance Insights → docs nay mô tả **Database Insights** (chế độ Advanced mặc định, có lock diagnostics, execution plan, fleet monitoring). Giá: kiểm tra trang pricing.

### Parameter group, option group & Blue/Green
- **Là gì:** parameter group chứa cấu hình engine (static cần reboot, dynamic áp ngay); option group bật tính năng riêng của engine; **Blue/Green Deployments** tạo môi trường green đồng bộ để nâng cấp rồi switchover.
- **Dùng khi nào:** ép SSL, chỉnh tham số → custom parameter group; nâng major version ít downtime, có đường lùi → Blue/Green.
- **Vì sao quan trọng:** **default parameter group không sửa được**; tạo group riêng, gắn vào DB rồi reboot nếu là tham số static.

### Amazon Aurora
- **Là gì:** engine tương thích MySQL/PostgreSQL; storage 6 bản sao / 3 AZ, tự lớn tới **256 TiB**; 1 writer + tới 15 reader, lag dưới 10 ms.
- **Dùng khi nào:** dùng **writer endpoint** cho ghi, **reader endpoint** cho đọc (cân bằng kết nối), không hard-code endpoint instance.
- **Vì sao quan trọng:** backup tự động 1–35 ngày, **không tắt được**. Theo dõi `AuroraReplicaLag` (lag cao → người dùng thấy dữ liệu khác nhau tùy reader).

### Backtrack, Cloning, Global Database, Serverless
- **Là gì:** Backtrack tua cụm tại chỗ tới 72 giờ (Aurora MySQL); Cloning tạo cụm mới copy-on-write; Global Database có Region chính đọc/ghi và tối đa 10 Region phụ chỉ đọc, replication thường dưới 1 giây; Aurora Serverless v2 co giãn capacity tại chỗ.
- **Dùng khi nào:** lỡ xóa dữ liệu, cần quay lại nhanh → Backtrack; môi trường test từ dữ liệu prod → Cloning; DR đa Region, đọc độ trễ thấp toàn cầu → Global Database.
- **Vì sao quan trọng:** PITR tạo cụm **mới**, Backtrack làm **tại chỗ**. ❓ Aurora Serverless v1 đã bị thay bằng v2; ngày ngừng chính xác của v1 chưa tra được.

### RDS Proxy
- **Là gì:** proxy managed gộp và chia sẻ connection, serverless, Multi-AZ, hỗ trợ RDS MySQL/PostgreSQL/MariaDB/SQL Server và Aurora.
- **Dùng khi nào:** Lambda hoặc ứng dụng mở quá nhiều connection, connection storm, cần failover nhanh hơn mà không sửa code.
- **Vì sao quan trọng:** **không public**, chỉ truy cập trong VPC; ép IAM auth, credential trong Secrets Manager. ❓ Con số "giảm thời gian failover tới 66%" lấy từ slide, chưa tìm thấy trong docs hiện hành.

### ElastiCache: Valkey/Redis OSS vs Memcached
- **Là gì:** Valkey/Redis OSS có Multi-AZ auto-failover, read replica, persistence, backup/restore, kiểu dữ liệu phong phú. Memcached có sharding, đa luồng, không replication, không persistence (node cluster mất dữ liệu khi thay).
- **Dùng khi nào:** cache DB (lazy loading, write-through, TTL) và **session store** để ứng dụng stateless, scale ngang không cần sticky session.
- **Vì sao quan trọng:** yêu cầu có failover, backup, replica → loại Memcached (trừ Memcached serverless có backup). Dùng cache cần sửa code ứng dụng.

### Scale & metric ElastiCache
- **Là gì:** Redis cluster mode disabled: thêm tới 5 replica hoặc đổi node type; cluster mode enabled: resharding, rebalancing. Memcached: thêm/bớt node, client dùng **Auto Discovery** qua configuration endpoint.
- **Dùng khi nào:** `Evictions` cao (bộ nhớ đầy) → eviction policy, node lớn hơn, thêm node/shard; `CPUUtilization` cao → scale.
- **Vì sao quan trọng:** `SwapUsage` không nên vượt 50 MB; theo dõi thêm `CurrConnections`, `DatabaseMemoryUsagePercentage`, `ReplicationLag`.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Read replica | Bản sao async để đọc | "offload reporting queries" |
| Multi-AZ instance | Standby sync, không đọc được | "automatic failover, high availability" |
| Multi-AZ DB cluster | 2 standby đọc được | "readable standbys" |
| PITR | Khôi phục tới thời điểm bất kỳ | "restore to 10:42 yesterday" |
| Storage auto scaling | Tự tăng storage | "running out of storage" |
| Enhanced Monitoring | Metric OS theo process | "per-process CPU / memory" |
| Database Insights | Phân tích DB load theo SQL/wait | "which query causes load" |
| Blue/Green Deployments | Môi trường green để nâng cấp | "upgrade with minimal downtime" |
| Aurora Backtrack | Tua cụm tại chỗ | "undo bad delete quickly" |
| Aurora Global Database | Region phụ chỉ đọc | "cross-Region DR, RTO < 1 min" |
| RDS Proxy | Connection pooling | "too many connections from Lambda" |
| ElastiCache session store | Session dùng chung | "stateless app, users stay logged in" |
| `Evictions` | Item bị đẩy vì đầy bộ nhớ | "cache memory full" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Read replica | Multi-AZ | Scale đọc, async; HA, sync | Scale reads → replica |
| Multi-AZ instance | Multi-AZ DB cluster | Standby không đọc; 2 standby đọc được | "Readable standby" → DB cluster |
| Backup tự động | Snapshot thủ công | 0–35 ngày, PITR; giữ đến khi xóa, share được | Share → manual |
| PITR | Backtrack | Cụm/DB mới; tại chỗ (Aurora MySQL) | "In place" → Backtrack |
| CloudWatch metrics | Enhanced Monitoring | Từ hypervisor; agent, theo process | "Per process" → Enhanced |
| Enhanced Monitoring | Database Insights | Metric OS; tải theo SQL/wait | "Which SQL" → Database Insights |
| Reader endpoint | Instance endpoint | Cân bằng kết nối đọc; cố định một instance | Đọc phân tán → reader |
| RDS Proxy | Read replica | Gộp connection; scale đọc | "Too many connections" → Proxy |
| Valkey/Redis OSS | Memcached | Replication, failover, backup; sharding, đa luồng | Cần HA → Redis/Valkey |
| Lazy loading | Write-through | Nạp khi miss, có thể cũ; ghi luôn vào cache | Luôn mới → write-through |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "reporting slows production DB" | Read replica | Đổi connection string |
| "survive AZ failure automatically" | Multi-AZ | |
| "test failover" | Reboot with failover | |
| "share snapshot with another account" | Snapshot thủ công (+ share KMS key) | |
| "encrypt existing RDS" | Snapshot → copy mã hóa → restore | |
| "notify on failover / low storage" | RDS event subscription → SNS / EventBridge | |
| "per-process memory on DB host" | Enhanced Monitoring | |
| "identify top SQL by load" | Database Insights | |
| "parameter change has no effect" | Static parameter → reboot; default group không sửa được | |
| "undo accidental delete in Aurora MySQL quickly" | Backtrack | |
| "copy prod Aurora for testing fast" | Aurora cloning | |
| "Lambda exhausts DB connections" | RDS Proxy | |
| "cache must failover and persist" | ElastiCache Valkey/Redis OSS Multi-AZ | |
| "high `Evictions`" | Eviction policy / scale node | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Truy vấn báo cáo làm chậm DB RDS MySQL production. Cách giảm tải với thay đổi nhỏ nhất?
- A. Bật Multi-AZ và trỏ báo cáo vào standby
- B. Tạo read replica, cho báo cáo dùng endpoint của replica
- C. Tăng backup retention
- D. Bật storage auto scaling

<details><summary>Đáp án</summary>

**B.**
- A sai: standby của Multi-AZ instance không đọc được.
- C, D sai: không giảm tải đọc.
</details>

**Câu 2.** Cần kiểm thử ứng dụng xử lý failover của RDS Multi-AZ. Cách làm?
- A. Xóa instance chính
- B. Reboot DB với tùy chọn failover
- C. Stop DB
- D. Tạo read replica rồi promote

<details><summary>Đáp án</summary>

**B.**
- A sai: phá hủy dữ liệu.
- C sai: stop không tạo failover.
- D sai: promote replica tạo DB độc lập, không phải failover Multi-AZ.
</details>

**Câu 3.** (Chọn 2) Cần chia sẻ snapshot của RDS mã hóa bằng customer managed key cho account khác. Hai bước nào cần?
- A. Copy snapshot tự động thành snapshot thủ công (nếu nguồn là snapshot tự động)
- B. Cho account đích quyền dùng KMS key trong key policy
- C. Chuyển sang AWS managed key `aws/rds`
- D. Bật Multi-AZ
- E. Public snapshot

<details><summary>Đáp án</summary>

**A, B.**
- C sai: AWS managed key không share cross-account được.
- D sai: không liên quan.
- E sai: snapshot mã hóa không public được, cũng trái yêu cầu bảo mật.
</details>

**Câu 4.** DBA muốn biết **câu SQL nào** gây tải cao nhất trên RDS PostgreSQL trong giờ cao điểm. Chọn gì?
- A. Enhanced Monitoring
- B. CloudWatch Database Insights
- C. RDS event subscription
- D. CloudTrail

<details><summary>Đáp án</summary>

**B.** Cắt DB load theo SQL, wait, user.
- A sai: metric OS theo process, không theo câu SQL.
- C sai: sự kiện vận hành.
- D sai: ghi API call của AWS.
</details>

**Câu 5.** Đổi một tham số trong custom parameter group nhưng DB không đổi hành vi. Nguyên nhân khả dĩ nhất?
- A. Tham số static, cần reboot DB
- B. Parameter group phải gắn với snapshot
- C. Phải bật Multi-AZ
- D. Tham số chỉ áp cho read replica

<details><summary>Đáp án</summary>

**A.**
- B sai: parameter group gắn với DB instance/cluster.
- C, D sai: không liên quan.
</details>

**Câu 6.** Hàng nghìn Lambda đồng thời làm RDS báo "too many connections". Giải pháp ít sửa code nhất?
- A. Tăng instance class
- B. RDS Proxy
- C. Read replica
- D. ElastiCache Memcached

<details><summary>Đáp án</summary>

**B.** Gộp và chia sẻ connection.
- A sai: chỉ đẩy giới hạn lên, vẫn có thể cạn.
- C sai: không gộp connection ghi.
- D sai: phải sửa code đáng kể, không giải quyết connection.
</details>

**Câu 7.** Kỹ sư lỡ xóa dữ liệu trong Aurora MySQL 20 phút trước. Cần quay lại nhanh nhất, giữ nguyên endpoint. Chọn gì?
- A. PITR
- B. Backtrack
- C. Restore snapshot thủ công
- D. Promote replica

<details><summary>Đáp án</summary>

**B.** Backtrack tua cụm tại chỗ, không tạo cụm mới.
- A, C sai: tạo cụm mới, đổi endpoint.
- D sai: replica cũng đã nhận thao tác xóa.
</details>

**Câu 8.** (Chọn 2) Ứng dụng web cần session dùng chung giữa các instance và cache phải tự failover khi node lỗi. Hai phát biểu nào đúng?
- A. ElastiCache Valkey/Redis OSS có Multi-AZ auto-failover
- B. Memcached có replication và failover
- C. Lưu session trong ElastiCache giúp ứng dụng stateless
- D. Sticky session là cách duy nhất giữ đăng nhập
- E. Memcached có persistence

<details><summary>Đáp án</summary>

**A, C.**
- B, E sai: Memcached không replication, không persistence.
- D sai: session store bỏ được sticky session.
</details>

---

### Nguồn đã đối chiếu
- Database Insights trên RDS: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.Overview.html
- RDS storage auto scaling (≤ 10%, 5 phút, dưới 4 lần / 24 giờ): https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIOPS.Autoscaling.html
- Multi-AZ DB cluster: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html
- Share DB snapshot: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html
- Aurora quotas (256 TiB): https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html
- Aurora Global Database: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html
- Aurora Backtrack: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html
- RDS Proxy: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html
- RDS Blue/Green Deployments: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/blue-green-deployments.html
