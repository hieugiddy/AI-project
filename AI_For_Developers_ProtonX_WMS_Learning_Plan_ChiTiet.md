# AI For Developers – ProtonX Learning Plan
## Học AI từ số 0 thông qua dự án WMS thực tế

> **Mục tiêu:** Không học AI theo kiểu học thuộc thuật ngữ.  
> Mỗi bài học bắt đầu bằng một **câu chuyện nghiệp vụ trong WMS**, sau đó mới đi đến bài toán kỹ thuật, code và khái niệm AI.

---

# 1. Câu chuyện tổng thể của khóa học

Bạn đang phát triển một hệ thống **WMS – Warehouse Management System** cho một công ty logistics.

Hệ thống hiện tại có các service:

- API Gateway – `8080`
- Auth Service – `8081`
- Product Service – `8082`
- Warehouse Service – `8083`
- Inventory Service – `8084`
- Order Service – `8085`
- Shipment Service – `8086`
- Notification Service – `8087`

Hệ thống đã có dữ liệu về:

- sản phẩm
- kho
- khu vực kho
- tồn kho
- đơn hàng
- chi tiết đơn hàng
- giao dịch tồn kho
- vận chuyển
- thông báo
- người dùng

## Vấn đề

Nhân viên kho và quản lý phải thao tác qua rất nhiều màn hình.

Ví dụ, quản lý hỏi:

> "Kiểm tra giúp tôi sản phẩm A100 hiện còn bao nhiêu hàng ở tất cả các kho."

Hiện tại họ phải:

```text
Mở màn hình Inventory
        ↓
Tìm SKU A100
        ↓
Chọn từng kho
        ↓
Xem tồn kho
        ↓
Tự cộng số lượng
```

Chúng ta muốn xây dựng:

```text
                    Người dùng
                        │
                        │ hỏi bằng tiếng Việt
                        ▼
                ┌─────────────────┐
                │   AI WMS        │
                │   Assistant     │
                └────────┬────────┘
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          WMS API       RAG       Database
             │           │           │
             ▼           ▼           ▼
          Tồn kho      Tài liệu    Lịch sử
          Đơn hàng     nghiệp vụ   giao dịch
             │           │           │
             └───────────┼───────────┘
                         ▼
                        AI
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Trả lời    Cảnh báo    Báo cáo
```

Cuối khóa học, người dùng có thể nói:

> "Kiểm tra tồn kho A100 ở tất cả các kho. Phân tích biến động trong 3 tháng gần đây. Nếu tồn dưới 100 thì tạo cảnh báo cho quản lý. Cuối cùng tạo báo cáo."

AI sẽ phải biết cách:

1. hiểu câu hỏi
2. lấy dữ liệu tồn kho
3. lấy lịch sử giao dịch
4. tìm tài liệu nghiệp vụ nếu cần
5. phân tích dữ liệu
6. quyết định có cần tạo cảnh báo hay không
7. gọi Notification Service
8. tạo báo cáo
9. trả lời người dùng

---

# 2. Nguyên tắc học

Mỗi bài học đều đi theo 7 bước:

## ① 🏢 Câu chuyện thực tế

Ai đang gặp vấn đề gì?

## ② 🎯 Yêu cầu

Người dùng muốn hệ thống làm gì?

## ③ 🤖 AI phải làm gì?

AI đóng vai trò gì trong bài toán?

## ④ 🔌 WMS/API liên quan

Service hoặc API nào được sử dụng?

## ⑤ 📦 Dữ liệu

Input và output cụ thể là gì?

## ⑥ 💻 Bài tập code

Bạn phải tự xây dựng phần nào?

## ⑦ 🧠 Kiến thức AI

Sau bài này bạn hiểu thêm khái niệm AI nào?

---

# 3. PHASE 0 – Chuẩn bị nền tảng

> **Mục tiêu:** Có đủ Python/FastAPI để bắt đầu xây AI service.

---

# Bài 0.1 – Nhân viên kho muốn biết tồn kho

## 🏢 Câu chuyện

Anh Nam là nhân viên quản lý kho.

