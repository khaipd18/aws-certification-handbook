# AIF-C01 — Amazon SageMaker AI

> Thuộc **Domain 1, Task 1.3** (AI/ML lifecycle, MLOps, chọn dịch vụ AWS cho từng giai đoạn pipeline) và một phần **Domain 4** (Clarify: bias, explainability). Dạng câu hỏi chủ đạo: **giai đoạn X của pipeline thì dùng tính năng SageMaker nào**.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời. Mốc kiến thức: 10/2026.

---

## 1. TL;DR

- **SageMaker AI** = dịch vụ **end-to-end, fully managed** để **tự build, train, deploy và monitor** model ML. Khác AI services (pre-trained) và Bedrock (dùng FM có sẵn).
- Pipeline → tính năng: chuẩn bị data (**Data Wrangler**), lưu feature (**Feature Store**), gắn nhãn (**Ground Truth**), train/tune (**built-in algorithms**, **AMT**), đánh giá/bias (**Clarify**), deploy (**4 inference options**), giám sát (**Model Monitor**), quản lý version (**Model Registry**), tự động hóa (**Pipelines**).
- **4 kiểu deploy:** **Real-time** (25 MB, 60s), **Serverless** (4 MB, 60s, chịu cold start), **Asynchronous** (1 GB, tới 1 giờ, request/response qua S3), **Batch Transform** (cả dataset, 100 MB mỗi mini-batch).
- **Clarify** = đánh giá FM, **giải thích** dự đoán (explainability), **phát hiện bias** trong data và model.
- **Governance:** **Model Cards** (tài liệu model), **Model Dashboard** (xem mọi model một chỗ), **Role Manager** (quyền theo persona), **Model Registry** (version, approval).
- **No-code / low-code:** **Canvas** (build model bằng giao diện, AutoML, ready-to-use models); **JumpStart** (hub FM và model pre-trained, deploy lên SageMaker với toàn quyền cấu hình).
- ⚠️ Data Wrangler và Autopilot không còn là app riêng trong Studio Classic → nay nằm **bên trong SageMaker Canvas**.

---

## 2. Core Concepts

### SageMaker AI – End-to-End ML Service
- **Là gì:** dịch vụ fully managed cho developer/data scientist làm toàn bộ vòng đời ML ở **một nơi**: Collect & prepare → Build & train → Deploy & monitor.
- **Dùng khi nào:** cần **model riêng** từ data riêng (vd: dự đoán điểm thi từ số năm kinh nghiệm IT, kinh nghiệm AWS, số giờ học) mà AI services có sẵn không đáp ứng được.
- **Vì sao quan trọng:** đề có "build, train, deploy custom ML model", "full control" → SageMaker AI.

### Built-in Algorithms
- **Là gì:** thuật toán dựng sẵn: **Supervised** (linear regression/classification, **KNN**), **Unsupervised** (**PCA** giảm số feature, **K-means** gom nhóm, **Anomaly Detection**), Text (NLP, summarization), Image (classification, detection).
- **Dùng khi nào:** dùng thẳng cho bài toán phổ biến, không cần tự viết thuật toán. **DeepAR** dự báo **time series** (dựa trên RNN).
- **Vì sao quan trọng:** đề hỏi "giảm số chiều/feature" → **PCA**; "forecast time series" → **DeepAR**.

### Automatic Model Tuning (AMT)
- **Là gì:** bạn định nghĩa **objective metric**; AMT tự chọn khoảng hyperparameter, chiến lược tìm kiếm, thời gian chạy tối đa, điều kiện **early stop**.
- **Dùng khi nào:** cần tìm hyperparameter tốt nhất mà không thử thủ công từng cấu hình.
- **Vì sao quan trọng:** tiết kiệm thời gian và tiền, tránh lãng phí vào cấu hình kém.

