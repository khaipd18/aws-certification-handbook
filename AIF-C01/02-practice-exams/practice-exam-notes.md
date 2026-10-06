# AWS AIF-C01 — Sổ tay cốt lõi & kiến thức dễ nhầm

> Tổng hợp ý tưởng cốt lõi + kiến thức liên quan/dễ nhầm từ **7 đề** (bộ "Practice Test #1–#4 Full-Exam" và Cloud Exam Pro "Practice Test 01–03"), theo nhóm chủ đề và dịch vụ.
> **🎯 = ý tưởng cốt lõi cần khắc sâu** · **⚠️ = kiến thức liên quan / dễ nhầm trong đề** · **📌 = cập nhật theo docs AWS hiện hành (10/2026)** · **❓ = các đề mâu thuẫn nhau hoặc đáp án đáng ngờ**.
> Xem **§30 (Phụ lục)** để tra nhanh dịch vụ/đáp án hay xuất hiện ở đáp án SAI.
>
> **Ôn cấp tốc trước khi thi:** §31 (Nguyên tắc làm bài) → §32 (Bảng so sánh dễ nhầm) → §33 (Từ khóa → đáp án) → §34 (Con số & đáp án bất khả thi).
>
> 5 domain AIF-C01: **D1 Fundamentals of AI/ML · D2 Fundamentals of GenAI · D3 Applications of FMs · D4 Responsible AI · D5 Security, Compliance & Governance**.
>
> **Khi có đề mới:** chèn kiến thức vào đúng mục (§1–§29), cập nhật §30–§34. Không tạo phần riêng cho từng đề.
>
> **Khi thi thật:** chọn đáp án đúng chức năng theo tên dịch vụ và tính năng hiện hành.

---

## 1. Nền tảng AI/ML

🎯 **AI > ML > Deep Learning > GenAI** (AI rộng nhất). ML là tập con của AI, học từ dữ liệu; AI còn gồm hệ thống rule-based.

🎯 **Không dùng ML** khi bài toán có đáp án tính được chính xác (xác suất rút lá bích) → ứng dụng **rule-based**.

⚠️ **Computer vision** = hiểu nội dung ảnh/video để ra quyết định (detect, classify, nhận diện). **Image processing** = xử lý ảnh (lọc nhiễu, tăng chất lượng). Đề hay đảo ngược hai khái niệm này. **NLP** = text/speech (phân loại phản hồi khách hàng thành chủ đề = NLP text classification).

⚠️ **ML algorithm** = công thức/thủ tục toán học; **ML model** = kết quả của algorithm sau khi train trên data. **Inference** = model đã train nhận input mới và trả dự đoán; **evaluation** = so sánh output để chọn model phù hợp nhất. Model ML có thể **deterministic, probabilistic, hoặc kết hợp**.

🎯 **Deep learning training** = điều chỉnh **weights và biases** của neural network qua nhiều vòng lặp (gradient descent) để giảm lỗi. **Traditional ML** cần người chọn feature thủ công; **deep learning** tự rút feature từ raw data (cần nhiều data, nhiều compute, không nhất thiết train nhanh hơn).

🎯 **Thách thức lớn nhất khi làm ML** = thu thập và chuẩn bị **dữ liệu chất lượng cao**. Muốn chọn đúng model → **định nghĩa use case hẹp, cụ thể** trước.

⚠️ **Loại bài toán:** dự đoán giá trị số → **Regression** · 2 lớp (fraud / not) → **Binary classification** · ≥ 3 lớp → **Multi-class** · mỗi mẫu nhiều nhãn → **Multi-label**.

---

## 2. Các loại learning

🎯 **Bảng chọn:**

| Loại | Dữ liệu | Ví dụ thuật toán / use case trong đề |
| --- | --- | --- |
| **Supervised** | Có nhãn | Linear/logistic regression, decision tree, neural network, SVM, **k-NN**; churn, fraud, phân loại ảnh, giá nhà |
| **Unsupervised** | Không nhãn | **K-means** (clustering, phân khúc khách hàng), **PCA** (giảm chiều), association rules (Apriori), anomaly detection |
| **Semi-supervised** | Ít nhãn + nhiều không nhãn (pseudo-labeling) | **Fraud identification, sentiment analysis, document classification** |
| **Self-supervised** | Không nhãn, model tự tạo nhãn | **Pre-training FM / LLM** (GPT, BERT) |
| **Reinforcement learning** | Agent + environment + reward | Robot, game, tối ưu hệ thống; chatbot học từ phản hồi tích cực |
| **Transfer learning** | Model task A → task B liên quan | Dùng lại kiến thức giữa các model của nhiều phòng ban |
| **Incremental training** | Model hiện có + dữ liệu mới, không quên kiến thức đã học | Cập nhật model với data mới |
| **Federated learning** | Data phân tán, **không gom về một chỗ** | Privacy, compliance khi huấn luyện |
| **Active learning** | Model chủ động xin người gắn thêm nhãn | Loại trừ khi đề nói "sẽ không gắn nhãn thêm" |

⚠️ **K-means** (unsupervised, gom cụm) ≠ **k-NN** (supervised, phân loại theo láng giềng gần nhất).

---

## 3. Dữ liệu & feature engineering

🎯 **Labeled** (có output, cho supervised) vs **unlabeled** (unsupervised). **Structured** (bảng hàng–cột) vs **unstructured** (text, ảnh, video). **Time series** → dự báo nhu cầu (SageMaker **DeepAR**, ARIMA).

🎯 **Đặc tính dataset:** **Diversity** (đủ mọi nhóm nhân khẩu học) · **Accuracy** (từng điểm dữ liệu đúng) · **Reliability** (ổn định, nhất quán) · **Recency bias** (lệch về dữ liệu mới, cần tránh).

🎯 **Feature engineering** = chọn, biến đổi, tạo feature để model tốt hơn (không phải thu thập data, tune hyperparameter hay visualize).
- **Feature extraction** = biến đổi sang không gian mới, giảm chiều (PCA).
- **Feature selection** = chọn tập con feature quan trọng từ feature có sẵn.
- Structured data: normalization, xử lý missing values, encoding. Unstructured: **tokenization, vectorization**.

🎯 **Train / validation / test:** train để học · **validation** để tune hyperparameter và chọn model (**optional**) · **test** đánh giá cuối khả năng generalize (**không optional**, không dùng để tune).

---

## 4. Vòng đời ML & MLOps

🎯 Thứ tự cơ bản: **Data collection → Data preprocessing → Model training → Model evaluation**. Theo Well-Architected ML: **Define business goal & frame ML problem → Develop → Deploy → Monitor**.

🎯 **Yêu cầu compliance và pháp lý** được xác định ở giai đoạn **business goal identification** (đầu tiên).

⚠️ **EDA** = thống kê tóm tắt, visualize, **correlation matrix** để hiểu data trước khi build model. Khác data preparation (làm sạch, xử lý missing), data augmentation (tạo thêm biến thể), model evaluation.

🎯 **Hyperparameter** (learning rate, epochs, batch size, regularization, dropout, số layer) = đặt trước, điều khiển quá trình train. **Model parameter** (weights, biases) = model tự học. Tăng **epochs** để tăng accuracy tới ngưỡng chấp nhận (cẩn thận overfit).

⚠️ **IaC trong MLOps** = triển khai ML workload **nhất quán, lặp lại được, scalable**; không tự chọn instance mạnh nhất, không thay hyperparameter tuning.

---

## 5. Kiến trúc model & model dễ diễn giải

🎯 **Kiến trúc hay gặp:**

| Kiến trúc | Dùng cho |
| --- | --- |
| **CNN** | Ảnh (phân loại ảnh đơn) |
| **RNN** | Dữ liệu tuần tự (time series, video, speech) |
| **Transformer** | **Self-attention** + contextual embeddings, xử lý song song (GPT, BERT, Claude, Llama; ChatGPT = Chat Generative Pre-trained Transformer) |
| **GAN** | 2 mạng (generator vs discriminator) đối kháng; **tạo synthetic data** giữ đặc tính thống kê của data gốc |
| **VAE** | Học latent space |
| **Diffusion** | Thêm nhiễu rồi khử nhiễu từng bước (Stable Diffusion, DALL-E, Adobe Firefly) |
| **BERT** | Đọc hai chiều, **contextual embeddings**, **điền từ bị thiếu** (masked LM) |
| **Word2Vec** | Static embeddings (một từ một vector) |
| **WaveNet** | Sinh âm thanh (speech synthesis) |
| **ResNet** | Ảnh |
| **XGBoost / SVM** | Tabular classification/regression, không sinh dữ liệu |

⚠️ **Object detection** = xác định và định vị nhiều vật thể trong ảnh (nhận diện mọi con vật trong ảnh bẫy camera). NER là kỹ thuật text; face recognition chỉ cho mặt người. PCA, SVD không hiểu ngữ cảnh.

🎯 **Model dễ diễn giải:** **Decision tree** → interpretable nhất, thấy feature nào dẫn tới quyết định. **Logistic/linear regression** → **xem và chỉnh tay được weights** từng biến. **Neural network, deep learning, SVM, ensemble** → "black box". Explainability tỷ lệ nghịch với **model complexity**.

