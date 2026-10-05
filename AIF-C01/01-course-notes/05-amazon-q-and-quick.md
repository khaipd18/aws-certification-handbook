# AIF-C01 — Amazon Q, Amazon Quick & Kiro

> Thuộc **Domain 1, Task 1.3.4** (exam guide 1.1 nêu tên trực tiếp: Amazon Bedrock, Amazon Q, **Amazon Quick**, **Kiro**, SageMaker AI) và Domain 3 (ứng dụng FM).
> Ký hiệu: `[+]` = bổ sung ngoài slide · `❓` = cần kiểm chứng · `⚠️` = lỗi thời. Mốc kiến thức: 10/2026.
> Lưu ý: họ sản phẩm "Q" đổi tên và thay thế liên tục. Học theo **chức năng** ("trợ lý cho nhân viên" hay "trợ lý cho developer"), đừng chỉ học tên.

---

## 1. TL;DR

- **Amazon Quick** = trợ lý **agentic AI cho nhân viên**: trả lời, nghiên cứu, phân tích, hành động trên dữ liệu và ứng dụng của công ty. Đây là **bản kế nhiệm Amazon Q Business**.
- Quick gồm: **Quick Index** (RAG managed), **Quick Research** (báo cáo có trích dẫn), **Quick Flows** (tự động hóa bằng ngôn ngữ tự nhiên), **Quick Automate** (quy trình nhiều bước, có human approval), **Quick Sight** (BI, trước là QuickSight), **Spaces**, **Apps**.
- Quick xây trên Bedrock; **dữ liệu của bạn không dùng để train model**. User chỉ nhận câu trả lời từ tài liệu **họ có quyền truy cập** (document-level ACL, xác thực qua IAM Identity Center).
- **Amazon Q Developer** = trợ lý **cho developer / người vận hành AWS**: hỏi về docs AWS, tài nguyên trong account, chi phí, lỗi; gợi ý lệnh CLI.
- ⚠️ Q Developer **IDE plugins** hết hỗ trợ ngày 30/04/2027, **Q Developer CLI** đã đổi tên thành **Kiro** → code companion trong IDE và CLI nay là **Kiro**.
- Q tích hợp vào nhiều dịch vụ: **Quick Sight** (hỏi dữ liệu bằng ngôn ngữ tự nhiên), **AWS Glue** (sinh và sửa ETL), **chat apps** Slack/Teams (trước là AWS Chatbot), **EC2 instance type finder**.
- **PartyRock** = sân chơi build app GenAI **không cần code, không cần AWS account**, chạy trên Bedrock.

---

## 2. Core Concepts

### Amazon Quick (kế nhiệm Amazon Q Business)
- ⚠️ Lỗi thời: **Amazon Q Business** không còn nhận khách hàng mới → nay dùng **Amazon Quick** (có lộ trình chuyển đổi, giữ được index cũ qua Bring Your Own Index).
- **Là gì:** trợ lý AI fully managed cho nhân viên; kết nối tài liệu, ứng dụng, database của công ty và Internet; trả lời, nghiên cứu, phân tích, **hành động** (vd: tạo Jira issue, gửi email báo cáo hàng tuần).
- **Vì sao quan trọng:** đề cho tình huống "nhân viên muốn hỏi đáp trên tài liệu nội bộ, **không muốn tự build**" thì đáp án là **Amazon Quick** (trước là Q Business), không phải Bedrock.

### Amazon Quick – Capabilities
- **Là gì:** **Quick Index** (knowledge base hợp nhất, RAG managed), **Quick Research** (agent nghiên cứu sâu, kết hợp dữ liệu nội bộ + web, ra báo cáo có trích dẫn), **Quick Sight** (dashboard BI có AI).
- **Dùng khi nào:** **Quick Flows** = tự động hóa đơn giản mô tả bằng lời (vd: gửi dashboard mỗi thứ Hai); **Quick Automate** = quy trình nhiều bước qua nhiều hệ thống, có **human approval**.
- **Vì sao quan trọng:** **Spaces** = không gian chung của team (tài liệu, dashboard, knowledge base, connector). `[+]` **Custom agents** gồm instructions + knowledge sources + tools, có thể gắn với một Space.

