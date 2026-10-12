# Đánh giá LLM Judge Phase B

Nhóm: Chu Văn Nhân

Người phụ trách: Chu Văn Nhân - MSSV: 2A202602668

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T10:11:59.296013+00:00.
Models generation/judge: Generation: `gpt-oss:120b-cloud`; Judge: `gpt-oss:120b-cloud`.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ
fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Candidate A | Candidate B | Winner Pass 1 | Winner Pass 2 | Final Winner | Consistent | Scores P1 (A/B) | Scores P2 (A/B) | Reasoning Pass 1 & 2 |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ **3 ngày làm việc** khi kết hôn. | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | B | B | B | Có | A: 0.83, B: 0.88 | A: 0.67, B: 0.87 | P1: Cả hai nêu đúng số ngày, B có thêm "có lương" gần reference hơn. P2: Hoán đổi vị trí, B vẫn thắng nhờ chi tiết "có lương". |
| 5 | Giao dịch trị giá **55 triệu VNĐ** vượt mức **50 triệu VNĐ**, do đó cần được **Tổng Giám đốc (CEO)** phê duyệt. | Cần Giám đốc phòng ban phê duyệt. | A | A | A | Có | A: 1.00, B: 0.00 | A: 0.95, B: 0.10 | P1: A nêu đúng thẩm quyền Tổng Giám đốc (CEO) cho đơn > 50 triệu; B sai thẩm quyền. P2: A tiếp tục thắng thuyết phục ở cả hai vị trí. |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có thời gian làm việc từ 6 tháng trở lên là **1 tháng lương**. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | B | tie | tie | Không | A: 0.97, B: 1.00 | A: 1.00, B: 1.00 | P1: Cả hai đúng nội dung, judge ưu tiên B vì ngắn gọn hơn. P2: Judge đánh giá hai đáp án ngang điểm (1.0 vs 1.0), kết luận hòa (tie). |
| 21 | **Số ngày phép:** 18 ngày (quy định 2024: 15 cơ bản + 1 ngày mỗi 3 năm thâm niên; 9 năm -> 18 ngày). **Khoảng lương:** Không tìm thấy. | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | B | B | B | Có | A: 0.60, B: 1.00 | A: 0.50, B: 1.00 | P1: B cung cấp đầy đủ cả ngày phép và khoảng lương; A thiếu phần lương. P2: Đổi vị trí, B vẫn thắng do thông tin đầy đủ theo reference. |
| 41 | Nhân viên được nghỉ **15 ngày phép năm** (theo chính sách nghỉ phép năm 2024, phiên bản 2.0). | Nhân viên được nghỉ 12 ngày phép năm. | A | A | A | Có | A: 0.90, B: 0.20 | A: 0.90, B: 0.20 | P1: A dùng chính sách hiện hành v2024 (15 ngày); B dùng chính sách cũ v2023 (12 ngày). P2: A giữ vững chiến thắng ở cả hai lượt. |

Position bias count/rate: 1 / 5 (20.0%).
Verbosity win rate: 0.75 (75.0%); decisive cases: 4 (trong đó 3 cases đáp án dài hơn chiến thắng: A thắng dài hơn ở câu 5, 41; B thắng dài hơn ở câu 21).

### Diễn giải và phân tích bias

1. **Phân tích Position Bias (1/5 cases = 20.0%)**:
   - Chỉ xuất hiện ở câu 12: Pass 1 judge chọn B thắng (A ở vị trí đầu, B ở vị trí sau), Pass 2 judge chọn Tie (B ở vị trí đầu, A ở vị trí sau). Vì cả hai đáp án đều đúng hoàn toàn về mặt ngữ nghĩa (thưởng Tết tối thiểu 1 tháng lương), sự khác biệt chỉ nằm ở cách hành văn. Khi đổi vị trí, judge không giữ nguyên lựa chọn mà chuyển thành hòa. Điều này phản ánh rõ ràng độ nhạy vị trí khi hai câu trả lời có chất lượng tương đương.
   - Nhờ kỹ thuật swap-and-average, kết quả chung cuộc của câu 12 được đưa về `tie`, loại bỏ hoàn toàn thiên vị vị trí và bảo đảm tính khách quan.

2. **Phân tích Verbosity Bias (3/4 decisive cases = 75.0%)**:
   - Trong 4 ca có kết quả phân định (decisive), 3 ca có đáp án dài hơn chiến thắng. Tuy nhiên, khi đối chiếu nội dung, nguyên nhân gốc rễ không phải là judge bị đánh lừa bởi độ dài đơn thuần:
     - Câu 5: Đáp án A dài hơn vì trích dẫn ngưỡng 50 triệu và chức danh CEO đúng theo chính sách; B ngắn nhưng sai nội dung (chọn Giám đốc phòng ban).
     - Câu 21: Đáp án B dài hơn vì trả lời đủ 2 vế (phép năm và lương Senior); A thiếu vế lương.
     - Câu 41: Đáp án A dài hơn vì giải thích rõ phiên bản v2024; B ngắn hơn nhưng dùng số liệu cũ v2023 đã hết hiệu lực.
   - Dấu hiệu thống kê 75.0% verbosity win rate thực chất phản ánh việc các đáp án đầy đủ và chính xác thường có xu hướng chi tiết và dài hơn các đáp án sai hoặc thiếu ý.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json;