### Model Deployment & Inference
- **Là gì:** deploy **một click**, auto scaling, không quản server (khác tự host): **Real-time** (từng request, latency thấp), **Serverless** (traffic thưa, chịu cold start; chọn RAM, scaling có sẵn).
- **Dùng khi nào:** **Asynchronous** (payload lớn tới 1 GB, xử lý lâu, gần real-time; request/response nằm trên S3, có hàng đợi); **Batch Transform** (dự đoán cho **cả dataset**, kết quả ra S3).
- **Vì sao quan trọng:** `[+]` async endpoint có thể **scale về 0** instance khi không có request; **SageMaker Inference Recommender** gợi ý instance và cấu hình endpoint tối ưu cho chi phí và hiệu năng.

### SageMaker Studio
- **Là gì:** giao diện web **hợp nhất** cho toàn bộ quá trình phát triển ML: notebook/IDE, cộng tác nhóm, tune và debug, deploy, workflow tự động.
- **Dùng khi nào:** team data science cần một môi trường chung thay vì nhiều công cụ rời rạc.
- **Vì sao quan trọng:** nhiều tính năng (Clarify FM evaluation, JumpStart, Canvas, MLflow) được truy cập **từ trong Studio**.

### Data Wrangler
- ⚠️ Lỗi thời: Data Wrangler như một app trong **Studio Classic** → nay dùng **Data Wrangler bên trong SageMaker Canvas** (có công cụ chuyển flow cũ sang).
- **Là gì:** chuẩn bị data **tabular và ảnh** cho ML: chọn, làm sạch, khám phá, trực quan hóa, biến đổi, **feature engineering** trong một giao diện; hỗ trợ SQL, có Data Quality report, "Quick Model" để xem nhanh độ quan trọng của feature.
- **Dùng khi nào:** data scientist muốn chuẩn bị data bằng point-and-click, rồi export data flow hoặc publish feature sang Feature Store.

### ML Features & Feature Store
- **Là gì:** **feature** = input của model, dùng cả lúc train lẫn lúc inference (vd: rating bài hát, thời lượng nghe, nhân khẩu học người nghe).
- **Dùng khi nào:** **Feature Store** lưu feature tập trung, ingest từ nhiều nguồn, định nghĩa phép biến đổi ngay trong store, nhận feature publish từ Data Wrangler; tìm feature được trong Studio.
- **Vì sao quan trọng:** feature chất lượng cao được **tái sử dụng** giữa các team và model, đảm bảo nhất quán giữa train và inference.

### SageMaker Clarify – Model Evaluation, Explainability, Bias
- **Là gì:** **đánh giá FM** (so sánh Model A với Model B; cả yếu tố "con người" như thân thiện, hài hước); dùng dataset có sẵn hoặc của bạn, metric có sẵn; human evaluation bằng work team (nhân viên của bạn). Slide nhắc "AWS-managed team" ❓ cần kiểm chứng.
- **Dùng khi nào:** **Explainability**: giải thích **vì sao** model dự đoán như vậy (vd: vì sao hồ sơ vay bị từ chối), cả trước deploy lẫn khi debug sau deploy.
- **Vì sao quan trọng:** **Detect bias**: đo bias trong dataset và model bằng metric thống kê, tự phát hiện khi bạn chỉ định feature cần xét (vd: giới tính, độ tuổi).

### Các loại bias trong data
- **Là gì:** **Sampling bias** (data không đại diện đủ cho toàn bộ quần thể); **Measurement bias** (công cụ đo/thu thập bị lỗi hoặc lệch); **Observer bias** (người thu thập/diễn giải mang định kiến cá nhân); **Confirmation bias** (chỉ tin thông tin khớp với định kiến, chủ yếu ở quyết định của con người).
- **Dùng khi nào:** vd thuật toán chỉ gắn cờ người thuộc một số nhóm sắc tộc → khả năng cao là **sampling bias** → cần **data augmentation** cho các lớp bị thiếu.
- **Vì sao quan trọng:** đề mô tả triệu chứng và hỏi đó là loại bias nào.

