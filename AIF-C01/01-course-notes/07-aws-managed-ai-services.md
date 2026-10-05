# AIF-C01 — AWS Managed AI Services

> Thuộc **Domain 1, Task 1.2** ("Explain the capabilities of AWS managed AI/ML services": SageMaker AI, Transcribe, Translate, Comprehend, Lex, Polly...). Dạng câu hỏi chủ đạo: **cho tình huống, chọn đúng dịch vụ**.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời. Mốc kiến thức: 10/2026.

---

## 1. TL;DR

- **AWS AI services** là các dịch vụ **pre-trained, fully managed, serverless**: gọi API là dùng được, không cần kiến thức ML. Chạy trên nhiều AZ và Region, trả theo mức dùng.
- Nhớ theo **loại dữ liệu đầu vào**: Text → **Comprehend** (phân tích), **Translate** (dịch); Tài liệu scan → **Textract**; Ảnh/video → **Rekognition**; Speech → text: **Transcribe**; Text → speech: **Polly**; Chatbot → **Lex**; Gợi ý sản phẩm → **Personalize**.
- Mỗi dịch vụ có bản **custom** khi cần đặc thù nghiệp vụ: Comprehend Custom Classification / Custom Entity Recognition, Rekognition Custom Labels / Custom Moderation Adapters, Transcribe Custom Vocabularies / Custom Language Models.
- **Con người trong vòng lặp:** **Mechanical Turk** = chợ crowdsourcing việc vặt (gắn nhãn); **Augmented AI (A2I)** = người duyệt các **dự đoán có độ tin cậy thấp** trong production.
- **Y tế (HIPAA):** **Transcribe Medical** (speech → text y khoa), **Comprehend Medical** (trích thông tin + **PHI** từ text lâm sàng), **AWS HealthScribe** (tự sinh clinical notes từ hội thoại bác sĩ–bệnh nhân).
- ⚠️ **Amazon Kendra** không nhận khách hàng mới từ 30/07/2026 → tìm kiếm tài liệu bằng ML nay dùng **Bedrock Managed Knowledge Base**.
- **Phần cứng AI:** GPU instance (P, G), **AWS Trainium** (train, Trn), **AWS Inferentia** (inference, Inf). Trn và Inf có **environmental footprint thấp nhất**.

---

## 2. Core Concepts

### Vì sao dùng AWS Managed AI Services
- **Là gì:** dịch vụ ML **đã train sẵn** cho các use case phổ biến; AWS lo hạ tầng, khả năng sẵn sàng (multi-AZ, multi-Region) và phần cứng chuyên dụng (CPU/GPU).
- **Dùng khi nào:** cần thêm năng lực AI (đọc tài liệu, nhận diện ảnh, chuyển giọng nói) **nhanh, không có đội ML**; trả theo mức dùng, hoặc provisioned throughput cho workload ổn định.
- **Vì sao quan trọng:** `[+]` thứ tự ưu tiên khi chọn: **AI service có sẵn** (ít công nhất) → **Bedrock** (GenAI với FM) → **SageMaker AI** (tự build/train model, nhiều kiểm soát nhất).

### Amazon Comprehend
- **Là gì:** dịch vụ **NLP** fully managed, serverless: phát hiện ngôn ngữ, trích **key phrases**, **entities** (người, địa điểm, tổ chức, thương hiệu...), **sentiment**, phân tích cú pháp (tokenization, parts of speech).
- **Dùng khi nào:** phân tích email/review khách hàng để biết điều gì tạo trải nghiệm tốt hay xấu. `[+]` Comprehend cũng **phát hiện PII** trong text.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: **topic modeling** (gom tài liệu theo chủ đề) và **event detection** không còn mở cho khách hàng mới → nay dùng **LLM trên Amazon Bedrock**.

### Comprehend: NER, Custom Classification, Custom Entity Recognition
- **Là gì:** **NER** trích các entity **có sẵn, phổ thông** (người, nơi chốn, tổ chức, ngày tháng); **Custom Entity Recognition** trích entity **riêng của doanh nghiệp** (số hợp đồng bảo hiểm, cụm từ báo hiệu khách sắp khiếu nại).
- **Dùng khi nào:** **Custom Classification** phân loại tài liệu vào **lớp do bạn định nghĩa** (vd: email khiếu nại / hỏi giá / hỗ trợ); hỗ trợ text, PDF, Word, ảnh.
- **Vì sao quan trọng:** custom model train từ data trong **S3**; chạy **real-time** (1 tài liệu, đồng bộ, qua endpoint) hoặc **async** (nhiều tài liệu, batch).