---

## 6. Overfitting vs Underfitting · Bias vs Variance

🎯 **Bảng chẩn đoán:**

| | Overfitting | Underfitting |
| --- | --- | --- |
| Triệu chứng | Train tốt, **test/production kém** | Kém **cả train lẫn test** |
| Bias / Variance | **High variance** | **High bias** |
| Nguyên nhân | Model quá phức tạp, học cả noise; data ít; train quá lâu | Model quá đơn giản; thiếu feature |
| Cách sửa | **Tăng regularization**, thêm data đa dạng, data augmentation, early stopping, cross-validation, pruning, giảm epochs, feature selection, ensembling | Model phức tạp hơn, thêm feature, giảm regularization, train lâu hơn |

⚠️ Bias và variance **có thể cùng cao** (vd giảm data train) → model vừa underfit vừa overfit.

---

## 7. Metrics

🎯 **Classification:**

| Metric | Công thức / ý nghĩa | Chọn khi |
| --- | --- | --- |
| **Accuracy** | (TP + TN) / tổng | Data **cân bằng** |
| **Precision** | TP / (TP + FP) | **False positive đắt** (nhân viên mất thời gian xem nhầm cảnh báo gian lận) |
| **Recall** | TP / (TP + FN) | **False negative đắt** (bỏ sót ung thư, gian lận) |
| **F1** | Trung bình điều hòa P và R | **Data mất cân bằng** (churn, fraud) |
| **Confusion matrix** | Bảng TP/FP/FN/TN theo lớp | Xem chi tiết lỗi phân loại |
| **AUC-ROC** | 0 → 1, so model ở nhiều threshold | So sánh classifier |

🎯 **Regression:** MAE, MSE, RMSE, R² (không phải classification).

🎯 **Text/GenAI:** **BLEU** → dịch máy (so với bản dịch người) · **ROUGE** → tóm tắt · **BERTScore** → **tương đồng ngữ nghĩa**, xử lý tốt văn phong không chuẩn (teen slang) · Accuracy/F1 → classification.

❓ Một đề đáp **BLEU** cho summarization vì không có ROUGE trong lựa chọn. Chuẩn là **ROUGE**.

🎯 **Runtime efficiency** → **average response time** (latency); không phải accuracy, throughput, precision-recall.

🎯 **Business metrics:** task completion rate, cost per interaction, CSAT, ARPU, ROI, average call duration (AHT), first-call resolution. Đo chatbot hỗ trợ nhân viên trong cuộc gọi → **average call duration**.

❓ Một đề đáp **Average handled time** cho "financial effect" của chatbot; đề khác coi **cost per interaction** là business metric. Có "cost per customer conversation" thì nó đo tài chính trực tiếp hơn.

---

## 8. SageMaker AI — Inference

🎯 **4 kiểu inference:**

| Kiểu | Đặc điểm | Đề chọn khi |
| --- | --- | --- |
| **Real-time** | Endpoint thường trực, latency thấp, payload ≤ 25 MB | Cần dự đoán ngay từng request (check-in bệnh nhân, chatbot) |
| **Serverless** | Không quản hạ tầng, traffic thưa, chịu cold start, scale về 0 | Workload gián đoạn, muốn rẻ |
| **Asynchronous** | Hàng đợi, payload **≤ 1 GB**, xử lý tới 1 giờ, near real-time | Payload lớn **từng request**, không cần ngay |
| **Batch transform** | Cả dataset, không endpoint thường trực | **Nhiều GB/TB**, chạy định kỳ |

⚠️ **Real-time = synchronous**, **batch = asynchronous**.

🎯 **Edge** → model nhỏ (**SLM**) chạy **trên thiết bị** để latency thấp nhất; gọi LLM/API trung tâm vẫn có network latency.

---

## 9. SageMaker AI — Tính năng

🎯 **Bảng khớp tính năng:**

| Tính năng | Dùng khi đề nói |
| --- | --- |
| **Data Wrangler** (nay trong Canvas) | Chuẩn bị data trực quan, **300+ transform**, không code; **chia train/validation/test**; **cân bằng data**; import/biến đổi data nhiều nguồn |
| **Feature Store** | **Kho feature tập trung**, chia sẻ/tái sử dụng giữa team |
| **Clarify** | **Bias** (data & model), **explainability** (SHAP, PDP), đánh giá FM |
| **Canvas** | **No-code**, business analyst, dự báo nhu cầu, AutoML |
| **JumpStart** | Hub FM & model pre-trained (public **và** proprietary), customize, ML Solutions dựng sẵn; **private curated hub** giới hạn model nhân viên thấy; data không train base model |
| **Model Monitor** | **Drift** và chất lượng model trong production |
| **Model Dashboard** | **Một nơi xem mọi model**, endpoint; tổng hợp từ Model Cards, Model Monitor, Endpoints |
| **Model Cards** | Tài liệu: **intended use, risk rating, training details, metrics**; schema cố định; tạo được cho model train ngoài SageMaker |
| **Model Registry** | **Version**, approval status |
| **Ground Truth** | **Gắn nhãn với con người** (MTurk, nhân viên, vendor), RLHF |
| **Automatic Model Tuning** | **Tự động tune hyperparameter** |
| **MLflow on SageMaker** | **Quản lý experiment** |
| **Pipelines** | Tự động hóa, retrain với **data mới** |
| **Endpoint** (real-time/serverless) | Deploy **không quản server** |
| **Network isolation** | Job chạy **không có Internet** |
| **Built-in algorithms** | Thuật toán có sẵn, vẫn cần code |
| **Role Manager** | Quyền theo persona ML |

🎯 Bộ governance của SageMaker: **Role Manager + Model Cards + Model Dashboard**.

⚠️ **JumpStart vs Bedrock:** JumpStart deploy model lên **SageMaker endpoint của bạn** (toàn quyền chọn instance), có ML Solutions dựng sẵn; Bedrock serverless.

---

## 10. GenAI, Foundation Model, LLM

🎯 **GenAI** = tạo nội dung **mới** (text, ảnh, audio, code) từ pattern học được. Use case hợp lệ: tạo ảnh từ text, tóm tắt, viết mô tả, sinh code. **Không hợp**: dự đoán giá nhà, dự báo chứng khoán, phân loại ảnh y tế (đó là discriminative ML).

⚠️ **Generative vs discriminative:** generative học phân phối dữ liệu để **tạo mới**; discriminative **phân loại** giữa các lớp.

🎯 **FM** = model lớn, đa năng, nền cho nhiều tác vụ; **LLM là một loại FM** chuyên ngôn ngữ (không phải ngược lại). FM pre-train bằng **self-supervised learning trên data không nhãn**; **fine-tuning là supervised** (data có nhãn).

🎯 **LLM non-deterministic** (cùng prompt có thể ra output khác) và là generative model. LLM dùng cho: sinh text, dịch, tóm tắt, hỏi đáp, **text-to-SQL**, **sinh code**. Không phải TTS, tạo video, 3D.

⚠️ **Multimodal model**: input **và** output đều có thể nhiều loại (text, ảnh, audio, video).

🎯 Ưu điểm GenAI trong đề: **adaptability** (thay logic hard-coded khó mở rộng), tự động tạo nội dung mới. Không đảm bảo bảo mật 100%, không thay toàn bộ con người. Best practice: **guardrails + transparency** (cho người dùng biết đang tương tác với AI), giám sát liên tục.

---

## 11. Token, embedding, context window

🎯 **Token** = đơn vị text model xử lý (từ, subword, dấu câu). **Tokenization** = chia text thành token.

🎯 **Embedding** = **vector số** nhiều chiều biểu diễn ngữ nghĩa; dùng để **so sánh văn bản bằng toán học**, semantic search, RAG. Không phải nén, mã hóa hay lưu trữ. **Text embeddings model** → chuyển dữ liệu riêng thành vector trước khi lưu vào vector DB.

🎯 **Context window** = số token tối đa model xem xét một lần. Input dài (sách) vượt context window → model thất bại.

⚠️ **Chunking** trong RAG → cải thiện **độ liên quan ngữ cảnh** của kết quả truy xuất (không phải để tiết kiệm storage).

⚠️ Chatbot hiểu cả **text + ảnh** với chi phí thấp → **multimodal embedding model** (rẻ hơn multimodal generative model).

---

## 12. Inference parameters

🎯 **Bảng tham số:**

| Tham số | Kiểm soát | Thấp | Cao |
| --- | --- | --- | --- |
| **Temperature** | Độ ngẫu nhiên/sáng tạo | Nhất quán, deterministic (0 = ổn định nhất) | Sáng tạo, đa dạng, dễ hallucinate hơn |
| **Top K** | **Số lượng** token ứng viên | Mạch lạc | Đa dạng |
| **Top P** | **% xác suất cộng dồn** của token ứng viên | Mạch lạc | Đa dạng |
| **Response length / Max tokens** | Số token tối đa của output | Ngắn, rẻ hơn | Dài |
| **Stop sequences** | Chuỗi báo dừng sinh | — | — |
| **CFG scale** (Stable Diffusion) | Mức bám sát prompt khi tạo ảnh | Ngẫu nhiên | **Cụ thể, bám prompt** |

