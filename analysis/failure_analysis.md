# Failure Analysis — Lab 18: Production RAG

**Nhóm:** Cá nhân

**Thành viên:** Ngô Huy Hoan — M1 đến M5

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|---|---:|---:|---:|
| Faithfulness | 0.8375 | 0.6817 | -0.1558 |
| Answer Relevancy | 0.7222 | 0.7322 | +0.0100 |
| Context Precision | 0.9250 | 0.9208 | -0.0042 |
| Context Recall | 0.9250 | 0.8417 | -0.0833 |

Production tăng nhẹ answer relevancy, nhưng faithfulness và recall giảm. Pipeline trả lời đúng trọng tâm hơn, song chưa luôn lấy đúng bằng chứng hoặc chưa buộc LLM bám sát context.

## Bottom-5 Failures

### #1 — Chu kỳ đổi mật khẩu

- **Question:** Bao lâu phải đổi mật khẩu một lần?
- **Expected:** Chính sách v2.0: 120 ngày; 90 ngày là chính sách cũ.
- **Got:** Faithfulness thấp nhất, score 0.4583; câu trả lời có rủi ro dùng thông tin cũ hoặc không được neo vào context.
- **Worst metric:** Faithfulness.
- **Error Tree:** Output không đáng tin → context có thể chứa cả v1.0/v2.0 → query chưa ưu tiên version hiện hành.
- **Root cause:** Retrieval chưa dùng metadata version/effective date.
- **Suggested fix:** Gắn `version`, `effective_date`, filter chính sách bị thay thế, bắt buộc citation trong answer.

### #2 — Nghỉ phép của nhân viên thử việc

- **Question:** Nhân viên thử việc có được nghỉ phép năm không?
- **Expected:** Không; nếu cần nghỉ thì xin nghỉ không lương và trưởng phòng phê duyệt.
- **Got:** Faithfulness 0.4583, cho thấy context có thể thiếu điều kiện “thử việc”.
- **Worst metric:** Faithfulness.
- **Error Tree:** Context nghiêng về nghỉ phép chung → mất ngoại lệ theo loại nhân viên.
- **Root cause:** Chunk/retrieval chưa ưu tiên điều kiện ngoại lệ.
- **Suggested fix:** Chunk theo section điều kiện, thêm HyQA và trọng số từ khóa `thử việc`.

### #3 — Bảo hiểm PVI của nhân viên thử việc

- **Question:** Nhân viên thử việc có được hưởng bảo hiểm sức khỏe PVI không?
- **Expected:** Không; chỉ tham gia bảo hiểm xã hội bắt buộc.
- **Got:** Faithfulness 0.5000; có rủi ro suy diễn từ quyền lợi của nhân viên chính thức.
- **Worst metric:** Faithfulness.
- **Error Tree:** Context chung về PVI không đủ để xác định đối tượng áp dụng.
- **Root cause:** Thiếu filter `employee_type` trước rerank.
- **Suggested fix:** Gắn metadata đối tượng áp dụng và yêu cầu LLM không suy luận khi thiếu evidence.

### #4 — Phê duyệt nghỉ không lương 20 ngày

- **Question:** Nghỉ phép không lương 20 ngày cần ai phê duyệt?
- **Expected:** CEO phê duyệt đối với nghỉ 16–30 ngày.
- **Got:** Context precision là metric thấp nhất, score 0.6227.
- **Worst metric:** Context Precision.
- **Error Tree:** Nhiều context về nghỉ phép nhưng không đúng khoảng 16–30 ngày.
- **Root cause:** BM25 khớp “nghỉ phép” nhưng chưa phân biệt ngưỡng số ngày; RRF giữ context nhiễu.
- **Suggested fix:** Rerank top-k, bảo toàn số ngày khi rewrite query, giảm context gửi vào LLM.

### #5 — Lương thử việc Junior

- **Question:** Lương thử việc của nhân viên Junior mức cao nhất là bao nhiêu?
- **Expected:** Junior tối đa 20 triệu; lương thử việc 85% = 17 triệu VNĐ/tháng.
- **Got:** Faithfulness thấp, score 0.7060.
- **Worst metric:** Faithfulness.
- **Error Tree:** Câu hỏi cần hai facts và phép tính; thiếu một context sẽ làm kết luận sai.
- **Root cause:** Facts lương Junior và quy tắc 85% có thể nằm ở chunk khác nhau.
- **Suggested fix:** Parent-child retrieval, context top-3 có rerank, và yêu cầu nêu phép tính.

## Case Study

**Question chọn phân tích:** Bao lâu phải đổi mật khẩu một lần?

1. Output đúng? → Không đủ đáng tin vì faithfulness thấp.
2. Context đúng? → Có nguy cơ lẫn v1.0 (90 ngày) với v2.0 (120 ngày).
3. Query rewrite OK? → Cần giữ ý “hiện hành” và ưu tiên version mới.
4. Fix → Metadata-aware retrieval, rerank theo version, citation bắt buộc.

**Nếu có thêm 1 giờ:** Bổ sung `version`, `effective_date`, `supersedes`; benchmark riêng nhóm câu hỏi versioned và đo lại faithfulness với top-3 context.
