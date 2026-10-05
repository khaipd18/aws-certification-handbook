# AIF-C01 — Generative AI with Amazon Bedrock

> Chương trọng tâm của **Domain 2 (Fundamentals of GenAI)** và **Domain 3 (Applications of Foundation Models)**. Đây là chương nặng nhất của AIF-C01.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời. Mốc kiến thức: 10/2026.

---

## 1. TL;DR

- **GenAI** là nhánh của Deep Learning, **tạo dữ liệu mới** (text, ảnh, audio, code, video) giống dữ liệu đã học. Nền tảng là **Foundation Model (FM)** được pre-train trên lượng lớn dữ liệu unlabeled.
- LLM sinh text **từng token một theo xác suất**, nên output **non-deterministic**: cùng một prompt có thể ra kết quả khác nhau.
- **Amazon Bedrock** là dịch vụ **fully managed, serverless** để dùng nhiều FM qua **một API thống nhất**. Data của bạn **không dùng để train** FM gốc; bản fine-tune là bản copy chỉ bạn dùng được.
- Thứ tự chi phí tăng dần: **Prompt engineering < RAG < Instruction-based fine-tuning < Domain adaptation**. Đề hỏi "rẻ nhất / ít công nhất" thì đi từ trái sang.
- **RAG** dùng Knowledge Base (embeddings + vector DB) để model trả lời từ **dữ liệu ngoài, cập nhật liên tục**, không đổi weights. **Fine-tuning** thì **thay đổi weights**.
- **Guardrails** lọc nội dung độc hại, PII, topic bị cấm, hallucination. **Agents** thực hiện tác vụ nhiều bước bằng cách gọi API/tool.
- Chi phí Bedrock chủ yếu do **số input/output tokens**. **Batch** rẻ hơn tới 50%; temperature, Top K, Top P **không ảnh hưởng giá**.
- Danh sách model thay đổi rất nhanh: nhiều model trong slide (Titan Text, Llama 2, Claude 2.1, SDXL, một số Nova đời 1) đã **EOL**. Xem mục Amazon Nova.

---

## 2. Core Concepts

### Generative AI & Foundation Model
- **Là gì:** FM được **pre-train** trên dữ liệu unlabeled khổng lồ, rồi **adapt** cho nhiều task (text generation, summarization, extraction, Q&A, chatbot, image generation).
- **Dùng khi nào:** cần một model đa năng thay vì train từng model riêng; train FM từ đầu tốn hàng chục triệu USD nên đa số doanh nghiệp **dùng lại FM có sẵn**.
- **Vì sao quan trọng:** FM có loại open-source (vd: Meta Llama, Google BERT) và loại commercial license (vd: Anthropic, OpenAI). Giấy phép là một tiêu chí chọn model.

### Large Language Model (LLM) & cách sinh text
- **Là gì:** FM chuyên text, có **hàng tỷ parameters**, train trên sách, báo, website; làm được translation, summarization, Q&A, content creation.
- **Dùng khi nào:** mỗi bước, model tạo danh sách **từ kế tiếp kèm xác suất**, rồi một thuật toán chọn từ (có yếu tố ngẫu nhiên), lặp lại đến hết câu.
- **Vì sao quan trọng:** đây là lý do output **non-deterministic** và có thể **hallucinate** (bịa ra thông tin nghe hợp lý nhưng sai).

### GenAI cho ảnh & Diffusion Models
- **Là gì:** text → image, image → image (vd: đổi phong cách anime), image → text (mô tả ảnh, đếm vật thể).
- **Dùng khi nào:** **Diffusion model** học bằng cách thêm nhiễu dần vào ảnh (forward diffusion), rồi sinh ảnh bằng cách khử nhiễu từ ảnh nhiễu theo prompt (reverse diffusion).
- **Vì sao quan trọng:** đề hỏi "kiến trúc nào tạo ảnh từ text" thì đáp án là **diffusion** (vd: Stable Diffusion).

### Amazon Bedrock
- **Là gì:** dịch vụ fully managed để build app GenAI: nhiều FM từ nhiều provider, **unified API**, pay-per-use, có sẵn Knowledge Bases (RAG), Agents, Guardrails, Evaluation.
- **Dùng khi nào:** muốn dùng FM **không quản hạ tầng**; thử nhanh model trong **Playground**.
- **Vì sao quan trọng:** **None of your data is used to train the FM.** Fine-tune tạo **bản copy riêng** chỉ account của bạn truy cập được.

