# AIF-C01 — Responsible AI, Security, Compliance & Governance

> Bao trùm **Domain 4 (Guidelines for Responsible AI, 14%)** và **Domain 5 (Security, Compliance, and Governance, 14%)**, tổng cộng 28% điểm. Câu hỏi thường cho tình huống rủi ro rồi hỏi **nguyên tắc / dịch vụ / biện pháp** phù hợp.
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời. Mốc kiến thức: 10/2026.

---

## 1. TL;DR

- **Responsible AI** = AI minh bạch, đáng tin, giảm rủi ro **suốt vòng đời** (design → develop → deploy → monitor → evaluate). **Security** = giữ **CIA** (Confidentiality, Integrity, Availability). **Governance** = chính sách và giám sát để AI tuân thủ luật. **Compliance** = tuân thủ quy định, nhất là y tế, tài chính, pháp lý.
- **8 dimension của Responsible AI (AWS):** Fairness, Explainability, Privacy & Security, Safety, Controllability, Veracity & Robustness, Governance, Transparency.
- Công cụ AWS: **Bedrock Guardrails** (lọc nội dung, PII, topic), **Bedrock Model Evaluation**, **SageMaker Clarify** (bias, explainability, FM eval), **Data Wrangler** (cân bằng data), **Model Monitor**, **A2I** (human review), **Model Cards / Model Dashboard / Role Manager**, **AWS AI Service Cards**.
- **Interpretability vs performance:** model dễ hiểu (linear regression, decision tree) thường kém hơn; model mạnh (neural network) là "black box", nên phải dùng **explainability** (vd: Partial Dependence Plots).
- **Thách thức GenAI:** toxicity, hallucination, plagiarism, nondeterminism, rủi ro pháp lý. **Prompt misuse:** poisoning, hijacking/prompt injection, exposure, prompt leaking, jailbreaking.
- **Governance & compliance trên AWS:** Config, Inspector, Artifact, CloudTrail, Trusted Advisor (⚠️ Audit Manager không nhận khách mới từ 30/04/2026 → dùng **Config Conformance Packs**). **Shared Responsibility:** với Bedrock, khách hàng lo data, access control, guardrails, mã hóa data ứng dụng.
- **Generative AI Security Scoping Matrix:** 5 scope từ **Consumer app** (sở hữu ít) → **Self-trained model** (sở hữu và trách nhiệm nhiều nhất).
- **MLOps** = DevOps cho ML: version control (data, code, model), automation, CI/CD, continuous retraining & monitoring.

---

## 2. Core Concepts

### Responsible AI, Security, Governance, Compliance
- **Là gì:** Responsible AI giảm rủi ro và kết quả tiêu cực; Security bảo vệ CIA của data, tài sản thông tin, hạ tầng; Governance tạo giá trị và quản lý rủi ro bằng chính sách và giám sát; Compliance đảm bảo tuân thủ quy định.
- **Dùng khi nào:** đặc biệt quan trọng với lĩnh vực nhạy cảm như y tế, tài chính, pháp lý.
- **Vì sao quan trọng:** governance giúp **tăng niềm tin** và bảo vệ khỏi rủi ro pháp lý và uy tín.

### Core dimensions of Responsible AI
- **Là gì:** **Fairness** (cân nhắc tác động lên các nhóm, chống phân biệt đối xử), **Explainability** (hiểu và đánh giá output), **Privacy & Security** (cá nhân kiểm soát việc data của mình được dùng), **Transparency** (cho stakeholder đủ thông tin để quyết định có dùng AI hay không).
- **Dùng khi nào:** **Veracity & Robustness** (output đúng kể cả khi input bất ngờ hoặc độc hại), **Governance** (định nghĩa và thực thi thực hành responsible AI), **Safety** (ngăn output có hại và lạm dụng), **Controllability** (có cơ chế giám sát và điều hướng AI theo giá trị con người).
- **Vì sao quan trọng:** đề mô tả một vấn đề rồi hỏi nó thuộc dimension nào (xem bảng mục 4).

### Responsible AI – AWS Services
- **Là gì:** **Bedrock** (model evaluation tự động hoặc bằng người; **Guardrails**: lọc nội dung độc hại, chặn topic, ẩn PII), **SageMaker Clarify** (đánh giá FM về accuracy, robustness, toxicity; phát hiện bias, vd data lệch về người trung niên).
- **Dùng khi nào:** **Data Wrangler** sửa bias bằng cách cân bằng dataset (augment data cho nhóm thiếu đại diện); **Model Monitor** phân tích chất lượng trong production; **A2I** cho người duyệt dự đoán.
- **Vì sao quan trọng:** governance: **SageMaker Role Manager, Model Cards, Model Dashboard**.

