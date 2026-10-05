# AIF-C01 — Kiến thức tổng hợp từ đề thi thử

> Gộp từ **7 đề** (Đ1–Đ4: bộ "Practice Test #1–#4 Full-Exam"; Đ5–Đ7: Cloud Exam Pro "Practice Test 01–03"). File chỉ ghi **kiến thức** bóc ra từ đáp án đúng và sai, không chép lại câu hỏi.
> Khi thêm đề mới: chèn vào đúng mục, cập nhật bảng tra nhanh và 3 phụ lục. Không tạo phần riêng cho từng đề.
> Ký hiệu: `⚠️` = đáp án/dịch vụ trong đề đã lỗi thời so với AWS 10/2026 · `❓` = đáp án đề đáng ngờ hoặc các đề mâu thuẫn nhau · `[Đx]` = đề có liên quan.

---

## 0. Đọc trước: đáp án đề lệch so với AWS hiện hành

Đề thi thử viết từ trước nên một số đáp án đã cũ. **Khi thi thật**: nếu đáp án "mới" có trong lựa chọn thì chọn đáp án mới; nếu chỉ có đáp án "cũ" thì chọn đáp án cũ.

| Chủ đề | Đề nói | AWS hiện hành (đã tra docs) |
|---|---|---|
| Custom model trên Bedrock | `[Đ7]` **bắt buộc** mua Provisioned Throughput | Chạy được **cả** Provisioned Throughput **lẫn on-demand** (custom model deployment) `[Đ2]` đã nêu đúng. Nếu lựa chọn chỉ có PT thì chọn PT |
| Vector store mặc định của Knowledge Base | `[Đ1]` OpenSearch Serverless là "default" | Quick create có 4 lựa chọn: **OpenSearch Serverless, Aurora PostgreSQL Serverless, Neptune Analytics, S3 Vectors**; AWS nay khuyên dùng **Bedrock Managed Knowledge Base** (datastore do Bedrock quản lý). Đề cũ vẫn đáp OpenSearch Serverless |
| Continued pre-training trên Bedrock | `[Đ1][Đ3][Đ5][Đ7]` là phương pháp customization của Bedrock | Trang customization hiện tại chỉ liệt kê **SFT, Reinforcement fine-tuning, Distillation** ❓. Khái niệm continued pre-training vẫn nằm trong exam guide 3.3, nên vẫn phải thuộc |
| AWS Audit Manager | `[Đ2][Đ6]` là đáp án đúng cho "continuously audit, collect evidence" | ⚠️ **Không nhận khách mới từ 30/04/2026** → thay bằng **AWS Config Conformance Packs**. Đề cũ vẫn đáp Audit Manager |
| Amazon Kendra | `[Đ4]` enterprise search | ⚠️ Không nhận khách mới từ 30/07/2026 → **Bedrock Managed Knowledge Base** |
| Amazon Q Business, Amazon Q Apps | Trợ lý doanh nghiệp | ⚠️ Không nhận khách mới → **Amazon Quick** (Apps in Quick) `[Đ3]` đã dùng Quick |
| Amazon Q in QuickSight | `[Đ4][Đ7]` | ⚠️ QuickSight nay là **Amazon Quick Sight** (một phần của Amazon Quick) |
| Amazon Q Developer IDE / CodeWhisperer | `[Đ1][Đ7]` gợi ý code | ⚠️ CodeWhisperer → Q Developer → nay khuyến nghị **Kiro** (Q Developer IDE plugins hết hỗ trợ 30/04/2027) |
| AWS Chatbot | Bot Slack/Teams | ⚠️ Đổi tên thành **Amazon Q Developer in chat applications** |
| Agents for Amazon Bedrock | `[Đ5][Đ6]` | ⚠️ Nay là **Agents Classic**, không nhận khách mới → **Bedrock AgentCore** |
| Amazon Fraud Detector | `[Đ5][Đ6]` đáp án nhiễu | ⚠️ Không nhận khách mới từ 07/11/2025 → SageMaker AI / AutoGluon |
| Amazon Forecast | Dịch vụ dự báo | ⚠️ Không nhận khách mới từ 29/07/2024 → **SageMaker Canvas** (forecast) |
| Guardrails content filter | `[Đ6]` 4 nhóm Hate/Insults/Sexual/Violence | Hiện có **6**: Hate, Insults, Sexual, Violence, **Misconduct**, **Prompt Attack** |
| "Bedrock không làm được RLHF" | `[Đ2]` | Bedrock nay có **Reinforcement fine-tuning** (reward function bằng Lambda hoặc judge model). RLHF "đúng nghĩa" (reward model học từ xếp hạng của người) vẫn không phải tính năng có sẵn của Bedrock |
| "Bedrock chỉ có on-demand và batch" | `[Đ3]` | Còn có **Provisioned Throughput** và service tiers (Standard/Priority/Flex/Reserved) |
| Quyền truy cập S3 theo team cho Bedrock | `[Đ3]` mỗi team một **service role** riêng · `[Đ5]` một role Bedrock full S3 + IAM role theo team | ❓ Hai đề mâu thuẫn. Theo **least privilege**: chọn **service role riêng cho từng team, chỉ truy cập data của team đó** `[Đ3]`. Đáp án của `[Đ5]` cấp quyền full S3 cho Bedrock nên vi phạm least privilege |
| Few-shot vs one-shot | `[Đ5]` "đưa ví dụ mô tả theo format" → one-shot | ❓ Đề cho **nhiều ví dụ** (descriptions, số nhiều) thì đáp án chuẩn là **few-shot**. Chỉ chọn one-shot khi đề nói rõ **một** ví dụ |
| Đo "financial effect" của chatbot | `[Đ6]` đáp **Average handled time** | ❓ `[Đ1]` coi **cost per interaction** là business metric. Nếu có lựa chọn "cost per customer conversation" thì nó đo tài chính trực tiếp hơn; key của `[Đ6]` gây tranh cãi |
| Metric cho summarization | `[Đ5]` đáp BLEU | Chuẩn là **ROUGE**. Đề đó không có ROUGE trong lựa chọn nên mới chọn BLEU |
| Giảm hallucination | `[Đ6]` đáp **giảm temperature** | Đúng trong phạm vi các lựa chọn của đề; biện pháp mạnh hơn là **RAG + contextual grounding check** |
| Chống prompt injection | `[Đ7]` "adversarial prompting" · `[Đ4]` thêm chỉ dẫn "ignore malicious content" vào prompt | Đáp án tốt nhất trên AWS là **Guardrails prompt attack filter + input tags**; nếu không có thì chọn chỉ dẫn phòng thủ trong prompt / prompt template |

---

## Bảng tra nhanh (các cặp hay bị đem làm bẫy)

### Cải thiện output của FM

| Kỹ thuật | Đổi weights? | Dữ liệu cần | Chi phí / công sức | Khi nào chọn |
|---|---|---|---|---|
| Prompt engineering (zero/few-shot, CoT, negative...) | Không | Không cần (vài ví dụ trong prompt) | **Thấp nhất** | Đổi giọng văn, format, phong cách theo người dùng; "least effort / most cost-effective" |
| RAG (Knowledge Bases) | Không | Tài liệu nội bộ (PDF, S3, wiki...) | Thấp–vừa | Dữ liệu **riêng, thay đổi thường xuyên**, cần trích dẫn nguồn |
| Fine-tuning (SFT / instruction-based) | **Có** (bản copy riêng) | **Labeled** (prompt–completion, Q&A pairs) | Cao | Task cụ thể, giọng thương hiệu cố định, thuật ngữ chuyên ngành |
| Continued pre-training / domain adaptation | **Có** | **Unlabeled**, khối lượng lớn, chuyên ngành | Cao nhất trong nhóm customization | Model cần "hiểu" lĩnh vực (genomics, y khoa...) |
| Distillation | **Có** (student) | Prompts (hoặc invocation logs) | Trả một lần, inference rẻ hơn | Cần model **nhỏ hơn, nhanh hơn, rẻ hơn**, accuracy gần bằng |
| Pre-train FM từ đầu | — | Khổng lồ | Cực đắt; **không làm trên Bedrock** (dùng SageMaker) | Gần như không bao giờ là đáp án "cost-effective" |

Thứ tự độ phức tạp tăng dần: **Prompt engineering → RAG → Fine-tuning** `[Đ3]`.

### Các loại learning