### Amazon Quick + IAM Identity Center
- **Là gì:** user xác thực qua **IAM Identity Center**, có thể nối với IdP ngoài (Google, Microsoft Active Directory / Entra ID...).
- **Dùng khi nào:** cần đảm bảo câu trả lời chỉ dựa trên tài liệu **user đó được phép xem** (document-level ACL từ nguồn như SharePoint, Confluence, Google Drive).
- **Vì sao quan trọng:** `[+]` ngoài cách provision qua AWS Console (IAM Identity Center / IAM Federation), cá nhân còn có thể **đăng ký Quick trực tiếp không cần AWS account** (gói Free/Plus).

### Amazon Quick – Integrations
- **Là gì:** **Knowledge bases** (S3, SharePoint, OneDrive, Google Drive, Confluence, Web Crawler) để index vào Quick; **structured data** cho Quick Sight (vd: Redshift, Athena, RDS, Snowflake).
- **Dùng khi nào:** **Actions / Connectors** để agent làm việc trên ứng dụng bên thứ ba (Jira, ServiceNow, Zendesk, Salesforce, Slack, Outlook...); có **100+ connector dựng sẵn**, và tự tạo connector từ **MCP server** hoặc **OpenAPI**.
- **Vì sao quan trọng:** `[+]` **Extensions** cho phép dùng Quick ngay trong Chrome, Slack, Microsoft Teams, Microsoft 365.

### Apps in Amazon Quick
- ⚠️ Lỗi thời: **Amazon Q Apps** (thuộc Q Business) → nay dùng **Apps in Amazon Quick**.
- **Là gì:** build và publish **web app không cần code** bằng cách mô tả bằng ngôn ngữ tự nhiên (project tracker, dashboard khách hàng, form nhập liệu...); quy trình là Describe → Generate → Preview → Iterate → Publish.
- **Dùng khi nào:** app có thể nhúng visual của Quick Sight, dùng Actions, dùng Spaces làm knowledge base, và gọi FM để tóm tắt hoặc phân loại.

### Amazon Q Developer (trong AWS Console)
- **Là gì:** trợ lý trả lời về **docs AWS** và chọn dịch vụ, về **tài nguyên trong account** (vd: "liệt kê Lambda functions của tôi"), gợi ý **lệnh CLI**.
- **Dùng khi nào:** phân tích **chi phí/bill**, giải thích lỗi, troubleshooting, hiểu và quản lý hạ tầng cloud.
- **Vì sao quan trọng:** đề phân biệt "trợ lý cho **developer/vận hành AWS**" (Q Developer) với "trợ lý cho **nhân viên** trên dữ liệu công ty" (Quick).

### Code companion trong IDE: Q Developer → Kiro
- ⚠️ Lỗi thời: **Amazon Q Developer IDE plugins** hết hỗ trợ ngày **30/04/2027**, **Q Developer CLI** đã đổi tên thành **Kiro** → nay dùng **Kiro** (inline suggestions, chat, code generation, agentic coding, MCP).
- **Là gì:** AI code companion (tương tự GitHub Copilot): gợi ý code real-time, sinh code, quét lỗ hổng bảo mật, debug, tối ưu, agent tự implement tính năng, sinh tài liệu, khởi tạo project.
- **Dùng khi nào:** developer muốn tăng tốc viết code trong IDE (VS Code, JetBrains, Visual Studio...) hoặc ở command line.

### Amazon Q trong Quick Sight
- ⚠️ Lỗi thời: "Amazon Q in **QuickSight**" → **QuickSight** nay là **Amazon Quick Sight**, một tính năng của Amazon Quick (API cũ vẫn chạy).
- **Là gì:** hỏi dữ liệu bằng **ngôn ngữ tự nhiên**, tạo **executive summary**, sinh và chỉnh visual cho dashboard.
- **Dùng khi nào:** business user không biết SQL nhưng muốn hỏi "doanh số quý trước theo vùng" và nhận được biểu đồ.

### Amazon Q tích hợp trong các dịch vụ AWS khác
- **EC2:** ⚠️ "Amazon Q for EC2" → nay là **EC2 instance type finder** trong console (nút "Get advice"), vẫn có thể hỏi Amazon Q; dùng để chọn instance type cho **workload mới**.
- **Chat apps:** ⚠️ **AWS Chatbot** → đổi tên thành **Amazon Q Developer in chat applications** (từ 19/02/2025); dùng trong Slack/Teams để nhận cảnh báo (alarm, security finding, billing), troubleshoot, hỏi về tài nguyên và chi phí.
- **AWS Glue:** **Amazon Q data integration in AWS Glue** trả lời câu hỏi về Glue, sinh script ETL (PySpark) từ ngôn ngữ tự nhiên, giải thích lỗi job và hướng dẫn sửa từng bước.

