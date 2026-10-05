# AIF-C01 — Introduction to AI

> Thuộc **Domain 1: Fundamentals of AI and ML** (20% điểm), Task 1.1 & 1.2 trong exam guide AIF-C01 (bản 1.1, 04/2026).
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **AI** là lĩnh vực khoa học máy tính giải quyết các bài toán vốn cần trí tuệ con người: nhìn, nghe, hiểu ngôn ngữ, ra quyết định.
- Quan hệ lồng nhau: **AI ⊃ Machine Learning ⊃ Deep Learning ⊃ Generative AI**. Đề rất hay hỏi "cái nào là tập con của cái nào".
- Vòng đời cơ bản: **Training dataset + Algorithm → Training → Model → Inference** (dự đoán trên dữ liệu mới).
- Generative AI (ChatGPT, DALL-E) là thứ mọi người nghĩ đến khi nói "AI", nhưng nó chỉ là **một nhánh nhỏ nhất** trong cây.
- `[+]` **Agentic AI** là nội dung mới trong exam guide 1.1: AI tự lập kế hoạch nhiều bước và gọi tool để hoàn thành mục tiêu.
- Intelligent Document Processing (IDP) là ví dụ điển hình **kết hợp nhiều kỹ thuật AI**: Computer Vision + Deep Learning + NLP.
- `[+]` AI/ML **không phải lúc nào cũng phù hợp**: cần kết quả xác định 100% hoặc chi phí > lợi ích thì dùng rule/logic thông thường.

---

## 2. Core Concepts

### Artificial Intelligence (AI)
- **Là gì:** máy tính giải các bài toán "giống người": nhận diện ảnh, chuyển giọng nói thành chữ, tạo ảnh, học từ dữ liệu.
- **Dùng khi nào:** bài toán có quy luật phức tạp, khó viết tay thành if/else, nhưng có nhiều dữ liệu mẫu.
- **Vì sao quan trọng:** là "chiếc ô" lớn nhất, mọi khái niệm khác (ML, DL, GenAI) đều nằm trong nó.

### How AI works: Training → Model → Inference
- **Là gì:** data scientist đưa **training dataset** (vd: ảnh trái cây đã gắn nhãn) vào **algorithm** (vd: classification) để tạo ra **model**.
- **Dùng khi nào:** model đã train được dùng để trả lời câu hỏi mới ("ảnh này là quả gì?" → "Apple"); bước này gọi là **inference**.
- **Vì sao quan trọng:** đề hay đánh đố giữa training (tốn tài nguyên, làm một lần/định kỳ) và inference (chạy liên tục khi phục vụ người dùng).

### History of AI (mốc chính)
- **1950s:** Alan Turing đề xuất Turing Test; John McCarthy đặt ra thuật ngữ "Artificial Intelligence". **1970s:** Expert Systems (vd: MYCIN, rule-based chẩn đoán vi khuẩn).
- **1990s:** Machine Learning & Data Mining; 1997 IBM Deep Blue thắng Kasparov. **2010s:** Deep Learning; 2016 AlphaGo thắng Lee Sedol.
- **2020s:** AI đi vào đời sống (trợ lý ảo, xe tự lái, chẩn đoán y tế) kèm tranh luận về ethics và regulation. Ý chính cần nhớ: AI đi **từ rule cứng → học từ dữ liệu**.

### AI Use Cases
- **Là gì:** speech recognition/generation, transcribe & translate, nhận diện ảnh, chơi game (Chess, Go, StarCraft), xe tự lái, gợi ý code, chẩn đoán y tế, fraud detection, tự động hóa quy trình.
- **Dùng khi nào:** khi cần **scale** việc con người làm, **hỗ trợ ra quyết định**, hoặc **tự động hóa** tác vụ lặp lại.
- **Vì sao quan trọng:** đề cho tình huống thực tế và hỏi dùng kỹ thuật / dịch vụ AWS nào (xem bảng mục 4).

