# AIF-C01 — AI & Machine Learning Overview

> Chương nền tảng của **Domain 1 (Fundamentals of AI and ML, 20%)**: các loại learning, dữ liệu, metric, fit, vòng đời ML. Phần lớn câu hỏi "lý thuyết thuần" của đề nằm ở đây.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời. Mốc kiến thức: 10/2026.

---

## 1. TL;DR

- **AI ⊃ ML ⊃ Deep Learning ⊃ GenAI.** AI không nhất thiết là ML: expert system (vd: MYCIN, khoảng 500 luật if-then) là AI nhưng **không học** từ dữ liệu.
- **Supervised** (data có nhãn): **Regression** (đoán số liên tục) và **Classification** (đoán nhãn). **Unsupervised** (không nhãn): **Clustering**, **Association rules**, **Anomaly detection**.
- Ngoài ra còn **Semi-supervised** (ít nhãn + pseudo-labeling), **Self-supervised** (tự tạo nhãn từ data, nền tảng của GPT/BERT), **Reinforcement Learning** (agent tối đa hóa reward), **RLHF** (reward từ đánh giá của con người).
- **Overfitting** = giỏi trên training, kém trên data mới (**high variance**). **Underfitting** = kém ngay trên training (**high bias**). Mục tiêu là low bias, low variance.
- Metric **classification**: Confusion matrix, **Precision** (FP tốn kém), **Recall** (FN tốn kém), **F1** (cân bằng, data mất cân bằng), Accuracy, **AUC-ROC**. Metric **regression**: **MAE, MAPE, RMSE, R²**.
- Inference: **real-time** (nhanh, từng request) và **batch** (lô lớn, không gấp); **edge** chạy model nhỏ (SLM) tại chỗ, offline, latency rất thấp.
- Vòng đời ML: Business goal → Problem framing → Data → Feature engineering → Training & tuning → Evaluation → Deployment → Monitoring → lặp lại.
- **Không dùng ML** cho bài toán **deterministic** (tính được chính xác bằng code): ML chỉ cho kết quả xấp xỉ.

---

## 2. Core Concepts

### AI, ML, DL, GenAI
- **Là gì:** AI là "ô" lớn gồm perception, reasoning, learning, problem solving, decision-making; ML là AI **học từ dữ liệu**, không lập trình luật tường minh.
- **Dùng khi nào:** DL dùng **neural network nhiều lớp**, cần nhiều data và **GPU**, mạnh cho Computer Vision và NLP; GenAI là DL dùng **foundation model** pre-train trên data unlabeled để **tạo nội dung**.
- **Vì sao quan trọng:** ví von của slide: luật if-then = AI; nhận ra thứ đã thấy nhiều = ML; suy ra từ khái niệm đã học = DL; sáng tạo nội dung mới = GenAI.

### AI Components (4 layer)
- **Là gì:** **Data layer** (thu thập dữ liệu) → **ML framework & algorithm layer** (chọn framework, thuật toán theo use case) → **Model layer** (cấu trúc, parameters, optimizer; train model) → **Application layer** (phục vụ model cho user).
- **Dùng khi nào:** dùng để định vị một công việc nằm ở tầng nào (vd: "deploy model ra API cho user" = application layer).
- **Vì sao quan trọng:** giúp phân vai giữa data scientist, data engineer và người dùng cuối.

### Neural Networks & Deep Learning
- **Là gì:** các **node** xếp thành **layer** (input → hidden → output); khi thấy nhiều data, mạng tự điều chỉnh **kết nối (weights)** giữa các node để nhận ra pattern.
- **Dùng khi nào:** vd nhận diện chữ số viết tay: input = pixel, hidden layer học nét thẳng/nét cong, output = chữ số có xác suất cao nhất.
- **Vì sao quan trọng:** "Deep" = **nhiều hơn một hidden layer**. Pattern do mạng **tự học**, không do người lập trình.

### Transformer, Diffusion, Multi-modal
- **Là gì:** **Transformer** xử lý **cả câu cùng lúc** (không từng từ một) và dùng **self-attention** để đánh trọng số độ quan trọng của từng từ; đây là nền tảng của LLM (GPT = Generative Pre-trained **Transformer**, BERT).
- **Dùng khi nào:** **Diffusion** sinh ảnh bằng cách khử nhiễu (vd: Stable Diffusion); **Multi-modal** nhận và/hoặc sinh **nhiều loại dữ liệu** (text + ảnh + audio → video...).
- **Vì sao quan trọng:** Transformer train nhanh hơn và cho câu mạch lạc hơn các kiến trúc tuần tự như RNN.

