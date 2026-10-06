# SAA-C03 — Databases in AWS: Choosing the Right Database

> Thuộc **Domain 3: Design High-Performing Architectures** và **Domain 4: Design Cost-Optimized Architectures**: chọn database theo mô hình dữ liệu, tải đọc/ghi, độ trễ, HA, chi phí.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- Hỏi trước khi chọn DB: tải **đọc nhiều / ghi nhiều / cân bằng**, throughput cần và có biến động không, **dung lượng và thời gian lưu**, độ bền, độ trễ, số user đồng thời, mô hình dữ liệu (join? cấu trúc?), schema cố định hay linh hoạt, license.
- **RDBMS/OLTP**: RDS, Aurora (SQL, join). **NoSQL**: DynamoDB (JSON/key-value), ElastiCache (key-value in-memory), Neptune (graph), DocumentDB (MongoDB), Keyspaces (Cassandra). **Object**: S3. **Data warehouse/OLAP**: Redshift, Athena, EMR. **Search**: OpenSearch. **Time series**: Timestream.
- ⚠️ Lỗi thời: slide ghi "RDS tối đa 5 read replica" → nay **15**.
- ⚠️ Lỗi thời: "Aurora 128 TB" → **256 TiB**; Aurora Global Database tối đa **10** Region phụ.
- ⚠️ Lỗi thời: **QLDB** (ledger bất biến) đã ngừng hoạt động 31/07/2025 (nguồn web); **Timestream for LiveAnalytics** đóng với khách mới từ 20/06/2025 (khách mới dùng **Timestream for InfluxDB**).
- **S3** cũng là "database" key-value cho object lớn; không hợp với nhiều object nhỏ hay truy vấn phức tạp.

---

## 2. Core Concepts

### Amazon RDS
- **Là gì:** PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Db2 managed; provisioned instance + EBS, storage auto scaling; read replica, Multi-AZ; IAM auth, KMS, SG; backup tự động + PITR (tới 35 ngày); maintenance window có downtime.
- **Dùng khi nào:** dữ liệu quan hệ, truy vấn SQL, giao dịch OLTP.
- **Vì sao quan trọng:** ⚠️ tối đa **15** read replica (slide cũ ghi 5). **RDS Custom** khi cần truy cập OS Oracle/SQL Server; **RDS Proxy** khi nhiều kết nối ngắn.

### Amazon Aurora
- **Là gì:** API tương thích PostgreSQL/MySQL, compute tách storage; storage 6 bản sao trên 3 AZ, tự sửa lỗi, tự co giãn tới 256 TiB; cluster: writer + tối đa 15 replica.
- **Dùng khi nào:** giống RDS nhưng cần hiệu năng, HA cao hơn, ít bảo trì; **Serverless v2** cho tải không đoán được; **Global Database** cho đa Region; **Aurora ML**; **cloning**.
- **Vì sao quan trọng:** ⚠️ 256 TiB (không phải 128 TB); Global Database 10 Region phụ (không phải 5).

### Amazon ElastiCache
- **Là gì:** Valkey, Redis OSS, Memcached managed; in-memory, độ trễ dưới mili giây; cluster mode để sharding; Multi-AZ, read replica (Valkey/Redis OSS).
- **Dùng khi nào:** key-value store, đọc nhiều, cache truy vấn DB, session, leaderboard.
- **Vì sao quan trọng:** **phải sửa code** ứng dụng để dùng cache; không phải DB chính cho dữ liệu cần bền vững.

### Amazon DynamoDB
- **Là gì:** NoSQL độc quyền AWS, độ trễ mili giây một chữ số; on-demand hoặc provisioned + auto scaling; DAX; Streams; Global Tables; PITR; item tới 400 KB.
- **Dùng khi nào:** app serverless, dữ liệu nhỏ (KB), schema linh hoạt, truy cập theo key, session, cart.
- **Vì sao quan trọng:** không join như SQL; mô hình truy vấn phải thiết kế trước theo partition key.

### Amazon S3 như kho dữ liệu
- **Là gì:** key-value cho object; tốt cho object lớn, độ bền cao; object tối đa **48.8 TiB**; nhiều storage class, lifecycle.
- **Dùng khi nào:** static file, data lake, nội dung media, backup.
- **Vì sao quan trọng:** không hợp cho nhiều object nhỏ ghi liên tục hoặc truy vấn kiểu DB; truy vấn bằng **Athena**.

### Amazon DocumentDB
- **Là gì:** DB tài liệu tương thích **MongoDB** (JSON), managed, HA trên 3 AZ, storage tự tăng theo 10 GB.
- **Dùng khi nào:** migrate ứng dụng MongoDB lên AWS.
- **Vì sao quan trọng:** "Aurora của MongoDB"; từ khóa "MongoDB compatible" → DocumentDB.