⚠️ Temperature, Top K, Top P **không ảnh hưởng giá và latency**. Chatbot sai giọng điệu → sửa **prompt**, không phải temperature/batch/max tokens.

❓ Một đề đáp **giảm temperature** để giảm hallucination — đúng trong phạm vi lựa chọn của đề; biện pháp mạnh hơn là **RAG + contextual grounding check**.

---

## 13. Prompt engineering

🎯 **4 thành phần prompt tốt:** **Instructions, Context, Input data, Output indicator** (không phải hyperparameters hay parameters).

🎯 **Kỹ thuật:**

| Kỹ thuật | Đặc điểm |
| --- | --- |
| **Zero-shot** | Không ví dụ |
| **One-shot** | 1 ví dụ |
| **Few-shot** | Vài ví dụ; với intent detection, ví dụ là cặp **user input + intent đúng** |
| **Chain-of-thought** | "Think step by step", lý luận nhiều bước |
| **Prompt chaining** | Chia task lớn thành subtask **gửi tuần tự**, output bước trước là input bước sau |
| **Tree of thoughts** | Nhiều nhánh suy luận song song |
| **Least-to-most** | Từ bài nhỏ tới bài lớn |
| **ReAct** | Reasoning + **acting** (gọi tool/API, vd kiểm tra tồn kho real-time) |
| **Negative prompting** | Chỉ rõ cái **không** được có (tên đối thủ, chủ đề nhạy cảm; với ảnh: loại chi tiết không liên quan) |

❓ Một đề đáp **one-shot** khi đề đưa "descriptions" (nhiều ví dụ) theo format. Nhiều ví dụ → chuẩn là **few-shot**; chỉ chọn one-shot khi đề nói rõ **một** ví dụ.

🎯 Đổi văn phong theo độ tuổi / giọng thương hiệu với **ít công nhất** → **prompt engineering động** (role/context trong prompt), không fine-tune. Mô tả sản phẩm cụ thể → prompt **chi tiết theo từng sản phẩm**.

⚠️ Agent cần ví dụ để chính xác hơn → **sửa advanced prompts của agent** (không dùng Guardrails, Ground Truth).

🎯 **Bedrock Prompt Management** → variables `{{...}}`, variants (so sánh model/tham số), versions (rollback không cần redeploy).

⚠️ **Prompt engineering vs context engineering:** prompt engineering = **hỏi thế nào**; context engineering = **cho model thấy gì** (profile, lịch sử hội thoại, tài liệu RAG, tool definition trong giới hạn context window). Rủi ro của prompt engineering: **prompt injection**.

---

## 14. RAG & Knowledge Bases

🎯 **RAG** = lấy thông tin từ nguồn ngoài, bổ sung vào prompt → câu trả lời **chính xác hơn, cập nhật**, **không đổi weights**. Dữ liệu **thay đổi nhanh** (tồn kho, chính sách, FAQ) → **RAG**, không fine-tune định kỳ.

🎯 **Knowledge Bases for Amazon Bedrock** = RAG **end-to-end fully managed** (ingest → embedding → vector store → retrieve → augment).

⚠️ Use case hợp RAG: chatbot CSKH, chatbot y tế, nghiên cứu pháp lý. **Không hợp**: tạo nội dung gốc, tạo ảnh từ text, gợi ý sản phẩm theo sở thích. Gắn PDF vào mọi request → đắt và tràn context window.

🎯 **Citations** của **RetrieveAndGenerate** → map từng đoạn câu trả lời với tài liệu nguồn (S3 URI) để kiểm chứng; hiển thị link nguồn → tăng niềm tin. Tăng số kết quả retrieve, giảm temperature hay invocation logging **không** cho trích dẫn.

⚠️ **RAG pipeline:** batch/offline = **tạo content embeddings + tạo search index**; online = embedding câu hỏi, retrieve, generate.

🎯 **Vector DB cho RAG:** **OpenSearch Service** (kNN, full-text, similarity), **Aurora PostgreSQL** (pgvector), Neptune Analytics, S3 Vectors, RDS for PostgreSQL. **Không** phải vector DB trong đề: DynamoDB, Redshift, Athena, EMR (DocumentDB cũng bị coi là sai).

📌 Vector store khi quick create Knowledge Base: **OpenSearch Serverless, Aurora PostgreSQL Serverless, Neptune Analytics, S3 Vectors**; AWS khuyên dùng **Bedrock Managed Knowledge Base** (datastore do Bedrock quản lý).

📌 **Amazon Kendra** (enterprise search) không nhận khách mới từ 30/07/2026 → **Bedrock Managed Knowledge Base**.

---

## 15. Customization: fine-tuning, continued pre-training, distillation, RLHF

🎯 **Cải thiện output của FM — bảng chọn:**

| Kỹ thuật | Đổi weights? | Dữ liệu cần | Chi phí / công sức | Khi nào chọn |
| --- | --- | --- | --- | --- |
| **Prompt engineering** | Không | Không cần (vài ví dụ trong prompt) | **Thấp nhất** | Đổi giọng văn, format; "least effort / most cost-effective" |
| **RAG** | Không | Tài liệu nội bộ (PDF, S3, wiki…) | Thấp–vừa | Dữ liệu **riêng, thay đổi thường xuyên**, cần trích dẫn |
| **Fine-tuning** (SFT / instruction-based) | **Có** (bản copy riêng) | **Labeled** (prompt–completion, Q&A pairs) | Cao | Task cụ thể, giọng thương hiệu cố định, thuật ngữ chuyên ngành |
| **Continued pre-training / domain adaptation** | **Có** | **Unlabeled**, khối lượng lớn, chuyên ngành | Cao nhất trong nhóm customization | Model cần "hiểu" lĩnh vực (genomics, y khoa…) |
| **Distillation** | **Có** (student) | Prompts (hoặc invocation logs) | Trả một lần, inference rẻ hơn | Cần model **nhỏ hơn, nhanh hơn, rẻ hơn**, accuracy gần bằng |
| **Pre-train FM từ đầu** | — | Khổng lồ | Cực đắt; **không làm trên Bedrock** (dùng SageMaker) | Gần như không bao giờ là đáp án "cost-effective" |

🎯 Thứ tự độ phức tạp tăng dần: **Prompt engineering → RAG → Fine-tuning → Continued pre-training → Pre-train**.

🎯 Hai cách làm FM thành chuyên gia genomics: **continued pre-training + domain adaptation fine-tuning**.

📌 Trang customization của Bedrock hiện chỉ liệt kê **SFT, Reinforcement fine-tuning, Distillation**. Continued pre-training vẫn nằm trong exam guide 3.3 → vẫn phải thuộc khái niệm.

🎯 Dữ liệu customization trên Bedrock đặt ở **Amazon S3** (không EBS, EFS, RDS), định dạng JSONL.

🎯 **Distillation**: teacher lớn → student nhỏ, dùng prompts/responses trong **invocation logs**; nhanh hơn tới 5 lần, rẻ hơn tới 75%, mất < 2% accuracy với RAG.

🎯 **RLHF**: con người **xếp hạng / so sánh** các response → train **reward model** → fine-tune bằng RL. Khác **human evaluation job** (chỉ chấm điểm, **không đổi model**).

📌 Bedrock có **Reinforcement fine-tuning** (reward function bằng Lambda hoặc judge model); RLHF "đúng nghĩa" (reward model học từ xếp hạng người) vẫn không phải tính năng có sẵn.

🎯 **Chuẩn bị data fine-tune:** loại trùng lặp, loại record không liên quan, sửa nhãn không nhất quán theo guideline rõ ràng (data nhỏ chất lượng > data lớn nhiễu; tăng epochs không sửa được lỗi data). Model kém ở dòng sản phẩm mới → thêm **dữ liệu có nhãn đại diện cho mọi dòng sản phẩm** rồi fine-tune lại.

🎯 **PII trong data training:** trước khi train → **mask / tokenize định danh**, hoặc thay bằng **synthetic data**; nếu đã train → **xóa model, làm sạch data, train lại**. Mã hóa KMS **không** ngăn model học PII. Model fine-tune nhả thông tin mật → cách **nhanh nhất** là **Guardrails sensitive information filter** (mask).

⚠️ Lợi ích fine-tuning = **cải thiện hiệu suất cho task cụ thể** (không làm model nhỏ hơn, không train lại từ đầu).

---

## 16. Đánh giá model

🎯 **Phương pháp đánh giá:**

| Phương pháp | Đặc điểm |
| --- | --- |
| **Automatic evaluation** (Bedrock) | Metric thống kê (**BERTScore, F1**…), dataset **built-in hoặc custom**; **định lượng**; ít công vận hành nhất |
| **LLM-as-a-judge** | Judge model chấm correctness, completeness, helpfulness; **không cần reference answer**, scale tới hàng nghìn prompt |
| **Human evaluation** (Bedrock) | **Work team của bạn** (SME, nhân viên), metric tùy chỉnh; **định tính** (tone, phong cách); **phải dùng custom dataset**; so sánh tối đa 2 model |
| **Benchmark datasets** | Bộ chuẩn có sẵn; đo accuracy, **phát hiện bias** với **ít công quản trị nhất** |
| **AgentCore Evaluations** | Online evaluation trên session production + human-in-the-loop cho response rủi ro cao |