| Loại | Dữ liệu | Ví dụ thuật toán / use case trong đề |
|---|---|---|
| Supervised | Có nhãn | Linear/logistic regression, decision tree, neural network, SVM, **k-NN**; churn, fraud, phân loại ảnh, giá nhà |
| Unsupervised | Không nhãn | **K-means** (clustering, phân khúc khách hàng), **PCA** (dimensionality reduction), association rules (Apriori), anomaly detection |
| Semi-supervised | Ít nhãn + nhiều không nhãn (pseudo-labeling) | Theo trang AWS: **fraud identification, sentiment analysis, document classification** |
| Self-supervised | Không nhãn, model tự tạo nhãn | **Pre-training FM / LLM** (GPT, BERT) |
| Reinforcement learning | Agent + environment + reward | Robot, game, tối ưu hệ thống; chatbot học từ phản hồi tích cực của khách |
| Transfer learning | Model đã train cho task A → task B liên quan | Dùng lại kiến thức giữa các model của nhiều phòng ban |
| Incremental training | Model cũ + dữ liệu mới, không quên kiến thức cũ | Cập nhật một model với data mới |
| Federated learning | Data phân tán, **không gom về một chỗ** | Privacy, compliance khi huấn luyện |
| Active learning | Model chủ động xin người gắn thêm nhãn | Loại trừ khi đề nói "sẽ không gắn nhãn thêm" |

### Overfitting vs Underfitting · Bias vs Variance

| | Overfitting | Underfitting |
|---|---|---|
| Triệu chứng | Train tốt, **test/production kém** | Kém **cả train lẫn test** |
| Bias / Variance | **High variance** | **High bias** |
| Nguyên nhân | Model quá phức tạp, học cả noise; data ít; train quá lâu | Model quá đơn giản; thiếu feature |
| Cách sửa | **Tăng regularization**, thêm data đa dạng, data augmentation, early stopping, cross-validation, pruning, giảm epochs, feature selection, ensembling | Model phức tạp hơn, thêm feature, giảm regularization, train lâu hơn |

Bias và variance **có thể cùng cao** (vd: giảm data train), khi đó model vừa underfit vừa overfit `[Đ3]`.

### Inference parameters

| Tham số | Kiểm soát | Thấp | Cao |
|---|---|---|---|
| **Temperature** | Độ ngẫu nhiên/sáng tạo | Nhất quán, deterministic (0 = ổn định nhất) | Sáng tạo, đa dạng |
| **Top K** | **Số lượng** token ứng viên | Mạch lạc | Đa dạng |
| **Top P** | **% xác suất cộng dồn** của token ứng viên | Mạch lạc | Đa dạng |
| **Response length / Max tokens** | Số token tối đa của output | Ngắn, rẻ hơn | Dài |
| **Stop sequences** | Chuỗi báo dừng sinh | — | — |
| **CFG scale** (Stable Diffusion) | Mức bám sát prompt khi tạo ảnh | Ngẫu nhiên | **Cụ thể, bám prompt** |

Temperature, Top K, Top P **không ảnh hưởng giá và latency**.

### Bedrock vs SageMaker AI vs AI services dựng sẵn

| | AI services (Comprehend, Rekognition...) | Amazon Bedrock | SageMaker AI |
|---|---|---|---|
| Là gì | Model pre-trained, gọi API | FM của nhiều provider qua API, serverless | Tự build/train/deploy model bất kỳ |
| Kiến thức ML | Không cần | Ít | Cần (trừ Canvas) |
| Tùy biến | Thấp (custom labels / classifier) | Prompt, RAG, fine-tune, distillation | Toàn quyền |
| Chọn khi | Task phổ biến, ít công nhất | GenAI, ít vận hành | ML riêng, train từ đầu, kiểm soát hạ tầng |

**JumpStart** khác Bedrock: JumpStart deploy model lên **SageMaker endpoint của bạn** (toàn quyền chọn instance), có ML Solutions dựng sẵn; Bedrock serverless `[Đ4]`.

### Classification metrics

| Metric | Công thức / ý nghĩa | Chọn khi |
|---|---|---|
| Accuracy | (TP + TN) / tổng | Data **cân bằng**, muốn % đúng tổng thể |
| Precision | TP / (TP + FP) | **False positive đắt** (vd: giảm thời gian nhân viên xem nhầm ca không gian lận) |
| Recall | TP / (TP + FN) | **False negative đắt** (bỏ sót ung thư, gian lận) |
| F1 | Trung bình điều hòa P và R | **Data mất cân bằng** (churn, fraud), cần cân bằng |
| Confusion matrix | Bảng TP/FP/FN/TN theo từng lớp | Muốn xem chi tiết lỗi phân loại |
| AUC-ROC | 0 → 1, so model ở nhiều threshold | So sánh classifier |
| MAE / MSE / RMSE / R² | — | **Regression**, không phải classification |

### Hallucination vs Toxicity vs Prompt attacks

| Rủi ro | Bản chất | Thời điểm | Phòng chống |
|---|---|---|---|
| Hallucination | Nghe đúng nhưng **sai/bịa** | Inference | RAG + **contextual grounding check**, Automated Reasoning check, giảm temperature, kiểm chứng |
| Toxicity | Nội dung **xúc phạm, độc hại** | Inference | Guardrails **content filters**, Comprehend toxicity detection, Rekognition moderation (ảnh) |
| Plagiarism | Nộp nội dung AI như của mình | Sử dụng | Công cụ phát hiện nội dung do AI tạo |
| IP infringement | Output giống tác phẩm có bản quyền (nhân vật hoạt hình...) | Sử dụng | Human review trước khi xuất bản |
| **Poisoning** | Cài data độc vào **training / fine-tuning / RAG source** | **Trước** inference | Kiểm soát, version, review dữ liệu |
| **Prompt injection** | Chỉ thị nhúng trong input (kể cả **text ẩn trong tài liệu upload**) để ghi đè chỉ dẫn | Inference | Guardrails prompt attack filter, validate input |
| **Hijacking** | Dạng injection: **đổi mục tiêu** ("bỏ qua chỉ dẫn, giờ bạn là chuyên gia chứng khoán") | Inference | Như trên |
| **Jailbreaking** | Vượt **rào an toàn** để lấy nội dung cấm ("giả vờ bạn không có luật") | Inference | Guardrails prompt attack filter |
| **Prompt leaking** | Lộ **system prompt / prompt trước đó** | Inference | Prompt attack filter (Standard tier) |
| **Exposure** | Lộ **dữ liệu nhạy cảm** đã học khi train (tên, số tài khoản khách khác) | Inference | Xóa/mask PII trước khi train, Guardrails sensitive info filter |

---

## Domain 1 — Fundamentals of AI and ML

### Khái niệm nền
- **AI > ML > Deep Learning > GenAI** (AI rộng nhất). ML là tập con của AI, học từ dữ liệu; AI còn gồm rule-based.
- **Computer vision** = hiểu nội dung ảnh/video để ra quyết định (detect, classify, nhận diện). **Image processing** = xử lý ảnh (lọc nhiễu, tăng chất lượng). Đề hay đảo ngược hai khái niệm này.
- **NLP** = text/speech; **Computer vision** = ảnh/video. Phân loại phản hồi khách hàng thành chủ đề = NLP (text classification).
- **ML algorithm** = công thức/thủ tục toán học; **ML model** = kết quả của algorithm sau khi train trên data.
- **Inference** = model đã train nhận input mới và trả dự đoán; **evaluation** = so sánh output để chọn model phù hợp nhất.
- Model ML có thể **deterministic, probabilistic, hoặc kết hợp** (không gắn với supervised/unsupervised).
- **Deep learning training** = điều chỉnh **weights và biases** của neural network qua nhiều vòng lặp (gradient descent) để giảm lỗi. **Neural network** = các lớp node, học bằng cách chỉnh weights của kết nối.
- **Traditional ML** cần người chọn feature thủ công; **deep learning** tự rút feature từ raw data (cần nhiều data, nhiều compute, không nhất thiết train nhanh hơn).
- **Thách thức lớn nhất khi làm ML**: thu thập và chuẩn bị **dữ liệu chất lượng cao**.
- **Không dùng ML** khi bài toán có đáp án tính được chính xác (xác suất rút lá bích) → ứng dụng **rule-based**.
- Muốn chọn đúng model → **định nghĩa use case hẹp, cụ thể** trước.