### SageMaker Ground Truth
- **Là gì:** đưa **con người** vào ML: **gắn nhãn data** (annotation), tạo data, đánh giá và review model, và **RLHF** (người đánh giá, kết quả đưa vào reward function để align model với sở thích của con người).
- **Dùng khi nào:** reviewer có thể là **Mechanical Turk**, **nhân viên của bạn**, hoặc **vendor bên thứ ba**.
- **Vì sao quan trọng:** **Ground Truth Plus** = dịch vụ gắn nhãn do AWS quản lý đội ngũ giúp bạn ❓ cần kiểm chứng (không tìm thấy trang hiện hành trong docs).

### ML Governance: Model Cards, Model Dashboard, Role Manager
- **Là gì:** **Model Cards** = tài liệu bất biến về model: mục đích sử dụng, **risk rating**, chi tiết training, kết quả đánh giá.
- **Dùng khi nào:** **Model Dashboard** = cổng xem, tìm kiếm mọi model trong account, model nào đang deploy, model nào **vi phạm ngưỡng** về data quality, model quality, bias, explainability.
- **Vì sao quan trọng:** **Role Manager** = tạo IAM role **least-privilege theo persona** (data scientist, MLOps engineer...).

### Model Monitor
- **Là gì:** giám sát chất lượng model **trong production**, liên tục hoặc theo lịch; **cảnh báo** khi có sai lệch để sửa data và train lại.
- **Dùng khi nào:** vd model cho vay bắt đầu duyệt cho người không đủ điểm tín dụng (**drift**).
- **Vì sao quan trọng:** `[+]` theo dõi 4 loại drift: **data quality**, **model quality**, **bias drift**, **feature attribution drift**.

### Model Registry
- **Là gì:** kho **tập trung** để theo dõi, quản lý và **version** model; gắn metadata cho model.
- **Dùng khi nào:** quản lý **trạng thái phê duyệt** (approved/rejected), tự động hóa deploy, chia sẻ model.
- **Vì sao quan trọng:** tích hợp với MLflow: model ghi bằng MLflow có thể tự đăng ký vào SageMaker Model Registry.

### SageMaker Pipelines
- **Là gì:** workflow tự động hóa build → train → deploy model; là **CI/CD cho ML**; build, train, test, deploy hàng trăm model tự động, không bước thủ công, lặp lại được.
- **Dùng khi nào:** step types: **Processing** (xử lý data, feature engineering), **Training**, **Tuning** (HPO), **AutoML**, **Model** (tạo/đăng ký model), **ClarifyCheck** (drift bias và explainability), **QualityCheck** (drift data quality và model quality).
- **Vì sao quan trọng:** đề có "automate ML workflow", "MLOps", "CI/CD for ML" → Pipelines.

### SageMaker JumpStart
- **Là gì:** **ML Hub**: tìm FM, model computer vision, NLP pre-trained từ Hugging Face, Meta, Stability AI, Databricks...; **browse → experiment → customize (với data của bạn) → deploy**.
- **Dùng khi nào:** model deploy **trực tiếp trên SageMaker**, bạn **toàn quyền** chọn instance và cấu hình deploy. Có thêm **ML Solutions**: template end-to-end (qua CloudFormation) cho demand forecasting, dự đoán tín dụng, fraud detection, computer vision.
- **Vì sao quan trọng:** `[+]` khác **Bedrock**: Bedrock là serverless, gọi FM qua API, không quản hạ tầng; JumpStart deploy FM lên **endpoint của bạn**, nhiều kiểm soát hơn nhưng tự quản instance.

### SageMaker Canvas
- **Là gì:** build model ML bằng **giao diện trực quan, không code**; AutoML (trước gọi là SageMaker Autopilot, nay UI nằm trong Canvas); dùng Data Wrangler để chuẩn bị data.
- **Dùng khi nào:** business analyst muốn dự đoán (vd: churn, doanh số) mà không biết code; dùng **ready-to-use models** từ Rekognition, Comprehend, Textract, Bedrock, JumpStart.
- **Vì sao quan trọng:** đề có "no-code", "business analyst", "visual interface" → Canvas.