### Amazon Neptune
- **Là gì:** graph database managed, 3 AZ, tới 15 read replica, truy vấn hàng tỷ quan hệ với độ trễ mili giây.
- **Dùng khi nào:** mạng xã hội, gợi ý, knowledge graph, phát hiện gian lận, Wikipedia.
- **Vì sao quan trọng:** **Neptune Streams** ghi lại thay đổi theo thứ tự (gửi thông báo, đồng bộ sang OpenSearch, replicate cross-Region).

### Amazon Keyspaces
- **Là gì:** DB tương thích **Apache Cassandra**, serverless, dùng CQL; 3 bản sao đa AZ; on-demand hoặc provisioned; mã hóa, backup, PITR 35 ngày.
- **Dùng khi nào:** dữ liệu IoT, time-series, ứng dụng Cassandra hiện có.
- **Vì sao quan trọng:** "Cassandra" / "CQL" → Keyspaces.

### Amazon Timestream
- **Là gì:** DB time series managed, serverless, lưu dữ liệu gần ở bộ nhớ và dữ liệu cũ ở tier rẻ; SQL tương thích; tích hợp Kinesis, Prometheus, Grafana, Quick Sight.
- **Dùng khi nào:** IoT, giám sát vận hành, analytics real-time theo thời gian.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: **Timestream for LiveAnalytics** đóng với khách mới từ 20/06/2025 → khách mới dùng **Timestream for InfluxDB** hoặc dịch vụ khác AWS gợi ý.

### ⚠️ Amazon QLDB
- **Là gì:** ledger database (sổ cái bất biến, kiểm chứng bằng mật mã), tập trung.
- **Dùng khi nào:** đề cũ: "ghi lại lịch sử thay đổi tài chính bất biến, kiểm chứng được" → QLDB.
- **Vì sao quan trọng:** ⚠️ QLDB đã **ngừng hoạt động 31/07/2025** (nguồn web, không tìm thấy trang docs AWS còn hoạt động); không chọn QLDB cho thiết kế mới.

### `[+]` Amazon MemoryDB
- **Là gì:** DB in-memory tương thích Valkey/Redis OSS nhưng **bền vững** (transaction log Multi-AZ), dùng làm DB chính.
- **Dùng khi nào:** cần tốc độ in-memory mà không muốn có thêm DB phía sau cache.
- **Vì sao quan trọng:** phân biệt với ElastiCache (cache, có thể mất dữ liệu khi lỗi).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| OLTP | Giao dịch nhiều, nhỏ | "transactional application" |
| OLAP | Phân tích khối lượng lớn | "analytics, data warehouse" |
| RDS | SQL managed | "relational, joins" |
| Aurora | SQL hiệu năng cao, 6 bản sao/3 AZ | "high performance MySQL/PostgreSQL" |
| ElastiCache | Cache in-memory | "sub-millisecond, cache" |
| MemoryDB | DB in-memory bền vững | "durable in-memory database" |
| DynamoDB | NoSQL key-value, serverless | "single-digit ms, flexible schema" |
| DocumentDB | Tương thích MongoDB | "MongoDB" |
| Neptune | Graph database | "social network, relationships" |
| Keyspaces | Tương thích Cassandra | "Cassandra, CQL" |
| Timestream | Time series | "IoT time-series data" |
| QLDB | Ledger (đã ngừng) | "immutable ledger" |
| S3 | Object key-value | "large objects, static content" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| RDS | Aurora | Aurora storage tự co giãn 256 TiB, 15 replica, failover nhanh | HA/hiệu năng cao → Aurora |
| DynamoDB | DocumentDB | DynamoDB API riêng AWS; DocumentDB tương thích MongoDB | "MongoDB" → DocumentDB |
| ElastiCache | MemoryDB | ElastiCache là cache; MemoryDB là DB chính bền vững | "Primary database in-memory" → MemoryDB |
| Neptune | DynamoDB | Neptune graph, truy vấn quan hệ phức tạp; DynamoDB key-value | "Friends of friends" → Neptune |
| Keyspaces | DynamoDB | Keyspaces dùng CQL Cassandra; DynamoDB API riêng | "Cassandra" → Keyspaces |
| Timestream | DynamoDB | Timestream tối ưu time series; DynamoDB tổng quát | "Time series" → Timestream |
| RDS | Redshift | RDS OLTP; Redshift OLAP | "Analytics / BI" → Redshift |
| S3 | DynamoDB | S3 object lớn; DynamoDB item nhỏ ≤ 400 KB | Object lớn → S3 |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "relational, complex joins, OLTP" | RDS / Aurora | |
| "MySQL compatible, highest availability, auto-growing storage" | Aurora | |
| "MongoDB workload migration" | DocumentDB | |
| "social graph, recommendation" | Neptune | |
| "Apache Cassandra, CQL" | Keyspaces | |
| "IoT time-series metrics" | Timestream (for InfluxDB cho khách mới) | |
| "key-value, serverless, millisecond, flexible schema" | DynamoDB | |
| "in-memory cache for DB queries" | ElastiCache | |
| "durable in-memory primary database" | MemoryDB | |
| "large objects, static content, data lake" | S3 | |
| "data warehouse, BI queries" | Redshift | Chương 18 |
| "full-text search" | OpenSearch | Chương 18 |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng mạng xã hội cần truy vấn "bạn của bạn" và gợi ý kết nối với độ trễ thấp. Chọn DB nào?
- A. RDS MySQL
- B. Amazon Neptune
- C. DynamoDB
- D. Redshift

