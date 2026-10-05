# CLF-C02 — Machine Learning

> Thuộc **Domain 3** (AI/ML services). CLF chỉ hỏi mức "dịch vụ nào làm việc gì": ảnh/video, giọng nói, văn bản, chatbot, tài liệu, recommendation.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời.

---

## 1. TL;DR

- **Rekognition** = ảnh/video: vật thể, người, text, khuôn mặt, celebrity, kiểm duyệt nội dung.
- **Transcribe** = **speech → text** (phụ đề, cuộc gọi CSKH, tự xóa PII). **Polly** = **text → speech**.
- **Translate** = dịch ngôn ngữ. **Comprehend** = **NLP**: ngôn ngữ, key phrase, thực thể, **sentiment**.
- **Lex** = xây **chatbot/voicebot** (ASR + NLU, công nghệ của Alexa). **Connect** = **contact center** trên cloud.
- **Textract** = trích **text, chữ viết tay, form, bảng** từ tài liệu scan.
- **Personalize** = **recommendation** real-time (cùng công nghệ Amazon.com). **SageMaker AI** = nền tảng để developer/data scientist **tự build, train, deploy** model.
- ⚠️ **Kendra** (search tài liệu bằng ML) không nhận khách mới từ 30/07/2026; `[+]` exam guide hiện có **Amazon Q** trong danh sách in-scope.

---

## 2. Core Concepts

### Amazon Rekognition
- **Là gì:** dùng ML tìm **vật thể, người, text, cảnh** trong **ảnh và video**; phân tích và tìm kiếm khuôn mặt để xác minh người dùng, đếm người; tạo DB "khuôn mặt quen thuộc" hoặc so với người nổi tiếng.
- **Dùng khi nào:** gắn nhãn, **content moderation**, phát hiện text, phân tích khuôn mặt (giới tính, độ tuổi, cảm xúc), tìm và xác minh khuôn mặt, nhận diện celebrity.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: tính năng **Pathing** (theo dõi đường đi, vd: phân tích trận đấu) → **People Pathing đã hết hỗ trợ từ 31/10/2025**; Streaming Video và Bulk Image Analysis không nhận khách mới từ 30/04/2026.

### Amazon Transcribe
- **Là gì:** chuyển **giọng nói thành văn bản** bằng **ASR** (deep learning).
- **Dùng khi nào:** chép lời cuộc gọi CSKH, tạo **phụ đề** tự động, tạo metadata để tìm kiếm kho media.
- **Vì sao quan trọng:** tự **xóa PII** (Redaction), tự **nhận diện ngôn ngữ** trong audio đa ngôn ngữ.

### Amazon Polly
- **Là gì:** chuyển **văn bản thành giọng nói** tự nhiên bằng deep learning.
- **Dùng khi nào:** tạo ứng dụng "biết nói": đọc bài viết, thông báo bằng giọng nói.
- **Vì sao quan trọng:** cặp đối nghịch kinh điển với Transcribe.

### Amazon Translate
- **Là gì:** dịch ngôn ngữ tự nhiên, chính xác.
- **Dùng khi nào:** **localize** nội dung (website, ứng dụng) cho người dùng quốc tế, dịch khối lượng văn bản lớn.
- **Vì sao quan trọng:** "multilingual website" → Translate.

### Amazon Lex
- **Là gì:** cùng công nghệ với **Alexa**: ASR chuyển giọng nói thành text và **NLU** nhận diện **intent** của người dùng.
- **Dùng khi nào:** xây **chatbot**, **call center bot**.
- **Vì sao quan trọng:** "conversational interface", "chatbot" → Lex.

### Amazon Connect
- **Là gì:** **contact center ảo trên cloud**: nhận cuộc gọi, tạo contact flow, tích hợp CRM hoặc dịch vụ AWS.
- **Dùng khi nào:** dựng tổng đài CSKH; ví dụ: cuộc gọi vào Connect → Lex nhận intent "đặt lịch hẹn" → Lambda ghi vào CRM.
- **Vì sao quan trọng:** không trả trước, rẻ hơn giải pháp contact center truyền thống khoảng 80% (theo slide).

