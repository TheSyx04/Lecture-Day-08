# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Trần Sỹ Minh Quân  
**Vai trò trong nhóm:** Tech Lead  
**Ngày nộp:** 13/04/2026  
**Độ dài yêu cầu:** 500–800 từ

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

Trong vai trò Tech Lead, tôi tập trung ở Sprint 1 và Sprint 2 để giữ nhịp kỹ thuật và kết nối các phần việc thành một pipeline chạy được end-to-end. Ở `index.py`, tôi sửa logic preprocess phần header để xử lý đúng tiếng Việt có dấu, nhờ đó metadata không bị parse sai từ đầu vào. Ở `rag_answer.py`, tôi implement `retrieve_dense()` theo đúng nguyên tắc dùng cùng embedding model với indexing, rồi chuẩn hóa output thành danh sách chunk có score để các bước sau dùng trực tiếp. Tôi cũng thiết kế grounded prompt với rule abstain rõ ràng và implement `call_llm()` để output ổn định cho phần đánh giá. Ngoài code, tôi phối hợp với Retrieval Owner để bảo đảm dense/hybrid dùng chung data contract, và với Eval Owner để `eval.py` gọi pipeline theo config baseline/variant không bị mismatch tham số.

---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

Điều tôi hiểu rõ hơn là mối liên hệ giữa retrieval và generation trong RAG không phải hai bước độc lập, mà là một chuỗi ràng buộc. Nếu retrieval trả về chunk chưa đủ sát câu hỏi, prompt càng “strict” thì model càng dễ abstain; prompt càng “thoáng” thì model dễ suy diễn ngoài context. Nghĩa là chất lượng câu trả lời phụ thuộc vào chất lượng evidence và mức độ ép grounding.

Ngoài ra, tôi còn biết thêm là evaluation loop giúp nhìn đúng lỗi gốc. Trước đây khi thấy câu trả lời sai, tôi thường nghĩ là do LLM. Nhưng qua scorecard 4 metric, tôi thấy có nhiều trường hợp Context Recall là 5/5 mà Completeness thấp, tức là source đúng đã được tìm thấy nhưng chunk chọn vào prompt chưa trúng ý hỏi.

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

Điều làm tôi ngạc nhiên là baseline dense đã cho Context Recall trung bình 5/5, nhưng vẫn có nhiều câu fail nặng ở Completeness. Kỳ vọng ban đầu của tôi là chỉ cần retrieve đúng source thì model sẽ trả lời đúng phần lớn câu hỏi. Tuy nhiên nó lại không như vậy

Lỗi mất thời gian debug nhất là các câu pipeline trả về “Không tìm thấy thông tin...” dù tài liệu thật ra có nội dung liên quan. Ban đầu tôi giả thuyết lỗi nằm ở prompt quá chặt. Sau khi kiểm tra chunk và kết quả retrieve chi tiết, tôi thấy gốc vấn đề thường là chất lượng evidence đưa vào top-3: chunk đúng nguồn nhưng không chứa mệnh đề quyết định để trả lời chính xác. Từ đó tôi rút ra bài học rằng Tech Lead cần thiết kế log debug theo từng tầng ngay từ đầu, thay vì chỉ nhìn answer cuối.

---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)

**Câu hỏi:** q04 - “Sản phẩm kỹ thuật số có được hoàn tiền không?”

**Phân tích:**

Ở baseline (dense), hệ thống trả lời abstain: “Không tìm thấy thông tin này...”, trong khi expected answer là “Không” và nêu rõ digital goods là ngoại lệ không hoàn tiền. Điểm cho câu này rất thấp: Faithfulness 1/5, Relevance 2/5, Completeness 1/5; riêng Context Recall vẫn 5/5. Điều này cho thấy pipeline chạm đúng nguồn tài liệu nhưng chưa đưa được bằng chứng đúng trọng tâm vào phần context mà LLM dùng để sinh câu trả lời.

Tôi đánh giá lỗi chính nằm ở retrieval/select hơn là generation thuần túy. Prompt strict của tôi chỉ cho phép trả lời khi context “clearly stated”; nếu top-3 không có câu phủ định về digital products thì model bắt buộc abstain. Tức là generation đang làm đúng rule, nhưng rule này phơi bày điểm yếu ở bước chọn evidence.

Variant hybrid không cải thiện câu này (điểm gần như giữ nguyên), vì vấn đề không phải thiếu keyword match mà là chunk quan trọng chưa được ưu tiên trong tập final chunks. Đây là case điển hình cho thấy tune retrieval mode thôi chưa đủ; cần tune cách select/rerank theo intent phủ định và exception clauses.

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

Tôi sẽ thử 2 cải tiến cụ thể. Thứ nhất, thêm bước rerank nhẹ cho top-10 trước khi cắt top-3 để giảm trường hợp “đúng source nhưng sai evidence”, vì scorecard cho thấy Context Recall cao nhưng Completeness thấp ở nhiều câu. Thứ hai, bổ sung query expansion cho alias/legacy term (ví dụ “Approval Matrix”), vì q07 cho thấy pipeline chưa nối được tên tài liệu cũ và mới dù đã retrieve gần đúng domain. Hai thay đổi này bám trực tiếp vào lỗi đã đo được trong eval.

---

