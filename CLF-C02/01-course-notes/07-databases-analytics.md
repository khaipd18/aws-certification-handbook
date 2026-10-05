# CLF-C02 — Databases & Analytics

> Thuộc **Domain 3: Cloud Technology and Services** (database, analytics). Trọng tâm: chọn đúng database theo loại dữ liệu và workload (OLTP vs OLAP, relational vs NoSQL).
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Managed database** = AWS lo provisioning, HA, backup, **vá OS**, monitoring; tự chạy DB trên EC2 thì bạn tự lo tất cả.
- **Relational / OLTP:** **RDS** (Postgres, MySQL, MariaDB, Oracle, SQL Server, Db2) và **Aurora** (độc quyền AWS, nhanh hơn, tự tăng storage).
- **RDS Read Replicas** = scale **đọc**; **Multi-AZ** = **failover** (HA); **Multi-Region read replicas** = DR + đọc local toàn cầu.
- **NoSQL:** **DynamoDB** (key-value, serverless, ms latency) + **DAX** (cache riêng cho DynamoDB) + **Global Tables** (active-active nhiều Region); **DocumentDB** ("Aurora cho MongoDB", JSON); **Neptune** (graph).
- **In-memory cache:** **ElastiCache** (Valkey, Redis OSS, Memcached).
- **Analytics:** **Redshift** (data warehouse OLAP, columnar), **EMR** (Hadoop/Spark), **Athena** (SQL serverless trên S3), **Quick Sight** (dashboard BI), **Glue** (ETL serverless + Data Catalog).
- **DMS** = migrate database lên AWS, nguồn vẫn chạy trong lúc migrate; homogeneous hoặc heterogeneous.

---

## 2. Core Concepts

### Relational vs NoSQL
- **Là gì:** **Relational** = bảng có liên kết, truy vấn bằng SQL; **NoSQL** = phi quan hệ, schema linh hoạt, tối ưu cho một data model cụ thể (key-value, document, graph, in-memory, search).
- **Dùng khi nào:** NoSQL khi cần mô hình dữ liệu dễ thay đổi, scale out bằng cluster phân tán, hiệu năng cao; dữ liệu dạng JSON lồng nhau là ví dụ điển hình.
- **Vì sao quan trọng:** đề cho loại dữ liệu và hỏi dịch vụ phù hợp.

### Managed DB & Shared Responsibility
- **Là gì:** AWS cung cấp provisioning nhanh, HA, scale dọc/ngang, backup & restore tự động, vá OS, monitoring và alerting.
- **Dùng khi nào:** chạy DB trên EC2 chỉ khi cần quyền kiểm soát đặc biệt; khi đó bạn tự lo resiliency, backup, patching, HA, scaling.
- **Vì sao quan trọng:** "least operational overhead" → managed DB, không phải DB trên EC2.

### Amazon RDS
- **Là gì:** dịch vụ DB quan hệ managed dùng SQL: PostgreSQL, MySQL, MariaDB, Oracle, Microsoft SQL Server, IBM Db2, Aurora.
- **Dùng khi nào:** OLTP truyền thống. Lợi ích: provisioning tự động, vá OS, backup liên tục với **Point-in-Time Restore**, dashboard, read replicas, Multi-AZ, maintenance window, storage trên EBS.
- **Vì sao quan trọng:** **không SSH** được vào instance RDS.

### Amazon Aurora
- **Là gì:** DB quan hệ **độc quyền** của AWS, tương thích PostgreSQL và MySQL; nhanh hơn MySQL trên RDS khoảng 5 lần, Postgres trên RDS hơn 3 lần.
- **Dùng khi nào:** cần hiệu năng và HA cao hơn RDS thông thường; storage tự tăng theo bước 10 GB tới 256 TiB (giới hạn thực tế phụ thuộc engine version).
- **Vì sao quan trọng:** đắt hơn RDS khoảng 20% nhưng hiệu quả hơn.