### Amazon Comprehend
- **Là gì:** dịch vụ **NLP** fully managed, serverless: phát hiện ngôn ngữ, trích key phrase, địa điểm, người, thương hiệu, sự kiện; hiểu văn bản **tích cực hay tiêu cực** (sentiment); phân tích bằng tokenization và parts of speech.
- **Dùng khi nào:** phân tích email/phản hồi khách hàng để biết điều gì dẫn tới trải nghiệm tốt/xấu.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: "tự sắp xếp tài liệu theo chủ đề" (**topic modeling**) và event detection → **không nhận khách mới**; sentiment, entity, key phrase, PII vẫn hoạt động.

### Amazon SageMaker AI
- **Là gì:** dịch vụ fully managed để developer/data scientist **build ML model**: gom các bước (chuẩn bị dữ liệu, gắn nhãn, train, tune, deploy) vào một chỗ và tự provision server.
- **Dùng khi nào:** cần model riêng (vd: dự đoán điểm thi từ số năm kinh nghiệm IT, thời gian học).
- **Vì sao quan trọng:** khác các dịch vụ AI dựng sẵn: SageMaker là nền tảng **tự làm**, các dịch vụ khác là API dùng ngay.

### Amazon Kendra
- ⚠️ Lỗi thời: Kendra **không nhận khách mới từ 30/07/2026** → AWS khuyến nghị **Amazon Bedrock Knowledge Bases (managed)** cho tìm kiếm tài liệu nội bộ.
- **Là gì:** dịch vụ **search tài liệu** dùng ML: trích câu trả lời từ text, PDF, HTML, PowerPoint, Word, FAQ; tìm kiếm bằng ngôn ngữ tự nhiên; học từ tương tác người dùng.
- **Dùng khi nào:** đề cũ: "enterprise search", "where is the IT support desk?" → Kendra.

### Amazon Personalize
- **Là gì:** dịch vụ ML fully managed để làm **recommendation cá nhân hóa real-time**; cùng công nghệ của Amazon.com.
- **Dùng khi nào:** gợi ý sản phẩm, re-ranking, marketing trực tiếp; tích hợp website, app, SMS, email; triển khai trong vài ngày, không cần tự build model.
- **Vì sao quan trọng:** exam guide CLF-C02 hiện xếp Personalize vào **out-of-scope**, vẫn nên nhận diện được.

### Amazon Textract
- **Là gì:** tự động trích **text, chữ viết tay và dữ liệu** từ tài liệu scan bằng AI/ML; đọc **form và bảng**; xử lý PDF, ảnh.
- **Dùng khi nào:** tài chính (hóa đơn, báo cáo), y tế (hồ sơ bệnh án, bảo hiểm), khu vực công (tờ khai thuế, giấy tờ tùy thân, hộ chiếu).
- **Vì sao quan trọng:** "extract data from scanned forms" → Textract, không phải Rekognition.

