# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Ngô Quang Tăng 
**Vai trò trong nhóm:** Documentation Owner  
**Ngày nộp:** 13/04/2026  
**Độ dài:** 720 từ

---

## 1. Tôi đã làm gì trong lab này? (115 từ)

Vai trò của tôi là **Documentation Owner** — người phụ trách lập kế hoạch kiến trúc, đồng bộ hóa các quyết định từ 4 sprint, và đảm bảo tất cả thành viên có reference document rõ ràng.

**Cụ thể, tôi đã:**
- **Sprint 1-4:** Viết `architecture.md` toàn diện, từ tổng quan kiến trúc → chunking decisions → retrieval strategies → LLM config → troubleshooting guide
- **Mapping role-specific decisions:** Ghi rõ Indexing Owner chọn chunk size 400 tokens, Retrieval Owner chọn hybrid Dense+BM25, Generation Owner chọn gpt-4o-mini, Eval Owner chọn 4 metrics
- **End-to-end example:** Tạo workflow chi tiết "SLA xử lý ticket P1" để mọi người hiểu pipeline hoạt động như thế nào
- **Bridge giữa sprints:** Từng tuần cập nhật `tuning-log.md` + architecture.md template khi sprint kết thúc
- Công việc tôi là **enabler** — giúp các owner khác có foundation rõ ràng để implement

---

## 2. Điều tôi hiểu rõ hơn sau lab này (125 từ)

**Metadata design cho RAG retrieval:** Ban đầu tôi nghĩ metadata chỉ cần `source` thôi. Nhưng qua thực tế, **metadata là DNA của pipeline tốt**.

Ví dụ corpus policy của chúng ta:
- `source + section` → LLM cite chính xác "According to [refund_policy | Time Limit]"
- `effective_date` → Retrieval Owner có thể filter P1 SLA **2026** vs cũ, tránh stale data
- `department` → Eval Owner sau này có thể measure "Accuracy by Department"
- `access` → Nếu scale to production, có thể enforce "HR staff chỉ thấy HR policy"

Metadata không phải overhead — nó là **compass định hướng** cho cả retrieval + eval logic. Build kỹ từ đầu = debug ít sau.

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (130 từ)

**Khó khăn lớn nhất: Lập kế hoạch architecture cho ngôn ngữ VN không rõ ràng.**

Bộ câu hỏi test ( `test_questions.json`) có mix Tiếng Việt + Tiếng Anh. Khi viết prompt template, tôi phải quyết định:
- Prompt chính: Tiếng Việt hay Tiếng Anh?
- Retrieved chunks: Ngôn ngữ gốc (mixed)?
- LLM output: Follow ngôn ngữ query?

Ban đầu tôi chọn "chủ yếu Tiếng Việt" vì đây là hệ thống nội bộ VN. Nhưng downstream Retrieval Owner báo: "OpenAI `text-embedding-3-small` mạnh hơn trên tiếng Anh pure".

**Giải pháp:** Tôi cập nhật architecture doc ghi rõ: prompt template **tiếng Việt**, chunks **mixed** (policy cũ = Anh, policy mới = Việt), LLM output **follow query ngôn ngữ**. Trade-off rõ ràng giúp team alignment tránh conflict sau.

---

## 4. Phân tích một câu hỏi trong scorecard (190 từ)

**Câu hỏi (q02):** "Khách hàng có thể yêu cầu hoàn tiền trong bao nhiêu ngày?"

**Actual Scores từ Evaluation:**

| Metric | Baseline | Variant | Expected | Result |
|--------|----------|---------|----------|--------|
| **Faithfulness** | 4/5 | 5/5 | 4/5 ✓ | Baseline: "7 ngày làm việc" ✓ (correct). Variant: same, cũng 5/5 |
| **Relevance** | 5/5 | 5/5 | 5/5 ✓ | Cả 2 trả lời trực tiếp câu hỏi ✓ |
| **Context Recall** | 5/5 | 5/5 | 5/5 ✓ | Cả 2 retrieve đủ context ✓ |
| **Completeness** | 5/5 | 5/5 | 5/5 ✓ | Cả 2 cover full scope (time frame từ confirmation date) ✓ |

**Đánh giá:** Đây là câu hỏi **"easy" — cả Baseline lẫn Variant đều perfect 4.75/5**. Lý do:
- Query "hoàn tiền" là **exact keyword** trong policy
- Dense embedding catch tốt vì keyword rõ ràng
- Hybrid không bring value vì Dense đã rank chính xác (#1)

**Key insight:** Hybrid+Rerank chỉ giúp khi Dense rank **sai** (recall thứ 5-10). Câu q02 là counterexample. Những câu fail (q04, q07, q09, q10) không retrieve do **out-of-context**, không phải ranking problem.

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (70 từ)

1. **Implement metadata hierarchy:** Thay vì flat `source | section`, dùng tree path `policy > refund > time_limit`. Query "Hoàn tiền" → auto-filter "policy.refund.*" → precision +15% dự kiến.

2. **Version tracking in metadata:** Thêm `version: v4`, `last_updated: 2026-04-01`. Eval Owner có thể track "Policy v3 vs v4 accuracy delta" — chuẩn bị cho real-time document update workflow.

---

## 6. Tóm tắt Eval Results — So sánh Baseline vs Variant (100 từ)

**Overall Performance:**
- **Baseline (Dense):** 3.73/5 average
  - Faithfulness: 3.30, Relevance: 3.80, Context Recall: 5.00, Completeness: 2.80
- **Variant (Hybrid+Rerank):** 3.68/5 average
  - Faithful: 3.40, Relevance: 3.80, Context Recall: 5.00, Completeness: 2.50
- **Result:** Variant **KHÔNG cải thiện** (-1.3%) do corpus quá nhỏ

**Pass/Fail Breakdown:**
- ✅ **6/10 pass:** q01, q02, q03, q05, q06, q08 (well-covered policy questions)
- ❌ **4/10 fail:** q04, q07, q09, q10 (out-of-context — không có matching policy)

**Lesson Learned:** Hybrid+Rerank không phải silver bullet. Chỉ hiệu quả khi:
- Corpus > 1000 chunks
- Có ranking errors (relevant doc rank thứ 5+ thay vì #1)
- Hiện tại: corpus đủ nhỏ, dense đã đủ tốt

---

*File báo cáo cá nhân: `reports/individual/NgoQuangTang.md`*
*Corresponding architecture doc: `docs/architecture.md`*