### Amazon Translate
- **Là gì:** dịch máy tự nhiên, chính xác giữa nhiều ngôn ngữ.
- **Dùng khi nào:** **localize** website và ứng dụng cho người dùng quốc tế; dịch khối lượng text lớn.
- **Vì sao quan trọng:** đề có "translate / localize / multilingual content" → Translate (không phải Comprehend hay Transcribe).

### Amazon Transcribe
- **Là gì:** **speech → text** bằng **Automatic Speech Recognition (ASR)** (deep learning); có **PII Redaction** tự động và **Automatic Language Identification** cho audio nhiều ngôn ngữ.
- **Dùng khi nào:** transcribe cuộc gọi CSKH, phụ đề tự động (closed captioning), tạo metadata để tìm kiếm kho media.
- **Vì sao quan trọng:** **Toxicity Detection** dựa trên cả tín hiệu giọng (tone, pitch) lẫn nội dung text (quấy rối, hate speech, đe dọa, chửi thề...); hiện chỉ cho **batch, tiếng Anh Mỹ (en-US)**.

### Transcribe: Custom Vocabularies vs Custom Language Models
- **Là gì:** **Custom Vocabularies** (cho **từ**): thêm từ, cụm từ, tên thương hiệu, viết tắt, kèm gợi ý cách phát âm.
- **Dùng khi nào:** **Custom Language Models** (cho **ngữ cảnh**): train trên text chuyên ngành của bạn để Transcribe hiểu từ trong ngữ cảnh; hợp với khối lượng lớn speech chuyên ngành.
- **Vì sao quan trọng:** dùng **cả hai** để đạt độ chính xác cao nhất (vd: "AWS Microservices" không bị nhận thành "A USA My crow services").

### Amazon Polly
- **Là gì:** **text → speech** giọng tự nhiên bằng deep learning; làm ứng dụng biết nói.
- **Dùng khi nào:** **Lexicons**: định nghĩa cách đọc (AWS → "Amazon Web Services"); **SSML**: markup chỉ cách phát âm (ngắt nghỉ, nhấn mạnh); **Speech marks**: đánh dấu thời điểm bắt đầu/kết thúc từ (lip-sync, highlight chữ).
- **Vì sao quan trọng:** voice engines: **generative**, **long-form**, **neural**, **standard**.

### Amazon Rekognition
- **Là gì:** tìm **vật thể, người, text, cảnh** trong ảnh và video; **phân tích khuôn mặt** (độ tuổi, cảm xúc...), **face search & verification**, nhận diện người nổi tiếng.
- **Dùng khi nào:** labeling, **content moderation**, text detection, xác thực người dùng, đếm người.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: **People Pathing** (theo dõi đường đi, vd: phân tích trận đấu thể thao) ngừng hỗ trợ từ 31/10/2025; **Streaming Video** và **Bulk Image Analysis** không mở cho khách mới từ 30/04/2026 → nay xử lý từng frame/ảnh bằng **Rekognition Image API**.

### Rekognition Custom Labels & Content Moderation
- **Là gì:** **Custom Labels**: gắn nhãn vài trăm ảnh (hoặc ít hơn), upload lên S3, Rekognition tự train model nhận diện **đối tượng riêng của bạn** (logo, sản phẩm trên kệ).
- **Dùng khi nào:** **Content Moderation** tự phát hiện nội dung không phù hợp hoặc xúc phạm (mạng xã hội, quảng cáo, ảnh do GenAI tạo); **Custom Moderation Adapters** tăng độ chính xác bằng ảnh có nhãn của bạn.
- **Vì sao quan trọng:** tích hợp **Amazon A2I** để đưa ca khó cho người duyệt, giảm mạnh khối lượng cần duyệt thủ công.