### MLflow on SageMaker AI
- **Là gì:** **MLflow** = công cụ open-source quản lý vòng đời ML; SageMaker AI cung cấp **MLflow fully managed** để track experiment/run, so sánh model, và (với GenAI) **tracing** từng bước của ứng dụng.
- **Dùng khi nào:** team đã quen MLflow, muốn track experiment trên AWS; tích hợp Studio và Model Registry.
- **Vì sao quan trọng:** slide gọi là "MLflow Tracking Servers"; docs hiện nhấn mạnh "managed MLflow" ❓ cần kiểm chứng tên gọi hạ tầng hiện hành.

### Extra Features
- **Là gì:** **Network Isolation mode**: container của job SageMaker chạy **không có outbound Internet**, kể cả không truy cập được S3.
- **Dùng khi nào:** yêu cầu bảo mật nghiêm ngặt, không cho job gửi data ra ngoài.
- **Vì sao quan trọng:** **DeepAR**: thuật toán dự báo time series dựa trên RNN.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Amazon SageMaker AI | Nền tảng end-to-end build/train/deploy ML | "custom ML model", "full control over training" |
| Built-in algorithms | Thuật toán dựng sẵn (linear, KNN, PCA, K-means...) | "use a provided algorithm" |
| PCA | Principal Component Analysis: giảm số feature | "reduce dimensionality / number of features" |
| DeepAR | Dự báo time series bằng RNN | "forecast time series", "demand forecasting" |
| Automatic Model Tuning (AMT) | Tự tìm hyperparameter tối ưu theo objective metric | "tune hyperparameters automatically" |
| Real-time Inference | Endpoint latency thấp, payload ≤ 25 MB, ≤ 60s | "web/mobile app needs instant prediction" |
| Serverless Inference | Không quản instance, payload ≤ 4 MB, ≤ 60s, có cold start | "intermittent traffic", "idle periods" |
| Asynchronous Inference | Hàng đợi, payload ≤ 1 GB, xử lý ≤ 1 giờ, I/O qua S3 | "large payload", "long processing", "near real-time" |
| Batch Transform | Dự đoán cả dataset, 100 MB mỗi mini-batch | "entire dataset", "offline", "no persistent endpoint" |
| `[+]` Inference Recommender | Gợi ý instance/cấu hình endpoint tối ưu | "choose best instance type for endpoint" |
| SageMaker Studio | Giao diện web hợp nhất cho ML | "unified ML IDE", "team collaboration" |
| Data Wrangler | Chuẩn bị, làm sạch, trực quan hóa data, feature engineering (nay trong Canvas) | "prepare and clean data visually" |
| Feature | Input của model, dùng khi train và inference | "input variables" |
| Feature Store | Kho feature tập trung, tái sử dụng | "share and reuse features across teams" |
| SageMaker Clarify | Đánh giá FM, explainability, detect bias | "explain prediction", "detect bias", "compare models" |
| Explainability | Giải thích vì sao model ra dự đoán | "why was this loan rejected?" |
| Sampling bias | Data không đại diện cho toàn bộ quần thể | "underrepresented groups in training data" |
| Measurement bias | Công cụ đo/thu thập data bị lệch | "faulty sensors", "skewed measurement" |
| Observer bias | Định kiến của người thu thập/diễn giải data | "annotator's personal bias" |
| Confirmation bias | Ưu tiên thông tin khớp với định kiến | "favor information confirming beliefs" |
| SageMaker Ground Truth | Gắn nhãn data, human review, RLHF | "label data", "human feedback", "RLHF" |
| RLHF | Reinforcement Learning from Human Feedback | "align model with human preferences" |
| Model Cards | Tài liệu model: intended use, risk rating, training | "document model for audit / compliance" |
| Model Dashboard | Xem mọi model và vi phạm ngưỡng ở một nơi | "central view of all models" |
| Role Manager | IAM role least-privilege theo persona ML | "persona-based permissions for data scientists" |
| Model Monitor | Giám sát drift của model trong production | "detect drift", "alert when quality degrades" |
| `[+]` Data / model quality drift | Data đầu vào hoặc chất lượng dự đoán thay đổi theo thời gian | "production data differs from training data" |
| `[+]` Bias drift / Feature attribution drift | Bias hoặc mức quan trọng của feature thay đổi trong production | "bias increases over time" |
| Model Registry | Kho version model, trạng thái approval | "version and approve models before deployment" |
| SageMaker Pipelines | CI/CD workflow cho ML | "automate build-train-deploy", "MLOps" |
| ClarifyCheck / QualityCheck step | Bước kiểm tra drift bias/explainability / data & model quality trong Pipelines | "check drift in pipeline" |
| SageMaker JumpStart | Hub FM và model pre-trained, deploy lên SageMaker | "pre-trained models from Hugging Face", "deploy FM with control" |
| JumpStart ML Solutions | Template end-to-end cho bài toán phổ biến | "pre-built solution for fraud detection / forecasting" |
| SageMaker Canvas | Build model không code, AutoML | "no-code ML", "business analysts" |
| AutoML (Autopilot) | Tự động chọn thuật toán, train, tune | "automatically build best model" |
| MLflow on SageMaker | MLflow managed: track experiment, tracing | "track experiments", "MLflow" |
| Network Isolation mode | Container job không có outbound Internet | "no internet access for training job" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| SageMaker AI | Amazon Bedrock | SageMaker: **tự build/train/deploy** model bất kỳ, nhiều kiểm soát; Bedrock: **dùng FM có sẵn** qua API, serverless | "Train custom model on our data" → SageMaker; "GenAI with least ops" → Bedrock |
| SageMaker AI | AI services (Comprehend, Rekognition...) | AI services pre-trained, gọi API; SageMaker cần kiến thức ML | "No ML expertise" → AI services |
| `[+]` JumpStart | Bedrock | Cả hai cho dùng FM; JumpStart deploy lên **endpoint của bạn** (chọn instance); Bedrock serverless | "Full control of deployment / instance" → JumpStart |
| JumpStart | Canvas | JumpStart cho người biết ML (hub model, customize); Canvas cho **người không biết code** | "Business analyst, no code" → Canvas |
| Real-time | Serverless | Real-time: traffic **đều**, latency thấp ổn định, 25 MB; Serverless: traffic **thưa**, chịu cold start, 4 MB | "Idle between spikes" → Serverless |
| Asynchronous | Batch Transform | Async: **từng request** lớn/lâu, xếp hàng, gần real-time; Batch: **cả dataset một lần**, không cần endpoint thường trực | "1 GB video per request" → Async; "score whole dataset nightly" → Batch |
| Data Wrangler | Feature Store | Data Wrangler **chuẩn bị/biến đổi** data; Feature Store **lưu và chia sẻ** feature | "Reuse features across teams" → Feature Store |
| Data Wrangler | Ground Truth | Data Wrangler làm sạch/biến đổi data đã có; Ground Truth **tạo nhãn** cho data | "Data has no labels" → Ground Truth |
| Ground Truth | Augmented AI (A2I) | Ground Truth: gắn nhãn data **trước train** (và RLHF); A2I: người duyệt **dự đoán** trong production | Before training → Ground Truth |
| Clarify | Model Monitor | Clarify: **phân tích** bias và explainability (trước và sau deploy); Model Monitor: **giám sát liên tục** drift trong production và cảnh báo | "Why did model predict X?" → Clarify; "alert when quality degrades" → Model Monitor |
| Clarify (explainability) | Model Cards | Clarify **giải thích dự đoán**; Model Cards **ghi tài liệu** về model | "Document intended use and risk" → Model Cards |
| Model Cards | Model Dashboard | Cards: tài liệu **từng model**; Dashboard: **tổng quan mọi model** và vi phạm ngưỡng | "Single view of all models" → Dashboard |
| Model Registry | Model Dashboard | Registry: **version + approval** để deploy; Dashboard: **quan sát** model và cảnh báo | "Approve model version before deploy" → Registry |
| Model Registry | Feature Store | Registry lưu **model**; Feature Store lưu **feature** | |
| Role Manager | IAM thông thường | Role Manager tạo IAM role **theo persona ML**, least-privilege | "Persona-based permissions for ML users" → Role Manager |
| Pipelines | AMT | Pipelines tự động hóa **cả workflow**; AMT chỉ **tune hyperparameter** (có thể là một step trong Pipelines) | "CI/CD for ML" → Pipelines |
| Sampling bias | Measurement bias | Sampling: **thiếu đại diện** của một nhóm; Measurement: **công cụ đo** bị lệch | "Only flags one ethnic group" → sampling |
| Observer bias | Confirmation bias | Observer: người thu thập/diễn giải data mang định kiến; Confirmation: chỉ tìm thông tin khớp niềm tin sẵn có | |
| PCA | K-means | PCA **giảm số feature**; K-means **gom nhóm** data | "Reduce dimensions" → PCA |
| Network Isolation | VPC thông thường | Network Isolation: container **không có outbound** nào, kể cả S3 | "No internet access at all for training container" |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "build, train, deploy custom ML model" | SageMaker AI | |
| "reduce number of features" | PCA (built-in) | |
| "forecast time series" | DeepAR | |
| "automatically find best hyperparameters" | SageMaker AMT | |
| "web app needs low-latency predictions, steady traffic" | Real-time Inference | ≤ 25 MB |
| "unpredictable / intermittent traffic, pay only when used" | Serverless Inference | ≤ 4 MB, cold start |
| "large payload (up to 1 GB), long processing, near real-time" | Asynchronous Inference | |
| "predictions for entire dataset, offline" | Batch Transform | |
| "recommend instance type for endpoint" | `[+]` Inference Recommender | |
| "prepare, clean, visualize data, feature engineering (no code)" | Data Wrangler (trong Canvas) | |
| "store and reuse features across teams" | Feature Store | |
| "label training data with humans" | Ground Truth | |
| "incorporate human feedback (RLHF)" | Ground Truth | |
| "explain why model made a prediction" | Clarify (explainability) | |
| "detect bias in dataset or model" | Clarify (bias detection) | |
| "compare FMs, evaluate friendliness / humor" | Clarify (FM evaluation) | |
| "document intended use, risk rating" | Model Cards | |
| "single view of all deployed models and threshold violations" | Model Dashboard | |
| "least-privilege permissions by ML persona" | Role Manager | |
| "monitor model in production, alert on drift" | Model Monitor | |
| "version models, approval workflow" | Model Registry | |
| "CI/CD for ML", "automate build-train-deploy" | SageMaker Pipelines | |
| "pre-trained models from Hugging Face, deploy with full control" | SageMaker JumpStart | |
| "pre-built end-to-end ML solution template" | JumpStart ML Solutions | |
| "business analyst, no code, build model" | SageMaker Canvas | |
| "track ML experiments with MLflow" | MLflow on SageMaker AI | |
| "training container must have no internet access" | Network Isolation mode | |
| "model only flags certain ethnic groups" | Sampling bias → data augmentation | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một ứng dụng xử lý video y tế gửi file **khoảng 800 MB** mỗi request, mỗi request cần **10–20 phút** để model xử lý, và user chấp nhận nhận kết quả sau. Kiểu deploy SageMaker nào phù hợp?
- A. Real-time Inference
- B. Serverless Inference
- C. Asynchronous Inference
- D. Batch Transform