### AWS AI Service Cards & Model Cards
- **Là gì:** **AI Service Cards** = tài liệu responsible AI do **AWS công bố** cho dịch vụ của AWS: use case dự kiến, giới hạn, lựa chọn thiết kế responsible AI, best practice deploy và tối ưu.
- **Dùng khi nào:** **Model Cards** = định dạng chuẩn để **bạn** ghi lại thông tin model: dataset, nguồn, license, bias đã biết, intended use, risk rating, chi tiết training, metric; với GenAI có thể kèm source citation và nguồn gốc data.
- **Vì sao quan trọng:** **SageMaker Model Cards** lưu tập trung, hỗ trợ **audit**. Exam guide Task 4.2 nêu tên Model Cards là công cụ cho transparency.

### Interpretability vs Explainability
- **Là gì:** **Interpretability** = mức con người hiểu được **nguyên nhân** của một quyết định ("why and how"), cần nhìn được vào bên trong model; **Explainability** = giải thích hành vi model qua input/output mà **không cần hiểu chính xác** cơ chế bên trong.
- **Dùng khi nào:** trade-off: **transparency/interpretability cao thì performance thường thấp** (linear/logistic regression, decision tree, Naive Bayes) ↔ performance cao nhưng khó hiểu (SVM, ensemble, **neural network**).
- **Vì sao quan trọng:** khi bắt buộc phải giải thích được (vd: quyết định tín dụng) thì có thể chọn model đơn giản hơn; nhiều trường hợp chỉ cần explainability là đủ.

### Decision Trees & Partial Dependence Plots (PDP)
- **Là gì:** **Decision tree** = supervised (classification + regression), chia data thành nhánh theo luật đơn giản ("thu nhập > 50K?"); dễ hiểu, trực quan, nhưng dễ **overfit** nếu quá nhiều nhánh.
- **Dùng khi nào:** **PDP** cho thấy **một feature** ảnh hưởng thế nào lên kết quả dự đoán khi giữ các feature khác cố định.
- **Vì sao quan trọng:** PDP đặc biệt hữu ích với model **black box** (neural network), giúp tăng interpretability và explainability.

### Human-Centered Design (HCD) cho Explainable AI
- **Là gì:** thiết kế AI ưu tiên nhu cầu con người: **amplified decision-making** (giảm sai sót khi áp lực cao; rõ ràng, đơn giản, dễ dùng; reflexivity và accountability), **unbiased decision-making** (quy trình không bias, huấn luyện người ra quyết định nhận diện bias).
- **Dùng khi nào:** **human and AI learning**: cognitive apprenticeship (AI học từ chuyên gia), personalization, user-centered design (dễ tiếp cận cho nhiều người dùng).
- **Vì sao quan trọng:** exam guide Task 4.2 nêu HCD, gồm cơ chế **user feedback** và **minh bạch quyết định của AI**.

### GenAI Capabilities & Challenges
- **Là gì:** năng lực: adaptability, responsiveness, simplicity, creativity, data efficiency, personalization, scalability.
- **Dùng khi nào:** thách thức: regulatory violations, social risks, data security & privacy, **toxicity**, **hallucinations**, interpretability, **nondeterminism**, **plagiarism & cheating**.
- **Vì sao quan trọng:** `[+]` **rủi ro pháp lý** theo exam guide: vi phạm sở hữu trí tuệ (**IP infringement**), output thiên lệch, mất niềm tin khách hàng, rủi ro cho end user, hallucination.

### Toxicity, Hallucinations, Plagiarism
- **Là gì:** **Toxicity** = nội dung xúc phạm, gây khó chịu (ranh giới giữa kiểm duyệt và censorship khó xác định); giảm bằng **curate training data** và **guardrail models**. **Hallucination** = khẳng định nghe đúng nhưng sai, do lấy mẫu từ kế tiếp theo xác suất.
- **Dùng khi nào:** giảm hallucination: giáo dục user kiểm tra lại, xác minh bằng nguồn độc lập, **đánh dấu nội dung chưa xác minh**. `[+]` Kỹ thuật: **RAG grounding**, Guardrails **contextual grounding check** / **Automated Reasoning check**, output validation, confidence scoring.
- **Vì sao quan trọng:** **Plagiarism & cheating** (bài luận, hồ sơ xin việc): khó truy nguồn output của LLM, nên đã có công nghệ phát hiện nội dung do AI tạo.

### Prompt Misuses
- **Là gì:** **Poisoning** = cài data độc hoặc thiên lệch vào **training data**; **Hijacking / Prompt injection** = nhúng chỉ thị vào prompt để bẻ hành vi model (tin giả, code độc).
- **Dùng khi nào:** **Exposure** = model làm lộ data nhạy cảm đã thấy khi train hoặc inference (vd: lộ lịch sử mua của "John Smith"); **Prompt leaking** = lộ prompt/input trước đó hoặc cách model hoạt động.
- **Vì sao quan trọng:** **Jailbreaking** = vượt qua ràng buộc đạo đức và an toàn của model (vd: many-shot jailbreaking). Phòng chống: Guardrails (prompt attack filter), lọc, sanitize, validate input.