### Chọn Base Foundation Model
- **Là gì:** tiêu chí gồm model type, performance, capabilities, constraints, compliance, mức customization, model size, inference options, licensing, **context window**, latency, multimodal.
- **Dùng khi nào:** model **nhỏ hơn thường rẻ hơn**; cần input/output nhiều loại (text + ảnh + video) thì chọn **multimodal model**.
- **Vì sao quan trọng:** ⚠️ Lỗi thời: bảng so sánh trong slide (Titan Text Express, Llama-2 70b-chat, Claude 2.1, SDXL 1.0, kèm giá) → các model này đã EOL. Chỉ học **tiêu chí chọn**, không học thuộc tên hay giá.

### Amazon Titan
- ⚠️ Lỗi thời: các Titan **text generation** model (vd: Titan Text Express) không còn trong danh sách model của Bedrock → nay text/multimodal của AWS dùng **Amazon Nova**.
- **Là gì:** các model Titan còn trong danh sách model chủ yếu là **embeddings** (Titan Text Embeddings V2, Titan Multimodal Embeddings G1) và Titan Image Generator G1 v2.
- **Vì sao quan trọng:** đề cũ hay hỏi "Titan Embeddings dùng cho RAG". Vẫn đúng về khái niệm, nhưng tên model có thể đổi.

### Model customization trên Bedrock
- **Là gì:** 3 phương pháp hiện hành: **Supervised fine-tuning**, **Reinforcement fine-tuning**, **Distillation**. Data training đặt trong **Amazon S3**, theo format quy định.
- **Dùng khi nào:** không phải model nào cũng fine-tune được; fine-tuning **thay đổi weights** của bản copy FM.
- **Vì sao quan trọng:** ❓ cần kiểm chứng: slide nhắc "domain adaptation" (continued pre-training với data unlabeled), nhưng trang customization hiện hành của Bedrock chỉ liệt kê 3 phương pháp trên.

### Supervised Fine-Tuning (SFT)
- **Là gì:** train thêm bằng **labeled data** dạng cặp **prompt → completion** (input-output) để model giỏi hơn ở task/lĩnh vực cụ thể.
- **Dùng khi nào:** chatbot cần persona/giọng điệu riêng, dùng data độc quyền (email, lịch sử CSKH), task mục tiêu (phân loại, đánh giá).
- **Vì sao quan trọng:** thường **rẻ hơn** re-train FM vì cần ít data và ít compute hơn, nhưng vẫn cần ML engineer để chuẩn bị data, train và evaluate.

### Reinforcement Fine-Tuning (RFT)
- **Là gì:** bạn chỉ cung cấp **prompts** và một **reward function** chấm điểm response; model tự sinh nhiều response và học dần để đạt điểm cao.
- **Dùng khi nào:** task **khách quan** thì viết reward function bằng **AWS Lambda**; task **chủ quan** (empathy, giọng điệu) thì dùng **một model khác làm judge**.
- **Vì sao quan trọng:** SFT = **bạn đưa sẵn output đúng**; RFT = **model tự sinh output, được chấm điểm**. Đây là điểm phân biệt hay bị hỏi.

### Distillation
- **Là gì:** model lớn (**teacher**) truyền kiến thức cho model nhỏ (**student**); bạn chỉ cần đưa prompts, Bedrock tự sinh response từ teacher để fine-tune student.
- **Dùng khi nào:** cần model **nhanh hơn, rẻ hơn** cho một use case cụ thể và chấp nhận giảm chút accuracy.
- **Vì sao quan trọng:** AWS công bố distilled model **nhanh hơn tới 500%, rẻ hơn tới 75%**, giảm accuracy dưới 2% với use case như RAG.

### Chi phí của fine-tuning
- **Là gì:** tính tiền training theo tokens (số token data × số epoch) + phí lưu model theo tháng; **chạy** custom model cũng đắt hơn base model.
- **Dùng khi nào:** chạy custom model qua **on-demand** (custom model deployment, trả theo token) hoặc **Provisioned Throughput** (tính theo giờ, có thể cam kết 1 hoặc 6 tháng để giảm giá).
- **Vì sao quan trọng:** ⚠️ slide ghi Provisioned Throughput "billed per month" → docs hiện ghi **tính theo giờ**, thời hạn cam kết là không cam kết / 1 tháng / 6 tháng.

### Model Evaluation
- **Là gì:** **Automatic** (built-in task: summarization, Q&A, classification, open-ended generation; dùng prompt dataset của bạn hoặc dataset có sẵn) và **Human** (work team: nhân viên, SME; thumbs up/down, ranking).
- **Dùng khi nào:** automatic có thể dùng **LLM-as-a-judge** để chấm điểm; human dùng khi cần đánh giá chủ quan (giọng văn, độ hữu ích).
- **Vì sao quan trọng:** **benchmark datasets** đo accuracy, speed, scalability, và giúp **phát hiện bias**; có thể tự tạo benchmark riêng cho doanh nghiệp.