### Aurora Serverless
- **Là gì:** Aurora tự khởi tạo và **tự scale** theo mức dùng thực tế, hỗ trợ PostgreSQL và MySQL.
- **Dùng khi nào:** workload **không thường xuyên, gián đoạn, khó dự đoán**.
- **Vì sao quan trọng:** không cần capacity planning, ít việc vận hành nhất, trả theo giây.

### RDS Read Replicas
- **Là gì:** bản sao chỉ đọc để **scale đọc**, tối đa **15** read replica; dữ liệu chỉ ghi vào DB chính.
- **Dùng khi nào:** ứng dụng đọc nhiều (báo cáo, dashboard).
- **Vì sao quan trọng:** read replica không phải cơ chế failover chính (đó là Multi-AZ).

### RDS Multi-AZ
- **Là gì:** DB standby ở AZ khác, **tự failover** khi AZ chính gặp sự cố.
- **Dùng khi nào:** cần **high availability**, disaster recovery trong Region.
- **Vì sao quan trọng:** dữ liệu chỉ đọc/ghi vào DB chính; chỉ có **1 AZ khác** làm failover.

### RDS Multi-Region (Read Replicas)
- **Là gì:** read replica đặt ở Region khác.
- **Dùng khi nào:** **DR khi cả Region gặp sự cố** và cho phép đọc với latency thấp ở nhiều nơi trên thế giới.
- **Vì sao quan trọng:** có **chi phí replication** dữ liệu giữa Region.

### Amazon ElastiCache
- **Là gì:** cache **in-memory** managed, hiệu năng cao, latency thấp; giảm tải cho DB khi đọc nhiều.
- ⚠️ Lỗi thời: "managed Redis hoặc Memcached" → nay ElastiCache hỗ trợ **Valkey, Redis OSS, Memcached**, có cả tùy chọn **serverless**.
- **Dùng khi nào:** lưu session, cache kết quả truy vấn; AWS lo vá OS, tối ưu, cấu hình, monitoring, khôi phục lỗi, backup.

### Amazon DynamoDB
- **Là gì:** DB **NoSQL key-value**, fully managed, **serverless**, replicate trên 3 AZ, scale tới hàng triệu request/giây, latency **một chữ số mili giây**.
- **Dùng khi nào:** app web/mobile/gaming quy mô lớn cần tốc độ ổn định; tích hợp IAM; có table class **Standard** và **Infrequent Access**.
- **Vì sao quan trọng:** từ khóa "serverless NoSQL", "key-value", "single-digit millisecond" → DynamoDB.

### DynamoDB Accelerator (DAX)
- **Là gì:** cache in-memory fully managed **chỉ dành cho DynamoDB**, cải thiện tới 10 lần: từ mili giây xuống **micro giây**.
- **Dùng khi nào:** cần đọc DynamoDB nhanh hơn nữa mà không sửa logic nhiều.
- **Vì sao quan trọng:** DAX chỉ dùng cho DynamoDB; **ElastiCache** dùng cho DB khác.

### DynamoDB Global Tables
- **Là gì:** bảng DynamoDB truy cập với latency thấp ở **nhiều Region**, replication **active-active** (đọc/ghi ở mọi Region).
- **Dùng khi nào:** người dùng toàn cầu cần ghi dữ liệu tại Region gần nhất.
- **Vì sao quan trọng:** "multi-Region, active-active, read/write anywhere" → Global Tables.

### Amazon Redshift
- **Là gì:** **data warehouse** dựa trên PostgreSQL nhưng dành cho **OLAP** (phân tích), không phải OLTP; lưu theo **cột**, xử lý song song (MPP), scale tới PB.
- **Dùng khi nào:** nạp dữ liệu theo đợt (vd: mỗi giờ) rồi chạy truy vấn phân tích bằng SQL; tích hợp BI (Quick Sight, Tableau).
- **Vì sao quan trọng:** **Redshift Serverless** tự provision và scale capacity, chỉ trả cho phần dùng: hợp báo cáo, dashboard.