### Regulated Workloads & AI Compliance Challenges
- **Là gì:** ngành cần compliance thêm: tài chính, y tế, hàng không vũ trụ; vd báo cáo định kỳ cho cơ quan liên bang, kết quả bị quản lý (hồ sơ vay, tín dụng). Cần audit, lưu trữ, yêu cầu bảo mật đặc biệt thì đó là **regulated workload**.
- **Dùng khi nào:** thách thức: **complexity & opacity** (khó audit), **dynamism** (AI thay đổi theo thời gian), **emergent capabilities**, rủi ro riêng (**algorithmic bias**, **human bias**, vi phạm privacy, misinformation), **algorithm accountability**.
- **Vì sao quan trọng:** luật: **EU AI Act**, và nhiều quy định ở cấp bang/thành phố ở Mỹ; thúc đẩy fairness, non-discrimination, quyền con người.

### AWS Compliance
- **Là gì:** AWS hỗ trợ nhiều chuẩn bảo mật và chứng nhận compliance (slide nêu "140+" ❓ cần kiểm chứng con số): **NIST**, **ENISA**, **ISO**, **SOC**, **HIPAA**, **GDPR**, **PCI DSS**.
- **Dùng khi nào:** tải báo cáo compliance của AWS qua **AWS Artifact**.
- **Vì sao quan trọng:** AWS đạt chứng nhận cho **hạ tầng**; khách hàng vẫn chịu trách nhiệm cho workload của mình (shared responsibility).

### AWS Tools for Governance
- **Là gì:** **AWS Config** (ghi và đánh giá cấu hình tài nguyên theo rule); **Amazon Inspector** (quét lỗ hổng tự động); **AWS Artifact** (tải báo cáo compliance và thỏa thuận của AWS); **AWS CloudTrail** (log mọi API call để audit).
- **Dùng khi nào:** **AWS Trusted Advisor** (kiểm tra best practice: chi phí, bảo mật, hiệu năng, độ chịu lỗi, giới hạn dịch vụ); ⚠️ Lỗi thời: **AWS Audit Manager** (thu thập bằng chứng audit) không nhận khách mới từ 30/04/2026 → nay dùng **AWS Config Conformance Packs**.
- **Vì sao quan trọng:** exam guide Task 5.2 nêu đích danh Config, Inspector, Artifact, CloudTrail, Trusted Advisor.

### Governance Framework & Strategies
- **Là gì:** framework mẫu: lập **AI Governance Board** (legal, compliance, data privacy, SME), định nghĩa vai trò và trách nhiệm, ban hành policy cho cả vòng đời AI.
- **Dùng khi nào:** chiến lược: **Policies** (data, training, validate output, safety, human oversight, IP, bias, privacy), **Review cadence** (tháng/quý/năm; review kỹ thuật + phi kỹ thuật), **Transparency standards** (công bố thông tin model, giới hạn, kênh feedback), **Team training**.
- **Vì sao quan trọng:** exam guide Task 5.2 hỏi "processes to follow governance protocols".

### Data Governance & Data Management
- **Là gì:** data governance: responsible AI framework; **governance council**, vai trò **data steward / data owner / data custodian**; chia sẻ data an toàn (data sharing agreement, data virtualization/federation).
- **Dùng khi nào:** khái niệm quản lý data: **lifecycle** (thu thập → xử lý → lưu → dùng → lưu trữ dài hạn), **logging**, **residency** (data được xử lý và lưu ở đâu), **monitoring** (chất lượng, bất thường, drift), **analysis**, **retention**.
- **Vì sao quan trọng:** **Data lineage** = **source citation** (nguồn, license, điều khoản) + **documenting data origins** (thu thập, làm sạch, biến đổi) + **cataloging**; phục vụ transparency, traceability, accountability.

### Security & Privacy for AI Systems
- **Là gì:** **Threat detection** (nội dung giả, data bị thao túng, tấn công tự động; dùng AI để phát hiện), **Vulnerability management** (pentest, code review, patch), **Infrastructure protection** (access control, phân đoạn mạng, mã hóa, chịu lỗi).
- **Dùng khi nào:** **Prompt injection** → guardrails: lọc, sanitize, validate prompt; **Data encryption** at rest và in transit, bảo vệ khóa mã hóa.
- **Vì sao quan trọng:** `[+]` dịch vụ AWS theo exam guide Task 5.1: **IAM** (roles, policies), **encryption (KMS)**, **Amazon Macie** (tìm PII/data nhạy cảm trong **S3**), **AWS PrivateLink** (kết nối private tới Bedrock, không qua Internet), **AgentCore Identity**, **Policy in AgentCore**, Bedrock Guardrails.