### Amazon Lex
- **Là gì:** build **chatbot giọng nói và text**, đa ngôn ngữ (vd: đặt pizza, đặt phòng khách sạn).
- **Dùng khi nào:** bot tự hiểu **intent** của user và gọi **Lambda** để "fulfill the intent"; tự hỏi thêm **slots** (tham số còn thiếu). Tích hợp Amazon Connect, Comprehend.
- **Vì sao quan trọng:** đề có "chatbot / conversational interface / intent / slot" → Lex.

### Amazon Personalize
- **Là gì:** dịch vụ ML fully managed cho **gợi ý cá nhân hóa real-time** (cùng công nghệ của Amazon.com); triển khai trong vài ngày, không cần tự build model.
- **Dùng khi nào:** gợi ý sản phẩm, xếp hạng lại (re-ranking), marketing trực tiếp; data từ S3 hoặc real-time; tích hợp website, app, SMS, email.
- **Vì sao quan trọng:** **Recipes** = thuật toán dựng sẵn theo use case: User-Personalization-v2, Personalized-Ranking-v2, Trending-Now / Popularity-Count, Similar-Items, Next-Best-Action, Item-Affinity (user segments).

### Amazon Textract
- **Là gì:** trích **text, chữ viết tay, data** từ tài liệu scan (PDF, ảnh), kể cả **forms** (key-value) và **tables**.
- **Dùng khi nào:** tài chính (hóa đơn, báo cáo), y tế (hồ sơ, claim bảo hiểm), khu vực công (tờ khai thuế, giấy tờ tùy thân, hộ chiếu).
- **Vì sao quan trọng:** `[+]` có API chuyên dụng: **AnalyzeExpense** (hóa đơn, biên lai), **AnalyzeID** (giấy tờ tùy thân Mỹ), **Queries** (hỏi trực tiếp "tên khách hàng là gì?").

### Amazon Kendra
- ⚠️ Lỗi thời: **Amazon Kendra** vào Maintenance Mode từ 30/06/2026, **không nhận khách hàng mới từ 30/07/2026** → nay dùng **Amazon Bedrock Managed Knowledge Base** (RAG managed, connector dựng sẵn, hybrid search).
- **Là gì:** dịch vụ **tìm kiếm tài liệu** bằng ML: tìm kiếm ngôn ngữ tự nhiên, trích câu trả lời từ trong tài liệu, học từ phản hồi của user (incremental learning).
- **Vì sao quan trọng:** đề cũ có "intelligent enterprise search" → Kendra; với khách mới, đáp án tương đương là Bedrock Knowledge Bases (hoặc Amazon Quick nếu là trợ lý cho nhân viên).

### Amazon Mechanical Turk & Amazon Augmented AI (A2I)
- **Là gì:** **Mechanical Turk** = chợ crowdsourcing: chia việc nhỏ (gắn nhãn 10 triệu ảnh) cho lực lượng lao động ảo phân tán, bạn tự đặt thù lao mỗi task.
- **Dùng khi nào:** **A2I** = **human review cho dự đoán ML trong production**: dự đoán độ tin cậy cao trả về ngay, **độ tin cậy thấp** chuyển cho người duyệt; kết quả lưu S3 và có thể đưa lại vào training data.
- **Vì sao quan trọng:** reviewer của A2I có thể là nhân viên của bạn, vendor (một số đã qua sàng lọc bảo mật) hoặc Mechanical Turk; model có thể chạy trên AWS hoặc nơi khác. Mechanical Turk tích hợp với A2I và SageMaker Ground Truth.

### Dịch vụ AI cho y tế
- **Là gì:** **Transcribe Medical**: speech → text cho thuật ngữ y khoa (thuốc, thủ thuật, bệnh), real-time và batch, chỉ en-US. **Comprehend Medical**: trích thông tin từ text lâm sàng (ghi chú bác sĩ, kết quả xét nghiệm) và phát hiện **PHI** (DetectPHI API).
- **Dùng khi nào:** **AWS HealthScribe** (HIPAA-eligible): phân tích hội thoại bác sĩ–bệnh nhân để tạo transcript, xác định vai người nói, trích thuật ngữ y khoa, **sinh clinical notes**.
- **Vì sao quan trọng:** pipeline điển hình: Transcribe Medical (audio → text) → Comprehend Medical (text → thông tin lâm sàng); stream real-time qua ⚠️ Kinesis Data Firehose → nay là **Amazon Data Firehose**.