### `[+]` Amazon Q
- **Là gì:** trợ lý GenAI của AWS; **Amazon Q Developer** hỗ trợ trên AWS Console (hỏi về tài nguyên, chi phí, khắc phục lỗi) và trong chat app.
- **Dùng khi nào:** ⚠️ **Amazon Q Business** không nhận khách mới → thay bằng **Amazon Quick** (trợ lý agentic cho doanh nghiệp); Q Developer IDE plugin hết hỗ trợ 30/04/2027 → **Kiro**.
- **Vì sao quan trọng:** Amazon Q nằm trong danh sách in-scope của CLF-C02 (nhóm Machine Learning).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Amazon Rekognition | Phân tích ảnh/video | "detect objects/faces in images", "content moderation" |
| Content moderation | Phát hiện nội dung không phù hợp | "filter inappropriate images" |
| Amazon Transcribe | Speech-to-text (ASR) | "subtitles", "transcribe calls" |
| PII Redaction | Tự xóa thông tin cá nhân khỏi transcript | "remove personal info from transcripts" |
| Amazon Polly | Text-to-speech | "lifelike speech", "read text aloud" |
| Amazon Translate | Dịch máy | "localize website" |
| Amazon Lex | Chatbot/voicebot (ASR + NLU) | "conversational bot", "intent" |
| Amazon Connect | Contact center trên cloud | "call center", "contact flows" |
| Amazon Comprehend | NLP: sentiment, entity, key phrase | "sentiment analysis", "extract key phrases" |
| Amazon SageMaker AI | Build, train, deploy model ML | "build custom ML models" |
| Amazon Kendra | Search tài liệu bằng ML (không nhận khách mới) | "enterprise document search" |
| Amazon Personalize | Recommendation real-time | "personalized product recommendations" |
| Amazon Textract | Trích text/form/bảng từ tài liệu scan | "scanned documents", "handwriting", "forms" |
| Amazon Q | Trợ lý GenAI của AWS | "generative AI assistant" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Transcribe | Polly | Speech → text vs text → speech | **T**ranscribe = **T**ext ra; **P**olly = **P**hát ra tiếng |
| Transcribe | Translate | Chuyển giọng nói thành chữ vs dịch ngôn ngữ | Video tiếng Pháp → phụ đề tiếng Anh: Transcribe + Translate |
| Lex | Connect | Bot hội thoại vs tổng đài | Hay dùng cùng nhau |
| Lex | Comprehend | Hiểu **intent** để trả lời vs phân tích văn bản (sentiment, entity) | "Chatbot" → Lex |
| Comprehend | Textract | Phân tích **ý nghĩa** văn bản vs **trích** text từ tài liệu scan | "Scanned form" → Textract |
| Textract | Rekognition | Tài liệu, form, bảng vs ảnh/video tổng quát (text ngắn trong ảnh) | "Invoices, tax forms" → Textract |
| SageMaker AI | Dịch vụ AI dựng sẵn | Tự build model vs gọi API có sẵn | "Custom model, data scientists" → SageMaker |
| Kendra | Comprehend | Tìm kiếm tài liệu vs phân tích văn bản | "Search" → Kendra (⚠️) |
| Personalize | Comprehend | Recommendation vs NLP | "Recommend next product" → Personalize |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "detect faces / objects / celebrities in images" | Rekognition | |
| "moderate user-uploaded images" | Rekognition | |
| "generate subtitles from video audio" | Transcribe | |
| "convert articles into audio" | Polly | |
| "translate website into 10 languages" | Translate | |
| "build a chatbot" | Lex | |
| "cloud contact center" | Connect | |
| "analyze customer reviews for sentiment" | Comprehend | |
| "extract data from scanned invoices/forms" | Textract | |
| "personalized recommendations" | Personalize | Out-of-scope |
| "build and train custom ML models" | SageMaker AI | |
| "natural language search over internal documents" | Kendra (đề cũ) / Bedrock Knowledge Bases (hiện tại) | ⚠️ |
| "generative AI assistant in AWS console" | Amazon Q Developer | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Công ty muốn tự động tạo phụ đề cho video đào tạo. Chọn dịch vụ nào?
- A. Amazon Polly
- B. Amazon Transcribe
- C. Amazon Translate
- D. Amazon Rekognition

<details><summary>Đáp án</summary>

**B.** Transcribe chuyển giọng nói thành văn bản.
- A sai: ngược lại, text → speech.
- C sai: dịch văn bản.
- D sai: phân tích hình ảnh.
</details>

**Câu 2.** Ứng dụng đọc tin tức cần chuyển bài viết thành giọng đọc tự nhiên. Chọn gì?
- A. Amazon Polly
- B. Amazon Lex
- C. Amazon Comprehend
- D. Amazon Transcribe

<details><summary>Đáp án</summary>

**A.** Polly là text-to-speech.
- B sai: chatbot.
- C sai: NLP.
- D sai: speech-to-text.
</details>

**Câu 3.** Bộ phận marketing muốn biết khách hàng nghĩ tích cực hay tiêu cực về sản phẩm từ hàng nghìn đánh giá. Chọn gì?
- A. Amazon Comprehend
- B. Amazon Textract
- C. Amazon Personalize
- D. Amazon Polly

