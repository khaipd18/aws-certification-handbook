# SAA-C03 — RDS, Aurora, RDS Proxy & ElastiCache

> Thuộc **Domain 2: Design Resilient Architectures** và **Domain 3: Design High-Performing Architectures**: database quan hệ managed, read replica vs Multi-AZ, Aurora, backup/restore, bảo mật và cache.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **RDS** = database quan hệ managed (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Db2, Aurora); AWS lo patch, backup, Multi-AZ; **không SSH** được (trừ **RDS Custom** cho Oracle/SQL Server).
- **Read replica**: **async**, tối đa **15** cho mỗi DB, cùng AZ/khác AZ/khác Region → để **scale đọc**. Replica cùng Region không tính phí truyền dữ liệu.
- **Multi-AZ**: standby **sync** ở AZ khác, một DNS, tự failover → để **HA/DR**, standby không phục vụ đọc (trừ Multi-AZ DB cluster).
- **Aurora**: 6 bản sao trên 3 AZ, tự lớn tới **256 TiB**, tối đa **15 replica**, failover < 30 giây, writer endpoint + reader endpoint.
- **Aurora Global Database**: tối đa **10 Region phụ**, độ trễ replication thường < 1 giây, promote Region khác với RTO < 1 phút.
- **Backtrack** (tua lại không cần restore) chỉ có ở **Aurora MySQL**; **cloning** copy-on-write nhanh hơn restore snapshot.
- **RDS Proxy**: pool kết nối (rất hợp Lambda), rút ngắn thời gian failover, bắt buộc IAM/Secrets Manager, chỉ truy cập trong VPC.
- **ElastiCache**: **Valkey / Redis OSS** (replication, Multi-AZ, persistence, backup) vs **Memcached** (sharding đa luồng, không replication); cache theo **lazy loading** hoặc **write-through** + TTL.

---

## 2. Core Concepts

### Amazon RDS
- **Là gì:** DB quan hệ managed: provisioning, patch OS, backup liên tục và PITR, monitoring dashboard, read replica, Multi-AZ, scale dọc, storage gắn EBS.
- **Dùng khi nào:** ứng dụng OLTP cần SQL, không muốn tự quản DB trên EC2.
- **Vì sao quan trọng:** **Storage Auto Scaling** tự tăng dung lượng khi gần đầy (đặt Maximum Storage Threshold), hợp với tải khó đoán.

### Read Replicas
- **Là gì:** bản sao **asynchronous** (eventually consistent), tối đa 15 cho mỗi source; promote thành DB độc lập được.
- **Dùng khi nào:** báo cáo/analytics chạy trên replica để không ảnh hưởng production; chỉ cho **SELECT**.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: slide ghi "tối đa 5 read replica" → nay **15** (MySQL, MariaDB, PostgreSQL...). Replica khác AZ cùng Region không tính phí network; khác Region thì có phí.

### Multi-AZ
- **Là gì:** standby đồng bộ ở AZ khác, chung một DNS name; tự failover khi mất AZ, mạng hoặc instance/storage lỗi.
- **Dùng khi nào:** HA cho production; read replica cũng có thể bật Multi-AZ để làm DR.
- **Vì sao quan trọng:** chuyển Single-AZ → Multi-AZ **không downtime** (AWS chụp snapshot rồi dựng standby đồng bộ). Standby thường không đọc được.

### `[+]` Multi-AZ DB cluster
- **Là gì:** dạng triển khai RDS MySQL/PostgreSQL với 1 writer + **2 standby đọc được** ở 3 AZ.
- **Dùng khi nào:** vừa cần HA vừa muốn standby phục vụ đọc, failover nhanh hơn Multi-AZ instance.
- **Vì sao quan trọng:** tránh khẳng định tuyệt đối "standby Multi-AZ không bao giờ đọc được".

### RDS Custom
- **Là gì:** RDS cho Oracle và SQL Server nhưng cho quyền truy cập OS và DB (SSH, SSM Session Manager) để tùy chỉnh.
- **Dùng khi nào:** cần cài agent/patch đặc thù mà RDS thường không cho.
- **Vì sao quan trọng:** nên tắt automation mode trước khi tùy chỉnh và chụp snapshot.