### Dữ liệu
- **Labeled** (có output, dùng cho supervised) vs **unlabeled** (unsupervised). **Structured** (bảng hàng–cột) vs **unstructured** (text, ảnh, video).
- **Time series data** → dự báo nhu cầu (SageMaker **DeepAR**, ARIMA).
- Đặc tính dataset: **Diversity** (đủ mọi nhóm nhân khẩu học), **Accuracy** (từng điểm dữ liệu đúng), **Reliability** (ổn định, nhất quán), **Recency bias** (lệch về dữ liệu mới, cần tránh).
- **Feature engineering** = chọn, biến đổi, tạo feature để model tốt hơn (không phải thu thập data, không phải tune hyperparameter, không phải visualize).
  - **Feature extraction**: biến đổi sang không gian mới, giảm chiều (PCA).
  - **Feature selection**: chọn tập con feature quan trọng từ feature có sẵn.
  - Structured data: normalization, xử lý missing values, encoding; unstructured: **tokenization, vectorization**.
- **Train / validation / test**: train để học; **validation** để tune hyperparameter và chọn model (**optional**); **test** để đánh giá cuối khả năng generalize (**không optional**, không dùng để tune).

### Vòng đời ML
- Thứ tự cơ bản: **Data collection → Data preprocessing → Model training → Model evaluation**.
- Well-architected: **Define business goal & frame ML problem → Develop → Deploy → Monitor**.
- **Yêu cầu compliance và pháp lý** được xác định ở giai đoạn **business goal identification** (đầu tiên).
- **EDA** = thống kê tóm tắt, visualize, **correlation matrix** để hiểu data trước khi build model. Khác data preparation (làm sạch, xử lý missing), data augmentation (tạo thêm biến thể), model evaluation.
- **Hyperparameter** (learning rate, epochs, batch size, regularization, dropout, số layer) = đặt trước, điều khiển quá trình train. **Model parameter** (weights, biases) = model tự học.
- Tăng **epochs** để tăng accuracy tới ngưỡng chấp nhận (nhưng cẩn thận overfit).
- **IaC trong MLOps**: triển khai ML workload **nhất quán, lặp lại được, scalable**; không tự chọn instance mạnh nhất, không thay thế hyperparameter tuning.

### Các kiến trúc / model hay gặp
- **CNN** → ảnh (phân loại ảnh đơn); **RNN** → dữ liệu tuần tự (time series, video, speech).
- **Transformer** → **self-attention** + contextual embeddings, xử lý song song (GPT, BERT, Claude, Llama; ChatGPT = Chat Generative Pre-trained Transformer).
- **GAN** → 2 mạng (generator vs discriminator) đối kháng; dùng **tạo synthetic data** giữ đặc tính thống kê của data gốc.
- **VAE** → học latent space. **Diffusion** → thêm nhiễu rồi khử nhiễu từng bước (Stable Diffusion, DALL-E, Adobe Firefly).
- **BERT** → đọc hai chiều, **contextual embeddings**, **điền từ bị thiếu** (masked LM). **Word2Vec** = static embeddings (một từ một vector). PCA, SVD không hiểu ngữ cảnh.
- **WaveNet** → sinh âm thanh (speech synthesis). **ResNet** → ảnh. **XGBoost / SVM** → tabular classification/regression, không sinh dữ liệu.
- **K-means** (unsupervised, gom cụm) ≠ **k-NN** (supervised, phân loại theo láng giềng gần nhất).
- **Object detection** = xác định và định vị nhiều vật thể trong ảnh (nhận diện mọi con vật trong ảnh bẫy camera). NER là kỹ thuật text; face recognition chỉ cho mặt người.

### Model dễ diễn giải
- **Decision tree** → interpretable nhất, thấy được feature nào dẫn tới quyết định. **Logistic/linear regression** → **xem và chỉnh tay được weights** của từng biến.
- **Neural network, deep learning, SVM, ensemble** → "black box".
- Explainability tỷ lệ nghịch với **model complexity**.

### Metrics
- Classification: xem bảng tra nhanh. Business case "nhân viên mất thời gian xem nhầm cảnh báo gian lận" → **Precision**.
- Regression: MAE, MSE, RMSE, R².
- **Runtime efficiency** → **average response time** (latency); không phải accuracy, throughput, precision-recall.
- **Business metrics**: task completion rate, cost per interaction, CSAT, ARPU, ROI, average call duration (AHT), first-call resolution.
- Đo hiệu quả chatbot hỗ trợ nhân viên trong cuộc gọi → **average call duration**.

### Inference (SageMaker AI)

| Kiểu | Đặc điểm | Đề chọn khi |
|---|---|---|
| Real-time | Endpoint thường trực, latency thấp, payload ≤ 25 MB | Cần dự đoán ngay từng request (check-in bệnh nhân, chatbot) |
| Serverless | Không quản hạ tầng, traffic thưa, chịu cold start, scale về 0 | Workload gián đoạn, muốn rẻ |
| Asynchronous | Hàng đợi, payload **≤ 1 GB**, xử lý tới 1 giờ, near real-time | Payload lớn **từng request**, không cần ngay `[Đ1]` (dataset < 1 GB) |
| Batch transform | Cả dataset, không endpoint thường trực | **Nhiều GB/TB**, chạy định kỳ (hàng ngày, cuối tuần) |

- **Real-time = synchronous**, **batch = asynchronous** `[Đ3]`.
- **Edge**: model nhỏ (**SLM**) chạy **trên thiết bị** để latency thấp nhất; gọi LLM/API trung tâm vẫn có network latency.

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (Domain 1)
- "thống kê, visualize, correlation matrix" → **EDA**
- "dự đoán giá trị số" → **Regression**; "2 lớp (fraud / not)" → **Binary classification**; "≥ 3 lớp" → **multi-class**; "mỗi mẫu nhiều nhãn" → **multi-label**
- "nhóm khách hàng, không nhãn, petabytes" → **Unsupervised / K-means**
- "tự cải thiện từ feedback/reward" → **Reinforcement learning**
- "dùng lại kiến thức model khác" → **Transfer learning**
- "dữ liệu nhạy cảm, không gom về" → **Federated learning**
- "train tốt, production kém" → **Overfitting** → tăng regularization / thêm data
- "xem và chỉnh weights" → **Logistic regression**
- "phân loại ảnh sản phẩm" → **CNN**; "video / chuỗi" → **RNN**
- "dữ liệu tổng hợp giữ thống kê gốc" → **GAN**
- "điền từ bị thiếu" → **BERT**
- "few GB mỗi ngày, không cần ngay" → **Batch**; "payload lớn ≤ 1 GB, gần real-time" → **Async**; "traffic thất thường, chịu cold start" → **Serverless**; "ngay lập tức, endpoint thường trực" → **Real-time**

---

## Domain 2 — Fundamentals of Generative AI

### GenAI, FM, LLM
- **GenAI** = tạo nội dung **mới** (text, ảnh, audio, code) từ pattern học được. Không phải dự đoán số, không phải phân loại.
- Use case hợp lệ: tạo ảnh từ text, tóm tắt, viết mô tả, sinh code. Không hợp: dự đoán giá nhà, dự báo chứng khoán, phân loại ảnh y tế (đó là discriminative ML).
- **Generative vs discriminative**: generative học phân phối dữ liệu để **tạo mới** (tạo ảnh động vật chưa từng có); discriminative **phân loại** giữa các lớp.
- **FM** = model lớn, đa năng, nền cho nhiều tác vụ; **LLM** là **một loại FM** chuyên ngôn ngữ (không phải ngược lại).
- FM pre-train bằng **self-supervised learning trên data không nhãn**; **fine-tuning là supervised** (data có nhãn).
- **LLM non-deterministic** (cùng prompt có thể ra output khác) và là generative model.
- LLM dùng cho: sinh text, dịch, tóm tắt, hỏi đáp, **text-to-SQL**, **sinh code** (text generation). Không phải TTS, tạo video, 3D.
- **Multimodal model**: input **và** output đều có thể nhiều loại (text, ảnh, audio, video).
- Ưu điểm GenAI trong đề: **adaptability** (thay logic hard-coded khó mở rộng), tự động tạo nội dung mới. Không đảm bảo bảo mật 100%, không thay thế toàn bộ con người.
- Best practice khi áp dụng GenAI: **guardrails + transparency** (cho người dùng biết họ đang tương tác với AI), giám sát liên tục.