### Metrics đánh giá FM
- **Là gì:** **ROUGE** (summarization, translation; ROUGE-N đếm n-gram trùng, ROUGE-L dùng longest common subsequence), **BLEU** (translation; dựa trên precision, phạt câu quá ngắn).
- **Dùng khi nào:** **BERTScore** đo **semantic similarity** bằng embeddings và cosine similarity; **Perplexity** đo khả năng đoán token kế tiếp (**càng thấp càng tốt**).
- **Vì sao quan trọng:** business metrics gồm User satisfaction, **ARPU**, Cross-domain performance, **Conversion rate**, Efficiency.

### RAG & Knowledge Bases
- **Là gì:** **Retrieval-Augmented Generation**: tìm thông tin liên quan trong vector DB, ghép vào prompt (**augmented prompt**), rồi FM sinh câu trả lời.
- **Dùng khi nào:** cần dữ liệu **real-time / cập nhật / nội bộ** mà FM chưa học; use cases: chatbot CSKH, nghiên cứu pháp lý, hỏi đáp y tế.
- **Vì sao quan trọng:** Bedrock tự **chunk → embeddings → lưu vector DB**. `[+]` Có 2 loại: **Bedrock Managed** KB (Bedrock lo datastore, có agentic retrieval, 7 connector) và **Customer-managed** KB (bạn tự chọn và quản lý vector DB).

### RAG Vector Databases & Data Sources
- **Là gì:** vector DB của AWS: **OpenSearch Service** (Serverless & Managed Cluster; kNN search tốc độ cao), **Aurora PostgreSQL** (pgvector), **Neptune Analytics** (graph, **GraphRAG**), **S3 Vectors** (rẻ, bền, query dưới 1 giây). Ngoài ra có bên thứ ba như Pinecone, Redis Enterprise Cloud.
- **Dùng khi nào:** data sources (Managed KB): S3, SharePoint, Confluence, Web Crawler, Google Drive, OneDrive, Custom. Slide có nhắc Salesforce ❓ cần kiểm chứng (không có trong danh sách connector của Managed KB).
- **Vì sao quan trọng:** đề hay hỏi "lưu embeddings ở đâu" hoặc "rẻ nhất cho vector" (S3 Vectors) hoặc "quan hệ giữa các thực thể" (Neptune / GraphRAG).

### Tokenization, Context Window, Embeddings
- **Là gì:** **Tokenization** chia text thành token (word-based hoặc subword); **context window** là số token tối đa model xem xét một lần.
- **Dùng khi nào:** context window là **yếu tố đầu tiên** khi chọn model cho tài liệu dài, nhưng càng lớn càng tốn memory/compute.
- **Vì sao quan trọng:** **Embeddings** biến text/ảnh/audio thành **vector nhiều chiều**; từ gần nghĩa (dog, puppy) có vector gần nhau. Đó là nền tảng của semantic search và RAG.

### Amazon Bedrock Guardrails
- **Là gì:** kiểm soát tương tác user ↔ FM. `[+]` Gồm 6 loại filter: **Content filters** (hate, insults, sexual, violence, misconduct, prompt attack), **Denied topics**, **Word filters**, **Sensitive information filters** (PII: block/mask), **Contextual grounding checks**, **Automated Reasoning checks**.
- **Dùng khi nào:** chặn topic không liên quan, ẩn PII, giảm **hallucination** (contextual grounding cho RAG); tạo được nhiều guardrail và theo dõi các input vi phạm.
- **Vì sao quan trọng:** Guardrails áp được cho **cả input lẫn output**, và dùng được độc lập với model qua ApplyGuardrail API.

### Agents: Bedrock Agents → AgentCore
- ⚠️ Lỗi thời: **Amazon Bedrock Agents** (nay gọi là **Agents Classic**) không còn nhận khách hàng mới → nay dùng **Amazon Bedrock AgentCore**.
- **Là gì:** agent nhận task, dùng FM lập kế hoạch nhiều bước (chain of thought), gọi **action groups** (API, Lambda, schema OpenAPI) và **Knowledge Bases**, rồi tổng hợp kết quả thành câu trả lời.
- **Dùng khi nào:** AgentCore gồm các service như Runtime (host agent serverless), Memory, Gateway (biến API/Lambda thành tool MCP), Identity, Code Interpreter; dùng được với mọi framework và mọi model.

### Monitoring: Bedrock & CloudWatch
- **Là gì:** **Model Invocation Logging** gửi log mọi lần gọi model (text, ảnh, embeddings) tới **CloudWatch Logs** và/hoặc **S3**; phân tích bằng CloudWatch Logs Insights.
- **Dùng khi nào:** **CloudWatch Metrics** theo dõi Invocations, InvocationThrottles, Input/OutputTokenCount...; đặt **CloudWatch Alarms** trên metric.
- **Vì sao quan trọng:** ❓ cần kiểm chứng: slide nêu metric `ContentFilteredCount` để theo dõi Guardrails; docs hiện tại của Guardrails dùng metric **InvocationsIntervened** (lọc theo loại policy).