Buổi sáng, anh nhận được yêu cầu:

> "Kiểm tra giúp tôi sản phẩm A100 còn bao nhiêu trong kho Hà Nội."

WMS đã có Inventory Service.

Inventory Service cung cấp dữ liệu:

```text
SKU: A100
Tên: Laptop Dell
Kho: Hà Nội
Tồn: 500
```

## 🎯 Yêu cầu

Xây một API Python:

```http
GET /inventory/A100?warehouse=HN
```

API trả:

```json
{
  "sku": "A100",
  "warehouse": "HN",
  "quantity": 500
}
```

## 🤖 AI có xuất hiện chưa?

**Chưa.**

Đây là chủ ý.

Trước khi học AI, chúng ta cần hiểu cách một AI service giao tiếp với hệ thống backend.

## 🔌 WMS liên quan

```text
AI Service
    ↓
Inventory Service
    ↓
Database
```

## 💻 Bài tập

Tạo project:

```text
wms-ai-platform/
├── app/
│   ├── main.py
│   └── models.py
├── tests/
├── requirements.txt
└── README.md
```

Cài:

```text
fastapi
uvicorn
pydantic
httpx
```

Tạo endpoint:

```http
GET /inventory/{sku}
```

## 🧠 Bạn học được

- Python cơ bản
- FastAPI
- REST API
- Pydantic
- JSON
- HTTP request/response

---

# Bài 0.2 – Quản lý muốn xem tất cả kho

## 🏢 Câu chuyện

Quản lý nói:

> "Không phải chỉ kho Hà Nội. Tôi muốn biết A100 còn bao nhiêu ở tất cả các kho."

Dữ liệu:

| Kho | Tồn |
|---|---:|
| Hà Nội | 500 |
| Hải Phòng | 120 |
| Quảng Ninh | 80 |
| TP.HCM | 350 |

## 🎯 Yêu cầu

API trả:

```json
{
  "sku": "A100",
  "total": 1050,
  "warehouses": [
    {
      "name": "Hà Nội",
      "quantity": 500
    },
    {
      "name": "Hải Phòng",
      "quantity": 120
    },
    {
      "name": "Quảng Ninh",
      "quantity": 80
    },
    {
      "name": "TP.HCM",
      "quantity": 350
    }
  ]
}
```

## 💻 Bài tập

Viết Python để:

1. lấy dữ liệu
2. duyệt danh sách kho
3. tính tổng
4. trả JSON

## 🧠 Bạn học được

- list
- dict
- function
- class
- Pydantic model
- xử lý JSON

---

# Bài 0.3 – Lấy dữ liệu từ nhiều service

## 🏢 Câu chuyện

Quản lý hỏi:

> "Cho tôi thông tin đầy đủ của A100."

Muốn trả lời, hệ thống cần:

```text
Product Service
    ↓
Tên sản phẩm
Giá
Category

Inventory Service
    ↓
Tồn kho

Warehouse Service
    ↓
Thông tin kho
```

## 🎯 Yêu cầu

Gộp dữ liệu thành:

```json
{
  "sku": "A100",
  "name": "Laptop Dell",
  "category": "Laptop",
  "totalInventory": 1050,
  "warehouses": [...]
}
```

## 💻 Bài tập

Dùng `httpx` gọi nhiều API.

Sau đó học:

```python
async
await
asyncio
```

## 🧠 Bạn học được

Tại sao backend AI thường phải gọi nhiều service song song.

---

# 4. PHASE 1 – API AI

## Bài 1.1 – Lần đầu nói chuyện với AI

## 🏢 Câu chuyện

Bây giờ công ty muốn nhân viên không cần nhớ API.

Thay vì:

```text
GET /inventory/A100
```

Nhân viên chỉ cần nói:

> "A100 còn bao nhiêu?"

## 🎯 Yêu cầu

Tạo:

```http
POST /api/ai/chat
```

Input:

```json
{
  "message": "A100 còn bao nhiêu?"
}
```

AI trả lời:

```text
A100 hiện còn 1,050 sản phẩm trên tất cả các kho.
```