### Intelligent Document Processing (IDP)
- **Là gì:** tự động đọc tài liệu (vd: hóa đơn là ảnh trong PDF) → trích xuất seller, buyer, bảng items → ghi vào database.
- **Dùng khi nào:** xử lý số lượng lớn hóa đơn, hợp đồng, form, hồ sơ mà trước đây phải nhập tay.
- **Vì sao quan trọng:** kết hợp Computer Vision (đọc ảnh) + NLP (hiểu nội dung) + Deep Learning. `[+]` Trên AWS: **Amazon Textract** trích xuất text/form/table, **Amazon Comprehend** phân tích ngôn ngữ.

### AI hierarchy: AI ⊃ ML ⊃ DL ⊃ GenAI
- **Là gì:** ML = AI học từ dữ liệu; Deep Learning = ML dùng neural network nhiều lớp; GenAI = DL **tạo ra nội dung mới** (text, ảnh, audio, code).
- **Dùng khi nào:** dùng sơ đồ này để loại trừ đáp án: "mọi GenAI đều là ML" là **đúng**, "mọi ML đều là GenAI" là **sai**.
- **Vì sao quan trọng:** Objective 1.1.2 yêu cầu trực tiếp phân biệt AI, ML, DL, GenAI, agentic AI.

### `[+]` Agentic AI
- **Là gì:** hệ thống xây quanh LLM, có thêm tools, retrieval, memory, chạy theo vòng lặp **reason → act** cho tới khi đạt mục tiêu.
- **Dùng khi nào:** tác vụ nhiều bước cần tự quyết định gọi API/tool nào (vd: tự tra cứu, đặt lịch, cập nhật hệ thống).
- **Vì sao quan trọng:** mới được thêm vào exam guide 1.1. Khác GenAI thường ở chỗ **hành động**, không chỉ sinh ra câu trả lời.

