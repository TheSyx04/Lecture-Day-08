# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Trần Sỹ Minh Quân  
**Vai trò trong nhóm:** Tech Lead  
**Ngày nộp:** 13/04/2026  
**Độ dài yêu cầu:** 500–800 từ

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

Trong vai trò Tech Lead, tôi tập trung vào Sprint 1 và Sprint 2 để bảo đảm pipeline chạy ổn định. Ở `index.py`, tôi chỉnh preprocess header để parse đúng metadata từ tài liệu tiếng Việt có dấu, tránh lệch từ bước indexing. Ở `rag_answer.py`, tôi triển khai `retrieve_dense()` theo nguyên tắc dùng cùng embedding model với bước index, rồi chuẩn hóa đầu ra chunk có score cho các bước select và eval. Tôi cũng xây dựng grounded prompt với rule abstain rõ ràng và triển khai `call_llm()` theo hướng deterministic để output ổn định khi chấm điểm..

---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

Tôi hiểu rằng retrieval và generation trong RAG là một chuỗi phụ thuộc, không thể tối ưu tách rời. Nếu retrieval chưa đưa đúng mệnh đề quyết định vào context, prompt càng strict thì model càng abstain. Ngược lại, prompt lỏng lại tăng nguy cơ suy diễn ngoài tài liệu. Vì vậy chất lượng answer thực tế là kết quả của cả evidence quality lẫn mức ép grounding.

Tôi cũng thấy rõ giá trị của evaluation loop. Sau khi xem scorecard 4 metric, tôi nhận ra nhiều câu có Context Recall 5/5 nhưng Completeness thấp. Nghĩa là hệ thống tìm đúng nguồn, nhưng top chunk đưa vào prompt chưa đúng trọng tâm câu hỏi.

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

Điều làm tôi ngạc nhiên là baseline dense đạt Context Recall trung bình 5/5 nhưng vẫn fail nặng ở Completeness ở nhiều câu. Ban đầu tôi nghĩ chỉ cần retrieve đúng source thì model sẽ trả lời đúng phần lớn truy vấn, nhưng thực tế không đơn giản như vậy.

Khó khăn tốn thời gian nhất là các trường hợp pipeline trả về “Không tìm thấy thông tin...” dù tài liệu có nội dung liên quan. Lúc đầu tôi nghi prompt quá chặt, nhưng khi kiểm tra sâu theo từng tầng, vấn đề chính thường nằm ở chất lượng evidence top-3: đúng nguồn nhưng thiếu câu then chốt để kết luận. Bài học tôi rút ra là cần thiết kế log debug từ đầu cho từng lớp indexing, retrieval, select và generation, thay vì chỉ nhìn output cuối cùng.

---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)

**Câu hỏi:** q04 - “Sản phẩm kỹ thuật số có được hoàn tiền không?”

**Phân tích:**

Ở baseline dense, hệ thống trả lời abstain “Không tìm thấy thông tin...”, trong khi expected answer là “Không” và phải nêu rõ digital goods là ngoại lệ không hoàn tiền. Điểm câu này rất thấp: Faithfulness 1/5, Relevance 2/5, Completeness 1/5; riêng Context Recall vẫn 5/5. Điều này cho thấy pipeline đã chạm đúng nguồn nhưng chưa chọn được bằng chứng quyết định để LLM kết luận đúng.

Theo tôi, lỗi gốc nằm ở retrieval/select hơn là generation thuần túy. Prompt strict chỉ cho phép trả lời khi thông tin xuất hiện rõ trong context. Nếu top-3 không chứa câu phủ định về digital products, model buộc phải abstain để tránh bịa. Nói cách khác, generation đang tuân thủ rule, nhưng rule này làm lộ điểm yếu của bước chọn evidence.

Variant hybrid cũng không cải thiện q04 vì vấn đề không nằm ở keyword matching mà ở thứ tự ưu tiên chunk cuối cùng. Case này cho thấy chỉ đổi retrieval mode là chưa đủ; cần thêm cơ chế select/rerank tập trung vào mệnh đề ngoại lệ và câu phủ định trong policy.

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

Nếu có thêm thời gian, tôi sẽ thử hai cải tiến. Thứ nhất, thêm rerank nhẹ cho top-10 trước khi cắt top-3 để giảm tình trạng “đúng source nhưng sai evidence”, vì scorecard cho thấy Recall cao nhưng Completeness thấp. Thứ hai, bổ sung query expansion cho alias/legacy term (ví dụ “Approval Matrix”) để nối tên tài liệu cũ và mới tốt hơn. Cả hai thay đổi đều bám trực tiếp vào lỗi đã đo trong eval.

---