### `[+]` Bảo mật cho AI agents: AgentCore Identity & Policy in AgentCore
- **Là gì:** **AgentCore Identity** quản lý danh tính, quyền truy cập và xác thực cho agent, tương thích IdP sẵn có; **Policy in AgentCore** dùng **Cedar policy** để kiểm soát **mọi lần agent gọi tool** qua AgentCore Gateway (mặc định từ chối).
- **Dùng khi nào:** vd chỉ cho phép agent hoàn tiền khi user cụ thể yêu cầu và số tiền dưới một ngưỡng.
- **Vì sao quan trọng:** agent **hành động** thật trên hệ thống, nên cần kiểm soát xác định (deterministic), không chỉ dựa vào prompt.

### Monitoring AI Systems
- **Là gì:** **Performance metrics**: Accuracy = số dự đoán **đúng** / tổng; Precision = TP / (TP + FP); Recall = TP / (TP + FN); F1 = **trung bình điều hòa** của precision và recall; Latency.
- **Dùng khi nào:** **Infrastructure monitoring**: CPU/GPU, network, storage, system logs (bắt nghẽn và lỗi).
- **Vì sao quan trọng:** giám sát thêm **bias & fairness, compliance, responsible AI**, không chỉ độ chính xác.

### AWS Shared Responsibility Model (cho AI)
- **Là gì:** **AWS** = security **OF** the cloud: hạ tầng (phần cứng, phần mềm, cơ sở vật chất, mạng) cho managed services như Bedrock, SageMaker, S3.
- **Dùng khi nào:** **Khách hàng** = security **IN** the cloud: với Bedrock là data management, access control, cấu hình guardrails, mã hóa data ứng dụng.
- **Vì sao quan trọng:** **Shared controls**: Patch management, Configuration management, Awareness & training (cả hai bên cùng làm, mỗi bên ở tầng của mình).

### Secure Data Engineering – Best Practices
- **Là gì:** **Data quality**: Completeness, Accuracy, Timeliness, Consistency; data profiling, monitoring, lineage. **Privacy-enhancing technologies**: data masking, obfuscation, encryption, tokenization.
- **Dùng khi nào:** **Data access control**: RBAC, fine-grained permissions, SSO, MFA, log mọi truy cập, review quyền theo least privilege. **Data integrity**: backup và recovery, lineage, audit trail.
- **Vì sao quan trọng:** `[+]` đặc điểm dataset tốt theo exam guide: **inclusive, diverse, curated sources, balanced**; kiểm tra bằng **label quality analysis, human audits, subgroup analysis**.

### Generative AI Security Scoping Matrix
- **Là gì:** framework phân loại app GenAI theo **mức sở hữu**: **Scope 1** Consumer app (dịch vụ GenAI public) → **Scope 2** Enterprise app (SaaS có tính năng GenAI, vd Amazon Q Developer) → **Scope 3** Pre-trained models (build app trên model có sẵn, vd Bedrock base model) → **Scope 4** Fine-tuned models (Bedrock custom model, JumpStart) → **Scope 5** Self-trained models (train từ đầu, vd SageMaker AI).
- **Dùng khi nào:** Scope 1–2: tư duy **người mua** (quản trị data, đọc kỹ thỏa thuận); Scope 3–5: tư duy **người xây** (threat modeling, kiểm soát nhiều hơn nên trách nhiệm cũng nhiều hơn).
- **Vì sao quan trọng:** các mặt cần xét: Governance & Compliance, Legal & Privacy, Risk Management, Controls, Resilience.

### MLOps
- **Là gì:** đảm bảo model không chỉ được phát triển mà còn được **deploy, monitor, retrain có hệ thống và lặp lại được**; là phần mở rộng của DevOps.
- **Dùng khi nào:** nguyên tắc: **version control** (data, code, model, rollback được), **automation** (mọi giai đoạn), **CI** (test model liên tục), **CD**, **continuous retraining**, **continuous monitoring**.
- **Vì sao quan trọng:** ví dụ pipeline: Data pipeline → Build & test pipeline → Deployment pipeline → Monitoring pipeline, với Data repo, Code repo, Model repo. Trên AWS: SageMaker Pipelines + Model Registry + Model Monitor.

