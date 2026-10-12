# Phân tích lỗi Phase A

Nhóm: Chu Văn Nhân

Người phụ trách: Chu Văn Nhân - MSSV: 2A202602668

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T09:48:47.808304+00:00.
Generation mode/fallback và điều kiện chạy: Chạy `--live` toàn bộ 50 câu hỏi; generation mode 100% `llm` (không có `extractive_fallback`); pipeline gồm Dense Qdrant + BM25 Hybrid Search (RRF k=60), Cross-Encoder Reranker (`ms-marco-MiniLM-L-6-v2`, top-3), và LLM `gpt-oss:120b-cloud`.
Models generation/evaluator: Generation: `gpt-oss:120b-cloud`. Evaluator: `gpt-oss:120b-cloud`, `sentence-transformers/all-MiniLM-L6-v2` (từ m4_eval metadata).

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.8650 | 0.5062 | 0.9300 |
| answer_relevancy | 0.5167 | 0.3896 | 0.5492 |
| context_precision | 0.9333 | 0.7333 | 0.9333 |
| context_recall | 0.8250 | 0.6792 | 0.7500 |
| avg_score | 0.7850 | 0.5771 | 0.7906 |

## Bottom 10 và nguyên nhân

| Rank | ID | Question | Distribution | Avg Score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 40 | Nhân viên cấp Manager có thời hạn thử việc 90 ngày, đang trong tháng thứ 3, phát hiện vi phạm bảo mật. Họ nên và không nên làm gì theo chính sách? | multi_hop | 0.2147 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 2 | 33 | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024 là bao nhiêu? | multi_hop | 0.3164 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 3 | 38 | Nhân viên đi công tác nước ngoài 4 ngày tại thành phố lớn, ở khách sạn 200 USD/đêm (3 đêm). Công ty thanh toán bao nhiêu tiền khách sạn? | multi_hop | 0.4337 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 4 | 39 | So sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0 về độ dài tối thiểu, thời hạn đổi và MFA. | multi_hop | 0.4424 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 5 | 23 | Nhân viên được tài trợ khóa học 25 triệu, nghỉ việc sau 8 tháng hoàn thành khóa học. Phải hoàn trả bao nhiêu? | multi_hop | 0.4583 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 6 | 49 | Theo chính sách nghỉ phép cũ (v2023), nhân viên được nghỉ bao nhiêu ngày? Còn chính sách nào đang có hiệu lực hiện tại? | adversarial | 0.4784 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 7 | 35 | Nhân viên Junior P1 có lương cơ bản 12 triệu vừa vào thử việc: nhận lương bao nhiêu và phụ cấp gì tháng đầu tiên? | multi_hop | 0.5178 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 8 | 29 | Nhân viên tạm ứng 8 triệu, chưa thanh toán sau 30 ngày (quá hạn 15 ngày). Ai phê duyệt khoản này và phí phạt là bao nhiêu? | multi_hop | 0.5222 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 9 | 30 | So sánh quyền lợi bảo hiểm giữa nhân viên thử việc và nhân viên chính thức. | multi_hop | 0.5333 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 10 | 36 | Kết quả đánh giá hiệu suất tháng 6 và tháng 12 được dùng để làm gì khác nhau? | multi_hop | 0.5356 | answer_relevancy | Answer doesn't match question | Improve prompt template |

### Đối chiếu và bằng chứng nguyên nhân lỗi

1. **Câu 40 (Rank 1, ID 40 - multi_hop, avg=0.2147, worst=faithfulness 0.0)**:
   - *Question*: Hỏi về nhân viên Manager thử việc 90 ngày phát hiện vi phạm bảo mật cần làm gì và không được làm gì.
   - *Đối chiếu*: Cần kết hợp hai tài liệu: chính sách thử việc và quy trình ứng phó sự cố bảo mật (CSIRP). Retrieval chỉ lấy được chunk chung về thử việc và định nghĩa vi phạm bảo mật, thiếu chunk quy định chi tiết về việc không tự ý can thiệp log máy chủ. LLM tự suy diễn các hành động kỹ thuật không có trong văn bản, dẫn đến `faithfulness = 0.0`.
   - *Lỗi*: Multi-hop retrieval thiếu thông tin liên văn bản, LLM hallucination.