### Training Data: Labeled/Unlabeled, Structured/Unstructured
- **Là gì:** **Labeled** = có input + nhãn output (→ supervised); **Unlabeled** = chỉ có input (→ unsupervised). **Structured** = tabular, time series; **Unstructured** = text, ảnh, audio.
- **Dùng khi nào:** loại dữ liệu quyết định chọn được thuật toán nào. Data là **giai đoạn quan trọng nhất**: *garbage in, garbage out*.
- **Vì sao quan trọng:** `[+]` gắn nhãn dữ liệu cho supervised learning trên AWS: **Amazon SageMaker Ground Truth** (người gắn nhãn + gắn nhãn tự động).

### Supervised Learning: Regression & Classification
- **Là gì:** học hàm ánh xạ input → output từ **labeled data** để dự đoán cho data mới; mạnh nhưng tốn công gắn nhãn khi có hàng triệu điểm dữ liệu.
- **Dùng khi nào:** **Regression** cho output **liên tục** (giá nhà, giá cổ phiếu, nhiệt độ); **Classification** cho output **rời rạc**: binary (spam/không), multiclass (thú/chim/bò sát), multi-label (một phim vừa action vừa comedy).
- **Vì sao quan trọng:** thuật toán tiêu biểu: linear regression; **k-NN** cho classification. Fraud, chẩn đoán, customer churn đều là classification.

### Training / Validation / Test Set
- **Là gì:** **Training** (60–80%) để train; **Validation** (10–20%) để tune hyperparameter và kiểm tra trong lúc phát triển; **Test** (10–20%) để đánh giá **cuối cùng**.
- **Dùng khi nào:** test set phải là data model **chưa từng thấy**, để đo khả năng tổng quát hóa.
- **Vì sao quan trọng:** đề hỏi "tập nào dùng để tune hyperparameter?" thì đáp án là **validation**.

### Feature Engineering
- **Là gì:** dùng kiến thức nghiệp vụ để biến raw data thành feature có ý nghĩa: **Extraction** (ngày sinh → tuổi), **Selection** (giữ feature quan trọng), **Transformation** (normalize), **Creation** (giá / m²).
- **Dùng khi nào:** structured data: tạo, chọn, normalize feature; unstructured: text → số bằng **TF-IDF / word embeddings**, ảnh → cạnh, texture bằng **CNN**.
- **Vì sao quan trọng:** đặc biệt hữu ích cho supervised learning; normalize giúp thuật toán (vd: gradient descent) hội tụ nhanh hơn.

### Unsupervised Learning
- **Là gì:** tự tìm pattern trong **unlabeled data**; máy tự tạo nhóm, con người đặt tên nhóm sau.
- **Dùng khi nào:** **Clustering** (**K-means**; customer segmentation, targeted marketing, recommender); **Association rules** (**Apriori**; market basket: bánh mì → bơ); **Anomaly detection** (**Isolation Forest**; giao dịch gian lận).
- **Vì sao quan trọng:** đề cho tình huống "**không có nhãn**, muốn chia nhóm khách hàng" thì đáp án là clustering.

### Semi-supervised & Self-supervised Learning
- **Là gì:** **Semi-supervised** = ít data có nhãn + nhiều data không nhãn; model tự gắn nhãn phần còn lại (**pseudo-labeling**) rồi train lại.
- **Dùng khi nào:** **Self-supervised** = model tự tạo nhãn từ chính data qua **pretext task** (vd: đoán từ bị che), học biểu diễn ngôn ngữ, rồi giải **downstream task**.
- **Vì sao quan trọng:** self-supervised là cách **GPT và BERT** được pre-train trên lượng text khổng lồ mà không cần người gắn nhãn.

### Reinforcement Learning (RL)
- **Là gì:** **Agent** quan sát **State** của **Environment**, chọn **Action** theo **Policy**, nhận **Reward**, cập nhật policy để **tối đa reward tích lũy**.
- **Dùng khi nào:** game (Chess, Go), robotics, tài chính (danh mục, trading), y tế (tối ưu phác đồ), xe tự lái (lập lộ trình).
- **Vì sao quan trọng:** vd robot đi mê cung: -1 mỗi bước, -10 đâm tường, +100 tới lối ra; robot học dần qua nhiều lần mô phỏng.