### `[+]` Khi nào KHÔNG nên dùng AI/ML
- **Là gì:** AI/ML đưa ra **dự đoán có xác suất**, không đảm bảo đúng 100%.
- **Dùng khi nào:** chọn rule-based / code thường khi luật đã rõ ràng, cần kết quả xác định, hoặc chi phí xây/vận hành model > lợi ích.
- **Vì sao quan trọng:** Objective 1.2.2 hỏi trực tiếp: "situations when a specific outcome is needed instead of a prediction".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Artificial Intelligence (AI) | Lĩnh vực rộng nhất: máy mô phỏng trí tuệ con người | "broad field", "mimic human intelligence" |
| Machine Learning (ML) | Nhánh của AI, model **tự học quy luật từ dữ liệu** thay vì lập trình luật | "learn from data", "without being explicitly programmed" |
| Deep Learning (DL) | Nhánh của ML, dùng **neural network nhiều lớp**; mạnh với ảnh, âm thanh, text | "neural network", "many layers", "unstructured data" |
| `[+]` Neural network | Mạng các "nơ-ron" nhân tạo xếp theo lớp, nền tảng của Deep Learning | "layers", "nodes", "weights" |
| Generative AI (GenAI) | Nhánh của DL, **tạo nội dung mới** (text, ảnh, code); vd ChatGPT, DALL-E | "generate", "create new content", "summarize", "draft" |
| `[+]` Agentic AI | LLM + tools + memory, tự lập kế hoạch và **thực thi hành động** nhiều bước | "autonomously", "multi-step", "take actions", "call APIs/tools" |
| Model | Kết quả sau training, dùng để dự đoán trên dữ liệu mới | "trained model", "deploy the model" |
| Algorithm | Phương pháp/công thức để học từ dữ liệu (vd: classification) | "which algorithm/technique" |
| Training dataset | Dữ liệu (thường đã gắn nhãn) dùng để dạy model | "historical data", "labeled examples" |
| Training | Quá trình model học từ dữ liệu, tốn compute | "build", "teach", "train the model" |
| Inference | Dùng model đã train để **dự đoán trên dữ liệu mới** | "predict on new data", "real-time predictions", "serve" |
| Classification | Dự đoán **nhãn/loại** (apple / banana, fraud / not fraud) | "categorize", "which class", "yes/no" |
| Computer Vision (CV) | AI hiểu **ảnh/video**: nhận diện vật thể, đọc chữ trong ảnh | "images", "video", "detect objects", "scanned" |
| Natural Language Processing (NLP) | AI hiểu **ngôn ngữ con người** dạng text | "sentiment", "entities", "language understanding" |
| Speech recognition | Chuyển **giọng nói → text** (speech-to-text) | "transcribe", "call recordings", "subtitles" |
| Intelligent Document Processing (IDP) | Trích xuất dữ liệu có cấu trúc từ tài liệu (PDF, ảnh scan) | "invoices", "forms", "extract fields from documents" |
| Expert System | AI đời đầu dựa trên **luật do chuyên gia viết** (vd: MYCIN), không tự học | "rule-based", "if-then rules", "deterministic" |
| Turing Test | Phép thử máy có "trò chuyện" không phân biệt được với người hay không | ít gặp, chỉ cần nhận diện tên |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| AI | Machine Learning | AI là ô lớn (gồm cả rule-based); ML chỉ là phần **học từ dữ liệu** | Mọi ML là AI, **không** phải mọi AI là ML |
| Machine Learning | Deep Learning | DL là ML dùng **neural network nhiều lớp**, cần nhiều data + compute (GPU) | Ảnh/âm thanh/text phức tạp → nghiêng về DL |
| Deep Learning | Generative AI | GenAI là DL dùng để **tạo mới**; DL truyền thống thường để **phân loại/dự đoán** | "Create/generate" → GenAI; "classify/detect" → ML/DL truyền thống |
| Generative AI | `[+]` Agentic AI | GenAI **sinh ra** nội dung; Agentic AI **lập kế hoạch + hành động** qua tools | Chỉ trả lời → GenAI; tự gọi API, làm nhiều bước → Agentic |
| Training | Inference | Training = dạy model (một lần/định kỳ, tốn compute); Inference = dùng model trả lời dữ liệu mới | "New data comes in, get prediction" → inference |
| Expert System (rule-based) | Machine Learning | Rule-based: người viết luật, kết quả xác định; ML: tự học luật từ data, kết quả xác suất | Cần kết quả chắc chắn, luật rõ → **không cần ML** |
| Computer Vision | NLP | CV xử lý **pixel** (ảnh, video, scan); NLP xử lý **text** | IDP cần **cả hai**: CV đọc chữ trong ảnh, NLP hiểu ý nghĩa |
| `[+]` Amazon Textract | `[+]` Amazon Comprehend | Textract **lấy text/form/table ra khỏi tài liệu**; Comprehend **phân tích text** (sentiment, entities, key phrases) | Textract = "đọc", Comprehend = "hiểu". PDF scan → Textract trước |
| `[+]` Amazon Textract | `[+]` Amazon Rekognition | Textract chuyên **tài liệu** (form, bảng, hóa đơn); Rekognition chuyên **ảnh/video** (vật thể, khuôn mặt, nội dung) | Document → Textract; photo/video → Rekognition |
| `[+]` Amazon Transcribe | `[+]` Amazon Polly | Transcribe: **speech → text**; Polly: **text → speech** | **T**ranscribe = **T**o text; **P**olly = **P**arrot (nói ra) |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "scanned documents", "invoices", "forms", "tables in PDF" | `[+]` Amazon Textract | Bài toán IDP |
| "sentiment", "entities", "key phrases", "language of text" | `[+]` Amazon Comprehend | NLP trên text có sẵn |
| "call recordings to text", "subtitles", "speech-to-text" | `[+]` Amazon Transcribe | Speech recognition |
| "read text aloud", "lifelike voice", "text-to-speech" | `[+]` Amazon Polly | Speech generation |
| "translate between languages" | `[+]` Amazon Translate | |
| "detect objects / faces in images or video" | `[+]` Amazon Rekognition | Computer Vision |
| "chatbot", "conversational interface", "intent" | `[+]` Amazon Lex | |
| "code suggestions for developers" | `[+]` Kiro | Amazon Q Developer IDE plugins hết hỗ trợ ngày 30/04/2027, AWS khuyến nghị chuyển sang Kiro |
| "fraud detection" | Classification / anomaly detection; build bằng Amazon SageMaker AI | `[+]` Amazon Fraud Detector **không nhận khách hàng mới từ 07/11/2025**. Đề cũ có thể vẫn ghi tên này |
| "generate new text/images/code" | Generative AI (vd: Amazon Bedrock) | |
| "autonomously plan and take actions using tools" | Agentic AI | |
| "rules are fixed", "must be deterministic", "exact outcome required" | Rule-based / không dùng ML | Objective 1.2.2 |
| "neural network with many layers" | Deep Learning | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một công ty bảo hiểm nhận hàng nghìn hóa đơn y tế dạng PDF scan mỗi ngày. Họ cần tự động trích xuất tên bệnh nhân, bảng chi phí và tổng tiền vào database. Dịch vụ AWS nào phù hợp nhất?
- A. Amazon Comprehend
- B. Amazon Textract
- C. Amazon Rekognition
- D. Amazon Polly

