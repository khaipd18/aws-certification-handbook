# SAA-C03 — Data & Analytics: Athena, Redshift, OpenSearch, EMR, Quick Sight, Glue, Lake Formation, Flink, MSK

> Thuộc **Domain 3: Design High-Performing Architectures** và **Domain 4: Design Cost-Optimized Architectures**: truy vấn data lake, data warehouse, tìm kiếm, big data, ETL, quản trị data lake, streaming analytics.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Athena**: SQL serverless trên S3 (trả theo TB quét); tăng tốc & giảm phí: **định dạng cột** (Parquet/ORC), **nén**, **partition**, file lớn (> 128 MB); **Federated Query** qua Lambda connector.
- **Redshift**: data warehouse OLAP dạng cột, MPP; provisioned hoặc **Serverless**; **Spectrum** truy vấn S3 không cần load; snapshot tự động/thủ công, copy cross-Region cho DR.
- **OpenSearch**: tìm kiếm full-text, phân tích log; managed cluster hoặc **Serverless**; thường đi kèm DynamoDB/DB chính làm nguồn.
- **EMR**: cluster Hadoop/Spark/HBase/Presto/Flink managed; node **primary/core/task**; Spot cho task node; **EMR Serverless**.
- ⚠️ Lỗi thời: **QuickSight** → nay là **Amazon Quick Sight** (thuộc **Amazon Quick**); vẫn là BI serverless, engine **SPICE**, phân quyền **Users/Groups** (Enterprise), row-level security.
- **Glue**: ETL serverless, **Data Catalog** (crawler, dùng bởi Athena/Redshift Spectrum/EMR), **job bookmarks**, Glue Studio, DataBrew, Streaming ETL.
- **Lake Formation**: dựng data lake nhanh, **phân quyền tập trung** row/column-level trên Glue Catalog.
- ⚠️ Lỗi thời: **Kinesis Data Analytics** → **Managed Service for Apache Flink**; KDA for SQL đã ngừng (không tạo mới từ 15/10/2025, xóa ứng dụng từ 27/01/2026). **MSK** = Kafka managed (provisioned/Serverless).

---

## 2. Core Concepts

### Amazon Athena
- **Là gì:** truy vấn S3 bằng SQL chuẩn (Trino/Presto), không server; hỗ trợ CSV, JSON, ORC, Avro, Parquet; ~$5/TB quét.
- **Dùng khi nào:** ad-hoc query, BI, phân tích log VPC Flow Logs, ELB, CloudTrail; thường đi với Quick Sight.
- **Vì sao quan trọng:** tối ưu: Parquet/ORC, nén, partition (`s3://bucket/year=/month=/`), file lớn; dùng **CTAS** để chuyển sang Parquet. **Federated Query** truy vấn DynamoDB, RDS, on-prem... qua Lambda, kết quả lưu S3.

### Amazon Redshift
- **Là gì:** data warehouse dựa trên PostgreSQL nhưng cho **OLAP**, lưu dạng cột, xử lý song song MPP; leader node + compute node.
- **Dùng khi nào:** báo cáo, BI, join và aggregate trên dữ liệu lớn có cấu trúc; **Redshift Serverless** khi tải không đều.
- **Vì sao quan trọng:** so với Athena: Redshift nhanh hơn cho truy vấn phức tạp lặp lại nhờ index/load sẵn; Athena tốt cho ad-hoc trên S3.

### Redshift snapshots, DR & loading
- **Là gì:** Multi-AZ cho một số loại cluster (RA3); snapshot incremental lưu S3, tự động (theo giờ/dung lượng) hoặc thủ công; tự **copy snapshot sang Region khác** (KMS cần snapshot copy grant).
- **Dùng khi nào:** nạp dữ liệu: **Data Firehose**, **COPY từ S3** (qua Internet hoặc **Enhanced VPC Routing** đi trong VPC), JDBC (insert theo lô lớn).
- **Vì sao quan trọng:** **Redshift Spectrum** truy vấn dữ liệu trong S3 mà không load, cần cluster đang chạy.

### Amazon OpenSearch Service
- **Là gì:** kế thừa Elasticsearch: tìm kiếm mọi trường, kể cả khớp một phần; managed cluster hoặc Serverless; có OpenSearch Dashboards.
- **Dùng khi nào:** tìm kiếm sản phẩm, phân tích log real-time; nạp từ Firehose, IoT, CloudWatch Logs, Lambda.
- **Vì sao quan trọng:** mẫu "DynamoDB + Streams → Lambda → OpenSearch" để tìm kiếm trên dữ liệu NoSQL; không thay thế DB chính cho truy cập theo key.