### Bedrock Pricing & Cost savings
- **Là gì:** **On-Demand** (không cam kết; text tính theo input/output tokens, embeddings theo input tokens, image theo số ảnh), **Batch** (output là file trên S3, giảm tới **50%**), **Provisioned Throughput** (mua Model Units, đảm bảo throughput).
- **Dùng khi nào:** on-demand cho workload khó đoán; batch cho việc không gấp; Provisioned Throughput để **giữ capacity**, thường **không** phải để tiết kiệm.
- **Vì sao quan trọng:** `[+]` ngoài slide còn có **service tiers** (Standard, Priority nhanh hơn, Flex rẻ hơn, Reserved cam kết) và **prompt caching** (token đọc từ cache tính giá thấp hơn).

### `[+]` Inference parameters: Temperature, Top K, Top P
- **Là gì:** **Temperature** thấp thì output ổn định, có tính quyết định hơn; cao thì sáng tạo, ngẫu nhiên hơn. **Top K** = số token ứng viên; **Top P** = top % xác suất cộng dồn.
- **Dùng khi nào:** giảm giá trị khi cần câu trả lời chính xác, nhất quán (FAQ, trích xuất); tăng khi cần sáng tạo (viết quảng cáo, brainstorm).
- **Vì sao quan trọng:** đề hay hỏi "làm sao để output ít ngẫu nhiên hơn": giảm temperature. Và các tham số này **không đổi giá**.