## 🤖 AI phải làm gì?

Ở bước đầu, AI chỉ cần hiểu câu hỏi và tạo câu trả lời.

## 💻 Bài tập

1. tạo API
2. nhận message
3. gọi LLM API
4. nhận response
5. trả response cho user

## 🧠 Kiến thức

- LLM
- AI API
- API key
- system prompt
- user prompt
- model response
- timeout
- error handling

---

# 5. PHASE 2 – Prompt Engineering

# Bài 2.1 – AI trả lời sai định dạng

## 🏢 Câu chuyện

Bạn hỏi:

> "A100 còn bao nhiêu?"

AI có thể trả:

```text
Theo dữ liệu tôi thấy thì A100 còn khá nhiều...
```

Không tốt.

Backend cần dữ liệu rõ ràng.

## 🎯 Yêu cầu

Muốn AI trả:

```json
{
  "intent": "CHECK_INVENTORY",
  "sku": "A100"
}
```

## 🤖 AI phải làm gì?

AI phải biến câu nói tự nhiên thành dữ liệu có cấu trúc.

Ví dụ:

```text
"A100 còn bao nhiêu?"
```

↓

```json
{
  "intent": "CHECK_INVENTORY",
  "sku": "A100"
}
```

## 🧠 Kiến thức

- Prompt
- System prompt
- User prompt
- Context
- Constraints
- Output format
- Structured Output

---

# Bài 2.2 – AI phải biết khi nào cần gọi WMS

## 🏢 Câu chuyện

Nhân viên hỏi:

> "A100 còn bao nhiêu?"

AI không thể tự biết số tồn.

Nó phải gọi WMS.

## Luồng

```text
User
 ↓
AI
 ↓
Hiểu: CHECK_INVENTORY
 ↓
Cần dữ liệu thật
 ↓
Gọi get_inventory("A100")
 ↓
Inventory Service
 ↓
Kết quả
 ↓
AI
 ↓
Trả lời User
```

## 🎯 Yêu cầu

Tạo tool:

```text
get_inventory(sku)
```

## 🧠 Kiến thức

**Tool Calling / Function Calling**

Đây là một bước cực kỳ quan trọng.

AI không trực tiếp truy cập database.

AI yêu cầu backend thực hiện một function.

---

# Bài 2.3 – AI xử lý nhiều loại câu hỏi

Nhân viên hỏi:

> "A100 còn bao nhiêu?"

hoặc:

> "Thông tin sản phẩm A100 là gì?"

hoặc:

> "Đơn hàng nào đang chứa A100?"

AI phải nhận biết:

```text
CHECK_INVENTORY
GET_PRODUCT
SEARCH_ORDER
```

Mỗi loại câu hỏi tương ứng với tool khác nhau.

---

# 6. PHASE 3 – AGENT

# Bài 3.1 – AI tự thực hiện nhiều bước

## 🏢 Câu chuyện

Quản lý nói:

> "Kiểm tra A100. Nếu tồn dưới 100 thì cảnh báo cho tôi."

AI phải:

```text
Bước 1
Kiểm tra tồn A100
       ↓
Bước 2
Tồn = 80
       ↓
Bước 3
80 < 100
       ↓
Bước 4
Tạo cảnh báo
```

## Tools

```text
get_inventory()
create_notification()
```

## 🤖 Đây chính là Agent

Agent có thể:

```text
nhận mục tiêu
    ↓
suy luận bước tiếp theo
    ↓
gọi tool
    ↓
nhận kết quả
    ↓
quyết định tiếp
    ↓
gọi tool khác
```

## 🧠 Kiến thức

- Agent
- Tool
- State
- Loop
- Decision
- Error handling

---

# 7. PHASE 4 – Docker Compose

## 🏢 Câu chuyện

AI Service bắt đầu có nhiều thành phần:

```text
AI Service
PostgreSQL
Vector Database
Redis
```

Không muốn cài từng thứ thủ công.

## 🎯 Yêu cầu

Dùng Docker Compose:

```text
docker-compose.yml
```