<details><summary>Đáp án</summary>

**C.** Async hỗ trợ payload tới 1 GB, xử lý tới 1 giờ, request/response qua S3 và có hàng đợi.
- A sai: tối đa 25 MB và 60 giây.
- B sai: tối đa 4 MB và 60 giây.
- D sai: dùng cho cả dataset có sẵn, không phải từng request đến liên tục.
</details>

**Câu 2.** Một chatbot nội bộ chỉ được dùng nhiều vào cuối tháng, các ngày còn lại gần như không có traffic. Team muốn **không trả tiền cho lúc idle** và chấp nhận đôi khi chậm vì cold start. Nên chọn gì?
- A. Real-time Inference với instance lớn
- B. Serverless Inference
- C. Batch Transform
- D. Provisioned Throughput trên Bedrock

<details><summary>Đáp án</summary>

**B.** Serverless tự scale theo traffic thưa, chỉ trả tiền khi xử lý, chịu được cold start.
- A sai: trả tiền cả lúc idle.
- C sai: không phục vụ request tương tác.
- D sai: Provisioned Throughput là cam kết capacity trên Bedrock, không phải SageMaker và không hợp traffic thưa.
</details>

**Câu 3.** Một ngân hàng bị cơ quan quản lý yêu cầu giải thích **vì sao** model từ chối hồ sơ vay của một khách hàng cụ thể. Dùng tính năng nào?
- A. SageMaker Model Monitor
- B. SageMaker Clarify
- C. SageMaker Feature Store
- D. SageMaker Ground Truth

