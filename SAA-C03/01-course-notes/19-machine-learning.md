# SAA-C03 — Machine Learning Services

> Thuộc **Domain 3: Design High-Performing Architectures**: nhận diện dịch vụ AI/ML managed theo use case (ảnh/video, giọng nói, văn bản, tài liệu, gợi ý, dự báo, tìm kiếm).
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Rekognition**: phân tích ảnh/video (khuôn mặt, vật thể, chữ, **content moderation**, so khớp khuôn mặt, celebrity, đường đi trong video).
- **Transcribe**: giọng nói → văn bản (tự che **PII**, nhận diện ngôn ngữ tự động); **Polly**: văn bản → giọng nói (**lexicon**, **SSML**).
- **Translate**: dịch máy; **Comprehend**: NLP (ngôn ngữ, sentiment, entity, key phrase, topic); **Comprehend Medical**: thông tin y tế, PHI.
- **Lex**: chatbot giọng nói/văn bản (cùng công nghệ Alexa); **Connect**: contact center trên cloud, tích hợp Lex.
- **Textract**: trích xuất chữ, form, bảng từ tài liệu scan; **Personalize**: gợi ý real-time; **SageMaker AI**: tự xây, train, deploy model.
- ⚠️ Lỗi thời: **Kendra** đóng với khách mới từ 30/07/2026; **Lookout for Metrics** ngừng 10/10/2025, **Lookout for Vision** ngừng 31/10/2025 (nguồn web), **Lookout for Equipment** ngừng 07/10/2026.
- ❓ **Forecast** vẫn có trong whitepaper tổng quan; trạng thái mở cho khách mới chưa xác minh được trong docs.

---

## 2. Core Concepts

### Amazon Rekognition
- **Là gì:** computer vision managed cho ảnh và video: nhận diện vật thể, người, chữ, cảnh; phân tích khuôn mặt (giới tính, độ tuổi, cảm xúc), tìm và so khớp khuôn mặt, nhận diện người nổi tiếng, theo dõi đường đi (sports).
- **Dùng khi nào:** **content moderation** phát hiện nội dung không phù hợp (mạng xã hội, quảng cáo, game); xác minh danh tính.
- **Vì sao quan trọng:** đặt **minimum confidence threshold**; tích hợp **Amazon Augmented AI (A2I)** để người duyệt các trường hợp không chắc chắn.

### Amazon Transcribe
- **Là gì:** ASR (speech-to-text) bằng deep learning; **redaction PII** tự động; nhận diện ngôn ngữ tự động cho audio đa ngôn ngữ.
- **Dùng khi nào:** transcript cuộc gọi khách hàng, phụ đề, metadata cho media để tìm kiếm.
- **Vì sao quan trọng:** "audio → text" → Transcribe; "text → audio" → Polly (đừng đảo ngược).

### Amazon Polly
- **Là gì:** text-to-speech; **lexicon** định nghĩa cách đọc từ (vd: "AWS" → "Amazon Web Services"); **SSML** điều khiển ngắt, nhấn mạnh, thì thầm, phiên âm.
- **Dùng khi nào:** ứng dụng biết nói, đọc bài báo, hệ thống trả lời tự động.
- **Vì sao quan trọng:** từ khóa "lifelike speech", "pronunciation" → Polly.

### Amazon Translate & Comprehend
- **Là gì:** **Translate**: dịch máy chất lượng cao, khối lượng lớn; **Comprehend**: NLP serverless: ngôn ngữ, key phrase, entity, sentiment, phân tích cú pháp, gom chủ đề (topic modeling).
- **Dùng khi nào:** địa phương hóa nội dung website; phân tích tương tác khách hàng (email) tìm điểm tích cực/tiêu cực.
- **Vì sao quan trọng:** **Comprehend Medical** dành cho văn bản lâm sàng, phát hiện **PHI**; lưu tài liệu trên S3, có thể kết hợp Transcribe.

### Amazon Lex & Amazon Connect
- **Là gì:** **Lex**: chatbot với ASR + NLU, cùng công nghệ Alexa; **Connect**: contact center ảo, nhận cuộc gọi, tạo luồng xử lý, tích hợp CRM; trả theo dùng, rẻ hơn contact center truyền thống nhiều lần.
- **Dùng khi nào:** "đặt lịch hẹn qua điện thoại" → Connect nhận cuộc gọi → Lex hiểu ý định → Lambda đặt lịch trong CRM.
- **Vì sao quan trọng:** đề hay ghép bộ ba Connect + Lex + Lambda.

### Amazon SageMaker AI
- **Là gì:** dịch vụ managed để developer/data scientist chuẩn bị dữ liệu, gán nhãn, xây, train, tune và deploy model ML.
- **Dùng khi nào:** cần model tùy chỉnh mà dịch vụ AI có sẵn không đáp ứng (vd: dự đoán điểm thi từ dữ liệu riêng).
- **Vì sao quan trọng:** "custom ML model end-to-end" → SageMaker AI; dịch vụ còn lại là model dựng sẵn qua API.