### RLHF (Reinforcement Learning from Human Feedback)
- **Là gì:** đưa **đánh giá của con người** vào reward function để model sát với mong muốn của con người hơn; dùng rộng rãi cho LLM.
- **Dùng khi nào:** quy trình 4 bước: (1) thu thập prompt + câu trả lời do người viết → (2) supervised fine-tune model → (3) người chọn câu trả lời họ thích hơn để train **reward model** → (4) dùng reward model tối ưu LLM bằng RL (tự động hóa được).
- **Vì sao quan trọng:** vd chuyển bản dịch từ "đúng kỹ thuật" sang "tự nhiên như người viết".

### Model Fit, Bias & Variance
- **Là gì:** **Bias** = sai lệch giữa dự đoán và thực tế do chọn model sai (vd: linear cho data phi tuyến); **Variance** = model thay đổi bao nhiêu khi train trên data khác có cùng phân phối.
- **Dùng khi nào:** **Underfitting = high bias** → giảm bằng model phức tạp hơn, thêm feature. **Overfitting = high variance** → giảm bằng feature selection, chia train/test nhiều lần.
- **Vì sao quan trọng:** **Balanced = low bias + low variance**. Bias ở đây là **thống kê**, khác "bias" về công bằng trong Responsible AI.

### Classification Metrics: Confusion Matrix & AUC-ROC
- **Là gì:** **Precision** = TP / (TP + FP); **Recall** = TP / (TP + FN); **F1** = trung bình điều hòa của Precision và Recall; **Accuracy** = (TP + TN) / tổng.
- **Dùng khi nào:** Precision khi **FP đắt** (vd: chặn nhầm email quan trọng); Recall khi **FN đắt** (vd: bỏ sót ung thư, gian lận); F1 khi **data mất cân bằng**; Accuracy chỉ khi data **cân bằng**.
- **Vì sao quan trọng:** **AUC-ROC** (0 → 1, 1 = hoàn hảo, 0.5 = đoán ngẫu nhiên) vẽ TPR (sensitivity) theo FPR (1 − specificity) ở nhiều threshold, dùng để so sánh model và chọn threshold.

### Regression Metrics
- **Là gì:** **MAE** (sai số tuyệt đối trung bình), **MAPE** (sai số % trung bình), **RMSE** (căn của sai số bình phương trung bình, phạt mạnh sai số lớn) đều đo **độ sai**; **R²** đo **% biến thiên được model giải thích**.
- **Dùng khi nào:** vd RMSE = 5 nghĩa là dự đoán điểm thi lệch khoảng 5 điểm; R² = 0.8 nghĩa là 80% thay đổi của điểm thi giải thích được bằng số giờ học.
- **Vì sao quan trọng:** đề hay đánh tráo metric classification và regression (xem bảng mục 4).

### Inferencing: Real-time, Batch, Edge
- **Là gì:** **Real-time** = quyết định ngay khi data tới, ưu tiên tốc độ (chatbot); **Batch** = phân tích lô lớn một lần, ưu tiên độ chính xác hơn tốc độ.
- **Dùng khi nào:** **Edge**: **SLM** chạy trên thiết bị (vd: Raspberry Pi), latency rất thấp, chạy offline; hoặc gọi **LLM** trên server qua Internet, mạnh hơn nhưng latency cao và cần online.
- **Vì sao quan trọng:** `[+]` trên SageMaker AI có 4 lựa chọn: **Real-time** endpoint, **Serverless** (traffic thưa, chịu được cold start), **Asynchronous** (payload lớn tới 1 GB, xử lý lâu, gần real-time), **Batch Transform** (cả dataset).

### Phases of an ML Project
- **Là gì:** Business goal (KPI, ngân sách) → **ML problem framing** (ML có phù hợp không) → Data collection & preparation → **Feature engineering** → Training & tuning → Evaluation → Deployment → **Monitoring & debugging** → lặp lại.
- **Dùng khi nào:** **EDA** (Exploratory Data Analysis): trực quan hóa data, dùng **correlation matrix** để xem các biến liên quan nhau đến mức nào và chọn feature.
- **Vì sao quan trọng:** không đạt business goal thì quay lại **data augmentation / feature augmentation** và train lại. `[+]` Giám sát model trong production trên AWS: **SageMaker Model Monitor** (phát hiện drift về data quality, model quality, bias, feature attribution).