### PartyRock
- **Là gì:** **playground build app GenAI** chạy trên Amazon Bedrock, không cần code, **không cần AWS account**, có quota miễn phí hằng ngày.
- **Dùng khi nào:** học và thử prompt engineering, thử nhiều FM, chia sẻ app demo nhanh.
- **Vì sao quan trọng:** đề hỏi "cách **nhanh nhất, không cần AWS account** để thử build app GenAI" thì đáp án là **PartyRock**.

### `[+]` Chọn giữa Quick, Q Developer, Bedrock, Kiro
- **Là gì:** Quick = **SaaS dùng ngay** cho nhân viên; Q Developer = trợ lý cho AWS console/vận hành; Kiro = trợ lý viết code; Bedrock = **tự build** app GenAI riêng.
- **Dùng khi nào:** "ít công sức nhất, có sẵn connector, cho nhân viên" → Quick; "nhúng GenAI vào sản phẩm của mình, tùy biến sâu" → Bedrock.
- **Vì sao quan trọng:** đây là kiểu câu hỏi chọn dịch vụ phổ biến nhất của AIF-C01 cho nhóm sản phẩm này.

---

## 3. Key Terms

| Term | Giải thích ngắn | Dấu hiệu trong đề |
|---|---|---|
| Amazon Quick | Trợ lý agentic AI cho nhân viên trên dữ liệu và app công ty; kế nhiệm Q Business | "employees ask questions about company documents", "no custom development" |
| Amazon Q Business | Tên cũ, không nhận khách hàng mới, thay bằng Amazon Quick | Đề cũ: "enterprise assistant on company data" |
| Quick Index | Knowledge base hợp nhất của Quick (RAG fully managed) | "index company documents for AI answers" |
| Quick Research | Agent nghiên cứu sâu, kết hợp dữ liệu nội bộ + web, báo cáo có trích dẫn | "research competitors", "report with citations" |
| Quick Flows | Tự động hóa tác vụ lặp lại, mô tả bằng ngôn ngữ tự nhiên | "every Monday, email the report" |
| Quick Automate | Tự động hóa quy trình nhiều bước, nhiều hệ thống, có human approval | "multi-step business process", "approval step" |
| Quick Sight | BI dashboard có AI (trước là Amazon QuickSight) | "dashboards", "ask questions about data in natural language" |
| Spaces | Không gian chung của team: tài liệu, dashboard, knowledge base, agent | "shared team workspace" |
| Apps in Amazon Quick | Build web app không cần code bằng ngôn ngữ tự nhiên (thay Q Apps) | "build an internal app without coding" |
| Actions / Connectors | Cho agent thực hiện việc trên app bên thứ ba (Jira, Salesforce, Slack...) | "create a Jira ticket", "update CRM record" |
| MCP / OpenAPI connector | Tự kết nối API hoặc MCP server riêng vào Quick | "connect to our own internal API" |
| Document-level ACL | User chỉ nhận câu trả lời từ tài liệu mình có quyền xem | "users must only see documents they are permitted to access" |
| IAM Identity Center | Xác thực user cho Quick, nối với IdP ngoài (Google, Microsoft AD/Entra) | "single sign-on", "corporate identity provider" |
| Amazon Q Developer | Trợ lý AI cho developer/vận hành trong AWS Console | "list my Lambda functions", "explain my bill", "suggest CLI command" |
| Kiro | Trợ lý viết code (IDE + CLI) thay cho Q Developer IDE plugins/CLI | "code suggestions in IDE", "agentic coding" |
| Amazon Q Developer in chat applications | Q trong Slack/Teams (trước là AWS Chatbot) | "Slack/Teams channel", "alarm notifications", "ChatOps" |
| EC2 instance type finder | Gợi ý instance type cho workload **mới** (trước gọi là Amazon Q for EC2) | "choose instance type for a new workload" |
| `[+]` AWS Compute Optimizer | Gợi ý instance cho workload **đang chạy**, dựa trên số liệu sử dụng | "right-size existing instances" |
| Amazon Q data integration in AWS Glue | Sinh và sửa ETL script, giải thích lỗi Glue job | "generate ETL code", "troubleshoot Glue job" |
| PartyRock | Playground build app GenAI không code, không cần AWS account | "experiment without AWS account", "learn prompt engineering" |

