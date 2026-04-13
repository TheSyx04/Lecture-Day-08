# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Phạm Minh Khôi  
**Vai trò trong nhóm:** Eval Owner  
**Ngày nộp:** 13/04/2026  
**Độ dài yêu cầu:** 500–800 từ  

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

Trong dự án xây dựng hệ thống Trợ lý nội bộ lần này, tôi đảm nhận vai trò Eval Owner, tập trung chính vào Sprint 4 và các giai đoạn đánh giá của Sprint 3. Nhiệm vụ cốt lõi của tôi là thiết lập một quy trình đo lường chất lượng khách quan để xác định cấu hình nào tối ưu nhất cho bài toán hỗ trợ kỹ thuật.  

Tôi đã trực tiếp triển khai logic LLM-as-Judge trong file `eval.py`, sử dụng mô hình gpt-4o-mini để tự động hóa việc chấm điểm 4 chỉ số quan trọng: Faithfulness (Độ trung thực), Relevance (Sự liên quan), Context Recall (Độ phủ ngữ cảnh) và Completeness (Sự đầy đủ). Ngoài ra, tôi thực hiện việc chạy thử nghiệm A/B giữa Baseline (Dense Retrieval) và Variant (Hybrid Retrieval) để so sánh hiệu năng. Công việc của tôi đóng vai trò là “mắt xích” cuối cùng, cung cấp dữ liệu định lượng giúp nhóm đưa ra quyết định tuning hệ thống dựa trên các bảng Scorecard.

---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

Sau khi trực tiếp tham gia vào vòng lặp đánh giá (Evaluation Loop), tôi hiểu rõ hơn về khái niệm “Grounded Evaluation” thay vì chỉ đánh giá cảm tính. Trước đây, tôi nghĩ rằng chỉ cần LLM trả lời nghe “có vẻ đúng” là đủ, nhưng qua bài lab, tôi nhận ra sự cần thiết của việc tách biệt giữa khả năng truy xuất (Retrieval) và khả năng tổng hợp (Generation).  

Đặc biệt, tôi hiểu sâu hơn về mô hình LLM-as-Judge. Việc sử dụng một LLM mạnh với prompt được thiết kế chặt chẽ để chấm điểm giúp loại bỏ sự chủ quan của con người và cho phép đánh giá trên quy mô lớn. Ngoài ra, tôi nhận ra rằng điểm số tuyệt đối không quan trọng bằng việc phân tích các chỉ số Delta. Sự thay đổi của các chỉ số này phản ánh rõ tác động của các kỹ thuật như Hybrid Retrieval, giúp định hướng tối ưu pipeline một cách chính xác hơn.

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

Điều khiến tôi ngạc nhiên nhất là kết quả của cấu hình Variant (Hybrid Retrieval). Trái với kỳ vọng rằng Hybrid sẽ vượt trội hoàn toàn, số liệu cho thấy điểm Completeness giảm từ 2.80 xuống 2.50 so với Baseline. Điều này cho thấy việc kết hợp BM25 có thể gây nhiễu nếu không tinh chỉnh trọng số phù hợp.  

Khó khăn lớn nhất tôi gặp phải là xung đột mã nguồn (Git Conflict) khi làm việc nhóm. Khi push kết quả eval, hệ thống từ chối do code trên remote mới hơn. Tôi đã phải sử dụng `git pull --rebase` để đồng bộ thay đổi mà không làm hỏng log kết quả. Ngoài ra, lỗi indentation khi copy code từ các nguồn khác cũng gây mất thời gian debug, cho thấy tầm quan trọng của việc chuẩn hóa code.

---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)

**Câu hỏi:** [q01] SLA xử lý ticket P1 là bao lâu?  

**Phân tích:**  

Đây là một ví dụ điển hình cho sự khác biệt giữa hai chiến lược truy xuất.  

Về điểm số: Baseline (Dense) đạt 5/5/5/4, trong khi Variant (Hybrid) đạt 5/5/5/3. Cả hai đều chính xác về mặt thông tin (Faithfulness = 5), nhưng Variant bị giảm điểm Completeness.  

Nguyên nhân nằm ở bước Retrieval. Ở Baseline, semantic search giúp lấy được chunk chứa đầy đủ thông tin: “15 phút phản hồi ban đầu và 4 giờ xử lý”. Trong khi đó, Variant với BM25 lại ưu tiên các chunk có mật độ từ khóa cao như “SLA”, “P1”, “4 giờ”, dẫn đến việc chọn một đoạn tóm tắt ngắn chỉ chứa “4 giờ”.  

Do thuật toán RRF đẩy chunk này lên đầu, LLM chỉ nhận được thông tin không đầy đủ, làm mất phần “15 phút phản hồi ban đầu”.  

Kết luận: Hybrid Retrieval có thể cải thiện khả năng match keyword, nhưng dễ làm mất ngữ cảnh nếu không có cơ chế reranking tốt.

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

Nếu có thêm thời gian, tôi sẽ tối ưu bước reranking. Kết quả eval cho thấy Hybrid Retrieval bị giảm Completeness do chọn các chunk ngắn thiếu thông tin. Tôi sẽ thử tích hợp một mô hình Cross-Encoder để chấm lại mức độ liên quan giữa Query và Context sau bước retrieval. Điều này giúp ưu tiên các chunk đầy đủ ngữ nghĩa, khắc phục vấn đề mất thông tin do BM25.

---
*