### `[+]` Responsible model selection
- **Là gì:** khi chọn model, cân nhắc thêm **environmental considerations / sustainability** (năng lượng, phần cứng hiệu quả).
- **Dùng khi nào:** chọn model **nhỏ hơn đủ dùng**, dùng phần cứng tiết kiệm năng lượng (Trainium/Inferentia), tái sử dụng FM thay vì train lại từ đầu.
- **Vì sao quan trọng:** exam guide Task 4.1 nêu "environmental considerations, sustainability".

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Responsible AI | AI minh bạch, đáng tin, giảm rủi ro suốt vòng đời | "trustworthy", "mitigate risk" |
| Fairness | Không phân biệt đối xử, cân nhắc tác động lên các nhóm | "discriminates against a group" |
| Explainability | Hiểu và giải thích output của model | "explain why", "justify decision" |
| Privacy & Security | Cá nhân kiểm soát data, bảo vệ data và model | "consent", "personal data protection" |
| Transparency | Cho stakeholder biết AI hoạt động thế nào, giới hạn ra sao | "disclose AI use", "publish model info" |
| Veracity & Robustness | Output đúng kể cả với input bất ngờ/đối kháng | "reliable under unexpected inputs" |
| Safety | Ngăn output có hại và lạm dụng | "harmful output", "misuse" |
| Controllability | Giám sát và điều hướng AI theo giá trị con người | "steer model behavior", "human oversight" |
| Governance | Chính sách, vai trò, giám sát cho AI | "policies", "oversight board" |
| AWS AI Service Cards | Tài liệu responsible AI do AWS công bố cho dịch vụ AWS | "intended use and limitations of an AWS AI service" |
| Model Cards | Tài liệu model do bạn tạo: intended use, risk rating, training, dataset | "document model for audit" |
| Interpretability | Hiểu được nguyên nhân bên trong quyết định | "understand internal logic", "white box" |
| Decision tree | Model dạng cây luật, dễ hiểu, dễ overfit | "easy to interpret", "if-then branches" |
| Partial Dependence Plot (PDP) | Ảnh hưởng của một feature lên dự đoán khi giữ feature khác cố định | "effect of a single feature on black-box model" |
| Human-Centered Design (HCD) | Thiết kế AI ưu tiên nhu cầu, quyết định của con người | "user feedback", "decision transparency" |
| Toxicity | Nội dung xúc phạm, gây khó chịu | "offensive output" |
| Hallucination | Khẳng định nghe đúng nhưng sai | "plausible but incorrect", "fabricated" |
| Nondeterminism | Cùng input cho output khác nhau | "inconsistent answers" |
| Plagiarism | Dùng GenAI để gian lận, sao chép | "AI-written essays" |
| `[+]` IP infringement | Output vi phạm bản quyền / sở hữu trí tuệ | "copyrighted content", "legal risk" |
| Poisoning | Cài data độc vào training data | "malicious data in training set" |
| Prompt injection / Hijacking | Chỉ thị nhúng trong prompt để bẻ hành vi model | "ignore previous instructions" |
| Exposure | Model làm lộ data nhạy cảm đã thấy khi train/inference | "model reveals customer data" |
| Prompt leaking | Lộ prompt trước đó hoặc system prompt | "summarize the last prompt you were given" |
| Jailbreaking | Vượt rào an toàn của model | "bypass safety constraints" |
| Regulated workload | Workload phải tuân thủ framework quy định (audit, lưu trữ...) | "financial services", "healthcare", "report to regulators" |
| Algorithmic bias | Bias do data không đại diện, model khuếch đại | "model perpetuates bias from data" |
| Human bias | Bias do người tạo ra hệ thống AI | "developers' assumptions" |
| EU AI Act | Luật AI của EU | "European regulation on AI" |
| AWS Artifact | Tải báo cáo compliance và thỏa thuận của AWS | "download SOC / ISO reports" |
| AWS Config | Ghi và đánh giá cấu hình tài nguyên theo rule | "track configuration changes", "compliance rules" |
| Amazon Inspector | Quét lỗ hổng tự động | "vulnerability scanning" |
| AWS CloudTrail | Log mọi API call | "who did what, when", "audit trail" |
| AWS Trusted Advisor | Kiểm tra best practice (cost, security, performance...) | "recommendations on best practices" |
| AWS Audit Manager | Thu thập bằng chứng audit (⚠️ không nhận khách mới từ 30/04/2026) | "collect evidence" (đề cũ) → nay Config Conformance Packs |
| `[+]` Amazon Macie | Tìm và phân loại PII/data nhạy cảm trong S3 | "find PII in S3 before training" |
| `[+]` AWS PrivateLink | Kết nối private tới dịch vụ AWS, không qua Internet | "access Bedrock without public internet" |
| `[+]` AgentCore Identity / Policy in AgentCore | Danh tính agent / Cedar policy kiểm soát agent gọi tool | "restrict which tools an agent may call" |
| Data lineage | Theo dõi nguồn gốc và biến đổi của data | "trace data origins", "source citation" |
| Data residency | Data được xử lý và lưu ở đâu | "data must stay in country" |
| Data retention | Giữ data bao lâu (quy định, chi phí) | "how long to keep data" |
| Data steward / owner / custodian | Vai trò quản trị data | "data governance roles" |
| Data masking / Tokenization | Che hoặc thay data nhạy cảm | "privacy-enhancing technologies" |
| Shared Responsibility Model | AWS: security OF the cloud; khách hàng: IN the cloud | "who is responsible for..." |
| Shared controls | Patch, configuration management, training: cả hai bên | "both AWS and customer" |
| GenAI Security Scoping Matrix | 5 scope theo mức sở hữu model/app | "classify GenAI use case security scope" |
| MLOps | DevOps cho ML: version, automate, CI/CD, retrain, monitor | "operationalize ML", "repeatable" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Governance | Compliance | Governance: **tự đặt** chính sách và giám sát nội bộ; Compliance: **tuân thủ** luật và quy định bên ngoài | "Internal policy board" → governance; "HIPAA / GDPR" → compliance |
| Fairness | Bias (thống kê) | Fairness: đối xử công bằng giữa các nhóm người; bias thống kê: sai số do model quá đơn giản (underfitting) | Đọc ngữ cảnh: nhóm người hay độ chính xác |
| Transparency | Explainability | Transparency: **công khai** thông tin về hệ thống (data, giới hạn, việc dùng AI); Explainability: **giải thích** một output cụ thể | "Disclose that AI is used" → transparency |
| Interpretability | Explainability | Interpretability: hiểu **cơ chế bên trong**; Explainability: giải thích qua **input/output** | Decision tree = interpretable; neural network + PDP = explainable |
| Veracity & Robustness | Safety | Veracity: output **đúng**, chịu được input lạ; Safety: output **không gây hại** | "Correct under adversarial input" → robustness |
| Controllability | Governance | Controllability: cơ chế **kỹ thuật** để điều hướng model; Governance: **chính sách, tổ chức** | "Steer the model" → controllability |
| AI Service Cards | Model Cards | Service Cards: AWS viết cho **dịch vụ AWS**; Model Cards: **bạn** viết cho **model của bạn** | "Our model's intended use" → SageMaker Model Cards |
| Poisoning | Prompt injection | Poisoning tác động **training data** (trước); Prompt injection tác động **prompt lúc chạy** | Training data → poisoning |
| Exposure | Prompt leaking | Exposure: lộ **data nhạy cảm** (của khách hàng, từ training); Prompt leaking: lộ **prompt / system prompt** | "Reveal last prompt" → prompt leaking |
| Hijacking | Jailbreaking | Hijacking: bẻ **mục tiêu** sang việc khác; Jailbreaking: phá **rào an toàn** để lấy nội dung cấm | "Pretend you have no restrictions" → jailbreak |
| Hallucination | Toxicity | Hallucination: **sai sự thật**; Toxicity: **xúc phạm** | "Invented citation" → hallucination |
| Algorithmic bias | Human bias | Algorithmic: từ data / model; Human: từ người thiết kế hệ thống | |
| AWS Config | AWS CloudTrail | Config: **trạng thái cấu hình** tài nguyên và tuân thủ rule; CloudTrail: **ai gọi API gì, khi nào** | "Who deleted the bucket?" → CloudTrail |
| AWS Artifact | AWS Audit Manager | Artifact: tải **báo cáo compliance của AWS**; Audit Manager: thu thập **bằng chứng về workload của bạn** (⚠️ không nhận khách mới, thay bằng Config Conformance Packs) | "Download AWS SOC 2 report" → Artifact |
| Amazon Inspector | Trusted Advisor | Inspector: quét **lỗ hổng** phần mềm; Trusted Advisor: gợi ý **best practice** (cost, security, limits...) | "CVE / vulnerability" → Inspector |
| `[+]` Amazon Macie | Bedrock Guardrails (PII filter) | Macie: tìm PII trong **data lưu ở S3**; Guardrails: chặn hoặc ẩn PII trong **prompt / response** | "Scan training data in S3 for PII" → Macie |
| `[+]` Comprehend PII | Macie | Comprehend: phát hiện PII trong **text gửi vào API**; Macie: quét **bucket S3** | |
| Security OF the cloud | Security IN the cloud | OF: AWS (hạ tầng, nơi host model); IN: khách hàng (data, IAM, guardrails, mã hóa data ứng dụng) | Cấu hình guardrail → khách hàng |
| Scope 3 (Pre-trained) | Scope 4 (Fine-tuned) | Scope 3: dùng model base qua API; Scope 4: fine-tune model trên data của bạn | Bedrock base → 3; Bedrock custom → 4 |
| Scope 1 (Consumer) | Scope 2 (Enterprise) | Scope 1: app GenAI public cho người tiêu dùng; Scope 2: SaaS doanh nghiệp có tính năng GenAI | Amazon Q Developer → Scope 2 |
| Data residency | Data retention | Residency: **ở đâu**; Retention: **bao lâu** | |
| Data lineage | Data cataloging | Lineage: **nguồn gốc và biến đổi**; Cataloging: **tổ chức, mô tả** dataset | Cả hai phục vụ traceability |
| Accuracy | Precision | Accuracy: đúng / tổng; Precision: TP / (TP + FP) | Slide ghi accuracy = "ratio of positive predictions": cách diễn đạt **không chính xác** |
| MLOps | DevOps | MLOps thêm version **data + model**, **continuous retraining & monitoring** | "Retrain automatically when drift" → MLOps |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "model discriminates against a demographic group" | Fairness → Clarify (detect bias), cân bằng data | |
| "explain why the model made a decision" | Explainability → Clarify, PDP | |
| "regulators require fully interpretable model" | Model đơn giản (decision tree, linear/logistic regression) | Đánh đổi performance |
| "inform users they are interacting with AI" | Transparency | |
| "model must work correctly with unexpected inputs" | Veracity & Robustness | |
| "block harmful topics, redact PII in prompts/responses" | Bedrock Guardrails | |
| "find PII in S3 training data" | `[+]` Amazon Macie | |
| "access Bedrock privately without internet" | `[+]` AWS PrivateLink | |
| "restrict which tools an AI agent can call" | `[+]` Policy in AgentCore (Cedar) | |
| "intended use and limitations of an AWS AI service" | AWS AI Service Cards | |
| "document our model's intended use, risk rating" | SageMaker Model Cards | |
| "human review of low-confidence predictions" | Amazon A2I | |
| "monitor model quality / bias in production" | SageMaker Model Monitor | |
| "reduce hallucinations" | `[+]` RAG grounding, Guardrails contextual grounding / Automated Reasoning, human verification | |
| "attacker inserted biased data into training set" | Poisoning | |
| "user prompt: ignore instructions and..." | Prompt injection / hijacking | |
| "model reveals other customers' personal data" | Exposure | |
| "user asks model to reveal its system prompt" | Prompt leaking | |
| "bypass model's safety rules" | Jailbreaking | |
| "download AWS ISO / SOC / PCI reports" | AWS Artifact | |
| "record API calls for audit" | AWS CloudTrail | |
| "check resource configurations against compliance rules" | AWS Config | |
| "automated vulnerability scanning" | Amazon Inspector | |
| "best-practice recommendations (cost, security, limits)" | AWS Trusted Advisor | |
| "continuously collect evidence for audits" | ⚠️ Audit Manager (đề cũ) → AWS Config Conformance Packs | |
| "trace where training data came from" | Data lineage / source citation | |
| "data must be stored and processed in-country" | Data residency | |
| "classify GenAI use case by ownership level" | Generative AI Security Scoping Matrix | |
| "version data, code, models; automate retraining" | MLOps (SageMaker Pipelines, Model Registry) | |
| "choose model with lower environmental impact" | `[+]` Model nhỏ hơn, Trainium/Inferentia, tái sử dụng FM | |