### Amazon Nova
- ⚠️ Lỗi thời: Nova **Premier** và **Sonic** (đời 1) EOL ngày 14/09/2026; Nova **Canvas** (ảnh) và **Reel** (video) EOL ngày 30/09/2026 → nay dùng các model **Nova 2**.
- **Là gì:** dòng FM do AWS xây dựng, nhanh, rẻ, enterprise-ready, dùng qua Bedrock. Còn trong danh sách: Nova **Pro** (cân bằng), **Lite** (multimodal giá rẻ), **Micro** (chỉ text, latency thấp nhất).
- **Dùng khi nào:** **Nova 2** (tới 1M token context, có reasoning): **Nova 2 Lite** (reasoning, multimodal, rẻ), **Nova 2 Sonic** (speech-to-speech real-time), **Nova Multimodal Embeddings** (embeddings cho RAG / semantic search). **Nova 2 Omni** ❓ cần kiểm chứng (vẫn ở dạng preview).

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Generative AI | DL tạo nội dung mới giống dữ liệu đã học | "generate", "create new content" |
| Foundation Model (FM) | Model lớn pre-train trên data unlabeled đa dạng, adapt cho nhiều task | "general-purpose", "pre-trained on broad data" |
| LLM | FM chuyên text, hàng tỷ parameters | "human-like text", "billions of parameters" |
| Multimodal model | Model nhận/sinh nhiều loại dữ liệu (text, ảnh, video, audio) | "text and images in the same request" |
| Prompt | Input bạn gửi cho model | "instruction to the model" |
| Non-deterministic | Cùng prompt có thể ra output khác nhau | "different answers for the same question" |
| `[+]` Hallucination | Model trả lời tự tin nhưng sai hoặc bịa | "fabricated", "factually incorrect but plausible" |
| Token / Tokenization | Đơn vị text model xử lý; chia text thành token (word/subword) | "cost based on tokens", "text split into pieces" |
| Context window | Số token tối đa model xem xét một lần | "long documents", "how much text the model can consider" |
| Embeddings | Vector số biểu diễn ý nghĩa của text/ảnh/audio | "semantic similarity", "vector representation" |
| Vector database | DB lưu và tìm kiếm embeddings theo độ tương đồng (kNN) | "similarity search", "store embeddings" |
| Diffusion model | Sinh ảnh bằng cách khử nhiễu dần (reverse diffusion) | "image generation from text", "Stable Diffusion" |
| Amazon Bedrock | Dịch vụ serverless dùng nhiều FM qua unified API | "managed GenAI", "no infrastructure", "multiple FMs" |
| Playground | Giao diện thử model trực tiếp trên console | "experiment with models without code" |
| RAG | Lấy dữ liệu ngoài ghép vào prompt, không đổi weights | "up-to-date / company data", "reduce hallucination", "no retraining" |
| Knowledge Base | Tính năng RAG managed của Bedrock (ingest → embed → vector store) | "connect company documents to FM" |
| GraphRAG | RAG dựa trên graph quan hệ giữa thực thể (Neptune Analytics) | "relationships between entities" |
| Fine-tuning | Train thêm FM bằng data của bạn, **thay đổi weights** | "adapt model", "specific tone/persona", "proprietary data" |
| Supervised Fine-Tuning | Fine-tune bằng cặp prompt-completion **có nhãn** | "labeled input-output pairs" |
| Reinforcement Fine-Tuning | Fine-tune bằng **reward function** chấm điểm response | "reward function", "feedback-based", "score responses" |
| Reward function | Hàm chấm điểm response: Lambda (khách quan) hoặc judge model (chủ quan) | "evaluate response quality automatically" |
| Distillation | Teacher (lớn) → Student (nhỏ), nhanh và rẻ hơn | "smaller, faster, cheaper model with similar accuracy" |
| Domain adaptation | Train FM trên data chuyên ngành (thường unlabeled), tốn compute nhất | "industry-specific terminology", "domain corpus" |
| Instruction-based fine-tuning | Fine-tune bằng các instruction/ví dụ cụ thể | "follow specific instructions better" |
| Automatic evaluation | Chấm điểm tự động bằng metric hoặc LLM-as-a-judge | "quick, scalable evaluation" |
| Human evaluation | Work team (nhân viên/SME) chấm điểm thủ công | "subjective quality", "SME review" |
| Benchmark dataset | Bộ câu hỏi/đáp án chuẩn để đo model, kể cả bias | "standardized test set", "detect bias" |
| ROUGE | Đo overlap n-gram; cho **summarization** | "summarization quality" |
| BLEU | Đo precision n-gram, phạt câu ngắn; cho **translation** | "translation quality" |
| BERTScore | Đo **semantic similarity** bằng embeddings (cosine) | "meaning, not exact words" |
| Perplexity | Khả năng đoán token kế tiếp; **thấp = tốt** | "how well model predicts next token" |
| ARPU / Conversion rate | Business metrics: doanh thu/user, tỷ lệ chuyển đổi | "business impact of GenAI app" |
| Guardrails | Bộ lọc input/output: harmful content, denied topics, PII, grounding | "block topics", "mask PII", "safety" |
| Contextual grounding check | Phát hiện response không bám vào nguồn (hallucination trong RAG) | "response not supported by source" |
| Bedrock AgentCore | Nền tảng build, deploy, vận hành agent (thay Bedrock Agents) | "deploy agents at scale", "any framework" |
| Action group | Tập hành động agent được phép làm (API/Lambda + OpenAPI schema) | "agent calls company APIs" |
| Model Invocation Logging | Log toàn bộ request/response tới CloudWatch Logs / S3 | "audit all prompts and responses" |
| On-Demand | Trả theo token, không cam kết | "unpredictable workload" |
| Batch inference | Xử lý nhiều request một lần, output ra S3, rẻ hơn tới 50% | "not time-sensitive", "large volume", "lowest cost" |
| Provisioned Throughput | Mua Model Units, đảm bảo throughput, tính theo giờ | "guaranteed capacity", "consistent high throughput" |
| `[+]` Temperature | Độ ngẫu nhiên: thấp = ổn định, cao = sáng tạo | "more consistent / more creative output" |
| `[+]` Top K / Top P | Giới hạn pool token ứng viên theo số lượng / % xác suất | "limit to most likely words" |
| Amazon Nova | Dòng FM của AWS trên Bedrock | "AWS's own foundation model" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Prompt engineering | RAG | PE chỉ sửa cách hỏi; RAG **thêm dữ liệu ngoài** vào prompt. Cả hai **không đổi weights** | Cần data công ty / mới cập nhật → RAG |
| RAG | Fine-tuning | RAG: data ngoài, cập nhật ngay, không train; FT: **đổi weights**, tốn compute, data "đóng băng" lúc train | Data thay đổi thường xuyên → **RAG**; cần đổi giọng văn/hành vi → **FT** |
| Instruction-based fine-tuning | Domain adaptation | Instruction: dạy model làm theo hướng dẫn/task cụ thể; Domain: train trên corpus chuyên ngành, **tốn nhất** | Thuật ngữ y tế/pháp lý chuyên sâu → domain adaptation |
| Supervised FT | Reinforcement FT | SFT: **bạn đưa sẵn** output đúng (labeled pairs); RFT: model **tự sinh**, reward function **chấm điểm** | Có "reward function" / "score" → RFT |
| Fine-tuning | Distillation | FT: model giỏi hơn ở task; Distillation: model **nhỏ hơn, nhanh hơn, rẻ hơn**, giữ gần bằng accuracy | "Reduce cost and latency, keep accuracy" → Distillation |
| Bedrock | SageMaker AI | Bedrock: dùng FM có sẵn qua API, serverless; SageMaker AI: tự build/train/deploy model ML bất kỳ, nhiều kiểm soát hơn | GenAI + ít vận hành → Bedrock; ML tùy biến hoàn toàn → SageMaker AI |
| Pre-training | Fine-tuning | Pre-train: từ đầu trên data unlabeled khổng lồ (cực đắt); FT: train thêm trên data nhỏ của bạn | Doanh nghiệp gần như **không bao giờ** pre-train FM |
| Training | Inference | Training/fine-tuning: đổi weights; Inference: gọi model sinh output | Pricing on-demand / batch / PT là chuyện **inference** |
| ROUGE | BLEU | ROUGE (Recall-oriented) → **summarization**; BLEU (Bilingual) → **translation** | **B**LEU = **B**ilingual = dịch |
| ROUGE / BLEU | BERTScore | ROUGE/BLEU so khớp **từ** (n-gram); BERTScore so **ý nghĩa** (embeddings) | "Semantic", "meaning" → BERTScore |
| Automatic evaluation | Human evaluation | Auto: nhanh, scale, metric/LLM judge; Human: chủ quan, SME, chậm và đắt hơn | "Brand voice", "subjective" → Human |
| Model metrics | Business metrics | Model: ROUGE, BLEU, accuracy...; Business: ARPU, conversion rate, user satisfaction | "Revenue", "purchases" → business metrics |
| Guardrails | Fine-tuning | Guardrails **lọc** input/output lúc chạy; FT **dạy lại** model | Chặn topic / ẩn PII → Guardrails, không phải FT |
| Guardrails | IAM | Guardrails kiểm soát **nội dung**; IAM kiểm soát **ai được gọi** model | "Who can access" → IAM |
| Knowledge Base (RAG) | Agent | KB chỉ **tìm và trả lời**; Agent **lập kế hoạch + gọi API làm việc** (có thể dùng KB) | "Place order", "update system" → Agent |
| On-Demand | Provisioned Throughput | On-demand: trả theo token, không cam kết; PT: trả theo giờ, đảm bảo capacity | PT để **reserve capacity**, thường không phải để tiết kiệm |
| On-Demand | Batch | Batch: không real-time, output ra S3, rẻ hơn tới 50% | "Overnight", "not urgent", "thousands of documents" → Batch |
| Temperature cao | Temperature thấp | Cao: sáng tạo, đa dạng; Thấp: ổn định, nhất quán | Không ảnh hưởng **giá** |
| Context window | Embeddings dimension | Context window: số token đầu vào tối đa; dimension: độ dài vector | Tài liệu dài → context window |
| Nova đời 1 | Nova 2 | Premier, Sonic, Canvas, Reel đời 1 đã EOL; Nova 2 có 1M context + reasoning | Học khái niệm, đừng học thuộc tên model |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "least effort / lowest cost to improve output" | Prompt engineering | Bậc rẻ nhất |
| "use company documents", "up-to-date info", "without retraining" | RAG / Bedrock Knowledge Bases | |
| "reduce hallucinations by grounding in documents" | RAG + Guardrails contextual grounding | |
| "specific tone / persona / brand voice" | Fine-tuning (SFT) | |
| "labeled prompt-completion pairs" | Supervised fine-tuning | |
| "reward function", "score generated responses" | Reinforcement fine-tuning | Lambda (objective) / judge model (subjective) |
| "smaller, faster, cheaper model with similar accuracy" | Distillation | |
| "industry-specific corpus, unlabeled data" | Domain adaptation (continued pre-training) | ❓ cần kiểm chứng tính năng hiện hành trên Bedrock |
| "store embeddings", "similarity search" | Vector DB: OpenSearch, Aurora PostgreSQL, S3 Vectors... | |
| "cheapest durable vector storage" | Amazon S3 Vectors | |
| "relationships between entities / graph" | Neptune Analytics (GraphRAG) | |
| "convert text to vectors" | Embeddings model (vd: Titan Text Embeddings, Nova Multimodal Embeddings) | |
| "evaluate summarization" | ROUGE | |
| "evaluate translation" | BLEU | |
| "semantic similarity of generated text" | BERTScore | |
| "subjective quality, SMEs review" | Human evaluation | |
| "detect bias in model output" | Benchmark dataset (bias) | |
| "block a topic", "mask PII", "filter hate speech" | Bedrock Guardrails | |
| "multi-step tasks, call APIs, take actions" | Bedrock AgentCore (Agents Classic cho khách cũ) | |
| "audit / log all prompts and responses" | Model Invocation Logging → CloudWatch Logs / S3 | |
| "alarm when guardrail blocks too much" | CloudWatch Metrics + Alarm | Metric Guardrails: InvocationsIntervened |
| "unpredictable traffic, no commitment" | On-Demand | |
| "process thousands of documents overnight cheaply" | Batch inference | Rẻ hơn tới 50% |
| "guaranteed throughput", "custom model in production" | Provisioned Throughput | |
| "more deterministic / consistent output" | Giảm Temperature (Top P / Top K) | Không đổi giá |
| "main driver of Bedrock cost" | Số input/output tokens | |
| "generate image from text" | Diffusion model (image generation FM) | |
| "AWS's own FM", "real-time voice conversation" | Amazon Nova; Nova 2 Sonic cho speech-to-speech | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một công ty muốn chatbot nội bộ trả lời câu hỏi dựa trên các chính sách HR được cập nhật hàng tuần. Họ muốn chi phí thấp và không phải train lại model mỗi khi chính sách thay đổi. Nên dùng gì?
- A. Fine-tune FM mỗi tuần với chính sách mới
- B. Amazon Bedrock Knowledge Bases (RAG)
- C. Tăng temperature của model
- D. Distillation từ model lớn hơn