### Amazon Aurora
- **Là gì:** DB tương thích MySQL/PostgreSQL do AWS xây; 6 bản sao trên 3 AZ (ghi cần 4/6, đọc cần 3/6), tự sửa lỗi, storage tự tăng tới 256 TiB.
- **Dùng khi nào:** cần hiệu năng cao (AWS công bố ~5x MySQL, ~3x PostgreSQL), HA sẵn có, nhiều replica.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Aurora tối đa 128 TB" → nay **256 TiB**. Writer endpoint trỏ primary; reader endpoint cân bằng tải ở mức kết nối cho replica; custom endpoint cho nhóm replica cấu hình khác.

### Aurora advanced
- **Là gì:** **Replica auto scaling**, **Aurora Serverless v2** (scale theo ACU, nay xuống được **0 ACU** khi auto-pause), **Global Database**, **Aurora ML** (gọi SageMaker/Bedrock từ SQL), **Babelfish** (app SQL Server nói chuyện với Aurora PostgreSQL).
- **Dùng khi nào:** tải gián đoạn khó đoán → Serverless v2; DR đa Region → Global Database.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "Global Database tối đa 5 Region phụ" → nay **10**.

### Backup, restore, cloning, Backtrack
- **Là gì:** automated backup 1–35 ngày (RDS đặt 0 để tắt; Aurora không tắt được), PITR; manual snapshot giữ đến khi xóa; restore luôn tạo **DB mới**.
- **Dùng khi nào:** DB dừng vẫn trả tiền storage → dùng lâu không chạy thì snapshot rồi xóa; **Aurora cloning** để tạo staging từ production nhanh; **Backtrack** để tua về thời điểm trước mà không restore.
- **Vì sao quan trọng:** Backtrack **chỉ Aurora MySQL**, không có ở Aurora PostgreSQL. Import từ on-premises: MySQL dùng Percona XtraBackup → S3 → Aurora.

### Bảo mật RDS/Aurora
- **Là gì:** mã hóa at rest bằng KMS (đặt lúc tạo; master không mã hóa thì replica cũng không), in-flight TLS, **IAM database authentication**, SG, audit log gửi CloudWatch Logs.
- **Dùng khi nào:** mã hóa DB chưa mã hóa: snapshot → copy có mã hóa → restore.
- **Vì sao quan trọng:** không SSH vào RDS (trừ Custom); user/quyền bên trong DB vẫn do bạn quản lý.

### Amazon RDS Proxy
- **Là gì:** proxy managed, gom và chia sẻ kết nối, serverless, Multi-AZ; hỗ trợ RDS (MySQL, PostgreSQL, MariaDB, SQL Server) và Aurora.
- **Dùng khi nào:** Lambda hoặc ứng dụng mở/đóng rất nhiều kết nối gây "too many connections"; muốn rút ngắn thời gian failover.
- **Vì sao quan trọng:** bắt buộc IAM authentication + credential lưu ở Secrets Manager; **không bao giờ public**, chỉ trong VPC; không phải sửa code nhiều.

### Amazon ElastiCache
- **Là gì:** cache in-memory managed: **Valkey/Redis OSS** (Multi-AZ auto-failover, read replica, persistence AOF, backup/restore, sorted set) và **Memcached** (nhiều node, sharding, đa luồng, không replication).
- **Dùng khi nào:** giảm tải DB đọc nhiều, lưu **session** để ứng dụng **stateless**, leaderboard bằng sorted set.
- **Vì sao quan trọng:** cần sửa code ứng dụng; bảo mật bằng SG, TLS, Redis AUTH / IAM auth (Valkey, Redis OSS). Backup/snapshot có ở Valkey, Redis OSS và **Memcached serverless**; Memcached node-based thì không.