Chạy:

```text
AI Service
PostgreSQL
Qdrant
```

## 🧠 Kiến thức

- Docker
- Docker Compose
- Environment variables
- Network
- Volume
- Service dependency

---

# 8. PHASE 5 – RAG

# Bài 5.1 – AI không biết quy định của công ty

## 🏢 Câu chuyện

Quản lý hỏi:

> "Theo quy định công ty, hàng tồn quá 90 ngày phải xử lý thế nào?"

Thông tin này **không nằm trong database tồn kho**.

Nó nằm trong file:

```text
inventory-policy.pdf
```

AI model không tự biết chính sách nội bộ của công ty.

## 🎯 Yêu cầu

Cho AI đọc tài liệu nội bộ.

Ví dụ tài liệu:

```text
inventory-policy.pdf
warehouse-guide.pdf
order-process.pdf
```

Người dùng hỏi:

> "Hàng tồn quá 90 ngày phải xử lý thế nào?"

AI phải tìm đoạn liên quan rồi trả lời.

## Luồng

```text
PDF
 ↓
Chunk
 ↓
Embedding
 ↓
Vector Database
 ↓
Search
 ↓
Relevant Context
 ↓
LLM
 ↓
Answer
```

## 🧠 Kiến thức

- RAG
- Embedding
- Chunking
- Vector Database
- Retrieval
- Context
- Citation

---

# 9. PHASE 6 – LangChain

## 🏢 Câu chuyện

Project bắt đầu lớn.

Bạn có:

```text
LLM
Tools
RAG
Prompt
Retriever
Agent
Memory
```

Việc tự quản lý tất cả trở nên phức tạp.

## 🎯 Yêu cầu

Dùng LangChain để tổ chức các thành phần.

## 🧠 Kiến thức

- Chain
- Retriever
- Tool
- Agent
- Memory
- RAG

---

# 10. PHASE 7 – LangGraph

## 🏢 Câu chuyện

Agent WMS bây giờ phải làm nhiều bước.

Ví dụ:

```text
User
 ↓
Kiểm tra tồn
 ↓
Kiểm tra lịch sử
 ↓
Phân tích
 ↓
Có tồn thấp?
 ├── YES → Tạo cảnh báo
 └── NO
 ↓
Tạo báo cáo
 ↓
Trả lời
```

Đây không còn là một lời gọi AI đơn giản.

## 🎯 Yêu cầu

Biểu diễn workflow thành graph:

```text
START
  ↓
Understand Request
  ↓
Inventory Tool
  ↓
History Tool
  ↓
Analysis
  ↓
Condition
 ├──────── YES ───────→ Notification
 │
 └──────── NO
          ↓
        Report
          ↓
         END
```

## 🧠 Kiến thức

- State
- Node
- Edge
- Conditional Edge
- Tool Node
- Retry
- Error Node
- Loop limit

---

# 11. PHASE 8 – Image API

## 🏢 Câu chuyện

Kho có hàng hóa được dán label.

Nhân viên chụp ảnh label:

```text
SKU: A100
LOT: 20260930
QTY: 50
```

Nhân viên muốn AI đọc ảnh.

## 🎯 Yêu cầu

Upload ảnh:

```text
label.jpg
```

AI trả:

```json
{
  "sku": "A100",
  "lot": "20260930",
  "quantity": 50
}
```

## 🧠 Kiến thức

- Vision model
- Image API
- Multimodal AI
- Image input

---

# 12. PHASE 9 – Audio + Video

## 🏢 Câu chuyện

Nhân viên kho đang đi kiểm kê.

Họ không muốn gõ.

Họ nói:

> "Kiểm tra A100 ở kho Quảng Ninh."

Hệ thống:

```text
Voice
 ↓
Speech To Text
 ↓
AI
 ↓
WMS Tool
 ↓
Response
 ↓
Text To Speech
 ↓
Voice
```

## 🧠 Kiến thức

- Speech-to-Text
- Text-to-Speech
- Audio AI
- Video AI

---

# 13. PHASE 10 – Frontend