2. **Câu 33 (Rank 2, ID 33 - multi_hop, avg=0.3164, worst=context_precision 0.0)**:
   - *Question*: Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024.
   - *Đối chiếu*: Cần 3 thông tin: phụ cấp Manager (3 triệu), phụ cấp thâm niên >= 10 năm (2 triệu), phép năm v2024 (12 ngày cơ bản + 2 ngày thâm niên = 14 ngày). Retrieval kéo về chunk chứa cả bảng lương cũ v2022 và các chính sách khen thưởng không liên quan khiến `context_precision` bằng 0.0. LLM tổng hợp bị lệch phép năm.
   - *Lỗi*: Retrieval precision thấp do tài liệu chứa nhiều phiên bản không được lọc metadata theo năm.

3. **Câu 38 (Rank 3, ID 38 - multi_hop, avg=0.4337, worst=faithfulness 0.0)**:
   - *Question*: Công tác nước ngoài 4 ngày thành phố lớn, khách sạn 200 USD/đêm (hạn mức 180 USD/đêm). Công ty thanh toán bao nhiêu tiền?
   - *Đối chiếu*: Reference quy định công ty thanh toán tối đa theo hạn mức 180 USD × 3 đêm = 540 USD, phần vượt 20 USD/đêm nhân viên tự chi trả trừ khi có phê duyệt trước của Giám đốc. LLM trả lời là công ty từ chối toàn bộ hoặc thanh toán toàn bộ 600 USD do nhầm lẫn điều kiện miễn trừ.
   - *Lỗi*: Logic suy luận toán học và điều kiện ngoại lệ của LLM bị sai lệch so với văn bản gốc.

4. **Câu 49 (Rank 6, ID 49 - adversarial, avg=0.4784, worst=answer_relevancy 0.4286)**:
   - *Question*: Hỏi về chính sách nghỉ phép cũ v2023 và chính sách nào đang có hiệu lực.
   - *Đối chiếu*: LLM nhận diện được v2023 đã hết hiệu lực và dẫn chiếu v2024. Tuy nhiên do LLM đưa ra câu trả lời chứa nhiều lời cảnh báo và phủ định dài dòng, RAGAS đánh giá `answer_relevancy` thấp vì embedding của câu trả lời phân tán khỏi vector câu hỏi.

## Matrix và nhóm yếu nhất

| Metric | factual | multi_hop | adversarial | Tổng |
|---|---|---|---|---|
| faithfulness | 2 | 5 | 2 | 9 |
| answer_relevancy | 15 | 10 | 5 | 30 |
| context_precision | 1 | 4 | 0 | 5 |
| context_recall | 2 | 1 | 3 | 6 |

Nhóm yếu theo mean thấp nhất: `multi_hop`; mean: `0.5771`.
Worst metric xuất hiện nhiều nhất: `answer_relevancy`; số câu: `30` câu (trên 50 câu).

### Giải thích vì sao nhóm multi_hop yếu