<details><summary>Đáp án</summary>

**B.** RAG truy xuất tài liệu mới nhất lúc trả lời, không cần train lại.
- A sai: tốn kém, và data bị "đóng băng" tại thời điểm train.
- C sai: temperature chỉ đổi độ ngẫu nhiên, không thêm kiến thức.
- D sai: distillation làm model nhỏ hơn, không cập nhật kiến thức.
</details>

**Câu 2.** (Chọn 2) Phát biểu nào **đúng** về Amazon Bedrock?
- A. Dữ liệu fine-tune của khách hàng được dùng để cải thiện FM gốc cho mọi người.
- B. Fine-tuning tạo ra một bản copy riêng của FM mà chỉ khách hàng đó truy cập được.
- C. Bedrock cung cấp một unified API để gọi nhiều FM từ nhiều provider.
- D. Khách hàng phải tự quản lý GPU instance để chạy inference on-demand.
- E. Temperature cao hơn làm tăng chi phí mỗi request.

<details><summary>Đáp án</summary>

**B, C.**
- A sai: data của bạn không dùng để train FM gốc.
- D sai: Bedrock là serverless.
- E sai: temperature, Top K, Top P không ảnh hưởng giá; chi phí phụ thuộc số token.
</details>

**Câu 3.** Một team muốn chatbot hỗ trợ kỹ thuật trả lời **đồng cảm và biết hỏi chẩn đoán**. Họ có nhiều prompt khách hàng nhưng **không có** câu trả lời mẫu; họ có thể mô tả tiêu chí một câu trả lời tốt để một model khác chấm điểm. Phương pháp customization nào phù hợp?
- A. Supervised fine-tuning
- B. Reinforcement fine-tuning
- C. Distillation
- D. Prompt caching