### Amazon Textract
- **Là gì:** trích xuất chữ, chữ viết tay, form, bảng từ tài liệu scan (PDF, ảnh).
- **Dùng khi nào:** xử lý hóa đơn, báo cáo tài chính, hồ sơ y tế, giấy tờ tùy thân, form thuế.
- **Vì sao quan trọng:** "extract data from scanned documents" → Textract, không phải Rekognition.

### Amazon Personalize
- **Là gì:** gợi ý real-time managed, cùng công nghệ Amazon.com; triển khai trong vài ngày, không phải xây model.
- **Dùng khi nào:** gợi ý sản phẩm, tái xếp hạng, marketing trực tiếp theo từng khách.
- **Vì sao quan trọng:** dữ liệu từ S3 hoặc API real-time.

### ⚠️ Kendra, Forecast và họ Lookout
- **Là gì:** **Kendra**: tìm kiếm doanh nghiệp bằng ML, hỏi đáp ngôn ngữ tự nhiên trên tài liệu; **Forecast**: dự báo chuỗi thời gian; **Lookout for Metrics/Equipment/Vision**: phát hiện bất thường trong số liệu kinh doanh, thiết bị công nghiệp, ảnh sản phẩm.
- **Dùng khi nào:** đề cũ vẫn dùng các dịch vụ này làm đáp án theo từ khóa.
- **Vì sao quan trọng:** ⚠️ Kendra đóng với khách mới từ 30/07/2026; Lookout for Metrics ngừng 10/10/2025; Lookout for Vision ngừng 31/10/2025 (nguồn web); Lookout for Equipment ngừng 07/10/2026. ❓ Forecast: chưa xác minh trạng thái khách mới.

### `[+]` Amazon Bedrock
- **Là gì:** dịch vụ serverless truy cập các foundation model (văn bản, ảnh, embedding) qua API, kèm Knowledge Bases (RAG), Agents, Guardrails.
- **Dùng khi nào:** ứng dụng generative AI, chatbot hỏi đáp trên tài liệu doanh nghiệp (thay thế phổ biến cho Kendra trong thiết kế mới).
- **Vì sao quan trọng:** đề SAA mới có thể nhắc Bedrock ở mức nhận diện.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Rekognition | Phân tích ảnh/video | "detect faces", "content moderation" |
| Augmented AI (A2I) | Người duyệt kết quả ML | "human review of low-confidence results" |
| Transcribe | Speech-to-text | "transcribe calls", "subtitles" |
| PII redaction | Che thông tin cá nhân trong transcript | "remove personal information" |
| Polly | Text-to-speech | "convert text to speech" |
| SSML / Lexicon | Điều khiển cách đọc | "customize pronunciation" |
| Translate | Dịch máy | "localize content" |
| Comprehend | NLP, sentiment, entity | "analyze customer emails" |
| Comprehend Medical | NLP y tế, PHI | "clinical notes" |
| Lex | Chatbot giọng nói/văn bản | "conversational interface" |
| Connect | Contact center cloud | "call center", "phone calls" |
| SageMaker AI | Xây, train, deploy model | "custom ML model" |
| Textract | Trích xuất dữ liệu tài liệu | "scanned forms, tables" |
| Personalize | Gợi ý real-time | "product recommendations" |
| Kendra | Tìm kiếm tài liệu bằng ML (đóng khách mới) | "enterprise document search" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Transcribe | Polly | Transcribe: nói → chữ; Polly: chữ → nói | Polly = "parrot" nói |
| Rekognition | Textract | Rekognition: ảnh/video, khuôn mặt, vật; Textract: tài liệu, form, bảng | Hóa đơn → Textract |
| Comprehend | Translate | Comprehend hiểu ý nghĩa; Translate dịch | Sentiment → Comprehend |
| Comprehend | Comprehend Medical | Thường vs y tế (PHI) | Clinical → Medical |
| Lex | Connect | Lex là bot hiểu ngôn ngữ; Connect là tổng đài | Gọi điện → Connect + Lex |
| Kendra | OpenSearch | Kendra hỏi đáp tài liệu bằng ML; OpenSearch tìm kiếm/log tổng quát | Kendra đóng với khách mới |
| SageMaker AI | Dịch vụ AI dựng sẵn | SageMaker tự xây model; dịch vụ dựng sẵn gọi API | Custom → SageMaker |
| Personalize | Forecast | Personalize gợi ý; Forecast dự báo chuỗi thời gian | |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "detect inappropriate images uploaded by users" | Rekognition content moderation | + A2I nếu cần người duyệt |
| "transcribe customer calls, remove PII" | Transcribe | |
| "read news articles aloud" | Polly | |
| "translate website into 10 languages" | Translate | |
| "sentiment of customer reviews" | Comprehend | |
| "extract PHI from doctor notes" | Comprehend Medical | |
| "chatbot for booking" | Lex | |
| "cloud call center" | Connect | |
| "extract tables from scanned invoices" | Textract | |
| "real-time product recommendations" | Personalize | |
| "build and train custom model" | SageMaker AI | |
| "natural-language search over internal docs (new design)" | Bedrock Knowledge Bases | Kendra đóng với khách mới |