### Amazon EMR
- **Là gì:** "Elastic MapReduce": tạo cluster **Hadoop** (hàng trăm EC2) để xử lý big data; hỗ trợ Spark, HBase, Presto, Flink.
- **Dùng khi nào:** xử lý dữ liệu lớn, ML, web indexing.
- **Vì sao quan trọng:** EMR lo provisioning, cấu hình, auto-scaling, tích hợp Spot Instances.

### Amazon Athena
- **Là gì:** dịch vụ truy vấn **serverless** bằng **SQL** chuẩn trên dữ liệu trong **S3** (CSV, JSON, ORC, Avro, Parquet); trả theo TB dữ liệu quét (slide: $5/TB).
- **Dùng khi nào:** BI, phân tích, báo cáo; phân tích VPC Flow Logs, ELB logs, CloudTrail.
- **Vì sao quan trọng:** dùng dữ liệu **nén hoặc dạng cột** để quét ít hơn, rẻ hơn. Exam tip: "analyze data in S3 using serverless SQL" → Athena.

### Amazon Quick Sight
- ⚠️ Lỗi thời: tên "Amazon QuickSight" → nay là **Amazon Quick Sight**, một phần của **Amazon Quick**; exam guide CLF-C02 hiện dùng tên mới.
- **Là gì:** dịch vụ **BI serverless** có ML để tạo dashboard tương tác, tự scale, nhúng được, tính phí theo session.
- **Dùng khi nào:** phân tích kinh doanh, trực quan hóa, phân tích ad-hoc; tích hợp RDS, Aurora, Athena, Redshift, S3.

### Amazon DocumentDB
- **Là gì:** DB **document** tương thích **MongoDB** (lưu, truy vấn, index JSON); khái niệm triển khai giống Aurora.
- **Dùng khi nào:** app đang dùng MongoDB muốn chuyển sang dịch vụ managed.
- **Vì sao quan trọng:** fully managed, HA trên 3 AZ, storage tự tăng theo bước 10 GB, scale tới hàng triệu request/giây.

### Amazon Neptune
- **Là gì:** DB **graph** fully managed; lưu hàng tỷ quan hệ và truy vấn với latency mili giây; HA trên 3 AZ, tới 15 read replica.
- **Dùng khi nào:** dữ liệu liên kết chặt: mạng xã hội, knowledge graph, phát hiện gian lận, recommendation engine.
- **Vì sao quan trọng:** từ khóa "highly connected data", "relationships" → Neptune.

### Amazon Timestream
- ⚠️ Lỗi thời: **Timestream for LiveAnalytics** không nhận khách mới từ **20/06/2025** → khách mới dùng **Timestream for InfluxDB** hoặc dịch vụ thay thế AWS đề xuất.
- **Là gì:** DB **time series** fully managed, serverless; lưu và phân tích hàng nghìn tỷ event/ngày, có hàm phân tích time series sẵn.
- **Dùng khi nào:** dữ liệu theo thời gian: IoT, metric vận hành.

### Amazon Managed Blockchain
- ⚠️ Sắp ngừng: Amazon Managed Blockchain (AMB) **ngừng nhận khách mới từ 29/10/2026** và **hết hỗ trợ 29/09/2027** → chuyển sang node tự quản trên AWS, nhà cung cấp bên thứ ba, hoặc database AWS.
- **Là gì:** dịch vụ managed để tham gia mạng blockchain public hoặc tạo mạng private; tương thích Hyperledger Fabric và Ethereum.
- **Dùng khi nào:** nhiều bên giao dịch mà không cần bên trung gian tin cậy.

### AWS Glue
- **Là gì:** dịch vụ **ETL** (Extract, Transform, Load) managed, **serverless**.
- **Dùng khi nào:** chuẩn bị và biến đổi dữ liệu cho phân tích (vd: S3/RDS → Glue → Redshift).
- **Vì sao quan trọng:** **Glue Data Catalog** là catalog dataset dùng chung cho Athena, Redshift, EMR.