<details><summary>Đáp án</summary>

**B.** Graph database tối ưu cho quan hệ nhiều tầng.
- A sai: join nhiều tầng chậm.
- C sai: không tối ưu duyệt đồ thị.
- D sai: OLAP.
</details>

**Câu 2.** Công ty chạy MongoDB on-prem, muốn chuyển lên dịch vụ managed tương thích. Chọn gì?
- A. DynamoDB
- B. Amazon DocumentDB
- C. Keyspaces
- D. Aurora PostgreSQL

<details><summary>Đáp án</summary>

**B.**
- A sai: API khác MongoDB.
- C sai: tương thích Cassandra.
- D sai: quan hệ.
</details>

**Câu 3.** Ứng dụng Cassandra cần chuyển lên AWS mà không quản lý cluster. Chọn gì?
- A. Amazon Keyspaces
- B. EMR
- C. Neptune
- D. ElastiCache

<details><summary>Đáp án</summary>

**A.**
- B sai: big data processing.
- C sai: graph.
- D sai: cache.
</details>

**Câu 4.** (Chọn 2) Cần database quan hệ có storage tự co giãn, 15 replica đọc độ trễ thấp và DR đa Region với RPO ~1 giây. Chọn gì?
- A. Amazon Aurora
- B. Aurora Global Database
- C. RDS Single-AZ
- D. DynamoDB
- E. Redshift

<details><summary>Đáp án</summary>

**A, B.**
- C sai: không HA, không đa Region.
- D sai: NoSQL.
- E sai: data warehouse.
</details>

**Câu 5.** Thiết bị IoT gửi số đo nhiệt độ mỗi giây; cần lưu và phân tích theo thời gian. Khách hàng mới của AWS năm 2026. Chọn gì?
- A. Timestream for LiveAnalytics
- B. Timestream for InfluxDB
- C. QLDB
- D. Neptune

<details><summary>Đáp án</summary>

**B.** Lựa chọn time series còn mở cho khách mới.
- A sai: đóng với khách mới từ 20/06/2025.
- C sai: đã ngừng hoạt động và là ledger.
- D sai: graph.
</details>

**Câu 6.** Ứng dụng cần DB in-memory làm nguồn dữ liệu chính, không chấp nhận mất dữ liệu khi node lỗi. Chọn gì?
- A. ElastiCache Memcached
- B. Amazon MemoryDB
- C. DAX
- D. S3

<details><summary>Đáp án</summary>

**B.** Bền vững nhờ transaction log Multi-AZ.
- A sai: không replication, không persistence.
- C sai: cache cho DynamoDB.
- D sai: không phải in-memory.
</details>

**Câu 7.** Team cần lưu ảnh và video kích thước GB, truy cập qua HTTP. Chọn gì?
- A. DynamoDB
- B. RDS BLOB
- C. Amazon S3
- D. ElastiCache

<details><summary>Đáp án</summary>

**C.**
- A sai: item tối đa 400 KB.
- B sai: tốn kém, không phù hợp file lớn.
- D sai: bộ nhớ đắt, không bền.
</details>

**Câu 8.** Read replica RDS PostgreSQL hiện có 6 cái, team muốn thêm nữa. Phát biểu nào đúng (10/2026)?
- A. Không thể, tối đa 5
- B. Có thể, tối đa 15 replica cho mỗi DB nguồn
- C. Phải chuyển sang Aurora mới có replica
- D. Chỉ thêm được replica cross-Region

<details><summary>Đáp án</summary>

**B.**
- A sai: con số 5 đã lỗi thời.
- C sai: RDS vẫn hỗ trợ replica.
- D sai: thêm được cả trong Region.
</details>

---

### Nguồn đã đối chiếu
- RDS read replicas (tối đa 15): https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html
- Aurora storage (256 TiB): https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html
- Aurora Global Database: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html
- Timestream for LiveAnalytics availability change (20/06/2025): https://docs.aws.amazon.com/timestream/latest/developerguide/AmazonTimestreamForLiveAnalytics-availability-change.html
- Amazon DocumentDB: https://docs.aws.amazon.com/documentdb/latest/developerguide/what-is.html
- Neptune Streams: https://docs.aws.amazon.com/neptune/latest/userguide/streams.html
- Amazon Keyspaces: https://docs.aws.amazon.com/keyspaces/latest/devguide/what-is-keyspaces.html
- Amazon MemoryDB: https://docs.aws.amazon.com/memorydb/latest/devguide/what-is-memorydb.html
- QLDB ngừng hoạt động 31/07/2025: nguồn web (thông báo end of support của AWS), không tìm thấy trang docs còn hoạt động.