### Hyperparameter Tuning
- **Là gì:** **hyperparameter** = thiết lập **trước khi train** (learning rate, batch size, epochs, regularization); khác `[+]` **parameter** (weights) mà model **tự học** trong lúc train.
- **Dùng khi nào:** **Learning rate** cao thì hội tụ nhanh nhưng dễ vượt qua điểm tối ưu; **batch size** nhỏ ổn định hơn nhưng chậm; **epochs** quá ít → underfit, quá nhiều → overfit; tăng **regularization** → giảm overfitting.
- **Vì sao quan trọng:** tune bằng grid search, random search hoặc **SageMaker Automatic Model Tuning (AMT)**. `[+]` AMT hỗ trợ grid, random, **Bayesian optimization**, **Hyperband**.

### Chống Overfitting & khi nào KHÔNG dùng ML
- **Là gì:** overfitting do data ít hoặc không đại diện, train quá lâu, hoặc model quá phức tạp học cả "noise".
- **Dùng khi nào:** khắc phục bằng: **tăng data**, **early stopping**, **data augmentation**, chỉnh hyperparameter, **ensembling** (kết hợp nhiều model).
- **Vì sao quan trọng:** bài toán **deterministic** (vd: xác suất rút được lá bài xanh) thì viết code tính trực tiếp, vì ML (kể cả LLM có reasoning) chỉ cho kết quả **xấp xỉ**.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Expert system | AI dựa trên luật viết tay (vd: MYCIN), không học từ data | "rule-based", "if-then rules" |
| Deep Learning | ML dùng neural network nhiều lớp, cần GPU và nhiều data | "multiple hidden layers", "complex patterns", "GPU" |
| Neural network | Node xếp theo layer, học bằng cách điều chỉnh kết nối | "input/hidden/output layer", "nodes" |
| Transformer | Kiến trúc xử lý cả câu cùng lúc, dùng self-attention | "attention", "LLM architecture", "GPT/BERT" |
| GPT | Generative Pre-trained Transformer: sinh text/code từ prompt | "generate text from prompt" |
| BERT | Bidirectional Encoder Representations from Transformers: đọc text **hai chiều** | "bidirectional", "understand context both directions" |
| RNN | Recurrent Neural Network cho dữ liệu **tuần tự** (time series, text, speech) | "sequential data", "time-series prediction" |
| CNN / ResNet | Convolutional NN (ResNet là một loại) cho **ảnh**: nhận diện, phát hiện vật thể, khuôn mặt | "image recognition", "object detection" |
| SVM | Support Vector Machine: thuật toán classification và regression | "classic ML classifier" |
| WaveNet | Model sinh **sóng âm thanh thô**, dùng cho speech synthesis | "generate raw audio", "text-to-speech model" |
| GAN | Generative Adversarial Network: sinh **dữ liệu tổng hợp** (ảnh, video, âm thanh) | "synthetic data", "data augmentation" |
| XGBoost | Extreme Gradient Boosting, mạnh với data dạng bảng | "gradient boosting", "tabular data" |
| Diffusion model | Sinh ảnh bằng khử nhiễu | "image generation", "Stable Diffusion" |
| Multi-modal model | Nhận/sinh nhiều loại data cùng lúc | "text + image + audio input" |
| Labeled / Unlabeled data | Có nhãn output / chỉ có input | "annotated", "no labels" |
| Structured data | Tabular (hàng-cột), time series | "rows and columns", "stock prices daily" |
| Unstructured data | Text, ảnh, audio, video | "reviews", "images", "social media posts" |
| Regression | Dự đoán giá trị **số liên tục** | "predict price / temperature / quantity" |
| Classification | Dự đoán **nhãn rời rạc** (binary, multiclass, multi-label) | "spam or not", "which category" |
| k-NN | K-nearest neighbors, thuật toán classification theo láng giềng gần nhất | "classify by similarity to nearest points" |
| Training / Validation / Test set | Để train / để tune hyperparameter / để đánh giá cuối | "tune hyperparameters" → validation |
| Feature engineering | Biến raw data thành feature có ý nghĩa | "derive age from birth date", "normalize" |
| TF-IDF / Word embeddings | Biến text thành vector số | "convert text to numeric features" |
| Clustering (K-means) | Gom nhóm data tương tự, không cần nhãn | "segment customers", "group similar" |
| Association rules (Apriori) | Tìm sản phẩm hay được mua cùng nhau | "market basket", "frequently bought together" |
| Anomaly detection (Isolation Forest) | Phát hiện điểm bất thường | "outlier", "unusual transaction" |
| Semi-supervised learning | Ít nhãn + nhiều không nhãn, pseudo-labeling | "small labeled set, large unlabeled set" |
| Pseudo-labeling | Model tự gắn nhãn cho data chưa có nhãn | "model labels remaining data" |
| Self-supervised learning | Tự tạo nhãn từ data qua pretext task | "predict masked word", "pre-training LLM" |
| Reinforcement Learning | Agent học qua trial-and-error để tối đa reward | "agent", "environment", "reward", "policy" |
| RLHF | RL với reward model học từ đánh giá của người | "human preferences", "align with human values" |
| Overfitting | Giỏi trên training, kém trên data mới (high variance) | "high training accuracy, low test accuracy" |
| Underfitting | Kém ngay trên training (high bias) | "poor on training data", "too simple" |
| Bias (thống kê) | Sai số do giả định sai của model | "model too simple" |
| Variance | Độ nhạy của model với thay đổi của training data | "sensitive to training data" |
| Confusion matrix | Bảng TP, FP, FN, TN cho classification | "evaluate classification model" |
| Precision | TP / (TP + FP): trong các dự đoán positive, bao nhiêu đúng | "false positives are costly" |
| Recall (Sensitivity) | TP / (TP + FN): trong các positive thật, bắt được bao nhiêu | "false negatives are costly", "don't miss" |
| F1 score | Cân bằng precision và recall | "imbalanced dataset", "balance" |
| Accuracy | (TP + TN) / tổng | "balanced dataset" |
| AUC-ROC | Diện tích dưới đường ROC, 0 → 1 | "compare models across thresholds" |
| MAE / MAPE / RMSE | Đo sai số của regression | "average error of numeric prediction" |
| R² | % biến thiên được model giải thích (gần 1 = tốt) | "explains variance" |
| Real-time inference | Dự đoán ngay, latency thấp | "immediate response", "chatbot" |
| Batch inference | Dự đoán lô lớn, không gấp | "large dataset at once", "overnight" |
| `[+]` Serverless inference | Endpoint không cần quản instance, hợp traffic thưa | "intermittent traffic", "tolerate cold start" |
| `[+]` Asynchronous inference | Xếp hàng request lớn/lâu, trả kết quả sau | "large payload", "long processing time" |
| Edge inference / SLM | Model nhỏ chạy trên thiết bị, offline, latency thấp | "limited connectivity", "on-device" |
| EDA / Correlation matrix | Khám phá data, xem độ liên quan giữa các biến | "visualize", "which features are related" |
| Hyperparameter | Thiết lập trước khi train (learning rate, batch size, epochs, regularization) | "set before training" |
| `[+]` Parameter (weights) | Giá trị model **tự học** trong lúc train | "learned during training" |
| Learning rate | Độ lớn bước cập nhật weights | "overshoot", "converge slowly" |
| Batch size | Số mẫu mỗi lần cập nhật weights | "examples per iteration" |
| Epoch | Một lần duyệt hết training set | "iterations over entire dataset" |
| Regularization | Phạt độ phức tạp để giảm overfitting | "reduce overfitting", "simpler model" |
| Early stopping | Dừng train trước khi overfit | "stop when validation performance degrades" |
| Data augmentation | Tạo thêm biến thể data để tăng đa dạng | "increase diversity of training data" |
| Ensembling | Kết hợp nhiều model | "combine multiple models" |
| SageMaker Automatic Model Tuning (AMT) | Tự động tìm hyperparameter tốt nhất | "automate hyperparameter tuning" |
| `[+]` SageMaker Ground Truth | Gắn nhãn dữ liệu (người + tự động) | "label training data" |
| `[+]` SageMaker Model Monitor | Giám sát drift của model trong production | "detect data drift in production" |
| `[+]` Transfer learning | Dùng lại model đã train cho task mới, chỉ train thêm phần nhỏ | "reuse pre-trained model for new task" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| AI | ML | AI gồm cả rule-based (expert system); ML phải **học từ data** | Có "rules written by experts" → AI nhưng không phải ML |
| Supervised | Unsupervised | Supervised cần **labeled data**, dự đoán output đã biết; Unsupervised tự tìm cấu trúc trong **unlabeled data** | "Historical labeled outcomes" → supervised |
| Semi-supervised | Self-supervised | Semi: con người gắn nhãn **một phần**; Self: **không người gắn nhãn**, model tự tạo nhãn từ data | Pre-training GPT/BERT → self-supervised |
| Reinforcement Learning | Supervised Learning | RL: **reward** qua tương tác với environment, không có đáp án đúng sẵn; Supervised: có nhãn đúng | "Agent", "reward", "trial and error" → RL |
| RL | RLHF | RLHF: reward model học từ **sở thích của con người** | "Human ranks responses" → RLHF |
| Regression | Classification | Regression: **số liên tục**; Classification: **nhãn rời rạc** | "How much / how many" → regression; "which / yes-no" → classification |
| Classification | Clustering | Classification: **có nhãn sẵn**, gán vào lớp đã biết; Clustering: **không nhãn**, tự gom nhóm | "Groups not known in advance" → clustering |
| Clustering | Anomaly detection | Clustering gom nhóm điểm giống nhau; Anomaly detection tìm điểm **khác biệt** | "Outlier / unusual" → anomaly |
| Clustering | Association rules | Clustering gom **khách hàng** giống nhau; Association tìm **sản phẩm hay đi cùng nhau** | "Bought together" → Apriori |
| Binary / Multiclass | Multi-label | Binary: 2 lớp; Multiclass: chọn 1 trong nhiều lớp; Multi-label: **nhiều nhãn cùng lúc** | Phim vừa action vừa comedy → multi-label |
| Validation set | Test set | Validation: **tune hyperparameter** trong lúc phát triển; Test: đánh giá **cuối cùng** trên data chưa thấy | Tune → validation |
| Overfitting | Underfitting | Over: train tốt / test tệ (high variance); Under: train tệ (high bias) | "Memorized the training data" → overfitting |
| Bias (thống kê) | Bias (fairness) | Thống kê: sai số do model quá đơn giản; Fairness: model đối xử bất công với một nhóm người | Đọc kỹ ngữ cảnh: "underfitting" hay "discrimination" |
| Precision | Recall | Precision lo **FP** (báo nhầm); Recall lo **FN** (bỏ sót) | Ung thư, gian lận (không được bỏ sót) → **Recall**; spam filter (không được chặn nhầm) → **Precision** |
| F1 | Accuracy | F1 cho data **mất cân bằng**; Accuracy chỉ đáng tin khi data **cân bằng** | 99% giao dịch hợp lệ → accuracy gây hiểu lầm, dùng F1 |
| Classification metrics | Regression metrics | Classification: Precision, Recall, F1, Accuracy, AUC; Regression: MAE, MAPE, RMSE, R² | "RMSE cho model spam" là đáp án **sai** |
| MAE / RMSE | R² | MAE/RMSE đo **độ sai** (thấp = tốt); R² đo **% biến thiên được giải thích** (gần 1 = tốt) | "Explains variance" → R² |
| Parameter | Hyperparameter | Parameter (weights) model tự học; Hyperparameter **người đặt trước** khi train | Learning rate, epochs → hyperparameter |
| Learning rate cao | Learning rate thấp | Cao: nhanh, dễ vượt điểm tối ưu; Thấp: chậm nhưng chính xác | "Overshoot" → giảm learning rate |
| Epochs quá ít | Epochs quá nhiều | Quá ít → underfit; quá nhiều → overfit | Overfit → giảm epochs / early stopping |
| Real-time inference | Batch inference | Real-time: latency thấp, từng request; Batch: lô lớn, không gấp | "Immediate" → real-time; "nightly report" → batch |
| `[+]` Serverless inference | `[+]` Asynchronous inference | Serverless: traffic **thưa, không đều**; Async: **payload lớn, xử lý lâu**, kết quả trả sau | "Idle periods" → serverless; "1 GB payload, minutes to process" → async |
| SLM on edge | LLM on server | SLM: offline, latency thấp, ít compute; LLM: mạnh hơn, cần Internet, latency cao | "No / limited Internet" → SLM on edge |
| Transformer | RNN | Transformer: xử lý **song song cả câu** + attention; RNN: xử lý **tuần tự** | LLM hiện đại → Transformer |
| GAN | Diffusion | Cả hai sinh ảnh; GAN: generator vs discriminator; Diffusion: khử nhiễu dần | GAN hay gắn với "synthetic data / data augmentation" |
| ML | Code deterministic | Bài toán tính được chính xác thì dùng code; ML chỉ xấp xỉ | "Exact answer can be computed" → không dùng ML |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "predict house price / sales amount / temperature" | Regression | |
| "spam or not spam", "fraud or not", "churn yes/no" | Binary classification | |
| "assign multiple tags to one item" | Multi-label classification | |
| "group customers without predefined labels" | Clustering (K-means) | |
| "products frequently bought together" | Association rule learning (Apriori) | |
| "detect unusual transactions without labels" | Anomaly detection (Isolation Forest) | |
| "few labeled + many unlabeled examples" | Semi-supervised learning | |
| "pre-train language model on raw text" | Self-supervised learning | |
| "agent, environment, maximize reward" | Reinforcement learning | |
| "align LLM responses with human preferences" | RLHF | |
| "high training accuracy, poor on new data" | Overfitting → thêm data, early stopping, augmentation, regularization, ensembling | |
| "poor accuracy even on training data" | Underfitting → model phức tạp hơn, thêm feature, train lâu hơn | |
| "false negatives are very costly (miss cancer/fraud)" | Recall | |
| "false positives are very costly" | Precision | |
| "imbalanced dataset", "balance precision and recall" | F1 score | |
| "compare classifiers across thresholds" | AUC-ROC | |
| "average error of a numeric prediction" | MAE / RMSE | |
| "% of variance explained" | R² | |
| "tune hyperparameters" | Validation set; SageMaker AMT | |
| "convert birth date to age", "normalize values" | Feature engineering | |
| "check which variables are correlated" | EDA / correlation matrix | |
| "label images for training" | `[+]` SageMaker Ground Truth | |
| "chatbot needs immediate answers" | Real-time inference | |
| "score millions of records overnight" | Batch inference (SageMaker Batch Transform) | |
| "intermittent traffic, avoid paying for idle endpoint" | `[+]` Serverless inference | |
| "large payloads, long processing time" | `[+]` Asynchronous inference | |
| "device with limited Internet, very low latency" | SLM on edge | |
| "monitor model drift in production" | `[+]` SageMaker Model Monitor | |
| "sequential data, speech, time series" (kiến trúc) | RNN | |
| "image recognition" (kiến trúc) | CNN / ResNet | |
| "generate synthetic data for augmentation" | GAN | |
| "generate raw audio / speech synthesis" (kiến trúc) | WaveNet | |
| "problem has an exact computable answer" | Không dùng ML, viết code | |
| "reuse a pre-trained model for a new related task" | `[+]` Transfer learning | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một ngân hàng có dữ liệu giao dịch lịch sử, mỗi giao dịch đã được gắn nhãn "gian lận" hoặc "hợp lệ". Họ muốn dự đoán giao dịch mới có gian lận không. Đây là loại bài toán nào?
- A. Clustering
- B. Regression
- C. Binary classification (supervised)
- D. Association rule learning