### Token, embedding, context window
- **Token** = đơn vị text model xử lý (từ, subword, dấu câu). **Tokenization** = chia text thành token.
- **Embedding** = **vector số** nhiều chiều biểu diễn ngữ nghĩa; dùng để **so sánh văn bản bằng toán học**, semantic search, RAG. Không phải nén, mã hóa hay lưu trữ.
- **Context window** = số token tối đa model xem xét một lần. Input dài (sách) vượt context window → model thất bại.
- **Chunking** trong RAG → cải thiện **độ liên quan ngữ cảnh** của kết quả truy xuất (không phải để tiết kiệm storage).
- **Text embeddings model** → chuyển dữ liệu riêng thành vector trước khi lưu vào vector DB.
- Chatbot hiểu cả **text + ảnh** với chi phí thấp → **multimodal embedding model** (rẻ hơn multimodal generative model).

### Agentic AI
- **Multi-agent (supervisor pattern)**: một agent điều phối chia việc cho các agent chuyên biệt, mỗi agent ít tool → giảm chọn nhầm tool; người dùng vẫn có một điểm tương tác.
- **Predefined workflow** (thứ tự cố định, gọi FM bên trong từng bước) khi cần **dự đoán được, lặp lại được, dễ audit**; autonomous agent hoặc swarm khi task mở.
- **Model Context Protocol (MCP)** = chuẩn mở để agent khám phá và gọi tool/data; giảm tích hợp từ M×N xuống M+N.
- **Strands Agents** = SDK open-source của AWS, **model-driven** (model tự chọn tool, kể cả tool từ MCP), vài dòng Python, chạy trên AgentCore Runtime.
- **Context engineering** = lắp ráp **đúng thông tin** (profile, lịch sử hội thoại, tài liệu RAG, tool definition) vào mỗi request trong giới hạn context window. Prompt engineering = **hỏi thế nào**; context engineering = **cho model thấy gì**.
- LLM **stateless**: muốn nhớ hội thoại → **đưa lịch sử tin nhắn vào prompt** hoặc dùng **AgentCore Memory** (short-term trong session, long-term giữa các session).
- **ReAct** prompting = reasoning + **acting** (gọi tool/API, vd: kiểm tra tồn kho real-time).

### Bedrock AgentCore (khớp dịch vụ)

| Dịch vụ | Chức năng |
|---|---|
| Runtime | Chạy agent serverless, session isolation |
| Gateway | Biến API / Lambda thành **tool MCP** |
| Memory | Short-term + long-term memory |
| Identity | OAuth token vault, agent hành động **thay mặt user**, không hard-code credential |
| Policy | Cedar policy, gateway **chặn mọi tool call** vi phạm (vd: đơn hàng > $10.000) |
| Evaluations | Đánh giá chất lượng agent, **online evaluation** trên session production |
| Observability | Trace, metric (CloudWatch) |

### Bedrock: chi phí và dữ liệu
- **On-demand** → không cam kết, dùng bao nhiêu trả bấy nhiêu; hợp **usage thấp / khó đoán / đang thử nghiệm**.
- **Provisioned Throughput** → mua capacity theo giờ (cam kết 1 hoặc 6 tháng để rẻ hơn); hợp **lưu lượng ổn định, cao**; giữ capacity. Dùng cho 1 lần gọi/ngày là lãng phí.
- **Batch inference** → rẻ hơn tới **50%** on-demand, khi chấp nhận trễ.
- Chi phí chính = **số token input + output**. Giảm chi phí: rút ngắn prompt, ít few-shot hơn, giới hạn **max output tokens**, yêu cầu trả lời ngắn gọn, **prompt caching** cho phần system prompt lặp lại.
- **Model nhỏ rẻ hơn** model lớn. Temperature/Top P không đổi giá.
- **Spot instances, Reserved Instances** là khái niệm của EC2, **không áp dụng cho Bedrock**.
- Dữ liệu trên Bedrock **không dùng để train base model và không chia sẻ với model provider**; fine-tuning tạo **bản copy private**. Không thể train base FM hay tạo model từ đầu trên Bedrock (dùng SageMaker).

### Công cụ GenAI khác trong đề
- **Kiro** → agentic IDE/CLI, **spec-driven development**: requirements.md (EARS) → design.md → tasks.md.
- **AWS Transform** → agent hiện đại hóa **.NET Framework, mainframe COBOL, VMware** lên AWS.
- **Amazon Quick** → trợ lý agentic cho nhân viên: Research (báo cáo có trích dẫn), Sight (BI), Flows/Automate, Index; không cần build app.

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (Domain 2)
- "tạo mới / generate" → GenAI; "dự đoán / phân loại" → ML truyền thống
- "FM học thế nào" → **self-supervised, data không nhãn**; "fine-tuning" → **supervised, labeled**
- "LLM so với FM" → LLM là **tập con** của FM
- "giới hạn số text model xem cùng lúc" → **context window**; "đơn vị text" → **token**; "vector ngữ nghĩa" → **embedding**
- "nhiều tool, agent chọn nhầm" → **multi-agent supervisor**; "auditor yêu cầu thứ tự cố định" → **predefined workflow**
- "chuẩn kết nối agent với tool" → **MCP**; "SDK open-source viết agent Python" → **Strands Agents**
- "agent nhớ ngày đi, sở thích ghế" → **AgentCore Memory**; "OAuth thay mặt user" → **AgentCore Identity**; "chặn đơn > ngưỡng" → **Policy in AgentCore**
- "chưa biết lượng dùng, không cam kết" → **On-demand**; "steady rate" → **Provisioned Throughput**; "chấp nhận trễ, rẻ nhất" → **Batch**
- "requirements, design, tasks cho từng feature" → **Kiro**; "chuyển .NET/COBOL" → **AWS Transform**

---

## Domain 3 — Applications of Foundation Models

### Prompt engineering
- **4 thành phần prompt tốt**: **Instructions, Context, Input data, Output indicator** (không phải hyperparameters hay parameters).
- **Zero-shot**: không ví dụ. **One-shot**: 1 ví dụ. **Few-shot**: vài ví dụ; với bài toán intent detection, ví dụ phải là cặp **user input + intent đúng**.
- **Chain-of-thought**: "think step by step", lý luận nhiều bước (toán, quy trình phức tạp).
- **Prompt chaining**: chia task lớn thành subtask **gửi tuần tự**, output bước trước làm input bước sau. **Tree of thoughts**: nhiều nhánh suy luận song song. **Least-to-most**: từ bài nhỏ tới bài lớn.
- **Negative prompting**: chỉ rõ cái **không** được có (tên đối thủ, chủ đề nhạy cảm; với ảnh: loại bỏ chi tiết không liên quan).
- Đổi văn phong theo độ tuổi / giọng thương hiệu với **ít công nhất** → **prompt engineering động** (role/context trong prompt), không fine-tune.
- Chatbot sai giọng điệu → **thử và chỉnh prompt lặp lại**; temperature, batch, max tokens không giải quyết giọng điệu.
- Mô tả sản phẩm cụ thể → prompt **chi tiết theo từng sản phẩm**.
- Agent cần ví dụ để chính xác hơn → **sửa advanced prompts của agent** để thêm ví dụ (không dùng Guardrails, Ground Truth).
- **Bedrock Prompt Management** → variables `{{...}}`, variants (so sánh model/tham số), versions (rollback không cần redeploy).
- Rủi ro của prompt engineering: **prompt injection**.

### RAG & Knowledge Bases
- RAG = lấy thông tin từ nguồn ngoài, bổ sung vào prompt → câu trả lời **chính xác hơn, cập nhật**, **không đổi weights**.
- Dữ liệu **thay đổi nhanh** (tồn kho, chính sách, FAQ) → **RAG**, không fine-tune định kỳ.
- **Knowledge Bases for Amazon Bedrock** = RAG **end-to-end fully managed** (ingest → embedding → vector store → retrieve → augment).
- Use case hợp RAG: chatbot CSKH, chatbot y tế, nghiên cứu pháp lý. **Không hợp**: tạo nội dung gốc, tạo ảnh từ text, gợi ý sản phẩm theo sở thích.
- Gắn PDF vào mọi request → đắt và tràn context window; RAG rẻ hơn.
- **Citations** của **RetrieveAndGenerate** → map từng đoạn câu trả lời với tài liệu nguồn (S3 URI) để kiểm chứng. Tăng số kết quả retrieve, giảm temperature hay invocation logging không cho trích dẫn.
- Hiển thị **link tài liệu tham chiếu** trong câu trả lời → tăng niềm tin khách hàng.
- **RAG pipeline**: batch/offline = **tạo content embeddings + tạo search index**; online = embedding câu hỏi, retrieve, generate.
- **Vector DB** cho RAG: **OpenSearch Service** (kNN, full-text, similarity), **Aurora PostgreSQL** (pgvector), Neptune Analytics, S3 Vectors, RDS for PostgreSQL. **Không** phải vector DB trong đề: DynamoDB, Redshift, Athena, EMR (và DocumentDB bị coi là sai).
- Quick create vector store: xem mục 0.