## 🏢 Câu chuyện

Backend đã chạy.

Bây giờ nhân viên cần một giao diện:

```text
┌──────────────────────────────────────┐
│          AI WMS ASSISTANT            │
├──────────────────────────────────────┤
│                                      │
│ User: A100 còn bao nhiêu?            │
│                                      │
│ AI: A100 còn 1,050 sản phẩm.         │
│                                      │
│ ┌──────────────────────────────────┐ │
│ │ Hà Nội             500           │ │
│ │ Hải Phòng          120           │ │
│ │ Quảng Ninh          80           │ │
│ │ TP.HCM             350           │ │
│ └──────────────────────────────────┘ │
│                                      │
│ [ Nhập câu hỏi...                 ]  │
└──────────────────────────────────────┘
```

## 🧠 Kiến thức

- React
- TypeScript
- API integration
- Streaming
- SSE
- Chat UI

---

# 14. PHASE 11 – Dự án AI WMS Copilot

Đây là project chính.

## US01 – Kiểm tra tồn kho

User:

> "A100 còn bao nhiêu?"

AI:

```text
Inventory Tool
```

---

## US02 – Kiểm tra nhiều kho

User:

> "A100 còn bao nhiêu ở tất cả các kho?"

AI:

```text
Inventory Tool
+
Warehouse Tool
```

---

## US03 – Phân tích lịch sử

User:

> "Tồn A100 trong 3 tháng vừa rồi biến động thế nào?"

AI:

```text
Inventory
    ↓
Inventory Transactions
    ↓
SQL/Data Analysis
    ↓
AI
```

---

## US04 – Hỏi quy định

User:

> "Hàng tồn quá 90 ngày xử lý thế nào?"

AI:

```text
Question
 ↓
RAG
 ↓
Policy Documents
 ↓
Answer + Citation
```

---

## US05 – Tạo cảnh báo

User:

> "Nếu A100 dưới 100 thì báo quản lý."

AI:

```text
Check Inventory
       ↓
Quantity < 100?
       ↓
     YES
       ↓
Notification Tool
```

---

## US06 – Tạo báo cáo

User:

> "Tạo báo cáo tồn kho A100 tháng này."

AI:

```text
Database
   ↓
Data Analysis
   ↓
LLM
   ↓
Report
```

---

# 15. PHASE 12 – Training Model

## 🏢 Câu chuyện

Bạn nhận thấy có những câu hỏi nghiệp vụ rất đặc thù.

Ví dụ:

```text
"Hàng này đang bị hold."

"Cho tôi xem hàng bị block."

"SKU này đang treo."

"Kiểm tra hàng đang quarantine."
```

Các từ có thể có ý nghĩa nghiệp vụ riêng trong công ty.

## 🎯 Trước khi training

Phải biết phân biệt:

```text
Prompt Engineering
        ↓
RAG
        ↓
Fine-tuning
        ↓
Training Model
```

Không phải vấn đề nào cũng cần training.

## 🧠 Kiến thức

- Dataset
- Training
- Fine-tuning
- Instruction tuning
- LoRA
- PEFT
- Inference
- Quantization

---

# 16. PHASE 13 – Graph RAG

## 🏢 Câu chuyện

Quản lý hỏi một câu phức tạp:

> "Tại sao A100 ở Quảng Ninh giảm mạnh? Những đơn hàng nào làm tồn kho giảm và A100 thuộc nhóm sản phẩm nào?"

Lúc này chúng ta cần hiểu **quan hệ giữa các entity**.

```text
Product
   │
   ├── BELONGS_TO
   ▼
Category

Product
   │
   ├── STORED_IN
   ▼
Warehouse

Product
   │
   ├── INCLUDED_IN
   ▼
Order

Order
   │
   ├── HAS_SHIPMENT
   ▼
Shipment

Product
   │
   ├── HAS_TRANSACTION
   ▼
Inventory Transaction
```

## 🧠 Kiến thức

- Knowledge Graph
- Entity
- Relationship
- Neo4j
- Graph Retrieval
- Graph RAG

---