<details><summary>Đáp án</summary>

**C.** Có labeled data và output là 2 lớp rời rạc.
- A sai: clustering dùng cho data không nhãn.
- B sai: regression dự đoán số liên tục.
- D sai: association tìm các item hay đi cùng nhau.
</details>

**Câu 2.** Model phát hiện ung thư từ ảnh X-quang. Bệnh viện nói **bỏ sót một ca ung thư** nguy hiểm hơn nhiều so với báo nhầm. Metric nào cần ưu tiên tối đa?
- A. Precision
- B. Recall
- C. RMSE
- D. R²

<details><summary>Đáp án</summary>

**B.** Recall = TP / (TP + FN), tức là giảm false negative (bỏ sót).
- A sai: precision lo false positive.
- C, D sai: là metric của regression.
</details>

**Câu 3.** Một model đạt **98% accuracy trên training set** nhưng chỉ **65% trên test set**. Những cách nào giúp khắc phục? **(Chọn 2)**
- A. Tăng thêm dữ liệu training đa dạng hơn (data augmentation)
- B. Dùng early stopping
- C. Tăng số epochs
- D. Dùng model phức tạp hơn với nhiều layer hơn
- E. Giảm regularization

<details><summary>Đáp án</summary>

**A, B.** Đây là overfitting (high variance): cần thêm data đa dạng và dừng train sớm.
- C, D, E sai: đều làm overfitting **nặng hơn**.
</details>