### Fine-tuning & customization
- **Fine-tuning**: dữ liệu **labeled**, **đổi weights**, tạo **bản copy riêng** của base FM. Instruction-based fine-tuning cần **cặp câu hỏi–câu trả lời** theo domain.
- **Continued pre-training**: dữ liệu **unlabeled**, mở rộng kiến thức domain, cập nhật theo thời gian. Hai cách làm FM thành chuyên gia genomics: **continued pre-training + domain adaptation fine-tuning**.
- Dữ liệu customization trên Bedrock đặt ở **Amazon S3** (không EBS, EFS, RDS); file định dạng JSONL.
- **Distillation**: teacher lớn → student nhỏ, dùng prompts/responses trong **invocation logs**; nhanh hơn tới 5 lần, rẻ hơn tới 75%, mất < 2% accuracy với RAG.
- **RLHF**: con người **xếp hạng / so sánh** các response → train **reward model** → fine-tune bằng RL. Khác **human evaluation job** (chỉ chấm điểm, **không đổi model**).
- **Chuẩn bị data fine-tune**: loại trùng lặp, loại record không liên quan, sửa nhãn không nhất quán theo guideline rõ ràng (data nhỏ chất lượng > data lớn nhiễu; tăng epochs không sửa được lỗi data).
- Model kém ở dòng sản phẩm mới → thêm **dữ liệu có nhãn đại diện cho mọi dòng sản phẩm** rồi fine-tune lại.
- Data training có **PII / dữ liệu mật**: trước khi train → **mask / tokenize định danh**, hoặc thay bằng **synthetic data**; nếu đã train rồi → **xóa model, làm sạch data, train lại**. Mã hóa KMS không ngăn model học PII.
- Model fine-tune có thể nhả thông tin mật trong response → cách **nhanh nhất** là **Guardrails sensitive information filter** (mask) `[Đ1]`.
- Lợi ích fine-tuning: **cải thiện hiệu suất cho task cụ thể** (không làm model nhỏ hơn, không train lại từ đầu).

### Đánh giá model

| Phương pháp | Đặc điểm |
|---|---|
| Automatic evaluation (Bedrock) | Metric thống kê (**BERTScore, F1**...), dùng **built-in hoặc custom** prompt dataset; đánh giá **định lượng**; ít công vận hành nhất |
| LLM-as-a-judge | Judge model chấm correctness, completeness, helpfulness; **không cần reference answer**, scale tới hàng nghìn prompt |
| Human evaluation (Bedrock) | **Work team của bạn** (SME, nhân viên), metric tùy chỉnh; đánh giá **định tính** (tone, phong cách ưa thích); **phải dùng custom dataset**; so sánh tối đa 2 model |
| Benchmark datasets | Bộ chuẩn có sẵn; đo accuracy, **phát hiện bias** với **ít công quản trị nhất** |
| AgentCore Evaluations | Online evaluation trên session production + human-in-the-loop cho response rủi ro cao |

- Metric text: **BLEU** → dịch máy (so sánh **tương đối** với bản dịch người); **ROUGE** → tóm tắt; **BERTScore** → **tương đồng ngữ nghĩa**, xử lý tốt văn phong không chuẩn (teen slang); Accuracy/F1 → classification.
- Chọn model **nhân viên thích phong cách** → **human evaluation + custom prompts**. Chọn model + chặn nội dung có hại → **Model Evaluation + Guardrails**.
- FM có đáp ứng mục tiêu kinh doanh không → đánh giá **mức phù hợp với use case cụ thể**.
- Kiểm tra accuracy FM phân loại ảnh → **benchmark dataset** (không deploy production rồi hỏi user).

### SageMaker AI (theo những gì đề hỏi)

| Tính năng | Dùng khi đề nói |
|---|---|
| Data Wrangler (nay trong Canvas) | Chuẩn bị data trực quan, **300+ transform**, không code; **chia train/validation/test**; **cân bằng data (balance)**; import/biến đổi data từ nhiều nguồn (cho Personalize) |
| Feature Store | **Kho feature tập trung**, chia sẻ/tái sử dụng giữa team |
| Clarify | **Bias** (data & model), **explainability** (SHAP, PDP), đánh giá FM |
| Canvas | **No-code**, business analyst, dự báo nhu cầu, AutoML |
| JumpStart | Hub FM & model pre-trained (public **và** proprietary), customize, ML Solutions dựng sẵn (fraud detection...), **giới hạn model nhân viên thấy được** (private curated hub), data không dùng để train base model |
| Model Monitor | **Drift** và chất lượng model trong production |
| Model Dashboard | **Một nơi xem mọi model**, endpoint, tổng hợp dữ liệu từ Model Cards, Model Monitor, Endpoints |
| Model Cards | Tài liệu: **intended use, risk rating, training details, metrics**; cấu trúc cố định (không sửa schema); tạo được cho model train ngoài SageMaker |
| Model Registry | **Version**, approval status |
| Ground Truth | **Gắn nhãn với con người** (MTurk, nhân viên, vendor), RLHF |
| Automatic Model Tuning | **Tự động tune hyperparameter** |
| MLflow on SageMaker | **Quản lý experiment** |
| Pipelines | Tự động hóa, retrain với **data mới** |
| Endpoint (real-time/serverless) | Deploy **không quản server** |
| Network isolation | Job chạy **không có Internet** |
| Built-in algorithms | Thuật toán có sẵn, vẫn cần code |
| Role Manager | Quyền theo persona ML |

Bộ governance: **Role Manager + Model Cards + Model Dashboard** `[Đ4]`.

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (Domain 3)
- "dữ liệu công ty, cập nhật liên tục, rẻ nhất" → **RAG / Knowledge Bases**
- "thấy nguồn từng câu" → **citations của RetrieveAndGenerate**
- "model nhỏ, nhanh, rẻ, giữ accuracy, dùng invocation logs" → **Distillation**
- "unlabeled + chuyên ngành" → **Continued pre-training**; "labeled + task" → **Fine-tuning**
- "chuẩn bị data fine-tune có trùng lặp/nhãn sai" → **làm sạch + guideline nhãn**
- "loại trừ cách": Guardrails/Watermark/Continued pre-training **không** làm RAG
- "đánh giá nhiều prompt, không có reference" → **LLM-as-a-judge**
- "đánh giá định tính, SME" → **human evaluation**; "định lượng, BERTScore" → **automatic**
- "300 transform, không code, chia tập" → **Data Wrangler**; "chia sẻ feature" → **Feature Store**; "giải thích feature ảnh hưởng dự đoán" → **Clarify**; "xem mọi model" → **Model Dashboard**; "intended use, risk rating" → **Model Cards**; "experiment" → **MLflow**; "no-code" → **Canvas**

---

## Domain 4 — Guidelines for Responsible AI

### Dimensions & tình huống
- **Fairness** → giảm bias, quyết định tuyển dụng/cho vay công bằng, dữ liệu đa dạng mọi nhân khẩu học.
- **Transparency** → công khai việc dùng AI, cách model hoạt động, giới hạn.
- **Explainability** → lý do của từng đề xuất (điều trị, từ chối khoản vay).
- **Privacy & security** → ẩn danh dữ liệu cá nhân khi chuẩn bị data.
- **Safety** → Guardrails chặn output độc hại, lạm dụng.
- **Veracity & robustness** → vẫn đúng khi input có **typo, diễn đạt lạ, cố tình gây nhiễu**.
- **Controllability** → cơ chế giám sát, điều hướng hành vi AI.
- **Governance** → chính sách, quy trình, kiểm toán.
- Ngoài ra đề nhắc: **Accountability**. "Tolerance", "Flexibility", "Open source" **không** phải dimension.