⚠️ Chọn model **nhân viên thích phong cách** → **human evaluation + custom prompts**. Chọn model + chặn nội dung có hại → **Model Evaluation + Guardrails**. Kiểm tra accuracy FM phân loại ảnh → **benchmark dataset** (không deploy production rồi hỏi user). FM có đáp ứng mục tiêu kinh doanh → đánh giá **mức phù hợp với use case cụ thể**.

---

## 17. Amazon Bedrock — giá & dữ liệu

🎯 **Bedrock vs SageMaker AI vs AI services dựng sẵn:**

| | AI services (Comprehend, Rekognition…) | Amazon Bedrock | SageMaker AI |
| --- | --- | --- | --- |
| Là gì | Model pre-trained, gọi API | FM nhiều provider qua API, serverless | Tự build/train/deploy model bất kỳ |
| Kiến thức ML | Không cần | Ít | Cần (trừ Canvas) |
| Tùy biến | Thấp (custom labels / classifier) | Prompt, RAG, fine-tune, distillation | Toàn quyền |
| Chọn khi | Task phổ biến, ít công nhất | GenAI, ít vận hành | ML riêng, train từ đầu, kiểm soát hạ tầng |

🎯 **Mô hình giá:**

| Mô hình | Chọn khi |
| --- | --- |
| **On-demand** | Không cam kết; usage **thấp / khó đoán / đang thử nghiệm** |
| **Provisioned Throughput** | Mua capacity theo giờ (cam kết 1 hoặc 6 tháng rẻ hơn); lưu lượng **ổn định, cao**. Dùng cho 1 lần gọi/ngày là lãng phí |
| **Batch inference** | Rẻ hơn tới **50%** on-demand, chấp nhận trễ |

📌 Ngoài 3 mô hình trên còn có **service tiers** (Standard/Priority/Flex/Reserved).

📌 **Custom model trên Bedrock** chạy được **on-demand** (custom model deployment) hoặc **Provisioned Throughput**.

🎯 Chi phí chính = **số token input + output**. Giảm chi phí: rút ngắn prompt, ít few-shot hơn, giới hạn **max output tokens**, yêu cầu trả lời ngắn gọn, **prompt caching** cho phần system prompt lặp lại. **Model nhỏ rẻ hơn** model lớn.

⚠️ **Spot instances, Reserved Instances** là khái niệm của EC2, **không áp dụng cho Bedrock**.

🎯 Dữ liệu trên Bedrock **không dùng để train base model và không chia sẻ với model provider**; fine-tuning tạo **bản copy private**. Không thể train base FM hay tạo model từ đầu trên Bedrock (dùng SageMaker).

---

## 18. Agentic AI & Bedrock AgentCore

🎯 **Multi-agent (supervisor pattern)**: một agent điều phối chia việc cho các agent chuyên biệt, mỗi agent ít tool → giảm chọn nhầm tool; người dùng vẫn có một điểm tương tác.

🎯 **Predefined workflow** (thứ tự cố định, gọi FM trong từng bước) khi cần **dự đoán được, lặp lại được, dễ audit**; autonomous agent hoặc swarm khi task mở.

🎯 **Model Context Protocol (MCP)** = chuẩn mở để agent khám phá và gọi tool/data; giảm tích hợp từ M×N xuống M+N. **Strands Agents** = SDK open-source của AWS, **model-driven** (model tự chọn tool, kể cả tool MCP), vài dòng Python, chạy trên AgentCore Runtime.

🎯 LLM **stateless**: muốn nhớ hội thoại → **đưa lịch sử tin nhắn vào prompt** hoặc dùng **AgentCore Memory** (short-term trong session, long-term giữa các session).

🎯 **AgentCore — khớp dịch vụ:**

| Dịch vụ | Chức năng |
| --- | --- |
| **Runtime** | Chạy agent serverless, session isolation |
| **Gateway** | Biến API / Lambda thành **tool MCP** |
| **Memory** | Short-term + long-term memory |
| **Identity** | OAuth token vault, agent hành động **thay mặt user**, không hard-code credential |
| **Policy** | Cedar policy, gateway **chặn mọi tool call** vi phạm (vd đơn hàng > $10.000) |
| **Evaluations** | Đánh giá chất lượng agent, **online evaluation** trên session production |
| **Observability** | Trace, metric (CloudWatch) |

📌 **Agents for Amazon Bedrock** nay là **Agents Classic**, không nhận khách mới → **Bedrock AgentCore**.

---

## 19. Công cụ GenAI khác của AWS

🎯 **Bảng chọn:**

| Dịch vụ | Dùng khi |
| --- | --- |
| **Amazon Quick** | Trợ lý agentic cho nhân viên: Research (báo cáo có trích dẫn), Sight (BI), Flows/Automate, Index; không cần build app |
| **Kiro** | Agentic IDE/CLI, **spec-driven development**: requirements.md (EARS) → design.md → tasks.md |
| **AWS Transform** | Agent hiện đại hóa **.NET Framework, mainframe COBOL, VMware** lên AWS |
| **Amazon Q Developer** | Trợ lý AWS console/vận hành |
| **Amazon Q data integration in AWS Glue** | Sinh/sửa script ETL Glue bằng ngôn ngữ tự nhiên |
| **PartyRock** | Playground GenAI không code, không cần AWS account |

📌 Trợ lý AI doanh nghiệp, app không code → **Amazon Quick** (Amazon Q Business, Q Apps không nhận khách mới). BI có AI → **Amazon Quick Sight**. Trợ lý lập trình → **Kiro** (Q Developer IDE plugins hết hỗ trợ 30/04/2027). ChatOps trong Slack/Teams → **Amazon Q Developer in chat applications**.

---

## 20. Dịch vụ AI dựng sẵn

🎯 **Bảng khớp dịch vụ:**

| Dịch vụ | Use case |
| --- | --- |
| **Comprehend** | NLP: **sentiment**, entities, key phrases, ngôn ngữ, **PII detection/redaction**, **toxicity detection** (không cần data có nhãn) |
| **Comprehend Medical** | Trích thực thể y khoa, **PHI** từ ghi chú lâm sàng |
| **Textract** | OCR: text, **chữ viết tay**, form, bảng, **hóa đơn/biên lai**, PDF |
| **Rekognition** | Ảnh/video: vật thể, khuôn mặt, celebrity, **text trong ảnh (biển số)**, **content moderation**, custom labels |
| **Transcribe** | Speech-to-text, **phụ đề**, cuộc gọi tổng đài |
| **Transcribe Medical** | Speech-to-text **thuật ngữ y khoa**, HIPAA |
| **AWS HealthScribe** | Sinh **clinical notes** từ hội thoại/đọc chính tả của bác sĩ |
| **Polly** | Text-to-speech (khóa học audio đa ngôn ngữ) |
| **Translate** | Dịch văn bản, ứng dụng đa ngôn ngữ |
| **Lex** | Chatbot/voicebot (intent, slot) |
| **Personalize** | **Gợi ý cá nhân hóa** (vd recipe **Similar-Items**) |
| **Augmented AI (A2I)** | Human review dự đoán ML |
| **Mechanical Turk** | Lực lượng crowdsourcing (gắn nhãn) |
| **DeepRacer** | Xe đua mô hình để học reinforcement learning |

⚠️ **Textract = tài liệu/PDF/chữ viết tay**; **Rekognition = text ngắn trong ảnh/video (biển số)**, **không** đọc PDF. Sentiment cuộc gọi audio = **Transcribe → Comprehend**.

📌 Comprehend: tính năng *prompt safety classification* không nhận khách mới; **toxicity detection vẫn hoạt động**. **Amazon Forecast** không nhận khách mới từ 29/07/2024 → **SageMaker Canvas**. **Amazon Fraud Detector** không nhận khách mới từ 07/11/2025 → SageMaker AI / AutoGluon.

🎯 **Dịch vụ dữ liệu liên quan:** **Glue** (ETL, data catalog) · **Lake Formation** (quản trị, phân quyền data lake) · **Athena / EMR / Redshift** (SQL trên S3 / Spark / warehouse — **không** phải vector DB) · **OpenSearch Service** (search + vector DB).

---

## 21. Hạ tầng cho AI

🎯 **Bảng chọn:**

| Hạ tầng | Dùng khi |
| --- | --- |
| **AWS Trainium** (Trn) | Chip **train** deep learning; hiệu quả năng lượng tốt nhất cho training |
| **AWS Inferentia** (Inf) | Chip **inference** chi phí thấp |
| **EC2 P series** | GPU train |
| **EC2 G series** | GPU đồ họa & inference nhẹ |
| **EC2 C series** | CPU compute |

⚠️ **Lambda không host được LLM** (giới hạn bộ nhớ/thời gian).

---

## 22. Responsible AI — dimensions

🎯 **Dimension và tình huống:**