<details><summary>Đáp án</summary>

**B.** RFT chỉ cần prompts + reward function. Ở đây là task chủ quan, nên dùng judge model chấm điểm.
- A sai: SFT cần cặp prompt-completion có nhãn, mà team không có.
- C sai: distillation nhằm thu nhỏ model.
- D sai: prompt caching là tối ưu chi phí, không phải customization.
</details>

**Câu 4.** Một công ty đang dùng một FM lớn, chất lượng tốt nhưng chậm và đắt. Họ muốn model **nhanh hơn, rẻ hơn** cho đúng use case của mình mà accuracy gần như không đổi. Nên dùng gì?
- A. Tăng context window
- B. Provisioned Throughput
- C. Model distillation
- D. Domain adaptation fine-tuning

<details><summary>Đáp án</summary>

**C.** Teacher (model lớn) truyền kiến thức cho student (model nhỏ), nên nhanh hơn, rẻ hơn và giữ gần bằng accuracy.
- A sai: context window lớn hơn còn làm tốn compute hơn.
- B sai: PT đảm bảo capacity, không giảm giá mỗi token, thường còn đắt hơn.
- D sai: domain adaptation là phương pháp tốn kém nhất và không làm model nhỏ đi.
</details>

**Câu 5.** (Chọn 2) Ứng dụng tóm tắt hồ sơ y tế cần: (1) **không bao giờ** hiển thị số an sinh xã hội của bệnh nhân, và (2) phát hiện bản tóm tắt chứa thông tin **không có trong tài liệu gốc**. Cấu hình Guardrails nào?
- A. Sensitive information filters
- B. Word filters chứa tên đối thủ cạnh tranh
- C. Contextual grounding checks
- D. Denied topics về chính trị
- E. Provisioned Throughput

<details><summary>Đáp án</summary>

**A, C.** Sensitive information filter block/mask PII; contextual grounding check phát hiện response không bám vào nguồn (hallucination).
- B, D sai: không liên quan đến hai yêu cầu.
- E sai: là lựa chọn pricing, không phải guardrail.
</details>