Nhóm `multi_hop` đạt điểm trung bình thấp nhất (0.5771) so với `factual` (0.7850) và `adversarial` (0.7906). Có 9/10 câu trong Bottom 10 thuộc về nhóm `multi_hop`.
Nguyên nhân cốt lõi:
1. **Suy giảm chất lượng Retrieval**: Các câu hỏi multi_hop cần nối 2 đến 3 mẩu thông tin nằm rải rác ở các file markdown khác nhau (ví dụ: cấp bậc + thâm niên + phép năm). Bộ tìm kiếm hiện tại chỉ lấy top-3 chunks sau rerank; khi một câu hỏi cần nhiều hơn 2 chunks từ các nguồn khác nhau, xác suất thiếu ít nhất một chunk thiết yếu là rất lớn (`context_recall` multi_hop chỉ đạt 0.6792 so với 0.8250 của factual).
2. **Nhiễu chunk do thiếu metadata**: Retrieval kéo theo các chunk từ tài liệu phiên bản cũ (v1.0/v2022) hoặc chunk phụ cấp khác loại, kéo `context_precision` xuống 0.7333.
3. **Hallucination thứ cấp**: Khi context bị phân mảnh hoặc thiếu điều kiện logic, LLM cố gắng "điền vào chỗ trống" để hoàn thành câu trả lời, dẫn đến `faithfulness` của multi_hop giảm mạnh xuống 0.5062 (trong khi factual đạt 0.8650).

**Đề xuất khắc phục cụ thể:**
- *Retrieval*: Tăng số lượng candidate ban đầu trước khi rerank (tăng dense và BM25 top_k từ 5 lên 15) và nâng top-k đầu ra sau reranker từ 3 lên 5 đối với các câu hỏi phức hợp.
- *Query Decomposition*: Thêm bước phân tích câu hỏi: nếu phát hiện câu hỏi chứa nhiều thực thể / điều kiện (multi-hop), tự động phân rã thành các sub-queries độc lập để retrieval riêng trước khi hợp nhất context.
- *Prompt & Metadata Filter*: Bổ sung filter theo `metadata.version` để loại bỏ chunk phiên bản cũ; siết chặt prompt cấm tự suy diễn nếu context không ghi rõ số liệu.
- *Cách kiểm tra hiệu quả*: Chạy lại `python scripts/setup_answers.py --live` và `python scripts/run_experiments.py --phase a --live`, so sánh `per_distribution["multi_hop"]["avg_score"]` phải tăng từ 0.5771 lên > 0.75.

## Bẫy adversarial và giới hạn

- **So sánh với factual và multi_hop**: Nhóm adversarial đạt `avg_score` = 0.7906, cao nhất trong 3 nhóm (factual 0.7850, multi_hop 0.5771). Trong đó `faithfulness` đạt 0.9300 và `context_precision` đạt 0.9333. Điều này cho thấy với các câu hỏi bẫy về chính sách lỗi thời, tài liệu chính sách hiện hành được retrieve chính xác và LLM tuân thủ ngữ cảnh để từ chối các khẳng định sai lệch.
- **Hiện tượng version conflict & negation**: Chỉ có 1 câu adversarial lọt vào Bottom 10 (ID 49, rank 6, avg=0.4784). Điểm số của ID 49 bị kéo xuống do `answer_relevancy` (0.4286) vì LLM tập trung vào phần phủ định chính sách v2023 khiến độ tương đồng ngữ nghĩa với câu hỏi bị giảm.
- **Giới hạn nhận thức an toàn**: Điểm số RAGAS cao ở nhóm adversarial chỉ phản ánh RAG pipeline trả lời đúng ngữ cảnh tài liệu; **nó không chứng minh pipeline đã an toàn trước các cuộc tấn công jailbreak, prompt injection hay rò rỉ dữ liệu nhạy cảm**. Điểm thấp hay cao ở Phase A không thay thế được các lớp Guardrails (Presidio PII + NeMo Guardrails) được đo lường ở Phase C.
- **Giới hạn thực nghiệm**:
  - Kích thước tập test: 50 câu hỏi (20 factual, 20 multi_hop, 10 adversarial); quy mô đủ để xác định xu hướng nhưng có phương sai khi phân tích chi tiết từng cặp metric.
  - LLM sử dụng: `gpt-oss:120b-cloud` qua Ollama Cloud API; mô hình suy luận tốt nhưng latency trung bình lớn hơn các mô hình nhỏ. Không xảy ra trường hợp nào phải kích hoạt `extractive_fallback`.