### Amazon EC2 & phần cứng cho AI
- **Là gì:** EC2 = IaaS: thuê VM (EC2), ổ đĩa mạng (EBS), cân bằng tải (ELB), auto scaling (ASG); tự chọn OS, CPU, RAM, storage, security group, user data.
- **Dùng khi nào:** **GPU instances** (dòng P, G) cho train/inference nói chung; **AWS Trainium** (Trn1 tới 16 accelerator; Trn2) để **train** deep learning, tiết kiệm tới 50% chi phí train; **AWS Inferentia** (Inf1, Inf2) cho **inference** hiệu năng cao, chi phí thấp.
- **Vì sao quan trọng:** Inf1 rẻ hơn tới 70% mỗi lần inference; Inf2 throughput cao hơn tới 4 lần so với Inf1. **Trn và Inf có environmental footprint thấp nhất**, nên là đáp án cho câu hỏi về tính bền vững.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| AWS AI services | Dịch vụ ML pre-trained, gọi API, không cần chuyên môn ML | "no ML expertise", "pre-trained", "least effort" |
| Amazon Comprehend | NLP: sentiment, entities, key phrases, language, PII | "analyze customer reviews", "sentiment", "extract entities" |
| Named Entity Recognition (NER) | Trích entity phổ thông: người, nơi, tổ chức, ngày | "people, places, organizations in text" |
| Comprehend Custom Entity Recognition | Trích entity riêng của doanh nghiệp | "policy numbers", "business-specific terms" |
| Comprehend Custom Classification | Phân loại tài liệu theo lớp tự định nghĩa | "categorize emails / documents into our own categories" |
| Amazon Translate | Dịch máy, localize | "translate", "localize website" |
| Amazon Transcribe | Speech → text (ASR) | "transcribe calls", "subtitles", "captions" |
| PII Redaction (Transcribe) | Tự xóa thông tin cá nhân khỏi transcript | "remove personal info from call transcripts" |
| Automatic Language Identification | Tự nhận diện ngôn ngữ trong audio | "multilingual audio" |
| Custom Vocabularies | Dạy Transcribe **từ** mới (brand, viết tắt) | "brand names", "acronyms misrecognized" |
| Custom Language Models | Dạy Transcribe **ngữ cảnh** chuyên ngành | "domain-specific context", "large volume of domain speech" |
| Toxicity Detection | Phát hiện lời nói độc hại qua giọng + nội dung | "toxic speech in voice chat / gaming" |
| Amazon Polly | Text → speech giọng tự nhiên | "read text aloud", "lifelike voice" |
| Lexicon | Quy định cách đọc một từ / viết tắt | "pronounce AWS as Amazon Web Services" |
| SSML | Markup điều khiển phát âm (ngắt, nhấn) | "add pause", "control pronunciation" |
| Speech marks | Metadata vị trí từ/câu trong audio | "lip-sync", "highlight words as spoken" |
| Amazon Rekognition | Phân tích ảnh/video: vật thể, khuôn mặt, text, moderation | "detect objects / faces in images", "celebrity" |
| Rekognition Custom Labels | Train nhận diện đối tượng riêng từ vài trăm ảnh | "detect our logo / products" |
| Content Moderation | Phát hiện nội dung không phù hợp trong ảnh/video | "filter inappropriate images" |
| Custom Moderation Adapter | Tinh chỉnh moderation bằng ảnh có nhãn của bạn | "improve moderation accuracy for our use case" |
| Amazon Lex | Build chatbot voice/text | "chatbot", "conversational interface" |
| Intent | Mục đích của user mà bot cần hiểu | "book hotel", "order pizza" |
| Slot | Tham số bot cần hỏi thêm để thực hiện intent | "date, size, location" |
| Amazon Personalize | Gợi ý cá nhân hóa real-time | "recommendations", "customers who bought X" |
| Recipe (Personalize) | Thuật toán dựng sẵn theo use case gợi ý | "trending items", "similar items", "next best action" |
| Amazon Textract | Trích text, forms, tables từ tài liệu scan | "scanned documents", "invoices", "forms" |
| Amazon Kendra | Enterprise search bằng ML (không nhận khách mới) | "intelligent document search" (đề cũ) |
| Bedrock Managed Knowledge Base | RAG managed, thay thế Kendra cho khách mới | "search company documents + generate answers" |
| Amazon Mechanical Turk | Crowdsourcing việc vặt cho con người | "crowdsource labeling", "pay per task" |
| Amazon Augmented AI (A2I) | Người duyệt dự đoán ML có độ tin cậy thấp | "human review of low-confidence predictions" |
| Transcribe Medical | Speech → text y khoa, HIPAA | "physician dictation", "medical terminology" |
| Comprehend Medical | NLP cho text lâm sàng, phát hiện PHI | "clinical notes", "PHI", "discharge summaries" |
| PHI | Protected Health Information | "patient health data" |
| AWS HealthScribe | Sinh clinical notes từ hội thoại bác sĩ–bệnh nhân | "generate clinical notes from conversations" |
| AWS Trainium (Trn) | Chip ML cho **training** | "train large models cost-effectively" |
| AWS Inferentia (Inf) | Chip ML cho **inference** | "low-cost, high-performance inference" |
| GPU instances (P, G) | EC2 có GPU cho ML/đồ họa | "GPU", "deep learning training" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Comprehend | Textract | Comprehend **hiểu** text đã có; Textract **lấy** text/form/table ra khỏi tài liệu scan | Scan/PDF → Textract trước, rồi mới Comprehend |
| Comprehend | Translate | Comprehend phân tích (sentiment, entities); Translate **dịch** | "Detect language" → Comprehend; "convert to Spanish" → Translate |
| Transcribe | Polly | Transcribe: **speech → text**; Polly: **text → speech** | **T**ranscribe = **T**o text |
| Transcribe | Comprehend | Transcribe chỉ chuyển giọng thành chữ; phân tích sentiment là Comprehend | Call center sentiment → Transcribe + Comprehend |
| Custom Vocabularies | Custom Language Models | Vocabularies: thêm **từ** (brand, viết tắt); Language Models: học **ngữ cảnh** từ text chuyên ngành | Từ cụ thể bị nhận sai → Vocabularies; cả lĩnh vực chuyên sâu → CLM; dùng cả hai = chính xác nhất |
| Lexicon (Polly) | SSML (Polly) | Lexicon: cách đọc **một từ** cố định; SSML: điều khiển **cách nói** từng đoạn (ngắt, nhấn, tốc độ) | "AWS → Amazon Web Services" → Lexicon |
| NER | Custom Entity Recognition | NER: entity **phổ thông** có sẵn; Custom: entity **riêng doanh nghiệp**, phải train | "Policy number", "internal product code" → Custom |
| Custom Classification | Custom Entity Recognition | Classification gán **cả tài liệu** vào một lớp; Entity trích **cụm từ** bên trong tài liệu | "Which category is this email?" → Classification |
| Rekognition | Textract | Rekognition: ảnh/video nói chung (vật thể, mặt, text ngắn); Textract: **tài liệu** (form, bảng) | Hóa đơn → Textract; biển số xe trong ảnh → Rekognition |
| Rekognition (labels có sẵn) | Rekognition Custom Labels | Có sẵn: vật thể phổ biến; Custom: **đối tượng riêng** (logo, sản phẩm) | "Our own logo" → Custom Labels |
| Rekognition Content Moderation | Bedrock Guardrails | Rekognition: kiểm duyệt **ảnh/video**; Guardrails: kiểm soát **input/output của FM** | Ảnh GenAI tạo ra cần kiểm duyệt → Rekognition |
| Lex | Polly / Transcribe | Lex là **chatbot hoàn chỉnh** (hiểu intent, slot); Polly/Transcribe chỉ là chuyển đổi giọng ↔ text | "Conversational bot" → Lex |
| Personalize | Comprehend / Bedrock | Personalize: **gợi ý** dựa trên hành vi user; không phải phân tích text hay sinh nội dung | "Recommend next product" → Personalize |
| Kendra | Bedrock Managed Knowledge Base | Kendra: search tài liệu (không nhận khách mới); BMKB: RAG managed + sinh câu trả lời | Khách mới cần search + trả lời → Bedrock KB |
| Mechanical Turk | Augmented AI (A2I) | MTurk: chợ việc vặt (thường dùng **trước** khi train, vd gắn nhãn); A2I: workflow **duyệt dự đoán** trong production | "Low-confidence predictions reviewed by humans" → A2I |
| A2I | SageMaker Ground Truth | Ground Truth: **gắn nhãn data training**; A2I: **duyệt dự đoán** sau khi model đã chạy | Before training → Ground Truth; after prediction → A2I |
| Transcribe Medical | Comprehend Medical | Transcribe Medical: audio → text; Comprehend Medical: text → thông tin lâm sàng, PHI | Pipeline: Transcribe Medical → Comprehend Medical |
| Comprehend Medical | AWS HealthScribe | Comprehend Medical: phân tích text có sẵn; HealthScribe: từ **hội thoại** → transcript + **clinical notes** | "Generate clinical notes from conversation" → HealthScribe |
| Trainium | Inferentia | Trainium: **train**; Inferentia: **inference** | **T**rainium = **T**rain; **I**nferentia = **I**nference |
| GPU instance | Trainium / Inferentia | GPU linh hoạt, đa dụng; Trn/Inf chuyên dụng, rẻ hơn, footprint môi trường thấp nhất | "Lowest environmental impact" → Trn/Inf |
| AI services | SageMaker AI | AI services: pre-trained, gọi API; SageMaker AI: tự build/train/deploy model | "No ML expertise" → AI services |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "sentiment of reviews / emails", "key phrases", "entities", "detect language" | Amazon Comprehend | |
| "detect PII in text" | `[+]` Comprehend (PII detection) | Trong audio → Transcribe PII Redaction |
| "classify documents into our own categories" | Comprehend Custom Classification | |
| "extract business-specific terms (policy numbers)" | Comprehend Custom Entity Recognition | |
| "group documents by topic" | ⚠️ đề cũ: Comprehend topic modeling; khách mới: LLM trên Bedrock | |
| "translate / localize content" | Amazon Translate | |
| "convert call recordings to text", "subtitles" | Amazon Transcribe | |
| "remove PII from call transcripts" | Transcribe PII Redaction | |
| "brand names / acronyms transcribed wrong" | Transcribe Custom Vocabularies | |
| "domain-specific speech, need context" | Transcribe Custom Language Models | |
| "detect toxic speech in audio" | Transcribe Toxicity Detection | |
| "read text aloud", "lifelike voice" | Amazon Polly | |
| "customize pronunciation of abbreviations" | Polly Lexicons | |
| "add pauses / emphasis in speech" | Polly SSML | |
| "highlight words as they are spoken", "lip-sync" | Polly Speech marks | |
| "detect objects, faces, celebrities in images/videos" | Amazon Rekognition | |
| "find our logo / our products in images" | Rekognition Custom Labels | |
| "filter inappropriate images" | Rekognition Content Moderation | + A2I cho human review |
| "chatbot", "intent", "slot" | Amazon Lex | |
| "personalized product recommendations" | Amazon Personalize | |
| "extract text, forms, tables from scanned documents" | Amazon Textract | |
| "process invoices / receipts", "ID documents" | `[+]` Textract AnalyzeExpense / AnalyzeID | |
| "intelligent search over enterprise documents" | ⚠️ Kendra (đề cũ) → Bedrock Managed Knowledge Base | Trợ lý cho nhân viên → Amazon Quick |
| "crowdsource simple human tasks / labeling" | Amazon Mechanical Turk | |
| "human review of low-confidence ML predictions" | Amazon Augmented AI (A2I) | |
| "physician dictation, medical terms" | Transcribe Medical | |
| "extract information / PHI from clinical notes" | Comprehend Medical | |
| "generate clinical notes from patient-clinician conversation" | AWS HealthScribe | |
| "lowest cost training of large deep learning models" | AWS Trainium (Trn) | |
| "lowest cost high-performance inference" | AWS Inferentia (Inf) | |
| "lowest environmental footprint for ML" | Trainium / Inferentia | |
| "build custom model, full control" | `[+]` SageMaker AI | Không phải AI service |