<details><summary>Đáp án</summary>

**B.** Textract trích xuất text, form (key-value) và table từ tài liệu scan, đúng bài toán IDP.
- A sai: Comprehend phân tích text **đã có**, không đọc được ảnh scan.
- C sai: Rekognition dành cho ảnh/video nói chung (vật thể, khuôn mặt), không chuyên form/bảng.
- D sai: Polly là text-to-speech.
</details>

**Câu 2.** (Chọn 2) Phát biểu nào **đúng** về quan hệ giữa các khái niệm AI?
- A. Deep Learning là một tập con của Machine Learning.
- B. Machine Learning bao gồm toàn bộ AI.
- C. Generative AI là một tập con của Deep Learning.
- D. Mọi hệ thống AI đều dùng neural network.
- E. Expert system là một dạng Deep Learning.

<details><summary>Đáp án</summary>

**A, C.** Thứ tự đúng là AI ⊃ ML ⊃ DL ⊃ GenAI.
- B sai: ngược chiều, ML nằm **trong** AI.
- D sai: AI rule-based hay ML truyền thống không cần neural network.
- E sai: expert system dựa trên luật viết tay, không học từ dữ liệu.
</details>

**Câu 3.** Một ngân hàng cần tính phí chuyển khoản theo biểu phí cố định do pháp chế ban hành. Kết quả phải chính xác tuyệt đối và giải thích được từng bước. Cách tiếp cận nào phù hợp nhất?
- A. Train một Deep Learning model trên lịch sử giao dịch
- B. Dùng Generative AI để tính phí
- C. Dùng logic rule-based (code thông thường), không dùng ML
- D. Dùng clustering để nhóm giao dịch rồi gán phí

<details><summary>Đáp án</summary>

**C.** Luật đã rõ ràng và cần kết quả xác định, nên ML (vốn trả về dự đoán có xác suất) là thừa và có rủi ro sai.
- A, B, D sai: đều cho kết quả xác suất, tốn chi phí, khó giải thích và không đảm bảo đúng 100%.
</details>

**Câu 4.** Một ứng dụng đã train xong model phân loại ảnh trái cây. Mỗi khi người dùng upload ảnh mới, model trả về tên loại quả. Giai đoạn này gọi là gì?
- A. Training
- B. Data labeling
- C. Inference
- D. Feature engineering

<details><summary>Đáp án</summary>