| Dimension | Dấu hiệu trong đề |
| --- | --- |
| **Fairness** | Giảm bias, tuyển dụng/cho vay công bằng, dữ liệu đa dạng mọi nhân khẩu học |
| **Transparency** | Công khai việc dùng AI, cách model hoạt động, giới hạn |
| **Explainability** | Lý do của từng đề xuất (điều trị, từ chối khoản vay) |
| **Privacy & security** | Ẩn danh dữ liệu cá nhân khi chuẩn bị data |
| **Safety** | Guardrails chặn output độc hại, lạm dụng |
| **Veracity & robustness** | Vẫn đúng khi input có **typo, diễn đạt lạ, cố tình gây nhiễu** |
| **Controllability** | Cơ chế giám sát, điều hướng hành vi AI |
| **Governance** | Chính sách, quy trình, kiểm toán |
| **Accountability** | Trách nhiệm giải trình |

⚠️ "Tolerance", "Flexibility", "Open source" **không** phải dimension.

🎯 **Human-in-the-loop** → giảm bias/toxicity ở **post-processing**; duyệt **thư rủi ro cao hoặc confidence thấp** trước khi gửi, thư thường vẫn tự động.

🎯 **Sustainability**: chọn **model nhỏ hơn** đủ chất lượng; train trên **Trainium**; không chạy song song nhiều model cho mọi request.

⚠️ **Rủi ro pháp lý:** ảnh giống nhân vật có bản quyền → **IP infringement**; dùng AI viết bài luận → **plagiarism**. ❓ Giải thích đề có nhắc AWS bồi thường IP không giới hạn cho output của Amazon Nova — cần kiểm chứng.

---

## 23. Bias & fairness

🎯 **Các loại bias:**

| Loại | Dấu hiệu |
| --- | --- |
| **Sampling bias** | Data không đại diện (camera chỉ gắn cờ một nhóm sắc tộc, 90% hồ sơ là nam) |
| **Measurement bias** | Công cụ đo lỗi |
| **Observer bias** | Người thu thập/diễn giải |
| **Confirmation bias** | Tìm thông tin khớp định kiến |
| **Algorithmic bias** | Model thiên vị một giới dù năng lực ngang nhau |
| **Human bias** | Người chọn feature theo niềm tin cá nhân |

🎯 **Sửa bias:** **data augmentation cho nhóm thiếu đại diện**; dataset **cân bằng, đa dạng, curated, kiểm tra label quality**; đo **class imbalance** rồi điều chỉnh training; fine-tune lại với data đa dạng (rẻ hơn pre-train).

🎯 **Điều tra bias:** **subgroup analysis** (error rate theo nhóm) + **human audit nhãn**. Accuracy tổng 92% có thể che giấu bias. Xóa cột nhân khẩu học **không đủ** (còn biến proxy như mã bưu chính).

⚠️ Regularization **không** sửa data imbalance; đổi model khác cũng không. Đánh giá bias định kỳ ít công nhất → **benchmark datasets**.

---

## 24. Explainability & transparency

🎯 **Interpretability** = hiểu **cơ chế bên trong**; **explainability** = giải thích **lý do output** cho stakeholder.

🎯 **SHAP (Shapley values)** → **local**, đóng góp từng feature cho **một dự đoán**. **PDP** → **global**, ảnh hưởng biên của một feature trên toàn dataset. SageMaker Clarify dùng cả hai.

⚠️ Lợi ích model minh bạch: **dễ debug, tối ưu** và **tăng niềm tin**. Không tự động ít compute hơn, không bảo mật hơn. **Trade-off**: model đơn giản dễ hiểu nhưng có thể kém chính xác hơn.

🎯 **Human-centered design**: nói rõ đây là đề xuất của AI, giải thích yếu tố chính, cho người dùng phản hồi; không che giấu, không đổi logic âm thầm.

⚠️ **Model Cards** (bạn viết cho model của bạn) vs **AWS AI Service Cards** (AWS công bố intended use, giới hạn, best practice của dịch vụ AWS).

🎯 Cơ quan cần **xem weights, tài liệu training data, license** → **open-weight model** có tài liệu và license cho phép kiểm tra.

⚠️ **Transparency vs safety**: công bố mục đích, giới hạn, loại safeguard ở mức tổng quát; **không** công bố nguyên system prompt và ngưỡng guardrail.

---

## 25. Guardrails for Amazon Bedrock

🎯 **Bảng filter:**

| Filter | Dùng khi |
| --- | --- |
| **Content filters** | Hate, Insults, Sexual, Violence, Misconduct, Prompt Attack; **strength riêng cho prompt và response** (None/Low/Medium/High) |
| **Denied topics** | Chặn **chủ đề** (tư vấn đầu tư/pháp lý, ý tưởng chính trị) |
| **Word filters** | Chặn **từ/cụm từ** cụ thể (tên đối thủ, từ tục) |
| **Sensitive information filters** | **Mask/block PII**, regex tùy chỉnh |
| **Contextual grounding check** | Response **không có trong nguồn** (grounding) hoặc **không trả lời câu hỏi** (relevance), ngưỡng 0–0.99 |
| **Automated Reasoning checks** | Kiểm tra response theo **luật logic** rút từ tài liệu chính sách, giải thích luật nào đúng/sai (bằng chứng audit), **detect mode** |
| **Prompt attack filter + input tags** | Jailbreak, prompt injection, prompt leakage; tag phần user input để không chặn nhầm system prompt |

📌 Content filter có **6** nhóm: **Hate, Insults, Sexual, Violence, Misconduct, Prompt Attack**.

🎯 Guardrails + **CloudWatch alarms** → lọc và **cảnh báo** khi vi phạm.

---

## 26. Rủi ro GenAI: hallucination, toxicity, prompt attacks

🎯 **Bảng rủi ro:**

| Rủi ro | Bản chất | Thời điểm | Phòng chống |
| --- | --- | --- | --- |
| **Hallucination** | Nghe đúng nhưng **sai/bịa** | Inference | RAG + **contextual grounding check**, Automated Reasoning check, giảm temperature, kiểm chứng |
| **Toxicity** | Nội dung **xúc phạm, độc hại** | Inference | Guardrails **content filters**, Comprehend toxicity detection, Rekognition moderation (ảnh) |
| **Plagiarism** | Nộp nội dung AI như của mình | Sử dụng | Công cụ phát hiện nội dung AI |
| **IP infringement** | Output giống tác phẩm có bản quyền | Sử dụng | Human review trước khi xuất bản |
| **Poisoning** | Cài data độc vào **training / fine-tuning / RAG source** | **Trước** inference | Kiểm soát, version, review dữ liệu |
| **Prompt injection** | Chỉ thị nhúng trong input (kể cả **text ẩn trong tài liệu upload**) để ghi đè chỉ dẫn | Inference | Guardrails prompt attack filter, validate input |
| **Hijacking** | Dạng injection: **đổi mục tiêu** ("bỏ qua chỉ dẫn, giờ bạn là chuyên gia chứng khoán") | Inference | Như trên |
| **Jailbreaking** | Vượt **rào an toàn** để lấy nội dung cấm ("giả vờ bạn không có luật") | Inference | Guardrails prompt attack filter |
| **Prompt leaking** | Lộ **system prompt / prompt trước đó** | Inference | Prompt attack filter (Standard tier) |
| **Exposure** | Lộ **dữ liệu nhạy cảm** đã học khi train | Inference | Xóa/mask PII trước khi train, Guardrails sensitive info filter |

❓ Chống prompt injection: một đề đáp "adversarial prompting", đề khác đáp thêm chỉ dẫn "ignore malicious content" vào prompt. Đáp án tốt nhất trên AWS là **Guardrails prompt attack filter + input tags**; không có thì chọn chỉ dẫn phòng thủ trong prompt/prompt template.

🎯 Chặn ảnh không phù hợp do chatbot trả về → **moderation API** (Rekognition).

---

## 27. Bảo mật: shared responsibility, IAM, mã hóa, mạng

🎯 **Shared responsibility (Bedrock):** **AWS** lo hạ tầng (phần cứng, mạng, cơ sở vật chất), vá lỗi dịch vụ. **Khách hàng** lo **dữ liệu (mã hóa in transit/at rest), IAM, cấu hình guardrails, ứng dụng**.

🎯 **IAM:**
- Giới hạn nhân viên chỉ dùng **model cụ thể** trên Bedrock → **IAM policies** (không phải STS, Inspector, service role).
- Mỗi team chỉ truy cập data của team trong RAG → **service role riêng cho từng team** (least privilege).
- Giới hạn FM nhân viên khám phá được trên SageMaker → **JumpStart private hub**.

❓ Một đề đáp "một role Bedrock full S3 + IAM role theo team" — cấp full S3 cho Bedrock vi phạm least privilege. Chọn **service role riêng cho từng team**.

🎯 **Mã hóa:** training data + customization job + custom model với **khóa công ty kiểm soát** (rotate, audit, disable) → **AWS KMS customer managed keys**. Mặc định Bedrock dùng **AWS owned keys** (không kiểm soát được). **TLS** chỉ mã hóa **in transit**. **Secrets Manager** dùng cho credential, không phải dataset.

🎯 **Mạng:**
- App trong VPC không Internet gọi Bedrock → **interface VPC endpoint (PrivateLink) cho bedrock-runtime**.
- SageMaker trong VPC đọc S3 không qua Internet → **VPC endpoint cho S3** (gateway hoặc interface).
- Fine-tune riêng tư, data ở lại Region, rẻ nhất → **Bedrock API + PrivateLink/VPC** (Outposts quá đắt).
- **NAT Gateway và Internet Gateway vẫn đi qua Internet** → đáp án sai.