---

## 5. Câu hỏi tình huống

**Câu 1.** Model duyệt hồ sơ vay từ chối tỷ lệ cao bất thường với người trên 60 tuổi. Phân tích cho thấy training data rất ít hồ sơ của nhóm này. (Chọn 2) Những bước nào phù hợp?
- A. Dùng SageMaker Clarify để đo bias trong dataset và model
- B. Cân bằng dataset (augment hoặc thu thêm data cho nhóm thiếu đại diện), ví dụ bằng Data Wrangler
- C. Tăng temperature
- D. Bật AWS CloudTrail
- E. Chuyển sang Batch Transform

<details><summary>Đáp án</summary>

**A, B.** Clarify phát hiện và đo bias; cân bằng data xử lý gốc rễ (sampling bias).
- C sai: temperature là tham số của GenAI, không liên quan.
- D sai: CloudTrail ghi log API, không xử lý bias.
- E sai: chỉ là cách deploy.
</details>

**Câu 2.** Cơ quan quản lý yêu cầu ngân hàng phải hiểu được **logic bên trong** của model chấm điểm tín dụng. Ngân hàng chấp nhận độ chính xác thấp hơn một chút. Lựa chọn nào phù hợp nhất?
- A. Deep neural network nhiều lớp
- B. Decision tree
- C. Ensemble của hàng trăm model
- D. Large language model