---

## 4. Dễ nhầm & bẫy đề

### 4.1 Các cặp dễ lẫn

| Concept A | Concept B | Khác nhau ở đâu | Mẹo nhớ |
|---|---|---|---|
| Amazon Quick | Amazon Bedrock | Quick: **sản phẩm dùng ngay** (SaaS) cho nhân viên; Bedrock: **nền tảng để tự build** app GenAI | "No development / out-of-the-box" → Quick; "build our own app" → Bedrock |
| Amazon Quick | Amazon Q Developer | Quick: nhân viên, dữ liệu doanh nghiệp; Q Developer: developer, tài nguyên AWS và code | Hỏi về **công ty** → Quick; hỏi về **AWS account** → Q Developer |
| Amazon Q Developer (console) | Kiro | Q Developer: chat trong AWS Console về account, chi phí, docs; Kiro: viết code trong IDE/CLI | Code trong IDE → Kiro |
| Amazon Q Business | Amazon Quick | Q Business là tên cũ, không nhận khách mới; Quick kế nhiệm, thêm Research, Flows, Automate, Sight, Apps | Đề cũ ghi Q Business = Quick ngày nay |
| Quick Flows | Quick Automate | Flows: tự động hóa **đơn giản**, mô tả bằng lời; Automate: quy trình **nhiều bước, nhiều hệ thống**, có approval | Có "human approval / complex process" → Automate |
| Quick Index | Bedrock Knowledge Bases | Quick Index: RAG có sẵn trong Quick cho nhân viên; Bedrock KB: RAG bạn tích hợp vào app tự build | Tự build app → Bedrock KB |
| Quick Research | Quick (chat) | Research: điều tra sâu, ra **báo cáo có trích dẫn**; Chat: hỏi đáp nhanh | "Comprehensive cited report" → Research |
| Quick Sight | Quick Index | Quick Sight: dữ liệu **có cấu trúc** (DB, warehouse) → dashboard; Quick Index: **tài liệu** (unstructured) → RAG | Chart/dashboard → Quick Sight |
| IAM Identity Center | Document-level ACL | Identity Center: **xác thực ai là user**; ACL: **user được xem tài liệu nào** | Cả hai cùng đảm bảo user chỉ thấy đúng dữ liệu của mình |
| Actions (connectors) | Knowledge bases | KB: **đọc** dữ liệu để trả lời; Actions: **làm việc** trên hệ thống khác (tạo ticket, gửi tin) | "Create / update / send" → Actions |
| EC2 instance type finder | `[+]` AWS Compute Optimizer | Finder: workload **mới**, dựa trên mô tả nhu cầu; Compute Optimizer: workload **đang chạy**, dựa trên số liệu sử dụng | New → finder; existing → Compute Optimizer |
| PartyRock | Amazon Bedrock Playground | PartyRock: **không cần AWS account**, build và chia sẻ app; Bedrock Playground: trong console, cần account | "No AWS account" → PartyRock |
| Amazon Q Developer in chat applications | Amazon Quick extension cho Slack/Teams | Q Developer in chat: ChatOps cho **tài nguyên AWS** (alarm, troubleshoot); Quick extension: trợ lý nhân viên trên **dữ liệu công ty** | Alarm, AWS resource → Q Developer in chat |

### 4.2 Từ khóa trong đề → chọn gì

| Từ khóa trong đề | Chọn | Ghi chú |
|---|---|---|
| "employees ask questions over company documents (SharePoint, S3, Confluence)" | Amazon Quick (trước là Q Business) | |
| "fully managed, no ML expertise, no custom code" + nhân viên | Amazon Quick | |
| "only return answers from documents the user can access" | Quick + IAM Identity Center + document-level ACL | |
| "in-depth research report with citations" | Quick Research | |
| "automate weekly report email using natural language" | Quick Flows | |
| "multi-step workflow across systems with human approval" | Quick Automate | |
| "BI dashboards, ask questions about data in natural language" | Quick Sight (trước là QuickSight + Amazon Q) | |
| "build internal web app without coding" | Apps in Amazon Quick (trước là Q Apps) | |
| "agent creates Jira issue / updates Salesforce" | Quick Actions (connectors) | |
| "connect Quick to our internal API" | Custom connector qua MCP / OpenAPI | |
| "list my AWS resources / explain my AWS bill / suggest CLI" | Amazon Q Developer | |
| "code suggestions, security scan in IDE" | Kiro (trước là Q Developer IDE plugin) | |
| "AWS notifications and troubleshooting in Slack/Teams" | Amazon Q Developer in chat applications (trước là AWS Chatbot) | |
| "choose EC2 instance type for a new workload" | EC2 instance type finder / hỏi Amazon Q | |
| "right-size running instances" | `[+]` AWS Compute Optimizer | |
| "generate / troubleshoot Glue ETL script" | Amazon Q data integration in AWS Glue | |
| "try GenAI apps without an AWS account or code" | PartyRock | |
| "build custom GenAI feature into our own product" | Amazon Bedrock | Không phải Quick |

