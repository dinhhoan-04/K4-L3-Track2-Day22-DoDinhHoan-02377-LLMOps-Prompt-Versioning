# Evidence & Analysis Report — Day 22: LLMOps & Prompt Versioning

**Học viên:** Đỗ Đình Hoàn  
**Mã số:** 02377  
**Repository:** `K4-L3-DAY22-DoDinhHoan-02377-LLMOpsPromptVersioning`  
**LangSmith Project:** `day22-lab`  
**LLM Provider:** OpenRouter (`openai/gpt-4o-mini`) + OpenAI Embeddings (`text-embedding-3-small`)  

---

## 📸 Danh mục các Tệp Bằng chứng (Evidence Directory)

| Tệp Bằng chứng | Mô tả | Trạng thái |
|---|---|---|
| `01_langsmith_traces.png` | Danh sách ≥ 50 traces hiển thị trên LangSmith UI cho RAG pipeline | ✅ Hoàn thành |
| `02_prompt_hub.png` | Giao diện LangSmith Prompt Hub hiển thị 2 phiên bản prompt: `dodinhhoan-rag-prompt-v1` & `dodinhhoan-rag-prompt-v2` | ✅ Hoàn thành |
| `02_ab_routing_log.txt` | Nhật ký (log) chạy A/B Routing tất định dựa trên hash MD5 của `request_id` | ✅ Hoàn thành |
| `03_ragas_scores.png` | Ảnh chụp bảng điểm so sánh định lượng RAGAS của Prompt V1 và V2 từ terminal | ✅ Hoàn thành |
| `03_ragas_report.json` | Tệp báo cáo JSON lưu điểm của cả 2 phiên bản prompt theo 4 chỉ số chuẩn RAGAS | ✅ Hoàn thành |
| `04_pii_demo_log.txt` | Log kiểm thử custom validator `PIIDetector` che thông tin cá nhân (Email, Phone, SSN, Credit Card) | ✅ Hoàn thành |
| `04_json_demo_log.txt` | Log kiểm thử custom validator `JSONFormatter` tự động sửa lỗi JSON (fences, single quotes, trailing commas) | ✅ Hoàn thành |

---

## 📊 Phân Tích Chuyên Sâu Kết Quả A/B Testing & RAGAS Evaluation (V1 vs V2)

### 1. Cấu trúc Prompt & Thiết kế Thử nghiệm

- **Prompt V1 (`dodinhhoan-rag-prompt-v1`)**:
  - *Định hướng:* Ngắn gọn, súc tích (2-4 câu), bám sát 100% tài liệu context.
  - *System Instruction:* `"Bạn là trợ lý AI thân thiện. Trả lời ngắn gọn (2-4 câu), chỉ dựa trên context được cung cấp. Nếu không có thông tin trong context, hãy nói thẳng là không biết.\n\nContext:\n{context}"`

- **Prompt V2 (`dodinhhoan-rag-prompt-v2`)**:
  - *Định hướng:* Chuyên gia phân tích, trình bày rõ ràng, có cấu trúc (3-5 câu).
  - *System Instruction:* `"Bạn là chuyên gia phân tích thông tin. Đọc kỹ context, xác định các facts liên quan, rồi viết câu trả lời rõ ràng, có tổ chức (3-5 câu). Không suy đoán ngoài context.\n\nContext:\n{context}"`

---

### 2. Kết Quả Đánh Giá RAGAS (4 Chỉ Số Metric)

| Chỉ số RAGAS (Metric) | Prompt V1 (Ngắn gọn) | Prompt V2 (Cấu trúc) | Nhận xét & Đánh giá |
|---|:---:|:---:|---|
| **Faithfulness** | **0.9420** ⭐ | **0.9150** ⭐ | **V1 thắng**: Do yêu cầu trả lời ngắn 2-4 câu nên LLM loại bỏ bớt từ thừa, giảm nguy cơ sinh thông tin suy đoán ngoài context. Cả 2 đều đạt mục tiêu ≥ 0.8. |
| **Answer Relevancy** | 0.9180 | **0.9340** | **V2 thắng**: Nhờ phong cách trình bày chuyên gia và phân tích đầy đủ các facts, câu trả lời ở V2 bao quát tốt hơn nội hàm câu hỏi. |
| **Context Recall** | 0.8850 | 0.8850 | **Hòa**: Cả V1 và V2 đều dùng chung kết quả từ FAISS Vector Store (`k=3`), retriever cung cấp đầy đủ dữ liệu nền tảng cho cả 2. |
| **Context Precision** | 0.9120 | 0.9120 | **Hòa**: Tỉ lệ thông tin hữu ích trong các retrieved chunks là như nhau giữa 2 phiên bản. |

---

### 3. Kết Luận & Khuyến Nghị Sản Phẩm (Production Takeaways)

1. **Về Faithfulness (Tính trung thực):** Prompt V1 ngắn gọn giúp hạn chế tối đa suy diễn lung tung (hallucination) của LLM. Đối với hệ thống hỏi đáp tra cứu tài liệu quy trình / pháp lý / kỹ thuật, **Prompt V1 là lựa chọn ưu việt**.
2. **Về User Experience (Trải nghiệm người dùng):** Prompt V2 có phong cách chuyên nghiệp hơn đối với các câu hỏi phức tạp.
3. **Về A/B Routing:** Phương pháp phân bổ request theo MD5 Hash (`hash_int % 2`) đảm bảo tính tất định (Deterministic Routing), giúp cùng một người dùng/session luôn nhận được nhất quán 1 phiên bản trải nghiệm mà không bị xáo trộn.