**Câu 4.** Một siêu thị muốn biết những sản phẩm nào thường được mua cùng nhau trong một hóa đơn để sắp xếp kệ hàng. Dữ liệu chỉ có lịch sử giao dịch, không có nhãn. Kỹ thuật nào phù hợp?
- A. K-means clustering
- B. Association rule learning (Apriori)
- C. Linear regression
- D. Reinforcement learning

<details><summary>Đáp án</summary>

**B.** Market basket analysis là ứng dụng kinh điển của association rules.
- A sai: K-means gom nhóm **khách hàng**, không tìm sản phẩm đi cùng nhau.
- C sai: dự đoán số.
- D sai: cần agent và reward.
</details>

**Câu 5.** Một team có 1.000 ảnh sản phẩm đã gắn nhãn và 500.000 ảnh chưa gắn nhãn. Họ train model trên phần có nhãn, để model tự gắn nhãn cho phần còn lại, rồi train lại trên toàn bộ. Đây là:
- A. Self-supervised learning
- B. Semi-supervised learning
- C. Reinforcement learning
- D. Unsupervised learning

<details><summary>Đáp án</summary>

**B.** Ít data có nhãn + nhiều data không nhãn, dùng pseudo-labeling.
- A sai: self-supervised không cần con người gắn nhãn chút nào.
- C sai: không có reward.
- D sai: unsupervised không dùng nhãn.
</details>