---

## 28. Logging, audit & governance services

🎯 **Logging:**

| Nhu cầu | Dịch vụ |
| --- | --- |
| **Ai** gọi API Bedrock nào, khi nào | **AWS CloudTrail** |
| **Toàn bộ prompt + response** | **Bedrock model invocation logging** → S3 / CloudWatch Logs (**tắt mặc định**) |
| Metric, alarm, hiệu suất ML system | **Amazon CloudWatch** |
| Phản ứng sự kiện | **EventBridge** (không có nội dung input/output chi tiết) |
| Lưu log 5 năm rẻ nhất (trong lựa chọn của đề) | CloudTrail + **S3 Intelligent-Tiering** |

🎯 **Governance & compliance services:**

| Dịch vụ | Dùng khi đề nói |
| --- | --- |
| **AWS Artifact** | Báo cáo compliance của AWS (SOC, ISO, PCI), **ISO/IEC 42001** (AI management system), báo cáo **ISV bên thứ ba** + **email thông báo** khi có báo cáo mới, thỏa thuận (BAA) |
| **AWS Config** | Theo dõi **thay đổi cấu hình** + đánh giá theo rule (Conformance Packs cho framework) |
| **Amazon Inspector** | **Tự động đánh giá lỗ hổng** (EC2, ECR, Lambda) |
| **AWS Audit Manager** | Thu thập bằng chứng audit liên tục |
| **AWS Trusted Advisor** | Gợi ý best practice: cost, performance, security, fault tolerance, service limits |
| **Amazon Macie** | Tìm **PII trong S3** (tự động, ít công nhất) |
| **Amazon GuardDuty** | **Threat detection** real-time |

📌 **AWS Audit Manager không nhận khách mới từ 30/04/2026** → "continuously audit, collect evidence" dùng **AWS Config Conformance Packs**.

⚠️ **PII ở đâu → dùng gì:** PII trong S3 → **Macie** · PII trong text/support ticket → **Comprehend PII redaction** · PII trong response của model → **Guardrails sensitive info filter**.

---

## 29. Data governance & Generative AI Security Scoping Matrix

🎯 **Khái niệm data governance:**

| Khái niệm | Trả lời câu hỏi |
| --- | --- |
| **Data residency** | Dữ liệu lưu/xử lý **ở đâu** (không rời quốc gia) |
| **Data retention** | Giữ **bao lâu** |
| **Data logging** | Ghi lại **truy cập và thay đổi** |
| **Data access control** | **Ai** được truy cập (xác thực, phân quyền) |
| **Data integrity** | Dữ liệu **đúng, nhất quán, không bị sửa** |
| **Data lineage** | Nguồn gốc và biến đổi dữ liệu (privacy, compliance) |

⚠️ **Threat detection** = phát hiện tấn công đang diễn ra; **vulnerability management** = tìm và vá điểm yếu. **Defense in depth** = nhiều lớp: input validation, access control, giám sát liên tục.

⚠️ Mở rộng AI chấm điểm tín dụng sang nước khác → xem **luật algorithm accountability** của nước đó. Governance strategy: **ethical AI guidelines + audit output AI** (không: dùng open-source không đánh giá, fine-tune tự do, chỉ dựa vào feedback người dùng).

🎯 **GenAI Security Scoping Matrix — 5 scope** theo mức sở hữu: **Consumer app → Enterprise app → Pre-trained model → Fine-tuned model → Self-trained model** (scope 5 = **train từ đầu**, trách nhiệm bảo mật **cao nhất**).

🎯 **5 discipline:** Governance & compliance, Legal & privacy, **Risk management** (threat modeling, đề xuất mitigation), Controls, Resilience.

---

## 30. Phụ lục — Tra cứu dịch vụ & bẫy distractor

> Dịch vụ/phát biểu hay xuất hiện ở **đáp án SAI**. Nhớ "nó thực chất làm gì" để loại nhanh.

### Bedrock & GenAI

| Đáp án nhiễu | Thực chất |
| --- | --- |
| Spot / Reserved Instances cho Bedrock | Mô hình giá của EC2, không có trên Bedrock |
| "Train base FM trên Bedrock", "tạo model từ đầu bằng Bedrock" | Bedrock chỉ customize bản copy; train từ đầu dùng SageMaker |
| "Public copy" của FM khi fine-tune | Bản copy luôn private |
| "Host Bedrock API on premises" | Không tồn tại |
| Lambda để host LLM | Giới hạn bộ nhớ/thời gian |
| Guardrails để "lưu thông tin khách cho lần sau" | Guardrails chỉ lọc, không lưu |
| Watermark detection, Guardrails, Continued pre-training để "làm RAG" | Không liên quan RAG |
| "Tăng temperature để chống tấn công / bớt lặp" | Không phải biện pháp bảo mật, không giảm chi phí |
| "Model lớn hơn tốn ít token hơn" | Sai; model lớn đắt hơn mỗi token |
| Encryption để "ngăn model nói ra PII" | Mã hóa không ngăn model học/nhả PII |

### Dịch vụ AI dựng sẵn

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Rekognition** | Ảnh/video, text ngắn trong ảnh | Text/NLP, dịch, đọc PDF |
| **Polly** | Text-to-speech | Speech-to-text (→ Transcribe) |
| **Transcribe** | Speech-to-text | Làm chatbot (→ Lex) |
| **Comprehend** | NLP trên text | Speech-to-text |
| **Personalize** | Gợi ý cá nhân hóa | Lưu lịch sử hội thoại, tóm tắt, dự báo time series |
| **Fraud Detector** (📌 đóng với khách mới) | Phát hiện gian lận | Truy cập tài liệu khiếu nại |

### SageMaker

| Tính năng | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Feature Store** | Lưu & chia sẻ feature | "Lưu response để duyệt" |
| **Data Wrangler** | Chuẩn bị data | "Biến đổi response gửi khách" |
| **SageMaker Inference endpoint** | Endpoint phục vụ model | "Nối VPC với S3" (đó là VPC endpoint) |

### Data & vector DB

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **DynamoDB / DocumentDB / Redshift / Athena / EMR** | NoSQL / document / warehouse / SQL trên S3 / Spark | Làm vector DB cho RAG (trong đề luôn sai) |

### Security & governance

| Dịch vụ | Thực chất là gì | Hay là distractor khi… |
| --- | --- | --- |
| **Inspector** | Quét lỗ hổng EC2/ECR/Lambda | "Scan prompt/response" |
| **Config** | Ghi cấu hình tài nguyên | "Ghi nội dung prompt" |
| **Trusted Advisor** | Gợi ý best practice hạ tầng | "Ghi ai gọi model", "loại bỏ bias" |
| **Macie** | PII trong S3 | "Xoay vòng credential", "quét output model real-time" |
| **CloudWatch / Audit Manager** | Metric / bằng chứng audit | "Lưu log API" thay CloudTrail |
| **NAT Gateway / Internet Gateway** | Ra Internet | "Truy cập Bedrock không qua Internet" |
| **TLS** | Mã hóa in transit | "Mã hóa at rest" |

### Phát biểu đảo ngược / kiến trúc sai

| Phát biểu | Thực tế |
| --- | --- |
| "Validation set đo generalization", "Test set optional" | Đảo ngược |
| WaveNet sinh text/SQL, ResNet hiểu ngôn ngữ, CNN sinh synthetic data dạng bảng | Sai kiến trúc |
| "Tolerance", "Flexibility", "Open source" là dimension responsible AI | Không phải |
| LLM là tập cha của FM | LLM là **tập con** của FM |
| Regularization sửa data imbalance | Không |

---

## 31. Nguyên tắc làm bài chung

🎯 **"Least effort / most cost-effective" cho GenAI** → theo thứ tự **prompt engineering → RAG → fine-tuning → continued pre-training → pre-train**; chọn mức thấp nhất đáp ứng yêu cầu.

🎯 **Dữ liệu riêng + thay đổi thường xuyên** → RAG. **Đổi hành vi/giọng cố định cho task** → fine-tuning (labeled). **Hiểu cả lĩnh vực mới** → continued pre-training (unlabeled). **Model nhỏ hơn, rẻ hơn** → distillation.

🎯 **Task phổ biến (OCR, dịch, sentiment, speech)** → **AI service dựng sẵn**, không Bedrock/SageMaker. **GenAI ít vận hành** → Bedrock. **ML tùy biến, train từ đầu, kiểm soát hạ tầng** → SageMaker AI.

🎯 **Đọc đúng tầng của rủi ro:** data (poisoning, bias, PII trong training) xử lý **trước khi train**; prompt (injection, jailbreak) xử lý bằng **Guardrails lúc inference**; output (hallucination, toxicity, PII) bằng **grounding/content/sensitive filter**.

🎯 **Bảo mật:** "không qua Internet" → **PrivateLink/VPC endpoint**; "khóa do công ty kiểm soát" → **KMS customer managed key**; "ai gọi API" → **CloudTrail**; "nội dung prompt/response" → **invocation logging**; "least privilege" → **role riêng, policy hẹp**.