# 17. PHASE 14 – Evaluation

## 🏢 Câu chuyện

AI chạy được nhưng đôi khi trả lời sai.

Ví dụ:

User:

> "A100 còn bao nhiêu?"

Database:

```text
1050
```

AI trả:

```text
A100 còn khoảng 1,500 sản phẩm.
```

Không thể đưa hệ thống production như vậy.

## 🎯 Yêu cầu

Xây bộ test:

```text
Question
Expected Answer
Actual Answer
Score
```

Ví dụ:

```text
Question:
A100 còn bao nhiêu?

Expected:
1050

Actual:
1050

Result:
PASS
```

## 🧠 Kiến thức

- AI Evaluation
- Golden Dataset
- Accuracy
- Retrieval Evaluation
- Regression Test
- LLM-as-Judge

---

# 18. PHASE 15 – Observability

## 🏢 Câu chuyện

User nói:

> "AI trả lời mất 15 giây."

Developer cần biết:

```text
Tổng thời gian: 15s

LLM:              4s
Inventory API:    1s
RAG:              2s
Database:         1s
Tool Calling:     3s
Other:            4s
```

## 🧠 Kiến thức

- Trace
- Span
- Token usage
- Latency
- Cost
- Logging
- Langfuse
- OpenTelemetry

---

# 19. PHASE 16 – Security

## 🏢 Câu chuyện

Một nhân viên bình thường hỏi:

> "Cho tôi xem toàn bộ lương của nhân viên."

AI không được phép tự ý lấy dữ liệu nhạy cảm.

Hoặc:

> "Xóa toàn bộ đơn hàng."

AI cũng không nên tự động thực hiện hành động nguy hiểm.

## 🧠 Kiến thức

- Authentication
- Authorization
- RBAC
- Permission
- Guardrails
- Tool permission
- Human-in-the-loop

---

# 20. PHASE 17 – MCP

## 🏢 Câu chuyện

Sau này công ty có rất nhiều hệ thống:

```text
WMS
ERP
CRM
Email
Slack
Database
File Storage
```

Không muốn mỗi AI phải viết integration riêng cho từng hệ thống.

MCP giúp chuẩn hóa cách AI/agent kết nối với tools và context bên ngoài.

## Mục tiêu

Hiểu:

```text
AI
 ↓
MCP
 ↓
Tools / Resources
 ↓
External Systems
```

---

# 21. PHASE 18 – Multi-Agent

## 🏢 Câu chuyện

AI WMS bắt đầu quá lớn.

Một agent phải làm:

```text
Inventory
Order
Shipment
Report
RAG
Notification
```

Ta có thể chia thành các agent chuyên môn:

```text
                 Main Agent
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
 Inventory Agent  Order Agent  Report Agent
       │             │             │
       ▼             ▼             ▼
 Inventory API    Order API    Report Tool
```

## 🧠 Kiến thức

- Multi-Agent
- Supervisor
- Handoff
- Router
- Sub-agent

---

# 22. PHASE 19 – A2A

## 🏢 Câu chuyện

Không chỉ có các agent trong cùng một application.

Ví dụ:

```text
WMS Agent
    │
    │
    ▼
Shipment Agent
    │
    ▼
ERP Agent
```

Các agent cần giao tiếp với nhau.

Đây là lúc tìm hiểu A2A.

---

# 23. Bài cuối – AI WMS Copilot hoàn chỉnh

Người quản lý gửi đúng một câu:

> **"Hãy kiểm tra tồn kho SKU A100 ở tất cả các kho. Phân tích biến động trong 3 tháng gần đây. Tìm nguyên nhân nếu tồn giảm mạnh. Nếu tồn dưới 100 thì tạo cảnh báo cho quản lý. Cuối cùng tạo báo cáo."**

Hệ thống phải thực hiện:

```text
                         USER
                          │
                          ▼
                    MAIN AGENT
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
       Inventory Tool   SQL Tool    RAG Tool
             │            │            │
             ▼            ▼            ▼
        Current Stock   History     Documents
             │            │            │
             └────────────┼────────────┘
                          ▼
                       ANALYSIS
                          │
                    ┌─────┴─────┐
                    │           │
                Stock < 100?    │
                    │           │
                   YES          NO
                    │           │
                    ▼           │
              Notification     │
                    │           │
                    └─────┬─────┘
                          ▼
                       REPORT
                          │
                          ▼
                         USER
```

---

# 24. Thứ tự học chính thức

Không học tất cả cùng lúc.

Đi theo thứ tự:

```text
PHASE 0
Python + FastAPI
        ↓
PHASE 1
AI API
        ↓
PHASE 2
Prompt
        ↓
Structured Output
        ↓
Tool Calling
        ↓
PHASE 3
Agent
        ↓
PHASE 4
Docker
        ↓
PHASE 5
RAG
        ↓
Embeddings + Vector DB
        ↓
PHASE 6
LangChain
        ↓
PHASE 7
LangGraph
        ↓
PHASE 8
Image
        ↓
PHASE 9
Audio + Video
        ↓
PHASE 10
Frontend
        ↓
PHASE 11
AI WMS Project
        ↓
PHASE 12
Training / Fine-tuning
        ↓
PHASE 13
Graph RAG
        ↓
Evaluation
        ↓
Observability
        ↓
Security
        ↓
MCP
        ↓
Multi-Agent
        ↓
A2A
```

---

# 25. Definition of Done

Sau mỗi bài, không chỉ xem video.

Bạn phải hoàn thành:

- [ ] Hiểu câu chuyện nghiệp vụ
- [ ] Hiểu vấn đề cần giải quyết
- [ ] Hiểu AI đóng vai trò gì
- [ ] Tự code lại
- [ ] Tự giải bài tập
- [ ] Test
- [ ] Commit Git
- [ ] Giải thích được code bằng lời của mình

---

# 26. Nguyên tắc quan trọng nhất

Bạn **không cần học thuộc**:

> Agent là gì?

> RAG là gì?

> Embedding là gì?

> Tool Calling là gì?

Thay vào đó phải hiểu:

> **"Trong câu chuyện này, tại sao tôi cần nó?"**

Ví dụ:

### User hỏi:

> "A100 còn bao nhiêu?"

Cần dữ liệu thật.

→ **Tool Calling**

### User hỏi:

> "Hàng tồn quá 90 ngày xử lý thế nào?"

Thông tin nằm trong tài liệu.

→ **RAG**

### User hỏi:

> "Kiểm tra tồn → phân tích → nếu thấp thì cảnh báo → tạo báo cáo."

Có nhiều bước.

→ **Agent / LangGraph**

### User hỏi:

> "A100 liên quan đến những đơn hàng nào và tại sao tồn giảm?"

Cần tìm quan hệ giữa entity.

→ **Graph RAG**

---

# 27. Mục tiêu cuối cùng

Sau khi hoàn thành lộ trình, bạn không chỉ biết gọi một LLM API.

Bạn sẽ có khả năng nhìn một bài toán backend như:

```text
User
 ↓
Business requirement
 ↓
AI có cần tham gia không?
 ↓
LLM?
 ↓
Tool?
 ↓
RAG?
 ↓
Agent?
 ↓
Workflow?
 ↓
Database?
 ↓
Evaluation?
 ↓
Security?
 ↓
Production?
```

và tự thiết kế được một **AI feature thực tế cho hệ thống Java/Spring WMS của mình**.

---

## 🚀 Bài đầu tiên chúng ta sẽ thực sự code

**Bài 0.1:**

> **"Anh Nam là nhân viên kho. Anh cần kiểm tra SKU A100 còn bao nhiêu tại kho Hà Nội. Hãy xây một Python/FastAPI service để nhận SKU và trả về số lượng tồn."**

Chưa có AI.

Chưa có LangChain.

Chưa có RAG.

Chưa có Agent.

**Chỉ xây viên gạch đầu tiên.**

Sau khi viên gạch này chạy được, chúng ta mới đưa AI vào và từng bước biến nó thành **AI WMS Copilot**.