---

## 5. Câu hỏi tình huống

**Câu 1.** Một công ty muốn nhân viên đặt câu hỏi bằng ngôn ngữ tự nhiên trên tài liệu nội bộ lưu ở SharePoint và Confluence. Họ không có đội ML và muốn giải pháp **có sẵn, ít công sức nhất**. Nên chọn gì?
- A. Amazon Bedrock Knowledge Bases + tự viết frontend
- B. Amazon Quick
- C. Amazon SageMaker AI
- D. Kiro

<details><summary>Đáp án</summary>

**B.** Quick là trợ lý fully managed cho nhân viên, có sẵn connector SharePoint/Confluence và Quick Index (RAG managed).
- A sai: làm được nhưng phải tự build, tốn công hơn.
- C sai: dùng để build và train model ML.
- D sai: trợ lý viết code cho developer.
</details>

**Câu 2.** (Chọn 2) Công ty ở câu 1 yêu cầu nhân viên phòng kinh doanh **không được** nhận câu trả lời dựa trên tài liệu lương của phòng HR. Cơ chế nào đáp ứng yêu cầu này?
- A. Xác thực user qua IAM Identity Center
- B. Document-level access control (ACL) từ nguồn dữ liệu
- C. Tăng temperature của model
- D. Dùng Batch inference
- E. Quick Flows

<details><summary>Đáp án</summary>

**A, B.** Identity Center xác định user là ai; ACL đảm bảo user chỉ nhận câu trả lời từ tài liệu mình có quyền xem.
- C, D sai: không liên quan đến phân quyền.
- E sai: Flows là tự động hóa tác vụ.
</details>

**Câu 3.** Một trưởng nhóm muốn **mỗi sáng thứ Hai**, hệ thống tự gửi email dashboard doanh số mới nhất cho cả team, và chỉ muốn mô tả yêu cầu bằng ngôn ngữ tự nhiên. Tính năng nào phù hợp nhất?
- A. Quick Research
- B. Quick Flows
- C. Quick Index
- D. Amazon Q Developer

<details><summary>Đáp án</summary>

**B.** Flows tự động hóa tác vụ lặp lại đơn giản, tạo bằng ngôn ngữ tự nhiên.
- A sai: Research dùng để tạo báo cáo nghiên cứu.
- C sai: Index chỉ là kho tri thức.
- D sai: Q Developer dành cho tài nguyên AWS.
</details>

**Câu 4.** Một kỹ sư DevOps muốn hỏi ngay trong AWS Console: *"Liệt kê tất cả Lambda function ở us-east-1"* và *"Vì sao hóa đơn tháng này tăng?"*. Nên dùng gì?
- A. Amazon Quick
- B. Amazon Q Developer
- C. PartyRock
- D. Amazon Bedrock Guardrails

<details><summary>Đáp án</summary>

**B.** Q Developer trả lời về tài nguyên trong account, phân tích chi phí và gợi ý lệnh CLI.
- A sai: Quick dành cho dữ liệu doanh nghiệp của nhân viên.
- C sai: PartyRock là playground học GenAI.
- D sai: Guardrails là bộ lọc an toàn.
</details>

**Câu 5.** Một team developer muốn gợi ý code real-time và quét lỗ hổng bảo mật ngay trong VS Code. Vào thời điểm 10/2026, AWS khuyến nghị dùng gì cho nhu cầu mới?
- A. Amazon Q Business
- B. Kiro
- C. Amazon Q Developer in chat applications
- D. AWS Glue

<details><summary>Đáp án</summary>