🎯 **Cách loại đáp án SAI:** (1) **dịch vụ sai loại dữ liệu** (Rekognition cho text, Polly cho speech-to-text); (2) **khái niệm EC2 áp sang Bedrock** (Spot, RI); (3) **đảo ngược định nghĩa** (LLM ⊃ FM, test optional, SHAP global); (4) **dịch vụ governance làm việc không phải của nó** (Inspector quét prompt, Config ghi prompt).

⚠️ **Chủ đề lặp lại ở gần như mọi đề (6–7/7):** inference parameters · RAG vs fine-tuning · overfitting/underfitting · SageMaker features · inference types · prompt techniques · metrics · loại learning · Guardrails · bias & fairness · explainability · governance services · Bedrock pricing · dịch vụ AI dựng sẵn. Ít hơn (4–5/7): prompt attacks, bảo mật mạng & mã hóa, token/embedding/context window, agentic AI (xu hướng mới).

---

## 32. Bảng so sánh dễ nhầm (tổng hợp)

| Cặp dễ nhầm | Khác nhau ở đâu | Dấu hiệu chọn |
| --- | --- | --- |
| AI vs ML vs DL vs GenAI | Lồng nhau, AI rộng nhất | "rule-based" → AI không phải ML |
| Computer vision vs Image processing | Hiểu nội dung / xử lý pixel | "detect, classify" → CV |
| Algorithm vs Model | Công thức / kết quả sau train | — |
| Hyperparameter vs Parameter | Đặt trước / model tự học | "learning rate, epochs" → hyperparameter |
| Supervised vs Unsupervised vs Self-supervised | Có nhãn / không nhãn / tự tạo nhãn | "pre-train FM" → self-supervised |
| K-means vs k-NN | Unsupervised gom cụm / supervised láng giềng | "phân khúc khách hàng" → K-means |
| Transfer vs Incremental vs Federated learning | Dùng lại model / thêm data mới / data không gom về | "privacy, không gom data" → Federated |
| Feature extraction vs Feature selection | Biến đổi sang không gian mới / chọn tập con | "PCA" → extraction |
| Validation vs Test set | Tune (optional) / đo generalization | "final evaluation" → test |
| EDA vs Data preparation vs Data augmentation | Hiểu data / làm sạch / tạo biến thể | "correlation matrix" → EDA |
| Overfitting vs Underfitting | High variance / high bias | "train tốt, production kém" → overfit |
| Precision vs Recall vs F1 | Giảm FP / giảm FN / data mất cân bằng | "bỏ sót ung thư" → Recall |
| BLEU vs ROUGE vs BERTScore | Dịch / tóm tắt / ngữ nghĩa | "summarization" → ROUGE |
| CNN vs RNN vs Transformer | Ảnh / chuỗi / self-attention | "video, time series" → RNN |
| GAN vs VAE vs Diffusion | Đối kháng / latent space / khử nhiễu | "synthetic data giữ thống kê" → GAN |
| BERT vs Word2Vec | Contextual / static embeddings | "điền từ bị thiếu" → BERT |
| Real-time vs Serverless vs Async vs Batch inference | Ngay / thưa, cold start / ≤1 GB hàng đợi / cả dataset | "nhiều GB mỗi ngày" → Batch |
| Generative vs Discriminative | Tạo mới / phân loại | "dự đoán giá nhà" → không phải GenAI |
| FM vs LLM | LLM là tập con của FM | — |
| Token vs Embedding vs Context window | Đơn vị text / vector ngữ nghĩa / giới hạn token | "vector so sánh văn bản" → embedding |
| Temperature vs Top K vs Top P | Sáng tạo / số lượng / phần trăm | "giới hạn số token ứng viên" → Top K |
| Zero vs One vs Few-shot | 0 / 1 / vài ví dụ | "nhiều ví dụ" → few-shot |
| Chain-of-thought vs Prompt chaining vs ReAct | Lý luận từng bước / tách subtask tuần tự / lý luận + gọi tool | "gọi API kiểm tra tồn kho" → ReAct |
| Prompt engineering vs Context engineering | Hỏi thế nào / cho model thấy gì | — |
| RAG vs Fine-tuning | Không đổi weights, data thay đổi / đổi weights, labeled | "data cập nhật liên tục" → RAG |
| Fine-tuning vs Continued pre-training | Labeled, task / unlabeled, domain | "unlabeled chuyên ngành" → CPT |
| Distillation vs Fine-tuning | Model nhỏ hơn / model chuyên hơn | "nhanh hơn, rẻ hơn" → distillation |
| RLHF vs Human evaluation | Đổi model / chỉ chấm điểm | "xếp hạng response để train" → RLHF |
| Automatic vs LLM-as-a-judge vs Human evaluation | Định lượng / không cần reference / định tính | "SME đánh giá tone" → human |
| On-demand vs Provisioned Throughput vs Batch | Khó đoán / ổn định cao / chấp nhận trễ | "steady high traffic" → PT |
| Bedrock vs SageMaker AI vs AI services | GenAI serverless / ML tùy biến / API dựng sẵn | "OCR hóa đơn" → Textract |
| JumpStart vs Bedrock | Deploy lên endpoint của bạn / serverless | "chọn instance" → JumpStart |
| Model Cards vs AI Service Cards vs Model Dashboard | Tài liệu model của bạn / tài liệu AWS / xem mọi model | "intended use, risk rating" → Model Cards |
| Clarify vs Model Monitor | Bias + explainability / drift | "drift production" → Model Monitor |
| Data Wrangler vs Feature Store vs Ground Truth | Chuẩn bị / chia sẻ feature / gắn nhãn người | "300 transform" → Data Wrangler |
| Textract vs Rekognition | Tài liệu, PDF, viết tay / ảnh, video, biển số | "hóa đơn" → Textract |
| Transcribe vs Polly | Speech → text / text → speech | "phụ đề" → Transcribe |
| Comprehend vs Comprehend Medical vs HealthScribe | NLP chung / PHI y khoa / clinical notes | "ghi chú lâm sàng từ hội thoại" → HealthScribe |
| Trainium vs Inferentia | Train / inference | "giảm carbon khi train" → Trainium |
| Interpretability vs Explainability | Cơ chế bên trong / lý do output | — |
| SHAP vs PDP | Local / global | "một dự đoán cụ thể" → SHAP |
| Sampling vs Algorithmic vs Human bias | Data không đại diện / model thiên vị / người chọn feature | "90% hồ sơ là nam" → sampling |
| Denied topics vs Word filters vs Content filters | Chủ đề / từ cụ thể / nhóm nội dung độc hại | "tên đối thủ" → word filter |
| Contextual grounding vs Automated Reasoning | Có trong nguồn không / đúng luật logic không | "luật chính sách phức tạp, audit" → Automated Reasoning |
| Prompt injection vs Hijacking vs Jailbreaking vs Leaking | Ghi đè chỉ dẫn / đổi mục tiêu / phá rào an toàn / lộ system prompt | "giả vờ không có luật" → jailbreak |
| Poisoning vs Exposure | Cài data độc trước train / lộ data đã học | "nhả số tài khoản khách khác" → exposure |
| CloudTrail vs Invocation logging | Ai gọi API / nội dung prompt-response | "prompts & responses" → invocation logging |
| AWS owned key vs Customer managed key | Mặc định, không kiểm soát / tự rotate, audit, disable | "công ty kiểm soát khóa" → customer managed |
| Interface endpoint vs NAT Gateway | Riêng tư / vẫn ra Internet | "không qua Internet" → endpoint |
| Artifact vs Config vs Inspector vs Audit Manager | Báo cáo compliance / cấu hình / lỗ hổng / bằng chứng audit | "ISO 42001 report" → Artifact |
| Macie vs Comprehend PII vs Guardrails sensitive info | PII trong S3 / trong text / trong response model | — |
| Data residency vs Retention vs Lineage | Ở đâu / bao lâu / nguồn gốc | "không rời quốc gia" → residency |

---

## 33. Từ khóa → đáp án (keyword mapping nhanh)

