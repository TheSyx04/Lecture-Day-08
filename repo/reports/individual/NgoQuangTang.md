# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Documentation Owner  
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

## 4. Phân tích một câu hỏi trong scorecard (195 từ)

**Câu hỏi:** "Khách hàng có thể yêu cầu hoàn tiền trong bao lâu?"

**Phân tích:**

| Metric | Baseline (Dense) | Variant (Hybrid+Rerank) | Ghi chú |
|--------|------------------|------------------------|--------|
| **Faithfulness** | 4/5 | 5/5 | Baseline retrieve đúng chunk refund_policy, nhưng rank thứ 5 (score 0.72). LLM chỉ lấy top-3 → miss. Variant: Hybrid + BM25 catch keyword "hoàn tiền" ở top-10, rerank đưa lên #2 → full context |
| **Context Recall** | 3/5 | 5/5 | Baseline only retrieve "Time Limit: 30 days". Variant also get "Proof of Payment", "Refund Process" → complete picture |
| **Answer Relevance** | 4/5 | 5/5 | Cả 2 trả lời đúng "30 ngày", nhưng baseline thiếu context "Từ ngày mua" → ambiguous. Variant rõ ràng "From purchase date" |
| **Completeness** | 3/5 | 5/5 | Baseline = fact only. Variant = fact + business logic (why 30 days, exceptions) |

**Root cause:** Câu hỏi này mix **exact keyword ("hoàn tiền")** + **semantic understanding ("refund timeframe")**. Dense-only embedding struggle với keyword pure. Hybrid + BM25 = solution.

**Impact:** Nếu company policy có nhiều câu hỏi keyword-heavy (tên chuyên ngành, mã lỗi, thủ ngữ nội bộ), Hybrid MUST > Dense only. Variant selection justified.

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (70 từ)

1. **Implement metadata hierarchy:** Thay vì flat `source | section`, dùng tree path `policy > refund > time_limit`. Query "Hoàn tiền" → auto-filter "policy.refund.*" → precision +15% dự kiến.

2. **Version tracking in metadata:** Thêm `version: v4`, `last_updated: 2026-04-01`. Eval Owner có thể track "Policy v3 vs v4 accuracy delta" — chuẩn bị cho real-time document update workflow.

---

*File này lưu tại: `reports/individual/DocumentationOwner.md`*
