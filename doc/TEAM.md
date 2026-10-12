# Nhóm thực hiện K4 Track 3A

Tên nhóm: Chu Văn Nhân

Repo chung: https://github.com/VanNahAI/K4-Track-3A-ChuVanNhan-2A202602668-Day24-Eval-Guard

Trưởng nhóm/người nộp: Chu Văn Nhân - 2A202602668

Ngày lab/deadline áp dụng: 2026-10-12 23:59:00 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: b9f8d5b53bc4f94899aa30fdc52c30be0e96eb6d

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Chu Văn Nhân | 2A202602668 | Trưởng nhóm solo: thực hiện toàn bộ Phase A, Phase B, Phase C và báo cáo tổng kết | Commits, artifacts JSON, failure_clusters.md, bias_report.md, blueprint.md |

Người tích hợp/chạy batch/nộp link: Chu Văn Nhân, độc lập hoàn thành tích hợp pipeline, kiểm thử và nộp repo.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11 64-bit, Python 3.11.9, Visual Studio C++ Build Tools 14.x.
Virtualenv/dependencies/imports: `.venv`, cài đặt đầy đủ theo `requirements/requirements.txt`, PyArrow 17.0.0, PyTorch 2.5, RAGAS, Presidio, NeMo Guardrails 0.10.0.
Models/Qdrant/50 answers đã chuẩn bị: Qdrant client 1.9.2 (in-memory mode tương thích search), Ollama Cloud API (`gpt-oss:120b-cloud`), embedding `sentence-transformers/all-MiniLM-L6-v2`. Sinh đủ 50 câu hỏi qua Hybrid Search + Reranker + LLM generation mode.
Nếu cài full NeMo chưa thành công, ghi lỗi thực tế; không dùng unit test pass hoặc pip resolve để khẳng định runtime đã chạy: NeMo Guardrails 0.10.0 hoạt động ổn định trên Python 3.11 Windows, đã tiêm ChatOpenAI adapter tương thích Ollama Chat completions API.

## INDIVIDUAL

### ChuVanNhan-2A202602668

- Phần được giao và phần thực tế đã làm: Solo hoàn thành toàn bộ Tasks 1-4 (Phase A RAGAS), Tasks 5-8 (Phase B Pairwise Judge, Swap & Average, Cohen's Kappa, Bias Report), Tasks 9-12 (Phase C Guardrails, Presidio PII scan, Input/Output rails, Latency Benchmark), và Task 13 (Hệ thống blueprint).
- Bằng chứng: Artifacts `artifacts/ragas_50q.json`, `artifacts/judge_results.json`, `artifacts/guard_results.json`, các file mã nguồn `src/phase_a_ragas.py`, `src/phase_b_judge.py`, `src/phase_c_guard.py` và các báo cáo Markdown.
- Cách kiểm tra: Đã chạy `pytest tests/test_phase_a.py`, `pytest tests/test_phase_b.py`, `pytest tests/test_phase_c.py`, `pytest tests/test_report_contracts.py` và các lệnh thực nghiệm live `python scripts/run_experiments.py --phase all --live`.
- Điều đã học: Hiểu sâu về phân tích ma trận lỗi RAGAS, các loại bias trong LLM-as-a-judge (vị trí, độ dài), cơ chế kiểm soát hallucination và bảo vệ an toàn đa tầng với Presidio và NeMo Guardrails.
- Giới hạn/cách xử lý: Xử lý tương thích Qdrant 1.9 (`search` thay vì `query_points`), cấu hình NeMo Guardrails kết nối endpoint Ollama Chat, và tinh chỉnh latency sequential.
- Nguồn/AI hỗ trợ nếu có: Tham khảo tài liệu RAGAS, NeMo Guardrails docs và Copilot SDK hỗ trợ soạn thảo. Tự kiểm chứng bằng hợp đồng schema và test suite.

## Cập nhật sau khi nộp

Không có cập nhật bổ sung sau deadline.