<details><summary>Đáp án</summary>

**B.** Clarify cung cấp explainability cho từng dự đoán và cho model nói chung.
- A sai: Model Monitor giám sát drift, không giải thích dự đoán.
- C sai: Feature Store chỉ lưu feature.
- D sai: Ground Truth dùng để gắn nhãn.
</details>

**Câu 4.** (Chọn 2) Một team MLOps cần: (1) lưu **các phiên bản model** cùng trạng thái phê duyệt trước khi deploy, và (2) **tự động hóa** quy trình xử lý data → train → đánh giá → đăng ký model mỗi khi có data mới. Dùng những gì?
- A. SageMaker Model Registry
- B. SageMaker Pipelines
- C. SageMaker Canvas
- D. SageMaker Model Cards
- E. Amazon Personalize

<details><summary>Đáp án</summary>

**A, B.** Model Registry quản lý version và approval; Pipelines là CI/CD tự động hóa workflow ML.
- C sai: Canvas là công cụ no-code.
- D sai: Model Cards là tài liệu model, không quản lý version hay approval.
- E sai: Personalize là dịch vụ gợi ý.
</details>

**Câu 5.** Sau 6 tháng chạy production, model dự đoán churn bắt đầu kém dần vì hành vi khách hàng thay đổi. Team muốn được **cảnh báo tự động** khi chất lượng model hoặc phân phối data đầu vào lệch so với baseline. Dùng gì?
- A. SageMaker Model Monitor
- B. SageMaker Clarify explainability
- C. SageMaker JumpStart
- D. SageMaker Data Wrangler