### AWS Database Migration Service (DMS)
- **Là gì:** migrate DB lên AWS nhanh, an toàn, tự phục hồi; **DB nguồn vẫn chạy** trong lúc migrate.
- **Dùng khi nào:** **homogeneous** (Oracle → Oracle) hoặc **heterogeneous** (SQL Server → Aurora).
- **Vì sao quan trọng:** sample question của khóa học: "dịch vụ nào đơn giản hóa việc migrate database" → DMS.

### `[+]` AWS Schema Conversion Tool (SCT)
- **Là gì:** chuyển đổi **schema** và code DB từ engine này sang engine khác.
- **Dùng khi nào:** migrate **heterogeneous** (vd: Oracle → Aurora PostgreSQL): SCT chuyển schema, DMS chuyển dữ liệu.
- **Vì sao quan trọng:** SCT nằm trong danh sách in-scope của CLF-C02.

### `[+]` Amazon OpenSearch Service
- **Là gì:** dịch vụ managed cho **search và phân tích log** (fork của Elasticsearch).
- **Dùng khi nào:** full-text search, phân tích log và clickstream gần real-time, dashboard.
- **Vì sao quan trọng:** in-scope CLF-C02; từ khóa "search engine", "full-text search" → OpenSearch.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| OLTP | Xử lý giao dịch online, đọc/ghi nhiều bản ghi nhỏ | "transactions", "orders" → RDS/Aurora |
| OLAP | Xử lý phân tích trên dữ liệu lớn | "analytics", "data warehouse" → Redshift |
| Amazon RDS | DB quan hệ managed (6 engine + Aurora) | "managed SQL database" |
| Point-in-Time Restore | Khôi phục DB về một thời điểm bất kỳ | "restore to 2:15 PM yesterday" |
| Amazon Aurora | DB quan hệ độc quyền AWS, tương thích MySQL/PostgreSQL | "cloud-optimized relational", "5x MySQL" |
| Aurora Serverless | Aurora tự scale, trả theo giây | "intermittent, unpredictable workload" |
| Read Replica | Bản sao chỉ đọc, scale đọc (tới 15) | "improve read performance" |
| Multi-AZ | Standby ở AZ khác, tự failover | "high availability", "AZ outage" |
| ElastiCache | Cache in-memory (Valkey, Redis OSS, Memcached) | "reduce DB load", "session store" |
| DynamoDB | NoSQL key-value serverless | "serverless", "single-digit ms", "key-value" |
| DAX | Cache in-memory riêng cho DynamoDB, micro giây | "microsecond latency for DynamoDB" |
| Global Tables | DynamoDB active-active nhiều Region | "multi-Region read/write" |
| Redshift | Data warehouse columnar, MPP | "data warehouse", "BI on petabytes" |
| Amazon EMR | Cluster Hadoop/Spark managed | "big data processing", "Hadoop" |
| Amazon Athena | SQL serverless trên S3 | "query S3 with SQL", "serverless" |
| Amazon Quick Sight | BI serverless, dashboard | "interactive dashboards", "visualization" |
| DocumentDB | Document DB tương thích MongoDB | "MongoDB", "JSON documents" |
| Neptune | Graph DB | "social network", "knowledge graph", "fraud ring" |
| Timestream | Time series DB | "IoT time-series data" |
| Managed Blockchain | Blockchain managed (Hyperledger Fabric, Ethereum) | "decentralized ledger, no central authority" |
| AWS Glue | ETL serverless + Data Catalog | "ETL", "data catalog" |
| DMS | Migrate DB, nguồn vẫn chạy | "migrate database with minimal downtime" |
| Homogeneous / Heterogeneous migration | Cùng engine / khác engine | "Oracle to Oracle" / "SQL Server to Aurora" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Read Replicas | Multi-AZ | Scale **đọc** vs **failover** HA | "Read performance" → Replica; "AZ outage" → Multi-AZ |
| Multi-AZ | Multi-Region replica | HA trong Region vs DR cho cả Region + đọc toàn cầu | "Regional disaster" → Multi-Region |
| RDS | Aurora | RDS nhiều engine; Aurora độc quyền, nhanh hơn, đắt hơn ~20% | "Cloud-optimized, MySQL/PostgreSQL compatible" → Aurora |
| RDS | DB trên EC2 | Managed (AWS vá OS, không SSH) vs tự quản toàn bộ | "Need OS access" → EC2 |
| DynamoDB | RDS | NoSQL key-value serverless vs SQL quan hệ | "Joins, complex queries" → RDS |
| DAX | ElastiCache | DAX chỉ cho DynamoDB; ElastiCache cho DB khác | "Cache for DynamoDB" → DAX |
| Redshift | RDS | OLAP data warehouse vs OLTP | "Analytics on large dataset" → Redshift |
| Redshift | Athena | Warehouse có cluster/serverless, dữ liệu nạp vào vs truy vấn trực tiếp trên S3 | "Ad-hoc SQL on S3, no infrastructure" → Athena |
| Athena | EMR | SQL serverless vs cluster Hadoop/Spark | "Hadoop, Spark" → EMR |
| Glue | DMS | ETL dữ liệu cho phân tích vs migrate database | "Migrate DB" → DMS |
| DMS | SCT | DMS chuyển dữ liệu; SCT chuyển schema khi khác engine | Heterogeneous → SCT + DMS |
| DocumentDB | DynamoDB | MongoDB-compatible document vs key-value AWS-native | "MongoDB" → DocumentDB |
| Neptune | DynamoDB | Graph vs key-value | "Relationships" → Neptune |
| Quick Sight | Athena | Dashboard/visualization vs truy vấn SQL | "Dashboard" → Quick Sight |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "managed relational database, SQL" | Amazon RDS | |
| "MySQL/PostgreSQL compatible, high performance, auto-growing storage" | Amazon Aurora | |
| "relational DB for intermittent/unpredictable workload" | Aurora Serverless | |
| "improve read performance of RDS" | Read Replicas | |
| "automatic failover to another AZ" | RDS Multi-AZ | |
| "serverless NoSQL, millisecond latency" | DynamoDB | |
| "microsecond reads from DynamoDB" | DAX | |
| "multi-Region active-active NoSQL" | DynamoDB Global Tables | |
| "in-memory cache to reduce DB load" | ElastiCache | |
| "data warehouse, OLAP, columnar" | Redshift | |
| "run SQL queries directly on S3 without servers" | Athena | |
| "Hadoop/Spark cluster" | EMR | |
| "BI dashboards" | Amazon Quick Sight | |
| "MongoDB workload" | DocumentDB | |
| "graph, social network, fraud detection relationships" | Neptune | |
| "time series, IoT metrics" | Timestream | Xem ⚠️ LiveAnalytics |
| "ETL, data catalog" | AWS Glue | |
| "migrate on-prem DB to AWS with minimal downtime" | DMS | |
| "convert Oracle schema to Aurora" | SCT | |
| "full-text search, log analytics" | OpenSearch Service | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Ứng dụng báo cáo làm DB RDS quá tải vì lượng truy vấn đọc lớn. Cách cải thiện hiệu năng đọc?
- A. Bật Multi-AZ
- B. Tạo Read Replicas
- C. Chuyển sang S3 Glacier
- D. Dùng AWS Glue