---

## 5. Câu hỏi tình huống

**Câu 1.** (Chọn 2) Một công ty muốn phân tích các **cuộc gọi tổng đài đã ghi âm** để biết khách hàng hài lòng hay không, đồng thời phải **xóa số thẻ tín dụng** khỏi transcript. Cần những gì?
- A. Amazon Transcribe với PII Redaction
- B. Amazon Comprehend để phân tích sentiment
- C. Amazon Polly
- D. Amazon Rekognition
- E. Amazon Personalize

<details><summary>Đáp án</summary>

**A, B.** Transcribe chuyển giọng nói thành text và xóa PII; Comprehend phân tích sentiment trên transcript.
- C sai: Polly làm text → speech.
- D sai: Rekognition dành cho ảnh/video.
- E sai: Personalize dùng để gợi ý sản phẩm.
</details>

**Câu 2.** Transcribe liên tục nhận sai tên sản phẩm "Xyliq" và các từ viết tắt nội bộ. Cách **nhanh nhất** để khắc phục?
- A. Custom Language Models
- B. Custom Vocabularies
- C. Amazon Translate
- D. Fine-tune một FM trên Bedrock

<details><summary>Đáp án</summary>

**B.** Custom Vocabularies thêm trực tiếp từ mới, brand name, viết tắt, kèm gợi ý phát âm.
- A sai: CLM cần train trên khối lượng lớn text chuyên ngành, phù hợp khi cần ngữ cảnh rộng.
- C sai: dịch thuật không liên quan.
- D sai: quá tốn công cho một vấn đề nhận diện từ.
</details>