**Câu 6.** Một team cần đánh giá model tóm tắt văn bản tự động, so với các bản tóm tắt tham chiếu do người viết. Metric nào phù hợp nhất?
- A. BLEU
- B. ROUGE
- C. ARPU
- D. Perplexity

<details><summary>Đáp án</summary>

**B.** ROUGE (Recall-Oriented) được thiết kế cho summarization, đo n-gram trùng với bản tham chiếu.
- A sai: BLEU chủ yếu cho translation.
- C sai: ARPU là business metric.
- D sai: perplexity đo khả năng đoán token kế tiếp, không so với bản tham chiếu.
</details>

**Câu 7.** Một công ty cần tạo mô tả sản phẩm cho **2 triệu SKU** trong catalog. Không cần kết quả ngay, chỉ cần xong trong vài ngày, với **chi phí thấp nhất**. Nên dùng chế độ nào trên Bedrock?
- A. On-Demand
- B. Provisioned Throughput với cam kết 6 tháng
- C. Batch inference
- D. Giảm temperature về 0

<details><summary>Đáp án</summary>

**C.** Batch xử lý số lượng lớn, output ra S3, rẻ hơn tới 50% so với on-demand.
- A sai: không có giảm giá.
- B sai: dùng để giữ capacity, không phải lựa chọn rẻ nhất cho job một lần.
- D sai: temperature không ảnh hưởng giá.
</details>

**Câu 8.** (Chọn 2) Một trợ lý mua sắm cần: tra lịch sử đơn hàng của khách qua API nội bộ, tra chính sách đổi trả trong tài liệu công ty, rồi **tự đặt đơn mới** khi khách đồng ý. Những thành phần nào cần dùng?
- A. Agent có action group gọi API đơn hàng
- B. Knowledge Base chứa chính sách đổi trả
- C. Distillation
- D. BLEU score
- E. Word filters

<details><summary>Đáp án</summary>

**A, B.** Agent lập kế hoạch nhiều bước và gọi API (action group) để tra cứu và đặt đơn; Knowledge Base cung cấp chính sách qua RAG. Với khách hàng mới, xây agent bằng **Bedrock AgentCore**.
- C sai: distillation là kỹ thuật tối ưu model.
- D sai: BLEU là metric đánh giá.
- E sai: word filters chỉ chặn từ ngữ.
</details>

---

### Nguồn đã đối chiếu
- Bedrock model lifecycle (Legacy/EOL: Nova Premier, Sonic, Canvas, Reel): https://docs.aws.amazon.com/bedrock/latest/userguide/model-lifecycle-legacy.html
- Bedrock model lifecycle (chính sách mới): https://docs.aws.amazon.com/bedrock/latest/userguide/model-lifecycle.html
- Bedrock Amazon model cards (Nova, Titan): https://docs.aws.amazon.com/bedrock/latest/userguide/model-cards-amazon.html
- What is Amazon Nova 2: https://docs.aws.amazon.com/nova/latest/nova2-userguide/what-is-nova-2.html
- What's new in Amazon Nova 2: https://docs.aws.amazon.com/nova/latest/nova2-userguide/whats-new.html
- Nova 2 Omni đang preview (nguồn báo chí, không phải docs AWS): https://www.eweek.com/news/amazon-nova-ai-overhaul/
- Bedrock model customization (SFT, RFT, Distillation): https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html
- Bedrock Model Distillation: https://docs.aws.amazon.com/bedrock/latest/userguide/model-distillation.html
- Distillation "500% faster, 75% less expensive": https://aws.amazon.com/bedrock/model-distillation/
- Provisioned Throughput: https://docs.aws.amazon.com/bedrock/latest/userguide/prov-throughput.html
- Bedrock pricing model & service tiers: https://docs.aws.amazon.com/prescriptive-guidance/latest/gen-ai-inference-architecture-and-best-practices-on-aws/amazon-bedrock.html
- Model evaluation (LLM-as-a-judge): https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation-judge.html
- Model evaluation (human workers): https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation-human.html
- Bedrock Managed vs Customer-managed Knowledge Bases: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-build-managed.html
- Vector store options for Knowledge Bases: https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-an-aws-vector-database-for-rag-use-cases/vector-db-options.html
- Bedrock Guardrails: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html
- Guardrails CloudWatch metrics: https://docs.aws.amazon.com/bedrock/latest/userguide/monitoring-guardrails-cw-metrics.html
- Bedrock runtime CloudWatch metrics: https://docs.aws.amazon.com/bedrock/latest/userguide/monitoring-runtime-metrics.html
- Bedrock Agents (Agents Classic, không nhận khách mới): https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html
- What is Amazon Bedrock AgentCore: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html
- Inference parameters (Temperature, Top K, Top P): https://docs.aws.amazon.com/bedrock/latest/userguide/inference-parameters.html