**Câu 6.** Một spam filter có confusion matrix: TP = 80, FP = 20, FN = 10, TN = 890. **Precision** là bao nhiêu?
- A. 0.80
- B. 0.89
- C. 0.97
- D. 0.20

<details><summary>Đáp án</summary>

**A.** Precision = TP / (TP + FP) = 80 / 100 = 0.80.
- B sai: 0.89 ≈ 80 / 90 là **recall**.
- C sai: 0.97 = (80 + 890) / 1000 là **accuracy**; accuracy cao ở đây gây hiểu lầm vì data mất cân bằng.
- D sai: 0.20 là tỷ lệ FP trong số dự đoán positive.
</details>

**Câu 7.** (Chọn 2) Phát biểu nào **đúng** về hyperparameter?
- A. Hyperparameter được đặt trước khi bắt đầu training.
- B. Weights của neural network là hyperparameter.
- C. Learning rate quá cao có thể làm model vượt qua điểm tối ưu.
- D. Test set được dùng để tune hyperparameter.
- E. Tăng regularization làm tăng overfitting.

<details><summary>Đáp án</summary>

**A, C.**
- B sai: weights là **parameter** do model tự học.
- D sai: tune hyperparameter dùng **validation set**; test set để đánh giá cuối.
- E sai: tăng regularization **giảm** overfitting.
</details>