**Câu 3.** Một hãng bảo hiểm muốn tự động trích **số hợp đồng** và **mã yêu cầu bồi thường** (định dạng riêng của công ty) từ email khách hàng. Nên dùng gì?
- A. Comprehend built-in NER
- B. Comprehend Custom Entity Recognition
- C. Comprehend Custom Classification
- D. Amazon Translate

<details><summary>Đáp án</summary>

**B.** Đây là entity riêng của doanh nghiệp, cần train custom recognizer.
- A sai: NER có sẵn chỉ nhận entity phổ thông (người, nơi, tổ chức).
- C sai: classification gán cả email vào một lớp, không trích cụm từ.
- D sai: dịch thuật không liên quan.
</details>

**Câu 4.** Một thương hiệu muốn tìm **logo của mình** trong ảnh đăng trên mạng xã hội. Họ có vài trăm ảnh đã gắn nhãn. Cách ít công nhất?
- A. Train CNN từ đầu trên SageMaker AI
- B. Amazon Rekognition Custom Labels
- C. Amazon Textract
- D. Amazon Comprehend

<details><summary>Đáp án</summary>

**B.** Custom Labels chỉ cần vài trăm ảnh có nhãn; Rekognition tự train model.
- A sai: tốn nhiều công và cần chuyên môn ML.
- C sai: Textract dành cho tài liệu.
- D sai: Comprehend dành cho text.
</details>