<details><summary>Đáp án</summary>

**B.** Read Replicas scale workload đọc.
- A sai: Multi-AZ cho failover, standby không phục vụ đọc.
- C sai: archive storage.
- D sai: ETL.
</details>

**Câu 2.** Database phải tự động chuyển sang AZ khác khi AZ chính gặp sự cố. Chọn gì?
- A. RDS Read Replica cùng AZ
- B. RDS Multi-AZ
- C. ElastiCache
- D. Redshift Serverless

<details><summary>Đáp án</summary>

**B.** Multi-AZ cung cấp standby và failover tự động.
- A sai: replica cùng AZ không giúp khi AZ hỏng; replica chủ yếu để đọc.
- C sai: cache.
- D sai: data warehouse.
</details>

**Câu 3.** Nhà phân tích muốn chạy SQL trên file CSV và Parquet trong S3 mà không quản lý hạ tầng. Chọn gì?
- A. Amazon Redshift (provisioned)
- B. Amazon Athena
- C. Amazon EMR
- D. Amazon RDS

<details><summary>Đáp án</summary>

**B.** Athena = SQL serverless trực tiếp trên S3.
- A sai: cần cluster và nạp dữ liệu.
- C sai: cluster Hadoop/Spark phải quản lý.
- D sai: DB quan hệ cho OLTP.
</details>