---

## 5. Câu hỏi tình huống

**Câu 1.** Mạng xã hội muốn tự động phát hiện ảnh nhạy cảm do user upload, người duyệt xem các trường hợp không chắc. Chọn gì?
- A. Textract
- B. Rekognition content moderation + Augmented AI
- C. Comprehend
- D. Polly

<details><summary>Đáp án</summary>

**B.**
- A sai: trích xuất văn bản tài liệu.
- C sai: NLP văn bản.
- D sai: text-to-speech.
</details>

**Câu 2.** Công ty cần chuyển ghi âm cuộc gọi thành văn bản và tự che số thẻ tín dụng. Chọn gì?
- A. Amazon Transcribe với PII redaction
- B. Amazon Polly
- C. Amazon Translate
- D. Amazon Lex

<details><summary>Đáp án</summary>

**A.**
- B sai: chiều ngược lại.
- C sai: dịch.
- D sai: chatbot.
</details>

**Câu 3.** (Chọn 2) Xây tổng đài nhận cuộc gọi đặt lịch khám, hiểu yêu cầu của khách và ghi vào hệ thống. Chọn gì?
- A. Amazon Connect
- B. Amazon Lex
- C. Amazon Rekognition
- D. Amazon Textract
- E. Amazon Personalize

<details><summary>Đáp án</summary>

**A, B.** Connect nhận cuộc gọi, Lex hiểu ý định (kèm Lambda ghi dữ liệu).
- C, D, E sai: không xử lý hội thoại.
</details>

**Câu 4.** Ngân hàng nhận hàng nghìn form vay vốn scan mỗi ngày, cần trích xuất trường và bảng. Chọn gì?
- A. Rekognition
- B. Textract
- C. Comprehend
- D. Transcribe

<details><summary>Đáp án</summary>

**B.**
- A sai: không tối ưu form/bảng.
- C sai: phân tích văn bản đã có.
- D sai: audio.
</details>

**Câu 5.** Phân tích sentiment hàng triệu review sản phẩm. Chọn gì?
- A. Comprehend
- B. Translate
- C. Polly
- D. Kendra

<details><summary>Đáp án</summary>

**A.**
- B sai: dịch.
- C sai: đọc văn bản.
- D sai: tìm kiếm tài liệu, và đóng với khách mới.
</details>

**Câu 6.** Website thương mại điện tử muốn gợi ý sản phẩm cá nhân hóa trong vài ngày mà không có đội ML. Chọn gì?
- A. SageMaker AI tự xây model
- B. Amazon Personalize
- C. Amazon Forecast
- D. Amazon Comprehend

<details><summary>Đáp án</summary>

**B.**
- A sai: cần chuyên môn ML và thời gian.
- C sai: dự báo chuỗi thời gian.
- D sai: NLP.
</details>

**Câu 7.** Ứng dụng đọc tin tức cần phát âm "AWS" thành "Amazon Web Services". Dùng gì?
- A. Polly lexicon
- B. Transcribe custom vocabulary
- C. Translate
- D. Lex slot

<details><summary>Đáp án</summary>

**A.** Lexicon định nghĩa cách đọc từ.
- B sai: dành cho nhận diện giọng nói.
- C, D sai: không điều khiển phát âm.
</details>

**Câu 8.** Bệnh viện cần trích xuất tên thuốc, liều lượng, chẩn đoán từ ghi chú bác sĩ. Chọn gì?
- A. Comprehend Medical
- B. Comprehend
- C. Textract
- D. Rekognition

<details><summary>Đáp án</summary>

**A.** Hiểu thuật ngữ y khoa, phát hiện PHI.
- B sai: NLP thông thường, không chuyên y tế.
- C sai: trích xuất chữ, không hiểu ngữ nghĩa y khoa.
- D sai: ảnh/video.
</details>

---

### Nguồn đã đối chiếu
- AWS Overview — Machine Learning (Lookout for Metrics ngừng 10/10/2025, danh sách dịch vụ): https://docs.aws.amazon.com/whitepapers/latest/aws-overview/machine-learning.html
- Amazon Kendra availability change (đóng với khách mới 30/07/2026): https://docs.aws.amazon.com/kendra/latest/dg/kendra-availability-change.html
- Amazon Lookout for Equipment discontinuation (07/10/2026): https://docs.aws.amazon.com/lookout-for-equipment/latest/ug/API_Reference.html
- Rekognition content moderation: https://docs.aws.amazon.com/rekognition/latest/dg/moderation.html
- Transcribe PII redaction: https://docs.aws.amazon.com/transcribe/latest/dg/pii-redaction.html
- Polly lexicons: https://docs.aws.amazon.com/polly/latest/dg/managing-lexicons.html
- Amazon Textract: https://docs.aws.amazon.com/textract/latest/dg/what-is.html
- Lookout for Vision ngừng 31/10/2025: nguồn web.