**Câu 5.** Model phân loại ảnh sản phẩm cho kết quả độ tin cậy thấp ở khoảng 3% trường hợp. Công ty muốn các trường hợp này được **người duyệt lại**, còn lại trả kết quả tự động. Dịch vụ nào phù hợp?
- A. Amazon Mechanical Turk
- B. Amazon Augmented AI (A2I)
- C. SageMaker Ground Truth
- D. Amazon Personalize

<details><summary>Đáp án</summary>

**B.** A2I chuyển dự đoán độ tin cậy thấp cho người duyệt, dự đoán độ tin cậy cao trả về ngay.
- A sai: MTurk là nguồn nhân lực (A2I có thể dùng MTurk), không phải workflow duyệt dự đoán.
- C sai: Ground Truth gắn nhãn data **trước** khi train.
- D sai: Personalize dùng để gợi ý.
</details>

**Câu 6.** (Chọn 2) Một bệnh viện muốn: (1) bác sĩ đọc ghi chú bằng giọng nói và chuyển thành text có thuật ngữ y khoa chính xác, (2) tự động trích tên thuốc, liều lượng và phát hiện PHI trong ghi chú đó. Dùng những dịch vụ nào?
- A. Amazon Transcribe Medical
- B. Amazon Comprehend Medical
- C. Amazon Polly
- D. Amazon Rekognition
- E. Amazon Lex

<details><summary>Đáp án</summary>

**A, B.** Transcribe Medical chuyển giọng nói thành text y khoa; Comprehend Medical trích thông tin lâm sàng và phát hiện PHI.
- C, D, E sai: không phục vụ hai yêu cầu này.
</details>