### Amazon EMR
- **Là gì:** cluster Hadoop managed (chạy Spark, HBase, Presto, Flink...), tự scale, tích hợp Spot.
- **Dùng khi nào:** xử lý dữ liệu, ML, web indexing, big data; **primary node** (điều phối), **core node** (chạy task + lưu HDFS), **task node** (chỉ chạy task, hợp Spot).
- **Vì sao quan trọng:** mua: On-Demand, Reserved (≥ 1 năm, hợp primary/core), Spot (task); cluster lâu dài hoặc tạm thời; **EMR Serverless** khi không muốn quản cluster.

### Amazon Quick Sight
- **Là gì:** BI serverless, dashboard tương tác, scale theo user, trả theo phiên; engine in-memory **SPICE** (khi import dữ liệu).
- **Dùng khi nào:** dashboard kinh doanh từ RDS, Aurora, Athena, Redshift, S3, OpenSearch, Timestream, SaaS (Salesforce, Jira), on-prem DB (JDBC).
- **Vì sao quan trọng:** ⚠️ đổi tên: QuickSight → **Amazon Quick Sight** trong bộ **Amazon Quick**. Enterprise có column-level security; dashboard chia sẻ cho Users/Groups (không phải IAM user).

### AWS Glue
- **Là gì:** ETL serverless (Spark); **Data Catalog** chứa metadata bảng, do **crawler** quét S3/RDS/DynamoDB/JDBC tạo ra; Athena, Redshift Spectrum, EMR dùng chung catalog.
- **Dùng khi nào:** chuyển CSV → Parquet cho Athena; **job bookmarks** để không xử lý lại dữ liệu cũ; **DataBrew** làm sạch dữ liệu không cần code; **Glue Studio** GUI; **Streaming ETL** trên Kinesis/Kafka/MSK.
- **Vì sao quan trọng:** từ khóa "serverless ETL", "metadata catalog" → Glue.

### AWS Lake Formation
- **Là gì:** dựng data lake trong vài ngày: thu thập, làm sạch, chuyển đổi, catalog; blueprint cho S3, RDS, DB on-prem, NoSQL; xây trên Glue.
- **Dùng khi nào:** cần **phân quyền tập trung** chi tiết tới **row và column** cho nhiều công cụ (Athena, Quick Sight, Redshift, EMR).
- **Vì sao quan trọng:** "fine-grained access control cho data lake" → Lake Formation, không phải bucket policy.

### Managed Service for Apache Flink
- **Là gì:** chạy ứng dụng Apache Flink (Java, Scala, SQL, Python) managed trên stream từ Kinesis Data Streams hoặc MSK; tự cung cấp tài nguyên, scale, checkpoint/snapshot.
- **Dùng khi nào:** phân tích real-time, tính toán theo cửa sổ thời gian, phát hiện bất thường.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Kinesis Data Analytics for SQL" → đã ngừng; Flink **không đọc trực tiếp từ Firehose**.

### Amazon MSK
- **Là gì:** Apache Kafka managed: tạo/cập nhật/xóa cluster, broker và ZooKeeper/KRaft do AWS quản, Multi-AZ (tới 3), dữ liệu lưu EBS lâu tùy ý; **MSK Serverless** không quản capacity.
- **Dùng khi nào:** ứng dụng đã dùng Kafka; consumer: Flink, Glue Streaming, Lambda, ứng dụng trên EC2/ECS/EKS.
- **Vì sao quan trọng:** so với Kinesis: MSK message mặc định 1 MB, cấu hình lớn hơn; topic + partition (thêm partition được, không bớt); mã hóa TLS tùy chọn. ⚠️ Bảng slide ghi "Kinesis giới hạn 1 MB" → Kinesis nay cấu hình tới **10 MiB**.