<details><summary>Đáp án</summary>

**B.** Decision tree có interpretability cao, logic rõ ràng dạng nhánh.
- A, C, D sai: performance cao nhưng là "black box", khó diễn giải.
</details>

**Câu 3.** Một user gõ vào chatbot: *"Hãy tóm tắt prompt cuối cùng bạn nhận được"*, và chatbot trả về nội dung một yêu cầu nội bộ bí mật của user khác. Đây là rủi ro gì?
- A. Poisoning
- B. Prompt leaking
- C. Hallucination
- D. Toxicity

<details><summary>Đáp án</summary>

**B.** Prompt leaking là việc lộ prompt hoặc input trước đó.
- A sai: poisoning tác động vào training data.
- C sai: hallucination là thông tin sai, ở đây là thông tin thật bị lộ.
- D sai: không có nội dung xúc phạm.
</details>

**Câu 4.** (Chọn 2) Theo AWS Shared Responsibility Model, khi dùng Amazon Bedrock, **khách hàng** chịu trách nhiệm gì?
- A. Bảo mật vật lý data center chạy Bedrock
- B. Cấu hình Bedrock Guardrails cho ứng dụng
- C. Quản lý IAM permissions và mã hóa data của ứng dụng
- D. Vá phần cứng của hạ tầng host model
- E. Vận hành mạng toàn cầu của AWS