### Caching strategies
- **Là gì:** **lazy loading** (cache-aside: miss mới đọc DB rồi ghi cache, dữ liệu có thể cũ), **write-through** (ghi DB thì ghi cache, không cũ nhưng cache chứa dữ liệu thừa), **TTL** để dữ liệu tự hết hạn.
- **Dùng khi nào:** dữ liệu ít đổi, đọc nhiều → lazy loading + TTL; cần cache luôn mới → write-through.
- **Vì sao quan trọng:** cache chỉ hữu ích khi dữ liệu đọc nhiều và cấu trúc phù hợp cho truy vấn.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Read replica | Bản sao async để scale đọc | "reporting workload slows production" |
| Multi-AZ | Standby sync, tự failover | "high availability", "AZ failure" |
| Multi-AZ DB cluster | 1 writer + 2 standby đọc được | "readable standbys" |
| Storage Auto Scaling | Tự tăng dung lượng RDS | "unpredictable storage growth" |
| RDS Custom | RDS Oracle/SQL Server có quyền OS | "access underlying OS" |
| Aurora endpoints | Writer, reader, custom | "load balance read connections" |
| Aurora Serverless v2 | Scale theo ACU, về 0 khi pause | "intermittent, unpredictable" |
| Aurora Global Database | Replication đa Region < 1 giây | "cross-Region DR, RPO 1s, RTO 1 min" |
| Aurora cloning | Copy-on-write, tạo DB thử nhanh | "staging from production quickly" |
| Backtrack | Tua Aurora MySQL về thời điểm trước | "undo bad query without restore" |
| RDS Proxy | Pool kết nối managed | "Lambda too many connections" |
| IAM DB authentication | Đăng nhập DB bằng token IAM | "no password in code" |
| Lazy loading | Cache khi miss | "stale data possible" |
| Write-through | Ghi DB đồng thời ghi cache | "cache always up to date" |
| Sorted set | Cấu trúc Redis cho xếp hạng | "real-time leaderboard" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Read Replica | Multi-AZ | Replica: async, đọc được, scale; Multi-AZ: sync, failover, HA | Scale đọc → replica; HA → Multi-AZ |
| RDS | Aurora | Aurora storage 6 bản/3 AZ, 256 TiB, failover nhanh hơn, đắt hơn ~20% | Hiệu năng/HA cao nhất → Aurora |
| Aurora Replica | Aurora Global DB | Replica trong Region; Global DB sang Region khác | Cross-Region DR → Global DB |
| Backtrack | PITR restore | Backtrack tua tại chỗ (Aurora MySQL); PITR tạo DB mới | "Without creating new cluster" → Backtrack |
| Cloning | Snapshot restore | Cloning copy-on-write, nhanh; restore copy toàn bộ | "Fast test copy" → cloning |
| RDS Proxy | ElastiCache | Proxy gom kết nối; ElastiCache cache dữ liệu | "Too many connections" → Proxy |
| Valkey/Redis OSS | Memcached | Redis: replication, persistence, HA; Memcached: sharding đa luồng, đơn giản | Cần HA/backup → Redis/Valkey |
| Lazy loading | Write-through | Lazy: dữ liệu có thể cũ; Write-through: luôn mới, tốn bộ nhớ | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "analytics queries slow down production DB" | Read replica | |
| "survive AZ failure, automatic failover" | RDS Multi-AZ | |
| "convert Single-AZ to Multi-AZ without downtime" | Modify DB → Multi-AZ | |
| "need OS-level access to Oracle on RDS" | RDS Custom | |
| "MySQL compatible, 15 low-latency replicas, auto storage growth" | Aurora | |
| "cross-Region DR, RPO ~1s, RTO < 1 min" | Aurora Global Database | |
| "unpredictable intermittent workload, pay per use" | Aurora Serverless v2 | |
| "rewind DB after bad DELETE, no restore" | Aurora Backtrack (MySQL) | |
| "Lambda exhausts DB connections" | RDS Proxy | |
| "store user sessions, make app stateless" | ElastiCache (hoặc DynamoDB) | |
| "real-time leaderboard" | ElastiCache Redis/Valkey sorted sets | |
| "encrypt existing unencrypted RDS" | Snapshot → copy encrypted → restore | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Team analytics chạy truy vấn nặng làm chậm DB production RDS MySQL. Giải pháp ít sửa code nhất?
- A. Bật Multi-AZ, truy vấn standby
- B. Tạo read replica và trỏ analytics vào replica
- C. Tăng instance type
- D. Dùng ElastiCache

<details><summary>Đáp án</summary>

**B.** Replica tách tải đọc khỏi primary.
- A sai: standby Multi-AZ instance không đọc được.
- C sai: tốn kém, vẫn chung tài nguyên.
- D sai: phải sửa code và không hợp truy vấn analytics đa dạng.
</details>

**Câu 2.** Công ty cần DB quan hệ chịu được mất một AZ với failover tự động, không cần thêm khả năng đọc. Chọn gì?
- A. RDS Multi-AZ
- B. Read replica cùng AZ
- C. Snapshot hằng ngày
- D. ElastiCache