<details><summary>Đáp án</summary>

**A.** Model Monitor giám sát liên tục hoặc theo lịch và cảnh báo khi có drift.
- B sai: explainability giải thích dự đoán, không tự giám sát và cảnh báo.
- C sai: hub model.
- D sai: chuẩn bị data.
</details>

**Câu 6.** Một business analyst không biết lập trình muốn dự đoán khách hàng nào sẽ hủy dịch vụ từ file CSV dữ liệu khách hàng. Cách phù hợp nhất?
- A. Viết notebook dùng built-in XGBoost trong SageMaker Studio
- B. SageMaker Canvas
- C. SageMaker Pipelines
- D. Amazon Bedrock Agents

<details><summary>Đáp án</summary>

**B.** Canvas cho phép build model bằng giao diện trực quan, AutoML, không code.
- A sai: cần biết code.
- C sai: Pipelines tự động hóa workflow cho MLOps.
- D sai: không phải công cụ build model dự đoán (và Bedrock Agents không nhận khách mới).
</details>

**Câu 7.** (Chọn 2) Một công ty thấy model tuyển dụng của họ gần như chỉ đề xuất ứng viên nam. Điều tra cho thấy 90% data training là hồ sơ nam giới. Phát biểu nào đúng?
- A. Đây là sampling bias
- B. Đây là confirmation bias của model
- C. Nên bổ sung data / data augmentation cho nhóm bị thiếu và kiểm tra lại bằng SageMaker Clarify
- D. Nên tăng learning rate để model học nhanh hơn
- E. Đây là measurement bias do cảm biến lỗi