**C.** Dùng model đã train để dự đoán trên dữ liệu **mới** là inference.
- A sai: training là giai đoạn dạy model trước đó.
- B, D sai: đều là bước chuẩn bị dữ liệu trước khi train.
</details>

**Câu 5.** (Chọn 2) Một call center muốn chuyển các bản ghi âm cuộc gọi thành văn bản rồi xác định khách hàng hài lòng hay bực bội. Nên kết hợp những dịch vụ nào?
- A. Amazon Polly
- B. Amazon Transcribe
- C. Amazon Comprehend
- D. Amazon Textract
- E. Amazon Rekognition

<details><summary>Đáp án</summary>

**B, C.** Transcribe làm speech-to-text, sau đó Comprehend phân tích sentiment trên text đó.
- A sai: Polly làm chiều ngược lại (text-to-speech).
- D sai: Textract dành cho tài liệu.
- E sai: Rekognition dành cho ảnh/video.
</details>

**Câu 6.** Tình huống nào là ứng dụng của **Generative AI**?
- A. Dự đoán doanh số quý tới dựa trên dữ liệu lịch sử
- B. Phân loại email là spam hay không spam
- C. Tự động viết mô tả sản phẩm từ danh sách thuộc tính
- D. Phát hiện giao dịch thẻ tín dụng bất thường

<details><summary>Đáp án</summary>

**C.** Viết mô tả là **tạo nội dung mới**, đúng đặc trưng của GenAI.
- A sai: forecasting là ML truyền thống (dự đoán số).
- B sai: classification.
- D sai: fraud/anomaly detection, cũng là dự đoán chứ không tạo nội dung.
</details>

**Câu 7.** Một công ty muốn xây trợ lý nhận yêu cầu "đặt chuyến công tác Hà Nội tuần sau trong ngân sách 10 triệu". Trợ lý tự tra chuyến bay, so sánh khách sạn, gọi API đặt chỗ và cập nhật lịch, tự quyết định bước tiếp theo cho đến khi xong. Đây là ví dụ của:
- A. Expert system
- B. Agentic AI
- C. Supervised classification
- D. Speech recognition

<details><summary>Đáp án</summary>

**B.** Hệ thống nhận một **mục tiêu**, tự lập kế hoạch nhiều bước và **gọi tools/API để hành động**. Đó là agentic AI.
- A sai: expert system chỉ áp luật cố định, không tự lập kế hoạch.
- C sai: classification chỉ gán nhãn.
- D sai: speech recognition chỉ chuyển giọng nói thành text.
</details>

**Câu 8.** (Chọn 2) Một trang tin tức muốn mỗi bài báo tiếng Việt có thêm bản **audio tiếng Anh** để người nước ngoài nghe. Cần những dịch vụ nào?
- A. Amazon Translate
- B. Amazon Transcribe
- C. Amazon Polly
- D. Amazon Comprehend
- E. Amazon Lex

<details><summary>Đáp án</summary>

**A, C.** Translate dịch text Việt → Anh, sau đó Polly đọc text tiếng Anh thành giọng nói.
- B sai: Transcribe là speech-to-text, đầu vào ở đây là text.
- D sai: Comprehend phân tích text, không dịch và không tạo audio.
- E sai: Lex dùng để xây chatbot.
</details>

---

### Nguồn đã đối chiếu
- AIF-C01 exam guide, Domain 1: https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain1.html
- AIF-C01 exam guide revisions (bản 1.1 thêm agentic AI): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/aif-01-revisions.html
- Amazon Textract: https://docs.aws.amazon.com/textract/latest/dg/what-is.html
- Agentic AI (Well-Architected Generative AI Lens): https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/agentic-ai.html
- Amazon Fraud Detector availability change: https://docs.aws.amazon.com/frauddetector/latest/ug/frauddetector-availability-change.html
- Amazon Q Developer IDE plugins end of support → Kiro: https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-developer-ide-end-of-support.html
