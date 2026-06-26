# Báo cáo Phân tích Đánh giá A/B Prompt (Day 22 Lab)

## 1. Tổng quan
Trong bài lab này, hai phiên bản System Prompt đã được triển khai và đánh giá định lượng bằng framework **RAGAS** trên bộ dataset 50 câu hỏi chuẩn (Ground Truth):
- **Prompt V1 (`antigravity-rag-prompt-v1`)**: Định hướng trợ lý trả lời ngắn gọn, trực diện (2-4 câu).
- **Prompt V2 (`antigravity-rag-prompt-v2`)**: Định hướng chuyên gia trả lời chi tiết, có tổ chức và cấu trúc (3-5 câu).

---

## 2. Kết quả Đánh giá RAGAS (50 mẫu)

| Chỉ số (Metric) | Prompt V1 (Ngắn gọn) | Prompt V2 (Cấu trúc) | Nhận xét |
| :--- | :---: | :---: | :--- |
| **Faithfulness** | **~0.96** | ~0.94 | V1 cao hơn |
| **Answer Relevancy** | **~0.90** | ~0.87 | V1 cao hơn |
| **Context Recall** | 1.00 | 1.00 | Ngang nhau |
| **Context Precision** | 1.00 | 1.00 | Ngang nhau |

*(Lưu ý: Điểm số cụ thể có thể dao động nhẹ giữa các lần chạy do bản chất sinh xác suất của mô hình LLM).*

---

## 3. Phân tích & Nhận xét

1. **Tại sao Prompt V1 đạt chỉ số Faithfulness cao hơn?**
   - Chỉ số **Faithfulness** đo lường tỷ lệ các "claims" (khẳng định) trong câu trả lời có thể được suy ra trực tiếp từ Context tra cứu được. 
   - Vì Prompt V1 yêu cầu trả lời cực kỳ ngắn gọn (2-4 câu), mô hình LLM có xu hướng cô đọng chính xác nguyên văn các ý chính trong retrieved context, không mở rộng hay diễn giải thêm. Điều này giúp giảm thiểu tối đa hiện tượng "hallucination" (suy diễn ngoài tài liệu).
   - Ngược lại, Prompt V2 khuyến khích trả lời kiểu "chuyên gia có tổ chức 3-5 câu", đôi khi khiến LLM tự động bổ sung từ ngữ liên kết hoặc kiến thức nền tảng sẵn có để câu văn mượt mà hơn, dẫn đến một vài claims nhỏ không nằm trong tài liệu gốc.

2. **Tại sao Prompt V1 đạt Answer Relevancy tốt hơn?**
   - **Answer Relevancy** đánh giá độ liên quan trực tiếp giữa câu trả lời và câu hỏi gốc (thông qua việc sinh ngược câu hỏi giả lập). Các câu trả lời ngắn, đi thẳng vào trọng tâm của V1 giúp vectơ embedding của câu trả lời khớp sát với vectơ câu hỏi hơn so với câu trả lời dài dòng, nhiều thông tin phụ của V2.

3. **Context Recall & Context Precision đạt tuyệt đối (1.0)**:
   - Điều này chứng tỏ **FAISS Retriever** kết hợp chiến lược chia chunk (`chunk_size=900, chunk_overlap=50`) đã hoạt động cực kỳ hiệu quả, luôn truy xuất chính xác và đầy đủ các đoạn văn bản chứa đáp án chuẩn cho toàn bộ 50 câu hỏi.

## 4. Kết luận
Cả hai phiên bản prompt đều vượt mục tiêu đề ra (**Faithfulness ≥ 0.8**). Trong môi trường thực tế:
- Nên ưu tiên **Prompt V1** cho các bot hỏi đáp tài liệu nội bộ/pháp lý cần độ chính xác tuyệt đối.
- Nên ưu tiên **Prompt V2** cho các trợ lý hỗ trợ khách hàng cần câu trả lời chi tiết, mượt mà và thân thiện hơn.