<details><summary>Đáp án</summary>

**A.**
- B sai: cùng AZ, failover không tự động.
- C sai: phải restore thủ công, mất dữ liệu.
- D sai: cache không phải DB chính.
</details>

**Câu 3.** Ứng dụng Lambda bị lỗi "too many connections" với Aurora khi tải tăng. Cách xử lý?
- A. Tăng memory cho Lambda
- B. Đặt RDS Proxy giữa Lambda và Aurora
- C. Thêm Aurora Replica
- D. Bật Backtrack

<details><summary>Đáp án</summary>

**B.** Proxy gom kết nối, Lambda dùng chung pool.
- A sai: không giảm số kết nối.
- C sai: replica chỉ giúp đọc, vẫn nhiều kết nối.
- D sai: dùng để tua dữ liệu.
</details>

**Câu 4.** (Chọn 2) Công ty cần DR đa Region cho DB MySQL với RPO khoảng 1 giây và RTO dưới 1 phút. Chọn gì?
- A. Aurora MySQL
- B. Aurora Global Database
- C. RDS Multi-AZ
- D. Snapshot copy sang Region khác hằng ngày
- E. ElastiCache Global Datastore

<details><summary>Đáp án</summary>

**A, B.** Aurora Global Database replication thường < 1 giây, promote Region phụ < 1 phút.
- C sai: chỉ trong một Region.
- D sai: RPO tính bằng giờ/ngày.
- E sai: là cache, không phải DB chính.
</details>

**Câu 5.** Developer lỡ chạy DELETE thiếu WHERE trên Aurora PostgreSQL. Phát biểu nào đúng?
- A. Dùng Backtrack để tua về trước khi xóa
- B. Dùng PITR để restore thành cluster mới tại thời điểm trước lỗi
- C. Bật Multi-AZ để lấy lại dữ liệu
- D. Dữ liệu mất vĩnh viễn

<details><summary>Đáp án</summary>

**B.** PITR tạo cluster mới tại thời điểm trước lỗi.
- A sai: Backtrack chỉ có cho Aurora MySQL.
- C sai: standby sync cũng đã bị xóa.
- D sai: còn automated backup.
</details>

**Câu 6.** Ứng dụng web dùng nhiều EC2 sau ALB; user bị đăng xuất khi request sang instance khác. Muốn app stateless. Chọn gì?
- A. Lưu session trên EBS
- B. Lưu session trong ElastiCache
- C. Dùng instance store
- D. Bật cross-zone

<details><summary>Đáp án</summary>

**B.** Session ở cache chung, instance nào cũng đọc được.
- A, C sai: ổ gắn một instance.
- D sai: không giải quyết session.
</details>

**Câu 7.** Cần môi trường staging giống production Aurora để test, tạo nhanh và rẻ. Chọn gì?
- A. Restore từ snapshot
- B. Aurora cloning
- C. Tạo read replica rồi promote
- D. Export sang S3

<details><summary>Đáp án</summary>

**B.** Cloning copy-on-write, tạo gần như tức thì, chỉ trả cho phần dữ liệu thay đổi.
- A sai: chậm hơn, copy toàn bộ.
- C sai: chậm hơn và không phải mục đích chính.
- D sai: không tạo được DB chạy.
</details>

**Câu 8.** Cần cache có Multi-AZ auto-failover, persistence và backup. Chọn gì?
- A. ElastiCache Memcached (node-based)
- B. ElastiCache for Valkey hoặc Redis OSS
- C. DAX
- D. CloudFront

<details><summary>Đáp án</summary>

**B.** Valkey/Redis OSS có replication, Multi-AZ, AOF, snapshot.
- A sai: Memcached node-based không replication, không backup.
- C sai: DAX chỉ dành cho DynamoDB.
- D sai: CDN cho nội dung HTTP.
</details>

---

### Nguồn đã đối chiếu
- RDS read replicas (tối đa 15): https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html
- Multi-AZ DB cluster deployments: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html
- Aurora storage (256 TiB): https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html
- Aurora Global Database (tối đa 10 secondary Region): https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html
- Aurora Serverless v2 scaling to zero ACUs: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html
- Aurora Backtrack (chỉ Aurora MySQL): https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html
- RDS Proxy: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html
- ElastiCache backups (Valkey, Redis OSS, Memcached serverless): https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/backups.html