**B.** Q Developer IDE plugins hết hỗ trợ ngày 30/04/2027; AWS khuyến nghị chuyển sang Kiro (inline suggestions, chat, code generation, agentic coding).
- A sai: Q Business không nhận khách mới và không dành cho code.
- C sai: dùng trong Slack/Teams.
- D sai: dịch vụ ETL.
</details>

**Câu 6.** Team vận hành muốn nhận cảnh báo CloudWatch alarm trong kênh Slack và hỏi ngay trong kênh về nguyên nhân sự cố trên tài nguyên AWS. Nên dùng gì?
- A. Amazon Q Developer in chat applications
- B. Quick Research
- C. EC2 instance type finder
- D. PartyRock

<details><summary>Đáp án</summary>

**A.** Trước gọi là AWS Chatbot; nhận notification (alarm, security finding, billing) và hỏi Amazon Q ngay trong Slack/Teams.
- B sai: Research tạo báo cáo nghiên cứu.
- C sai: chọn instance type.
- D sai: playground GenAI.
</details>

**Câu 7.** (Chọn 2) Ghép đúng nhu cầu với công cụ:
- A. Chọn instance type cho một workload **mới** → EC2 instance type finder
- B. Sinh script ETL PySpark từ mô tả bằng lời → Amazon Q data integration in AWS Glue
- C. Right-size các instance **đang chạy** → PartyRock
- D. Hỏi dữ liệu bán hàng và nhận biểu đồ → Kiro
- E. Build web app nội bộ không code → Amazon Q Developer in chat applications

<details><summary>Đáp án</summary>

**A, B.**
- C sai: right-size instance đang chạy là việc của AWS Compute Optimizer.
- D sai: hỏi dữ liệu và nhận biểu đồ là Quick Sight.
- E sai: build app không code là Apps in Amazon Quick.
</details>

**Câu 8.** Một giảng viên muốn sinh viên thử xây ứng dụng GenAI đơn giản (vd: tạo câu đố từ đoạn văn) **mà không cần tạo AWS account và không viết code**. Lựa chọn nào phù hợp nhất?
- A. Amazon Bedrock Playground trong AWS Console
- B. PartyRock
- C. Amazon SageMaker AI Studio
- D. Kiro

<details><summary>Đáp án</summary>

**B.** PartyRock là playground chạy trên Bedrock, không cần AWS account, không cần code.
- A sai: cần AWS account.
- C sai: cần AWS account và kiến thức ML.
- D sai: công cụ viết code.
</details>

---

### Nguồn đã đối chiếu
- AIF-C01 exam guide, Domain 1 (Task 1.3.4 nêu Amazon Q, Amazon Quick, Kiro): https://docs.aws.amazon.com/aws-certification/latest/ai-practitioner-01/ai-practitioner-01-domain1.html
- What is Amazon Quick: https://docs.aws.amazon.com/quick/latest/userguide/what-is.html
- How Amazon Quick works (agents, spaces, integrations): https://docs.aws.amazon.com/quick/latest/userguide/how-quicksuite-works.html
- Quick integrations (100+ pre-built action connectors): https://docs.aws.amazon.com/quick/latest/userguide/connecting-integrations-apps.html
- Quick document-level ACL: https://docs.aws.amazon.com/quick/latest/userguide/acl-best-practices-kb.html
- Quick data privacy (không train model trên dữ liệu khách hàng): https://docs.aws.amazon.com/quick/latest/userguide/teams-extension-user-guide.html
- Amazon Q Business availability change → Amazon Quick: https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/qbusiness-availability-change.html
- Amazon Q Developer IDE plugins end of support → Kiro: https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-developer-ide-end-of-support.html
- Upgrade to Kiro (Q Developer CLI rebrand): https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/upgrade-to-kiro.html
- Q Developer chat về tài nguyên và chi phí: https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/chat-actions.html, https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/chat-costs.html
- AWS Chatbot → Amazon Q Developer in chat applications: https://docs.aws.amazon.com/chatbot/latest/adminguide/service-rename.html
- EC2 instance type finder: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-ec2-instance-type-recommendations.html
- Amazon Q data integration in AWS Glue: https://docs.aws.amazon.com/glue/latest/dg/q.html
- PartyRock (nguồn web): https://aws.amazon.com/about-aws/whats-new/2023/11/partyrock-amazon-bedrock-playground/, https://innfactory.de/en/cloud/aws/products/partyrock/