### `[+]` Big data ingestion pipeline mẫu
- **Là gì:** IoT Core → **Data Firehose** → S3 (raw) → Lambda/Glue chuyển đổi → S3 (processed) → **Athena** → **Quick Sight**/Redshift.
- **Dùng khi nào:** cần pipeline serverless, gần real-time, chi phí thấp.
- **Vì sao quan trọng:** đề hay hỏi thành phần còn thiếu trong pipeline kiểu này.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Athena | SQL serverless trên S3 | "query S3 ad-hoc, no infrastructure" |
| Federated Query | Athena truy vấn nguồn ngoài S3 | "query across RDS, DynamoDB, on-prem" |
| Columnar format | Parquet/ORC | "reduce Athena cost" |
| Redshift | Data warehouse OLAP | "BI, complex analytics" |
| Redshift Spectrum | Redshift truy vấn S3 | "query S3 without loading" |
| Enhanced VPC Routing | COPY/UNLOAD đi trong VPC | "keep traffic private" |
| OpenSearch | Search & log analytics | "full-text search", "partial match" |
| EMR | Hadoop/Spark managed | "big data cluster", "Spark" |
| Task node | Node EMR chỉ tính toán | "use Spot safely" |
| Quick Sight / SPICE | BI serverless, engine in-memory | "interactive dashboards" |
| Glue Data Catalog | Metadata bảng | "central metadata", "crawler" |
| Job bookmarks | Không xử lý lại dữ liệu cũ | "process only new data" |
| Lake Formation | Data lake + quyền row/column | "fine-grained data lake access" |
| Managed Service for Apache Flink | Phân tích stream real-time | "real-time analytics on streams" |
| Amazon MSK | Kafka managed | "Apache Kafka" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Athena | Redshift | Athena ad-hoc trên S3, trả theo quét; Redshift warehouse, truy vấn phức tạp lặp lại | Ad-hoc → Athena |
| Redshift | Redshift Spectrum | Redshift query dữ liệu đã load; Spectrum query S3 trực tiếp | "Without loading" → Spectrum |
| Glue | EMR | Glue ETL serverless; EMR cluster tự quản cấu hình | "Serverless ETL" → Glue |
| Glue Data Catalog | Lake Formation | Catalog chứa metadata; Lake Formation thêm phân quyền tập trung | Row/column permissions → Lake Formation |
| OpenSearch | Athena | OpenSearch search/full-text, index; Athena SQL trên S3 | "Search" → OpenSearch |
| Kinesis Data Streams | MSK | Kinesis API AWS, shard; MSK Kafka, topic/partition | "Kafka" → MSK |
| Flink | Firehose | Flink phân tích real-time; Firehose nạp dữ liệu vào đích | "Analytics in real time" → Flink |
| Quick Sight | OpenSearch Dashboards | Quick Sight BI kinh doanh; Dashboards cho log/search | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "query logs in S3 with SQL, serverless" | Athena | |
| "reduce Athena cost and improve performance" | Parquet/ORC + partition + nén | |
| "query data in RDS and DynamoDB together" | Athena Federated Query | |
| "enterprise data warehouse, BI" | Redshift | |
| "query S3 from Redshift without loading" | Redshift Spectrum | |
| "DR for Redshift in another Region" | Cross-Region snapshot copy | |
| "full-text search on product catalog" | OpenSearch | |
| "Hadoop/Spark cluster" | EMR | |
| "cheapest EMR nodes, can be interrupted" | Spot cho task node | |
| "serverless dashboards for business users" | Quick Sight | |
| "convert CSV to Parquet serverless" | Glue ETL | |
| "row/column-level access to data lake" | Lake Formation | |
| "real-time analytics on Kinesis stream" | Managed Service for Apache Flink | |
| "migrate Kafka to managed" | Amazon MSK | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Team cần truy vấn log ALB trong S3 bằng SQL, không dựng hạ tầng. Chọn gì?
- A. Redshift provisioned
- B. Amazon Athena
- C. EMR
- D. OpenSearch

<details><summary>Đáp án</summary>

**B.**
- A sai: phải load và quản cluster.
- C sai: cluster.
- D sai: phải nạp vào index.
</details>

**Câu 2.** Chi phí Athena tăng vì quét quá nhiều dữ liệu CSV. Cách giảm hiệu quả nhất?
- A. Chuyển sang Parquet, nén, partition theo ngày
- B. Tăng số Lambda
- C. Dùng S3 Standard-IA
- D. Bật versioning

<details><summary>Đáp án</summary>

**A.** Ít byte quét hơn → rẻ và nhanh hơn.
- B sai: không liên quan.
- C sai: phí truy xuất có thể tăng, không giảm byte quét.
- D sai: không liên quan.
</details>