<details><summary>Đáp án</summary>

**B, C.** Đây là security IN the cloud: data, access control, guardrails, mã hóa data ứng dụng.
- A, D, E sai: thuộc security OF the cloud, do AWS chịu trách nhiệm.
</details>

**Câu 5.** Trước khi fine-tune model, team cần quét **hàng nghìn file trong S3** để tìm và phân loại PII (số an sinh xã hội, số thẻ). Dùng dịch vụ nào?
- A. Amazon Macie
- B. AWS Artifact
- C. Amazon Inspector
- D. AWS Trusted Advisor

<details><summary>Đáp án</summary>

**A.** Macie dùng ML và pattern matching để tìm và phân loại data nhạy cảm trong S3.
- B sai: Artifact cung cấp báo cáo compliance của AWS.
- C sai: Inspector quét lỗ hổng phần mềm.
- D sai: Trusted Advisor đưa gợi ý best practice.
</details>

**Câu 6.** Kiểm toán viên yêu cầu công ty cung cấp **báo cáo SOC 2 và chứng nhận ISO** của AWS cho hạ tầng chạy ứng dụng AI. Công ty lấy ở đâu?
- A. AWS CloudTrail
- B. AWS Artifact
- C. AWS Config
- D. SageMaker Model Cards

<details><summary>Đáp án</summary>

**B.** Artifact là cổng tải báo cáo compliance và thỏa thuận của AWS.
- A sai: CloudTrail ghi log API.
- C sai: Config theo dõi cấu hình tài nguyên.
- D sai: Model Cards là tài liệu về model của bạn.
</details>

**Câu 7.** Theo Generative AI Security Scoping Matrix, một công ty **fine-tune** một FM trên Amazon Bedrock bằng data khách hàng của họ. Ứng dụng này thuộc scope nào?
- A. Scope 1 – Consumer app
- B. Scope 2 – Enterprise app
- C. Scope 4 – Fine-tuned models
- D. Scope 5 – Self-trained models

<details><summary>Đáp án</summary>

**C.** Fine-tune model trên data của mình là Scope 4.
- A, B sai: dùng app có sẵn, không động vào model.
- D sai: train model từ đầu.
</details>

**Câu 8.** (Chọn 2) Một ứng dụng hỏi đáp pháp lý dùng GenAI đôi khi trích dẫn **án lệ không tồn tại**. Biện pháp nào giúp giảm rủi ro?
- A. Dùng RAG để grounding câu trả lời vào cơ sở dữ liệu án lệ thật, kết hợp Guardrails contextual grounding check
- B. Đánh dấu nội dung là "chưa xác minh" và yêu cầu chuyên gia kiểm tra trước khi dùng
- C. Tăng temperature để model sáng tạo hơn
- D. Bật Model Invocation Logging là đủ để ngăn hallucination
- E. Dùng AWS Trusted Advisor

<details><summary>Đáp án</summary>

**A, B.** Grounding bằng RAG + contextual grounding giảm hallucination; human verification và gắn nhãn chưa xác minh giảm tác hại.
- C sai: tăng temperature còn làm tăng nguy cơ bịa thông tin.
- D sai: logging chỉ ghi lại, không ngăn chặn.
- E sai: Trusted Advisor không liên quan đến output của model.
</details>

---

### Nguồn đã đối chiếu
- AIF-C01 exam guide, Domain 4 (Responsible AI): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain4.html
- AIF-C01 exam guide, Domain 5 (Security, Compliance, Governance): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain5.html
- 8 core dimensions of responsible AI (Amazon Nova responsible use): https://docs.aws.amazon.com/nova/latest/userguide/responsible-use.html
- AWS AI Service Cards (ví dụ HealthScribe): https://docs.aws.amazon.com/ai/responsible-ai/aws-healthscribe/overview.html
- Generative AI Security Scoping Matrix: https://docs.aws.amazon.com/whitepapers/latest/navigating-security-landscape-genai/scoping-generative-ai-use-cases.html
- Policy in AgentCore (Cedar): https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/policy-core-concepts.html
- Amazon Bedrock AgentCore (Identity): https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html
- Amazon Macie (sensitive data discovery): https://docs.aws.amazon.com/whitepapers/latest/navigating-gdpr-compliance/discovering-and-protecting-data-at-scale-with-amazon-macie.html
- Bedrock Guardrails: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html
- Bedrock security & shared responsibility: https://docs.aws.amazon.com/bedrock/latest/userguide/security.html
- AWS compliance programs (không ghi tổng số chứng nhận): https://aws.amazon.com/compliance/programs/
- AWS Audit Manager availability change → Config Conformance Packs: https://docs.aws.amazon.com/audit-manager/latest/userguide/audit-manager-availability-change.html