| Từ khóa trong đề | Đáp án |
| --- | --- |
| bài toán có đáp án tính chính xác | Rule-based, không dùng ML |
| thống kê, visualize, correlation matrix | EDA |
| dự đoán giá trị số | Regression |
| 2 lớp | Binary classification |
| nhóm khách hàng, không nhãn | Unsupervised / K-means |
| giảm chiều | PCA |
| tự cải thiện từ feedback/reward | Reinforcement learning |
| dùng lại kiến thức model khác | Transfer learning |
| dữ liệu nhạy cảm, không gom về | Federated learning |
| ít nhãn + nhiều không nhãn | Semi-supervised |
| dự báo nhu cầu, time series | DeepAR / SageMaker Canvas |
| train tốt, production kém | Overfitting → tăng regularization / thêm data |
| kém cả train lẫn test | Underfitting |
| xem và chỉnh weights | Logistic regression |
| interpretable nhất | Decision tree |
| phân loại ảnh | CNN |
| video / chuỗi | RNN |
| self-attention | Transformer |
| dữ liệu tổng hợp giữ thống kê gốc | GAN |
| điền từ bị thiếu | BERT |
| định vị nhiều vật thể trong ảnh | Object detection |
| false positive đắt | Precision |
| false negative đắt | Recall |
| data mất cân bằng | F1 |
| dịch máy | BLEU |
| tóm tắt | ROUGE |
| tương đồng ngữ nghĩa, văn phong lạ | BERTScore |
| runtime efficiency | Average response time |
| nhiều GB mỗi ngày, không cần ngay | Batch transform |
| payload lớn ≤ 1 GB, gần real-time | Async inference |
| traffic thất thường, chịu cold start | Serverless inference |
| ngay lập tức, endpoint thường trực | Real-time inference |
| latency thấp nhất trên thiết bị | SLM tại edge |
| 300 transform, không code, chia tập | Data Wrangler |
| chia sẻ feature giữa team | Feature Store |
| bias + giải thích feature | SageMaker Clarify |
| drift production | Model Monitor |
| xem mọi model một chỗ | Model Dashboard |
| intended use, risk rating | Model Cards |
| gắn nhãn với con người | Ground Truth |
| experiment tracking | MLflow on SageMaker |
| tự tune hyperparameter | Automatic Model Tuning |
| no-code cho business analyst | SageMaker Canvas |
| giới hạn model nhân viên thấy | JumpStart private hub |
| tạo mới / generate | GenAI |
| FM học thế nào | Self-supervised, data không nhãn |
| LLM so với FM | LLM là tập con |
| giới hạn số text model xem cùng lúc | Context window |
| vector ngữ nghĩa | Embedding |
| hiểu text + ảnh, rẻ | Multimodal embedding model |
| nhất quán, deterministic | Temperature thấp / 0 |
| giới hạn số token ứng viên | Top K |
| bám prompt khi tạo ảnh | CFG scale cao |
| giảm chi phí token | Prompt ngắn, max tokens, prompt caching, model nhỏ |
| think step by step | Chain-of-thought |
| tách task thành bước tuần tự | Prompt chaining |
| lý luận + gọi tool | ReAct |
| loại bỏ chi tiết/tên đối thủ | Negative prompting |
| đổi văn phong, ít công nhất | Prompt engineering |
| version, variant, variable cho prompt | Bedrock Prompt Management |
| dữ liệu công ty, cập nhật liên tục, rẻ | RAG / Knowledge Bases |
| thấy nguồn từng câu | Citations (RetrieveAndGenerate) |
| vector DB cho RAG | OpenSearch Service / Aurora PostgreSQL pgvector |
| labeled + task cụ thể | Fine-tuning |
| unlabeled + chuyên ngành | Continued pre-training |
| model nhỏ, nhanh, rẻ, dùng invocation logs | Distillation |
| con người xếp hạng response để train | RLHF |
| data fine-tune có trùng lặp/nhãn sai | Làm sạch + guideline nhãn |
| PII trong data training | Mask/tokenize hoặc synthetic data trước khi train |
| đánh giá nhiều prompt, không có reference | LLM-as-a-judge |
| đánh giá định tính, SME, tone | Human evaluation (custom dataset) |
| định lượng, ít công nhất | Automatic evaluation |
| đo accuracy/bias ít công | Benchmark datasets |
| chưa biết lượng dùng, không cam kết | On-demand |
| steady high traffic | Provisioned Throughput |
| chấp nhận trễ, rẻ nhất | Batch inference |
| nhiều tool, agent chọn nhầm | Multi-agent supervisor |
| auditor yêu cầu thứ tự cố định | Predefined workflow |
| chuẩn kết nối agent với tool | MCP |
| SDK open-source viết agent Python | Strands Agents |
| agent nhớ sở thích | AgentCore Memory |
| OAuth thay mặt user | AgentCore Identity |
| biến API thành tool | AgentCore Gateway |
| chặn tool call vượt ngưỡng | Policy in AgentCore |
| online evaluation agent | AgentCore Evaluations |
| requirements, design, tasks cho feature | Kiro |
| chuyển .NET/COBOL/VMware | AWS Transform |
| trợ lý nhân viên, research có trích dẫn | Amazon Quick |
| playground không cần account | PartyRock |
| sentiment / entities / PII trong text | Comprehend |
| PHI y khoa | Comprehend Medical |
| hóa đơn, form, chữ viết tay, PDF | Textract |
| biển số, vật thể, moderation ảnh | Rekognition |
| phụ đề, speech-to-text | Transcribe |
| clinical notes từ hội thoại | HealthScribe |
| text-to-speech | Polly |
| chatbot intent/slot | Lex |
| gợi ý sản phẩm tương tự | Personalize (Similar-Items) |
| human review dự đoán | Augmented AI (A2I) |
| chip train tiết kiệm năng lượng | Trainium |
| chip inference rẻ | Inferentia |
| dataset đủ mọi nhân khẩu học | Diversity / Fairness |
| một nhóm bị gắn cờ nhiều hơn | Sampling bias → augmentation |
| audit bias khi có khiếu nại | Subgroup analysis + human label audit |
| typo, câu lừa | Veracity & robustness |
| local explanation | SHAP |
| global explanation | PDP |
| xem weights, license | Open-weight model |
| chặn chủ đề | Denied topics |
| chặn từ | Word filters |
| che PII trong response | Sensitive information filter |
| không có trong tài liệu nguồn | Contextual grounding check |
| luật phức tạp, cần giải thích, audit | Automated Reasoning checks |
| jailbreak / injection / leakage | Prompt attack filter + input tags |
| giảm carbon | Model nhỏ hơn / Trainium |
| who called which API | CloudTrail |
| prompts & responses | Model invocation logging |
| không qua Internet | VPC endpoint / PrivateLink |
| khóa do công ty quản lý | KMS customer managed key |
| giới hạn model cụ thể cho nhân viên | IAM policy |
| mỗi team chỉ data của team | Service role riêng mỗi team |
| PII trong S3 | Macie |
| báo cáo SOC/ISO/ISO 42001, ISV | Artifact |
| thay đổi cấu hình | Config |
| lỗ hổng | Inspector |
| best practice cost/limits | Trusted Advisor |
| bằng chứng audit liên tục | Audit Manager (📌 nay Config Conformance Packs) |
| dữ liệu không rời quốc gia | Data residency |
| nguồn gốc dữ liệu | Data lineage |
| train từ đầu = trách nhiệm bảo mật nhiều nhất | Scope 5 |
| threat modeling | Risk management |

---

## 34. Con số cần thuộc & đáp án bất khả thi

### Con số phải nhớ

| Hạng mục | Con số |
| --- | --- |
| SageMaker real-time inference | Payload ≤ 25 MB |
| SageMaker async inference | Payload ≤ 1 GB, xử lý tới 1 giờ |
| Bedrock batch inference | Rẻ hơn tới 50% on-demand |
| Provisioned Throughput | Cam kết 1 hoặc 6 tháng (hoặc không cam kết) |
| Distillation | Nhanh hơn tới 5 lần, rẻ hơn tới 75%, mất < 2% accuracy (với RAG) |
| Data Wrangler | 300+ transform |
| Guardrails content filter | 6 nhóm: Hate, Insults, Sexual, Violence, Misconduct, Prompt Attack; strength None/Low/Medium/High |
| Contextual grounding threshold | 0–0.99 |
| Human evaluation | So sánh tối đa 2 model |
| Prompt tốt | 4 thành phần: Instructions, Context, Input data, Output indicator |
| Security Scoping Matrix | 5 scope, 5 discipline |
| Temperature | 0 = ổn định nhất |
| Invocation logging | Tắt mặc định |

### Đáp án BẤT KHẢ THI → loại ngay

- **Spot / Reserved Instances cho Bedrock**.
- **Train base FM từ đầu trên Bedrock** (dùng SageMaker).
- **Fine-tune tạo bản copy public** (luôn private).
- **Bedrock dùng data của bạn để train base model / chia sẻ với provider**.
- **Host Bedrock API on-premises**.
- **Lambda host LLM**.
- **Temperature/Top K/Top P thay đổi giá hoặc latency**.
- **Fine-tuning dùng data unlabeled** (đó là continued pre-training).
- **RAG đổi weights của model**.
- **Mã hóa KMS ngăn model học/nhả PII**.
- **TLS mã hóa at rest**.
- **NAT Gateway / Internet Gateway giúp truy cập không qua Internet**.
- **CloudTrail ghi nội dung prompt/response** (đó là invocation logging).
- **Inspector quét prompt; Config ghi prompt; Trusted Advisor loại bỏ bias; Macie quét output model real-time**.
- **Guardrails lưu thông tin khách hàng**.
- **DynamoDB / DocumentDB / Redshift / Athena / EMR làm vector DB cho RAG** (trong đề).
- **Rekognition đọc PDF / làm NLP; Polly làm speech-to-text; Comprehend làm speech-to-text**.
- **Regularization sửa data imbalance**.
- **Test set optional / validation set đo generalization**.
- **LLM là tập cha của FM**.
- **"Tolerance", "Flexibility", "Open source" là dimension responsible AI**.

---

### Nguồn đã đối chiếu (các ghi chú 📌 và ❓)

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
- Q Developer IDE plugins end of support (→ Kiro): https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-developer-ide-end-of-support.html