không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference.
Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| ID | Câu hỏi | Model Answer gốc | Nhãn Judge | Reasoning | Nhãn Human | Đồng thuận |
|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ bao nhiêu ngày khi kết hôn? | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | Đúng số ngày (3 ngày) và có lương, phù hợp reference. | 1 | Có |
| 5 | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | Cần Giám đốc phòng ban phê duyệt. | 0 | Sai thẩm quyền phê duyệt: trên 50 triệu phải do CEO phê duyệt. | 0 | Có |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là bao nhiêu? | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | Khớp reference: nhân viên chính thức từ 6 tháng được tối thiểu 1 tháng lương. | 1 | Có |
| 21 | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | Đầy đủ và chính xác cả số ngày phép (18 ngày) và khung lương (20-35 triệu). | 1 | Có |
| 23 | Nhân viên được tài trợ khóa học 25 triệu, nghỉ việc sau 8 tháng. Phải hoàn trả bao nhiêu? | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | Khớp reference: chưa đủ thời gian cam kết 1 năm nên hoàn trả 100% (25 triệu). | 1 | Có |
| 29 | Nhân viên tạm ứng 8 triệu, chưa thanh toán sau 30 ngày. Ai phê duyệt và phí phạt là bao nhiêu? | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | 0 | Sai thẩm quyền (cần cả Trưởng phòng và Kế toán trưởng) và thiếu số tiền phạt cụ thể. | 0 | Có |
| 33 | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm? | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | 1 | Cung cấp đúng số ngày phép (19 ngày) và phụ cấp hàng tháng (1.500.000 VNĐ). | 1 | Có |
| 41 | Nhân viên được nghỉ bao nhiêu ngày phép năm? | Nhân viên được nghỉ 12 ngày phép năm. | 0 | Dùng số liệu chính sách cũ v2023 (12 ngày) thay vì chính sách v2024 (15 ngày). | 0 | Có |
| 46 | Nhân viên thử việc có được nghỉ phép năm không? | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | 1 | Đúng quy định: thử việc không có phép năm, phải xin nghỉ không lương. | 1 | Có |
| 50 | Nhân viên Manager có thể dùng VPN cá nhân (NordVPN) khi WFH không? | Được, miễn là đảm bảo kết nối an toàn. | 0 | Sai hoàn toàn chính sách: cấm sử dụng VPN thương mại/cá nhân khi WFH. | 0 | Có |

Cohen’s κ: 1.0000.
Observed/chance agreement và diễn giải:
- Observed agreement: P_o = 10 / 10 = 1.0000 (100% các câu đánh giá trùng khớp hoàn hảo giữa Judge và Human).
- Chance agreement: Phân bố nhãn của Judge là 6 nhãn 1 và 4 nhãn 0 (p_1 = 0.6, p_0 = 0.4); phân bố của Human là 6 nhãn 1 và 4 nhãn 0 (q_1 = 0.6, q_0 = 0.4). Xác suất đồng thuận ngẫu nhiên P_e = (0.6 × 0.6) + (0.4 × 0.4) = 0.36 + 0.16 = 0.5200.
- Cohen's κ = (P_o - P_e) / (1.0 - P_e) = (1.0 - 0.52) / (1.0 - 0.52) = 1.0000.
- Diễn giải: Giá trị κ = 1.0000 đạt mức đồng thuận tuyệt đối (Almost Perfect / Perfect Agreement theo thang Landis & Koch). LLM Judge hiểu rõ các tiêu chí khắt khe về tính đúng đắn, phủ định và phiên bản chính sách theo reference.

## Giới hạn và cách dùng judge

1. **Giới hạn về mô hình và cỡ mẫu**:
   - Sử dụng mô hình `gpt-oss:120b-cloud` làm judge. Mặc dù đạt κ = 1.0 trên tập 10 câu, cỡ mẫu 10 câu nhãn human và 5 cặp pairwise là tương đối nhỏ; kết quả mang tính chất benchmark hiệu chuẩn cục bộ, chưa thể khẳng định khái quát hóa 100% trên toàn bộ miền dữ liệu.
   - Khi gặp các trường hợp câu trả lời tương đương về ngữ nghĩa (như câu 12), judge vẫn có thể dao động nhẹ về điểm số giữa hai lượt.

2. **Cách kiểm chứng và triển khai thực tế**:
   - Không được coi LLM-as-a-Judge là "chân lý tuyệt đối" (ground truth) thay thế hoàn toàn con người.
   - Bắt buộc áp dụng kỹ thuật Swap-and-Average trong môi trường production để khử triệt để position bias trước khi đưa ra quyết định chọn model hay cập nhật prompt.
   - Đối với các câu hỏi critical liên quan đến bảo mật, tài chính hoặc pháp lý, LLM Judge chỉ đóng vai trò bộ lọc sơ loại (triage); các ca có biên độ điểm số sít sao hoặc không nhất quán vị trí cần được chuyển tiếp cho chuyên gia con người (Human-in-the-loop) thẩm định cuối cùng.