**Câu 8.** Một nhà máy cần model kiểm tra lỗi sản phẩm trên dây chuyền, ở khu vực **kết nối Internet không ổn định** và yêu cầu **latency cực thấp**. Cách triển khai nào phù hợp nhất?
- A. Gọi LLM lớn trên cloud qua API
- B. Chạy Small Language Model / model nhỏ trực tiếp trên thiết bị edge
- C. Batch inference mỗi đêm
- D. Asynchronous inference trên cloud

<details><summary>Đáp án</summary>

**B.** Edge inference chạy offline với latency rất thấp và ít compute.
- A sai: cần Internet và latency cao hơn.
- C sai: không phải real-time.
- D sai: vẫn cần Internet, và kết quả trả về sau.
</details>

---

### Nguồn đã đối chiếu
- AIF-C01 exam guide, Domain 1 (loại inferencing, loại dữ liệu, loại learning, metric, MLOps): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain1.html
- AIF-C01 exam guide, Domain 3 (RLHF, transfer learning, continuous pre-training): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain3.html
- SageMaker AI model deployment options (real-time, serverless, async tới 1 GB, batch transform): https://docs.aws.amazon.com/sagemaker/latest/dg/how-it-works-deployment.html
- SageMaker AI hyperparameter tuning strategies (grid, random, Bayesian, Hyperband): https://docs.aws.amazon.com/sagemaker/latest/dg/automatic-model-tuning-how-it-works.html
- SageMaker Ground Truth (data labeling): https://docs.aws.amazon.com/sagemaker/latest/dg/data-label.html
- SageMaker Model Monitor FAQs (drift): https://docs.aws.amazon.com/sagemaker/latest/dg/model-monitor-faqs.html
- SageMaker AI features: https://docs.aws.amazon.com/sagemaker/latest/dg/whatis-features.html
- What is RLHF (link trong slide): https://aws.amazon.com/what-is/reinforcement-learning-from-human-feedback/