<details><summary>Đáp án</summary>

**A.** Comprehend phân tích sentiment.
- B sai: trích text từ tài liệu scan.
- C sai: recommendation.
- D sai: text-to-speech.
</details>

**Câu 4.** Công ty bảo hiểm cần trích dữ liệu từ các đơn yêu cầu bồi thường dạng scan có chữ viết tay và bảng. Chọn gì?
- A. Amazon Rekognition
- B. Amazon Textract
- C. Amazon Comprehend
- D. Amazon Translate

<details><summary>Đáp án</summary>

**B.** Textract trích text, chữ viết tay, form và bảng.
- A sai: phân tích ảnh/video tổng quát.
- C sai: phân tích ý nghĩa văn bản đã có.
- D sai: dịch.
</details>

**Câu 5.** (Chọn 2) Công ty muốn dựng tổng đài có bot tự hiểu yêu cầu đặt lịch của khách qua điện thoại. Dùng dịch vụ nào?
- A. Amazon Connect
- B. Amazon Lex
- C. Amazon Polly
- D. Amazon Textract
- E. Amazon Rekognition

<details><summary>Đáp án</summary>

**A, B.** Connect là contact center; Lex nhận intent của người gọi.
- C sai: chỉ chuyển text thành giọng nói (Lex/Connect đã có sẵn).
- D, E sai: tài liệu và ảnh.
</details>

**Câu 6.** Mạng xã hội cần tự động phát hiện ảnh không phù hợp do người dùng upload. Chọn gì?
- A. Amazon Rekognition
- B. Amazon Comprehend
- C. Amazon SageMaker AI
- D. Amazon Kendra

<details><summary>Đáp án</summary>

**A.** Rekognition có content moderation cho ảnh/video, dùng ngay không cần train.
- B sai: văn bản.
- C sai: phải tự build model, tốn công hơn.
- D sai: search tài liệu.
</details>

**Câu 7.** Đội data scientist muốn build, train và deploy model dự đoán churn trên dữ liệu riêng. Chọn gì?
- A. Amazon SageMaker AI
- B. Amazon Personalize
- C. Amazon Comprehend
- D. Amazon Lex

<details><summary>Đáp án</summary>

**A.** SageMaker AI là nền tảng tự build/train/deploy model.
- B sai: recommendation dựng sẵn.
- C sai: NLP dựng sẵn.
- D sai: chatbot.
</details>

**Câu 8.** Một website toàn cầu cần hiển thị nội dung bằng nhiều ngôn ngữ cho người dùng quốc tế. Chọn gì?
- A. Amazon Translate
- B. Amazon Transcribe
- C. Amazon Polly
- D. Amazon Connect

<details><summary>Đáp án</summary>

**A.** Translate dịch và localize nội dung.
- B sai: speech-to-text.
- C sai: text-to-speech.
- D sai: contact center.
</details>

---

### Nguồn đã đối chiếu
- CLF-C02 in-scope services (Machine Learning, có Amazon Q): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-in-scope-services.html
- CLF-C02 out-of-scope services (Amazon Personalize): https://docs.aws.amazon.com/aws-certification/latest/cloud-practitioner-02/clf-02-out-of-scope-services.html
- Rekognition People Pathing end of support (31/10/2025): https://docs.aws.amazon.com/rekognition/latest/dg/persons.html
- Rekognition feature availability changes: https://docs.aws.amazon.com/rekognition/latest/dg/rekognition-availability-changes.html
- Comprehend feature availability change (topic modeling, event detection): https://docs.aws.amazon.com/comprehend/latest/dg/comprehend-availability-change.html
- Amazon Kendra availability change: https://docs.aws.amazon.com/kendra/latest/dg/kendra-availability-change.html
- Amazon Q Business availability change → Amazon Quick: https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/qbusiness-availability-change.html
- Amazon Q Developer IDE plugins end of support → Kiro: https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-developer-ide-end-of-support.html
- Amazon Textract: https://docs.aws.amazon.com/textract/latest/dg/what-is.html