**Câu 4.** Game mobile cần DB NoSQL serverless, latency mili giây, scale tới hàng triệu request mỗi giây. Chọn gì?
- A. Amazon DynamoDB
- B. Amazon Aurora
- C. Amazon Neptune
- D. Amazon Redshift

<details><summary>Đáp án</summary>

**A.** DynamoDB là key-value serverless, latency một chữ số mili giây.
- B sai: quan hệ.
- C sai: graph.
- D sai: data warehouse.
</details>

**Câu 5.** (Chọn 2) Lợi ích của Amazon RDS so với tự cài database trên EC2?
- A. AWS tự vá hệ điều hành
- B. Bạn có thể SSH vào instance để tùy chỉnh OS
- C. Backup liên tục và Point-in-Time Restore
- D. Không bao giờ tốn chi phí
- E. Chạy được mọi engine database tồn tại

<details><summary>Đáp án</summary>

**A, C.**
- B sai: không SSH được vào RDS.
- D sai: vẫn trả tiền.
- E sai: chỉ hỗ trợ một số engine.
</details>

**Câu 6.** Mạng xã hội cần truy vấn nhanh quan hệ bạn bè, bài viết, lượt thích. Chọn DB nào?
- A. Amazon DocumentDB
- B. Amazon Neptune
- C. Amazon ElastiCache
- D. Amazon Timestream

<details><summary>Đáp án</summary>

**B.** Neptune là graph DB cho dữ liệu liên kết chặt.
- A sai: document DB.
- C sai: cache.
- D sai: time series.
</details>

**Câu 7.** Công ty chuyển SQL Server on-premises sang Aurora PostgreSQL, DB nguồn phải tiếp tục phục vụ trong lúc migrate. (Chọn 2) Dùng dịch vụ nào?
- A. AWS Database Migration Service
- B. AWS Schema Conversion Tool
- C. AWS Glue
- D. Amazon Athena
- E. AWS Storage Gateway

<details><summary>Đáp án</summary>

**A, B.** Khác engine (heterogeneous): SCT chuyển schema, DMS chuyển dữ liệu khi nguồn vẫn chạy.
- C sai: ETL cho phân tích.
- D sai: truy vấn S3.
- E sai: hybrid storage.
</details>

**Câu 8.** Bảng DynamoDB cần đọc với latency micro giây mà không thay đổi nhiều code. Chọn gì?
- A. ElastiCache for Memcached
- B. DynamoDB Accelerator (DAX)
- C. Read Replica
- D. DynamoDB Global Tables

<details><summary>Đáp án</summary>

**B.** DAX là cache in-memory tích hợp riêng cho DynamoDB, đưa latency xuống micro giây.
- A sai: dùng được nhưng cần tự tích hợp; DAX là lựa chọn native.
- C sai: khái niệm của RDS.
- D sai: replication đa Region, không phải cache.
</details>

---

### Nguồn đã đối chiếu
- ElastiCache engines (Valkey, Redis OSS, Memcached, serverless): https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/SelectEngine.html
- Aurora storage auto scaling (tới 256 TiB): https://docs.aws.amazon.com/rds/latest/auroraextendedcontent/aurora-features-scalability.html
- Aurora size limits theo engine version: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html
- Timestream for LiveAnalytics availability change: https://docs.aws.amazon.com/timestream/latest/developerguide/AmazonTimestreamForLiveAnalytics-availability-change.html
- Amazon Managed Blockchain end of support: https://docs.aws.amazon.com/managed-blockchain/latest/hyperledger-fabric-dev/managed-blockchain-end-of-support.html
- CLF-C02 in-scope services (Amazon Quick Sight, OpenSearch, SCT): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-in-scope-services.html