### Bias
- **Sampling bias** → data không đại diện (camera chỉ gắn cờ một nhóm sắc tộc, 90% hồ sơ là nam).
- **Measurement bias** → công cụ đo lỗi; **Observer bias** → người thu thập/diễn giải; **Confirmation bias** → tìm thông tin khớp định kiến.
- **Algorithmic bias** → model thiên vị một giới dù năng lực ngang nhau. **Human bias** → người chọn feature theo niềm tin cá nhân.
- Sửa: **data augmentation cho nhóm thiếu đại diện**, dataset **cân bằng, đa dạng, curated, kiểm tra label quality**; đo **class imbalance** rồi điều chỉnh training; fine-tune lại với data đa dạng (rẻ hơn pre-train).
- Điều tra bias: **subgroup analysis** (error rate theo nhóm) + **human audit nhãn**. Accuracy tổng 92% có thể che giấu bias. Xóa cột nhân khẩu học không đủ (còn biến proxy như mã bưu chính).
- Regularization **không** sửa data imbalance; đổi model khác cũng không.
- Đánh giá bias định kỳ ít công nhất → **benchmark datasets**.

### Explainability & transparency
- **Interpretability** = hiểu **cơ chế bên trong**; **explainability** = giải thích **lý do output** cho stakeholder.
- **SHAP (Shapley values)** → **local**, đóng góp từng feature cho **một dự đoán**; **PDP** → **global**, ảnh hưởng biên của một feature trên toàn dataset. Clarify dùng cả hai.
- Lợi ích model minh bạch: **dễ debug, tối ưu** và **tăng niềm tin**. Không tự động ít tốn compute hơn, không bảo mật hơn.
- **Trade-off**: model đơn giản dễ hiểu nhưng có thể kém chính xác hơn.
- **Human-centered design**: nói rõ đây là đề xuất của AI, giải thích yếu tố chính, cho người dùng phản hồi; không che giấu, không đổi logic âm thầm.
- **Model Cards** (bạn viết cho model của bạn) vs **AWS AI Service Cards** (AWS công bố: intended use, giới hạn, best practice của dịch vụ AWS).
- Cơ quan cần **xem weights, tài liệu training data, license** → **open-weight model** có tài liệu và license cho phép kiểm tra.
- **Transparency vs safety**: công bố mục đích, giới hạn, loại safeguard ở mức tổng quát; **không** công bố nguyên system prompt và ngưỡng guardrail.

### Guardrails for Amazon Bedrock

| Filter | Dùng khi |
|---|---|
| **Content filters** | Hate, Insults, Sexual, Violence, Misconduct, Prompt Attack; **strength riêng cho prompt và response** (None/Low/Medium/High) |
| **Denied topics** | Chặn **chủ đề** (tư vấn đầu tư/pháp lý, ý tưởng chính trị trong bản tin) |
| **Word filters** | Chặn **từ/cụm từ** cụ thể (tên đối thủ, từ tục) |
| **Sensitive information filters** | **Mask/block PII**, regex tùy chỉnh |
| **Contextual grounding check** | Response **không có trong nguồn** (grounding) hoặc **không trả lời câu hỏi** (relevance), có ngưỡng 0–0.99 |
| **Automated Reasoning checks** | Kiểm tra response theo **luật logic** rút từ tài liệu chính sách, giải thích luật nào đúng/sai (bằng chứng audit), **detect mode** |
| **Prompt attack filter** + **input tags** | Jailbreak, prompt injection, prompt leakage; tag phần user input để không chặn nhầm system prompt |

Guardrails + **CloudWatch alarms** → lọc và **cảnh báo** khi vi phạm.

### Khác
- **Human-in-the-loop** → giảm bias/toxicity **ở giai đoạn post-processing**; duyệt **thư rủi ro cao hoặc confidence thấp** trước khi gửi, thư thường vẫn tự động.
- **Sustainability**: chọn **model nhỏ hơn** đủ chất lượng; train trên **Trainium** (hiệu quả năng lượng tốt nhất); không chạy song song nhiều model cho mọi request.
- Rủi ro pháp lý: ảnh giống nhân vật có bản quyền → **IP infringement**. Giải thích đề có nhắc AWS bồi thường IP không giới hạn cho output của Amazon Nova ❓ cần kiểm chứng.
- Dùng AI viết bài luận → **plagiarism**.
- Chặn ảnh không phù hợp do chatbot trả về → **moderation API** (Rekognition).
- Comprehend **toxicity detection** → phát hiện bình luận độc hại **không cần data có nhãn** (vẫn hoạt động; chỉ tính năng *prompt safety classification* không nhận khách mới).

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (Domain 4)
- "dataset đủ mọi nhân khẩu học" → **Diversity / Fairness**
- "một nhóm bị gắn cờ nhiều hơn" → **Sampling bias** → augmentation
- "local vs global explanation" → **SHAP local, PDP global**
- "giải thích feature ảnh hưởng dự đoán" → **SageMaker Clarify**
- "typo, câu lừa" → **Veracity & robustness**
- "chặn chủ đề" → **Denied topics**; "chặn từ" → **Word filters**; "che PII" → **Sensitive info filter**; "không có trong tài liệu" → **Contextual grounding**; "luật phức tạp, cần giải thích, audit" → **Automated Reasoning**
- "audit bias khi có khiếu nại" → **subgroup analysis + human label audit**
- "giảm carbon" → **model nhỏ hơn / Trainium**

---

## Domain 5 — Security, Compliance & Governance for AI

### Shared responsibility (Bedrock)
- **AWS**: hạ tầng (phần cứng, mạng, cơ sở vật chất) chạy Bedrock, vá lỗi dịch vụ.
- **Khách hàng**: **dữ liệu (mã hóa in transit/at rest), IAM, cấu hình guardrails, ứng dụng**.

### IAM & quyền
- Giới hạn nhân viên chỉ dùng **model cụ thể** trên Bedrock → **IAM policies** (không phải STS, Inspector, service role).
- Mỗi team chỉ truy cập data khách hàng của team trong RAG → **service role riêng cho từng team** (least privilege). Xem mục 0 về mâu thuẫn giữa các đề.
- Giới hạn FM mà nhân viên khám phá được trên SageMaker → **JumpStart private hub**.

### Mã hóa
- Training data + customization job + custom model với **khóa công ty kiểm soát** (rotate, audit, disable) → **AWS KMS customer managed keys**. Mặc định Bedrock dùng **AWS owned keys** (không kiểm soát được).
- **TLS** chỉ mã hóa **in transit**, không phải at rest. **Secrets Manager** dùng cho credential, không phải dataset.

### Mạng
- App trong VPC không Internet gọi Bedrock → **interface VPC endpoint (PrivateLink) cho bedrock-runtime**.
- SageMaker trong VPC đọc S3 không qua Internet → **VPC endpoint cho S3** (gateway hoặc interface).
- Fine-tune riêng tư, data ở lại Region, rẻ nhất → **Bedrock API + PrivateLink/VPC** (Outposts quá đắt).
- NAT Gateway và Internet Gateway **vẫn đi qua Internet** nên là đáp án sai.

### Logging & audit

| Nhu cầu | Dịch vụ |
|---|---|
| **Ai** gọi API Bedrock nào, khi nào | **AWS CloudTrail** |
| **Toàn bộ prompt + response** | **Bedrock model invocation logging** → S3 / CloudWatch Logs (**tắt mặc định**) |
| Metric, alarm | **Amazon CloudWatch** (hiệu suất ML system, scalable) |
| Phản ứng sự kiện | EventBridge (không có nội dung input/output chi tiết) |
| Lưu log 5 năm rẻ nhất (trong lựa chọn của đề) | CloudTrail + **S3 Intelligent-Tiering** |

### Governance & compliance services

| Dịch vụ | Dùng khi đề nói |
|---|---|
| **AWS Artifact** | Báo cáo compliance của AWS (SOC, ISO, PCI), **ISO/IEC 42001** (AI management system), báo cáo **ISV bên thứ ba** + **email thông báo** khi có báo cáo mới, thỏa thuận (BAA) |
| **AWS Config** | Theo dõi **thay đổi cấu hình** + đánh giá theo rule (Conformance Packs cho framework) |
| **Amazon Inspector** | **Tự động đánh giá bảo mật / lỗ hổng** (EC2, ECR, Lambda) |
| **AWS Audit Manager** ⚠️ | Thu thập bằng chứng audit liên tục (không nhận khách mới từ 30/04/2026) |
| **AWS Trusted Advisor** | Gợi ý best practice: cost, performance, security, fault tolerance, service limits |
| **Amazon Macie** | Tìm **PII trong S3** (tự động, ít công nhất) |
| **Amazon GuardDuty** | **Threat detection** real-time |