<details><summary>Đáp án</summary>

**A, C.** Data không đại diện đủ cho nhóm nữ là sampling bias; khắc phục bằng bổ sung hoặc augmentation data và đo lại bias bằng Clarify.
- B sai: confirmation bias chủ yếu là bias trong quyết định của con người.
- D sai: không liên quan đến bias.
- E sai: không có công cụ đo nào bị lỗi.
</details>

**Câu 8.** Một team data science muốn dùng một FM open-source từ Hugging Face, fine-tune trên data riêng, và **tự chọn loại instance GPU** cùng cấu hình endpoint khi deploy. Nên dùng gì?
- A. Amazon Bedrock on-demand
- B. SageMaker JumpStart
- C. Amazon Quick
- D. PartyRock

<details><summary>Đáp án</summary>

**B.** JumpStart là hub FM/model pre-trained, cho customize và deploy lên SageMaker với toàn quyền cấu hình.
- A sai: Bedrock là serverless, không cho chọn instance.
- C sai: Quick là trợ lý cho nhân viên.
- D sai: PartyRock là playground học GenAI.
</details>

---

### Nguồn đã đối chiếu
- AIF-C01 exam guide, Domain 1 (Task 1.3 MLOps, dịch vụ theo giai đoạn pipeline): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain1.html
- SageMaker AI hosting FAQs (real-time 25 MB/60s, async 1 GB/60 phút, batch 100 MB/3600s, scale to zero, Inference Recommender): https://docs.aws.amazon.com/sagemaker/latest/dg/hosting-faqs.html
- Serverless endpoint payload 4 MB, timeout 1 phút: https://docs.aws.amazon.com/sagemaker/latest/dg/serverless-endpoints-invoke.html
- Model deployment options: https://docs.aws.amazon.com/sagemaker/latest/dg/how-it-works-deployment.html
- AMT tuning strategies: https://docs.aws.amazon.com/sagemaker/latest/dg/automatic-model-tuning-how-it-works.html
- Data Wrangler trong SageMaker Canvas (migrate từ Studio Classic): https://docs.aws.amazon.com/sagemaker/latest/dg/studio-updated-migrate-ui.html
- Autopilot UI tích hợp vào Canvas: https://docs.aws.amazon.com/sagemaker/latest/dg/use-auto-ml.html
- Clarify foundation model evaluations: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-foundation-model-evaluate-whatis.html
- Clarify human evaluation (work team): https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-foundation-model-evaluate-human.html
- Ground Truth workforce (MTurk, private, vendor): https://docs.aws.amazon.com/sagemaker/latest/dg/sms.html
- Model governance (Role Manager, Model Cards, Model Dashboard): https://docs.aws.amazon.com/sagemaker/latest/dg/governance.html
- Model Monitor FAQs (drift types): https://docs.aws.amazon.com/sagemaker/latest/dg/model-monitor-faqs.html
- MLflow & Model Registry: https://docs.aws.amazon.com/sagemaker/latest/dg/mlflow.html, https://docs.aws.amazon.com/sagemaker/latest/dg/mlflow-track-experiments-model-registration.html
- JumpStart solution templates: https://docs.aws.amazon.com/sagemaker/latest/dg/jumpstart-solutions.html
- SageMaker AI features (Canvas, Studio, A2I, Autopilot...): https://docs.aws.amazon.com/sagemaker/latest/dg/whatis-features.html