**Câu 7.** Một startup cần **train** một model deep learning rất lớn với chi phí thấp nhất và tác động môi trường thấp nhất. Sau đó họ cần **inference** hiệu năng cao, chi phí thấp. Lựa chọn nào đúng thứ tự?
- A. Inferentia để train, Trainium để inference
- B. Trainium để train, Inferentia để inference
- C. GPU P-series cho cả hai, vì luôn rẻ nhất
- D. Instance general-purpose cho cả hai

<details><summary>Đáp án</summary>

**B.** Trainium chuyên cho training, Inferentia chuyên cho inference, cả hai có environmental footprint thấp nhất.
- A sai: đảo ngược vai trò.
- C sai: GPU linh hoạt nhưng không phải lựa chọn rẻ hay xanh nhất.
- D sai: không tối ưu cho deep learning.
</details>

**Câu 8.** Một website thương mại điện tử muốn hiển thị "Sản phẩm bạn có thể thích" dựa trên lịch sử xem và mua của từng user, triển khai nhanh, không có đội ML. Nên chọn gì?
- A. Amazon Comprehend
- B. Amazon Personalize
- C. Amazon Lex
- D. Amazon Kendra

<details><summary>Đáp án</summary>

**B.** Personalize cho gợi ý cá nhân hóa real-time với recipe dựng sẵn (vd: User-Personalization).
- A sai: Comprehend dành cho NLP.
- C sai: Lex dùng để build chatbot.
- D sai: Kendra dùng để tìm kiếm tài liệu, và không còn nhận khách hàng mới.
</details>

---

### Nguồn đã đối chiếu
- AIF-C01 exam guide, Domain 1 (Task 1.2 managed AI services): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain1.html
- Amazon Comprehend feature availability change (topic modeling, event detection): https://docs.aws.amazon.com/comprehend/latest/dg/comprehend-availability-change.html
- Comprehend custom classification / custom entity recognition (real-time): https://docs.aws.amazon.com/comprehend/latest/dg/custom-sync.html, https://docs.aws.amazon.com/comprehend/latest/dg/detecting-cer-real-time.html
- Transcribe toxicity detection (batch, en-US): https://docs.aws.amazon.com/transcribe/latest/dg/toxicity.html
- Transcribe Medical: https://docs.aws.amazon.com/transcribe/latest/dg/transcribe-medical.html
- AWS HealthScribe: https://docs.aws.amazon.com/transcribe/latest/dg/health-scribe.html
- Polly voice engines: https://docs.aws.amazon.com/polly/latest/dg/voice-engines-polly.html
- Rekognition feature availability changes (Streaming Video, Bulk Image Analysis): https://docs.aws.amazon.com/rekognition/latest/dg/rekognition-availability-changes.html
- Rekognition People Pathing end of support (31/10/2025): https://docs.aws.amazon.com/rekognition/latest/dg/persons.html
- Rekognition Custom Labels: https://docs.aws.amazon.com/rekognition/latest/customlabels-dg/limits.html
- Amazon Kendra availability change → Bedrock Managed Knowledge Base: https://docs.aws.amazon.com/kendra/latest/dg/kendra-availability-change.html
- Amazon Lex V2 (migration từ V1): https://docs.aws.amazon.com/lexv2/latest/dg/migration.html
- Amazon Personalize recipes: https://docs.aws.amazon.com/personalize/latest/dg/how-new-data-influences-recommendations.html
- Amazon Textract: https://docs.aws.amazon.com/textract/latest/dg/what-is.html
- Amazon Augmented AI: https://docs.aws.amazon.com/sagemaker/latest/dg/a2i-getting-started.html
- Comprehend Medical: https://docs.aws.amazon.com/comprehend-medical/latest/dev/comprehendmedical-welcome.html
- Amazon Data Firehose (tên mới của Kinesis Data Firehose): https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-kinesisfirehose.html
- EC2 instance types (Trainium Trn1/Trn2): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html
- AWS Inferentia (Inf1 rẻ hơn tới 70%, Inf2 throughput gấp tới 4 lần) — nguồn web: https://aws.amazon.com/machine-learning/inferentia/, https://aws.amazon.com/blogs/machine-learning/aws-inferentia2-builds-on-aws-inferentia1-by-delivering-4x-higher-throughput-and-10x-lower-latency/