### Data governance
- **Data residency** = dữ liệu lưu/xử lý **ở đâu** (không rời quốc gia). **Data retention** = giữ **bao lâu**. **Data logging** = ghi lại **truy cập và thay đổi**.
- **Data access control** = xác thực, phân quyền **ai được truy cập**. **Data integrity** = dữ liệu **đúng, nhất quán, không bị sửa**.
- **Data lineage** → theo dõi nguồn gốc và biến đổi dữ liệu, phục vụ **privacy và compliance**.
- **Threat detection** = phát hiện tấn công đang diễn ra; **vulnerability management** = tìm và vá điểm yếu.
- **Defense in depth** = nhiều lớp: input validation, access control, giám sát liên tục.
- Mở rộng AI chấm điểm tín dụng sang nước khác → xem **luật algorithm accountability** của nước đó.
- Governance strategy: **ethical AI guidelines + audit output AI**. Không: dùng open-source không đánh giá, fine-tune tự do, chỉ dựa vào feedback người dùng.

### Generative AI Security Scoping Matrix
- 5 scope theo mức sở hữu: **Consumer app → Enterprise app → Pre-trained model → Fine-tuned model → Self-trained model** (scope 5 = **train từ đầu**, trách nhiệm bảo mật **cao nhất**).
- 5 discipline: Governance & compliance, Legal & privacy, **Risk management** (threat modeling, đề xuất mitigation), Controls, Resilience.

### ⚠️ Bẫy đề thi & Từ khóa nhận diện (Domain 5)
- "who called which API" → **CloudTrail**; "prompts & responses" → **invocation logging**
- "không qua Internet" → **VPC endpoint / PrivateLink**
- "khóa do công ty quản lý" → **KMS customer managed key**
- "PII trong S3" → **Macie**; "PII trong text/support ticket" → **Comprehend PII redaction**; "PII trong response của model" → **Guardrails sensitive info filter**
- "báo cáo SOC/ISO, ISV, notification" → **Artifact**; "thay đổi cấu hình" → **Config**; "lỗ hổng" → **Inspector**; "best practice cost/limits" → **Trusted Advisor**
- "dữ liệu không rời quốc gia" → **Data residency**
- "train từ đầu = sở hữu bảo mật nhiều nhất" → **Scope 5**; "threat modeling" → **Risk management**

---

## Danh mục dịch vụ AI/ML của AWS (use case một dòng)

| Dịch vụ | Use case |
|---|---|
| Amazon Bedrock | Dùng FM qua API, serverless; RAG (Knowledge Bases), Guardrails, Evaluation, customization |
| Bedrock AgentCore | Build, chạy, bảo mật, đánh giá AI agent (Runtime, Gateway, Memory, Identity, Policy, Evaluations) |
| Amazon SageMaker AI | Build/train/deploy model ML tùy biến (xem bảng tính năng ở Domain 3) |
| SageMaker JumpStart | Hub FM/model pre-trained và solution dựng sẵn, deploy lên SageMaker |
| SageMaker Canvas | ML no-code cho business analyst (kể cả forecast) |
| Amazon Quick | Trợ lý agentic cho nhân viên: research có trích dẫn, BI (Quick Sight), automation |
| Amazon Q Developer | Trợ lý AWS console/vận hành; ⚠️ phần IDE chuyển sang Kiro |
| Amazon Q data integration in AWS Glue | Sinh/sửa script ETL Glue bằng ngôn ngữ tự nhiên |
| Kiro | Agentic IDE/CLI, spec-driven development |
| AWS Transform | Agent hiện đại hóa .NET, mainframe, VMware |
| PartyRock | Playground GenAI không code, không cần AWS account |
| Amazon Comprehend | NLP: **sentiment**, entities, key phrases, ngôn ngữ, **PII detection/redaction**, **toxicity detection** |
| Amazon Comprehend Medical | Trích thực thể y khoa, **PHI** từ ghi chú lâm sàng |
| Amazon Textract | OCR: text, **chữ viết tay**, form, bảng, **hóa đơn/biên lai**, PDF |
| Amazon Rekognition | Ảnh/video: vật thể, khuôn mặt, celebrity, **text trong ảnh (biển số)**, **content moderation**, custom labels |
| Amazon Transcribe | Speech-to-text, **phụ đề**, cuộc gọi tổng đài |
| Amazon Transcribe Medical | Speech-to-text **thuật ngữ y khoa**, HIPAA |
| AWS HealthScribe | Sinh **clinical notes** từ hội thoại/đọc chính tả của bác sĩ |
| Amazon Polly | Text-to-speech, khóa học audio đa ngôn ngữ |
| Amazon Translate | Dịch văn bản, ứng dụng đa ngôn ngữ, mô tả sản phẩm nhiều thứ tiếng |
| Amazon Lex | Chatbot/voicebot (intent, slot) |
| Amazon Personalize | **Gợi ý cá nhân hóa** (vd: recipe **Similar-Items**) |
| Amazon Kendra ⚠️ | Enterprise search (không nhận khách mới) → Bedrock Managed Knowledge Base |
| Amazon Forecast ⚠️ | Dự báo chuỗi thời gian (không nhận khách mới) → SageMaker Canvas |
| Amazon Fraud Detector ⚠️ | Phát hiện gian lận (không nhận khách mới) → SageMaker AI |
| Amazon OpenSearch Service | Search + **vector DB** (kNN, similarity) cho RAG |
| Amazon Aurora PostgreSQL | Vector DB với **pgvector** |
| AWS Glue | ETL, data catalog |
| AWS Lake Formation | Quản trị và phân quyền data lake |
| Amazon Athena / EMR / Redshift | Truy vấn SQL trên S3 / Big Data (Spark) / data warehouse; **không** phải vector DB |
| Amazon Macie | Phát hiện PII trong S3 |
| AWS Trainium (Trn) | Chip **train** DL; hiệu quả năng lượng nhất cho training |
| AWS Inferentia (Inf) | Chip **inference** chi phí thấp |
| EC2 P / G / C series | GPU train / GPU đồ họa & inference nhẹ / CPU compute |
| Amazon DeepRacer | Xe đua mô hình để học reinforcement learning |
| Amazon Augmented AI (A2I) | Human review dự đoán ML |
| Amazon Mechanical Turk | Lực lượng crowdsourcing (gắn nhãn) |

---

## Phụ lục A — Bẫy phải thuộc lòng

1. **FM pre-train = self-supervised, unlabeled**; **fine-tune = supervised, labeled**; **continued pre-training = unlabeled**.
2. **Fine-tuning đổi weights**; **RAG và prompt engineering không đổi weights**.
3. Thứ tự rẻ → đắt: **Prompt engineering < RAG < Fine-tuning < Continued pre-training < Pre-train**.
4. **Top K = số lượng** token ứng viên; **Top P = phần trăm**; **Temperature = độ sáng tạo**; không cái nào đổi giá.
5. **Temperature thấp / 0** → deterministic; **cao** → sáng tạo (và dễ hallucinate hơn).
6. **BLEU = dịch**, **ROUGE = tóm tắt**, **BERTScore = ngữ nghĩa**, **F1 = classification**, **RMSE = regression**.
7. **Precision** → giảm false positive; **Recall** → giảm false negative; **F1** → data mất cân bằng.
8. **Overfitting = high variance** → **tăng** regularization; **underfitting = high bias**.
9. **Validation set optional**, dùng tune; **test set** đo generalization.
10. **K-means unsupervised ≠ k-NN supervised**.
11. **CNN = ảnh**, **RNN = chuỗi/video**, **Transformer = self-attention**, **GAN = 2 mạng đối kháng**, **Diffusion = khử nhiễu**, **VAE = latent space**.
12. **SHAP = local**, **PDP = global**.
13. **Interpretability = cơ chế bên trong**; **explainability = giải thích output**.
14. **CloudTrail = ai gọi API**; **invocation logging = nội dung prompt/response**.
15. **NAT/IGW vẫn qua Internet**; muốn private → **VPC endpoint / PrivateLink**.
16. **TLS = in transit**, **KMS = at rest** (CMK khi cần tự kiểm soát khóa).
17. **On-demand** = chưa biết usage; **PT** = steady; **Batch** = rẻ 50% khi chấp nhận trễ; **Spot/RI không có trên Bedrock**.
18. **Data residency = ở đâu**; **retention = bao lâu**; **logging = ai truy cập/sửa gì**.
19. **Poisoning = trước inference (data)**; injection/hijacking/jailbreak/leaking = **lúc inference (prompt)**; **exposure = lộ data đã học**.
20. **Hijacking** = đổi mục tiêu; **jailbreaking** = phá rào an toàn để lấy nội dung cấm.
21. **Model Cards** = tài liệu model của bạn; **AI Service Cards** = tài liệu AWS về dịch vụ AWS; **Model Dashboard** = xem mọi model.
22. **Data Wrangler** = chuẩn bị/cân bằng/chia data; **Feature Store** = lưu & chia sẻ feature; **Ground Truth** = gắn nhãn với con người.
23. **Async ≤ 1 GB/request**, **Batch = cả dataset nhiều GB**, **Serverless = traffic thưa + cold start**, **Real-time = endpoint thường trực**.
24. **Textract = tài liệu/PDF/chữ viết tay**; **Rekognition = text ngắn trong ảnh/video (biển số)**; Rekognition **không** đọc PDF.
25. Sentiment cuộc gọi audio = **Transcribe → Comprehend**.
26. **Data trên Bedrock không train base model, không chia sẻ với provider**; fine-tune tạo **bản copy private**.
27. **Scope 5 (self-trained) = sở hữu bảo mật nhiều nhất**.
28. **Trainium = train**, **Inferentia = inference**, cả hai tiết kiệm năng lượng.
29. **Business goal phase** quyết định yêu cầu compliance.
30. LLM là **tập con của FM**; LLM **non-deterministic, generative**.