**Câu 3.** Công ty muốn kho dữ liệu cho BI với truy vấn phức tạp, join nhiều bảng lớn, chạy thường xuyên. Chọn gì?
- A. Athena
- B. Amazon Redshift
- C. DynamoDB
- D. ElastiCache

<details><summary>Đáp án</summary>

**B.**
- A sai: phù hợp ad-hoc, không tối ưu cho tải BI liên tục phức tạp.
- C, D sai: không phải OLAP.
</details>

**Câu 4.** (Chọn 2) Cần data lake trên S3 với metadata tập trung và phân quyền theo cột cho các team phân tích dùng Athena. Chọn gì?
- A. Glue Data Catalog (crawler)
- B. Lake Formation
- C. Bucket policy theo prefix
- D. EMR HDFS
- E. ElastiCache

<details><summary>Đáp án</summary>

**A, B.**
- C sai: không phân quyền theo cột.
- D sai: không phải data lake S3.
- E sai: cache.
</details>

**Câu 5.** Pipeline cần phân tích luồng click real-time từ Kinesis Data Streams, tính số click theo cửa sổ 1 phút. Chọn gì?
- A. Kinesis Data Analytics for SQL
- B. Managed Service for Apache Flink
- C. Athena
- D. Glue crawler

<details><summary>Đáp án</summary>

**B.**
- A sai: đã ngừng.
- C sai: truy vấn dữ liệu nghỉ trên S3.
- D sai: tạo metadata.
</details>

**Câu 6.** Cụm EMR chạy job hằng đêm; muốn giảm chi phí mà không mất dữ liệu HDFS. Chọn gì?
- A. Spot cho core node
- B. Spot cho task node, On-Demand/Reserved cho primary và core
- C. Spot cho primary node
- D. Tắt EMR

<details><summary>Đáp án</summary>

**B.** Task node không chứa HDFS, mất cũng không mất dữ liệu.
- A sai: core node chứa HDFS.
- C sai: mất primary là mất cluster.
- D sai: không đáp ứng yêu cầu.
</details>

**Câu 7.** Dữ liệu sản phẩm lưu trong DynamoDB; cần tìm kiếm theo từ khóa trong mô tả. Kiến trúc?
- A. Scan DynamoDB mỗi lần tìm
- B. DynamoDB Streams → Lambda → OpenSearch, tìm kiếm trên OpenSearch
- C. DAX
- D. Athena trên DynamoDB

<details><summary>Đáp án</summary>

**B.**
- A sai: chậm và tốn RCU.
- C sai: cache theo key, không full-text.
- D sai: không phù hợp tìm kiếm thời gian thực.
</details>

**Câu 8.** Công ty đang chạy Kafka tự quản trên EC2, muốn giảm vận hành mà giữ API Kafka. Chọn gì?
- A. Kinesis Data Streams
- B. Amazon MSK
- C. SQS
- D. Amazon MQ

<details><summary>Đáp án</summary>

**B.**
- A sai: API khác, phải sửa code.
- C sai: hàng đợi, không phải Kafka.
- D sai: ActiveMQ/RabbitMQ.
</details>

---

### Nguồn đã đối chiếu
- Athena performance tuning (partition, columnar): https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html
- Athena Federated Query: https://docs.aws.amazon.com/athena/latest/ug/connect-to-a-data-source.html
- Redshift Spectrum: https://docs.aws.amazon.com/redshift/latest/dg/c-using-spectrum.html
- Redshift cross-Region snapshot copy: https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-snapshots.html
- Amazon EMR node types: https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-master-core-task-nodes.html
- Amazon Quick Sight (thuộc Amazon Quick): https://docs.aws.amazon.com/quick/latest/userguide/quick-bi.html
- AWS Glue job bookmarks: https://docs.aws.amazon.com/glue/latest/dg/monitor-continuations.html
- AWS Lake Formation: https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html
- Kinesis Data Analytics for SQL discontinuation: https://docs.aws.amazon.com/kinesisanalytics/latest/dev/discontinuation.html
- Kinesis Data Streams large records (tới 10 MiB): https://docs.aws.amazon.com/streams/latest/dev/large-records.html
- Amazon MSK: https://docs.aws.amazon.com/msk/latest/developerguide/what-is-msk.html