## Phụ lục B — Dịch vụ / tính năng không tồn tại hoặc bị dùng sai mục đích (loại ngay)

| Đáp án nhiễu trong đề | Vì sao loại |
|---|---|
| Spot / Reserved Instances cho Bedrock | Mô hình giá của EC2, không có trên Bedrock |
| "Train base FM trên Bedrock", "tạo model từ đầu bằng Bedrock" | Bedrock chỉ customize bản copy; train từ đầu dùng SageMaker |
| "Public copy" của FM khi fine-tune | Bản copy luôn private |
| "Host Bedrock API on premises" | Không tồn tại |
| AWS Lambda để host LLM | Giới hạn bộ nhớ/thời gian, không phù hợp |
| Amazon Inspector "scan prompt/response" | Inspector quét lỗ hổng EC2/ECR/Lambda |
| AWS Config "ghi nội dung prompt" | Config ghi cấu hình tài nguyên |
| Trusted Advisor "ghi ai gọi model", "loại bỏ bias" | Chỉ gợi ý best practice hạ tầng |
| Amazon Macie "xoay vòng credential", "quét output model real-time" | Macie chỉ phát hiện data nhạy cảm trong S3 |
| CloudWatch/Audit Manager "lưu log API" thay CloudTrail | Sai dịch vụ |
| Feature Store "lưu response để duyệt", Data Wrangler "biến đổi response gửi khách" | Sai mục đích |
| Guardrails để "lưu thông tin khách cho lần sau" | Guardrails chỉ lọc, không lưu trữ |
| Personalize để lưu lịch sử hội thoại / tóm tắt / dự báo time-series | Personalize chỉ gợi ý |
| Rekognition cho text, NLP, dịch, PDF; Polly cho speech-to-text; Transcribe cho chatbot; Comprehend cho speech-to-text | Nhầm dịch vụ (xem danh mục) |
| Fraud Detector để truy cập tài liệu khiếu nại | Sai mục đích (và đã ngừng nhận khách mới) |
| DynamoDB / DocumentDB / Redshift / Athena / EMR làm vector DB cho RAG | Trong đề luôn là đáp án sai |
| SageMaker Inference endpoint để "nối VPC với S3" | Endpoint inference không phải network endpoint |
| Watermark detection, Guardrails, Continued pre-training để "làm RAG" | Không liên quan RAG |
| Encryption để "ngăn model nói ra PII" | Mã hóa không ngăn model học/nhả PII |
| "Tăng temperature để chống tấn công / bớt lặp" | Không phải biện pháp bảo mật, không giảm chi phí |
| "Model lớn hơn tốn ít token hơn" | Sai; model lớn đắt hơn mỗi token |
| "Validation set dùng đo generalization", "Test set optional" | Đảo ngược |
| WaveNet sinh text/SQL, ResNet hiểu ngôn ngữ, CNN sinh synthetic data dạng bảng | Sai kiến trúc |
| "Tolerance", "Flexibility", "Open source" là dimension của responsible AI | Không phải |

## Phụ lục C — Chủ đề lặp lại nhiều lần

| Chủ đề | Số đề xuất hiện (ước lượng) | Ghi nhớ |
|---|---|---|
| Inference parameters (Temperature/Top K/Top P/length/stop) | 7/7 | Bảng tra nhanh |
| RAG / Knowledge Bases vs fine-tuning, chi phí | 7/7 | RAG cho data riêng, thay đổi, rẻ |
| Overfitting/underfitting, bias/variance | 6/7 | High variance = overfit |
| SageMaker features (Clarify, Data Wrangler, Feature Store, Canvas, Model Cards/Dashboard/Monitor, JumpStart) | 7/7 | Bảng Domain 3 |
| Inference types SageMaker (real-time/serverless/async/batch) | 6/7 | Payload và tính cấp bách |
| Prompt techniques (zero/few/one-shot, CoT, negative, chaining) | 7/7 | Few-shot = cặp input–label |
| Metrics (BLEU/ROUGE/BERTScore, Precision/Recall/F1, RMSE) | 7/7 | Ghép metric đúng task |
| Supervised / unsupervised / semi / self-supervised / RL | 7/7 | Bảng learning |
| Guardrails (các loại filter) | 6/7 | Bảng Domain 4 |
| Prompt attacks (poisoning, injection, hijacking, jailbreak, leaking, exposure) | 5/7 | Bảng rủi ro |
| Bias (sampling) & fairness, data diversity | 6/7 | Augmentation, balanced data |
| Explainability (Clarify, SHAP/PDP, decision tree) | 6/7 | SHAP local, PDP global |
| Governance services (Artifact, Config, Inspector, CloudTrail, Trusted Advisor, Audit Manager) | 6/7 | Bảng Domain 5 |
| Bedrock pricing (on-demand, PT, batch, token) | 6/7 | Token = cost driver |
| Bảo mật mạng & mã hóa (PrivateLink, VPC endpoint, KMS) | 5/7 | Không qua Internet |
| Token / embedding / context window / chunking | 5/7 | Định nghĩa |
| Dịch vụ AI dựng sẵn (Comprehend, Textract, Rekognition, Transcribe, Polly, Translate, Lex, Personalize) | 7/7 | Danh mục dịch vụ |
| Agentic AI (AgentCore, MCP, multi-agent, Strands) | 4/7 (đề mới) | Xu hướng mới của exam guide 1.1 |

---

### Nguồn đã đối chiếu khi viết mục 0
- Custom model inference (on-demand + PT): https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-use.html
- Knowledge Base quick-create vector stores: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-create.html
- Bedrock customization methods: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html
- Comprehend trust & safety (toxicity detection): https://docs.aws.amazon.com/comprehend/latest/dg/trust-safety.html
- Comprehend feature availability change: https://docs.aws.amazon.com/comprehend/latest/dg/comprehend-availability-change.html
- Amazon Forecast end of new customer access: https://docs.aws.amazon.com/managedservices/latest/onboardingguide/forecast.html
- JumpStart curated private hubs: https://docs.aws.amazon.com/sagemaker/latest/dg/jumpstart-curated-hubs-gated-model-access.html
- AWS ISO/IEC 42001 (tải certificate qua Artifact): https://aws.amazon.com/compliance/iso-42001-faqs/
- Audit Manager availability change: https://docs.aws.amazon.com/audit-manager/latest/userguide/audit-manager-availability-change.html
- Kendra availability change: https://docs.aws.amazon.com/kendra/latest/dg/kendra-availability-change.html
- Q Business availability change: https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/qbusiness-availability-change.html
- Bedrock Agents (Agents Classic): https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html
- Fraud Detector availability change: https://docs.aws.amazon.com/frauddetector/latest/ug/frauddetector-availability-change.html
- Bedrock Guardrails: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html
