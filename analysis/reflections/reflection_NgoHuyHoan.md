# Individual Reflection — Lab 18: Production RAG

**Tên:** Ngô Huy Hoan

**Module phụ trách:** M1–M5

## 1. Mapping bài giảng vào code

| Lecture Concept | Module | Hàm cụ thể | Observation |
|---|---|---|---|
| Semantic chunking | M1 | `chunk_semantic()` | Nhóm câu theo độ tương đồng, có lexical fallback để chạy offline và tránh chunk quá nhỏ. |
| Parent-child retrieval | M1 | `chunk_hierarchical()` | Child phục vụ truy hồi chính xác; `parent_id` giúp lấy context rộng hơn. |
| BM25 + Dense fusion | M2 | `reciprocal_rank_fusion()` | RRF kết hợp lexical và semantic ranking mà không cần chuẩn hóa score. |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | Rerank giảm context nhiễu trước khi sinh answer; fallback giữ test ổn định. |
| RAGAS | M4 | `evaluate_ragas()` | Relevancy 0.7322 nhưng faithfulness 0.6817, nên không thể chỉ nhìn một metric. |
| Contextual embeddings | M5 | `contextual_prepend()` | Context, summary và metadata làm rõ vai trò chunk trong tài liệu. |

## 2. Khó khăn & cách giải quyết

- **Lỗi gặp phải:** `FileExistsError: [WinError 183]`, marker merge conflict `<<<<<<< HEAD`, và timeout khi model cố tải từ mạng.
- **Cách debug:** Đọc traceback, dùng `git status`/tìm marker conflict, rồi chạy `python -m pytest tests/ -q` sau mỗi thay đổi.
- **Cách giải quyết:** Thay report an toàn khi rerun, xoá marker conflict, và dùng fallback lexical/heuristic khi không bật tải model.
- **Kiến thức cần bổ sung:** Retrieval có versioning, groundedness evaluation, và vận hành model offline/production.

## 3. Action Plan cho project

## Project: Production RAG cho tài liệu chính sách nội bộ

### Hiện tại

- Pipeline có chunking, hybrid search, reranking, enrichment và RAGAS.
- Known issues: faithfulness thấp ở câu versioned và retrieval có thể thiếu facts cho câu hỏi tổng hợp.

### Plan áp dụng

1. [ ] **Chunking:** Structure-aware + parent-child, giữ header/version/ngày hiệu lực trong metadata.
2. [ ] **Search:** Hybrid BM25 + dense + RRF, thêm filter `version`, `source`, loại tài liệu.
3. [ ] **Reranking:** Cross-encoder top-20 → top-3, benchmark latency trước khi bật trong production.
4. [ ] **Evaluation:** RAGAS regression cho nhóm versioned, multi-hop và ngoại lệ.
5. [ ] **Enrichment:** Contextual prepend, metadata extraction và HyQA cho section chứa rule.

### Timeline

- **Tuần 1:** Chuẩn hóa metadata version/effective date và test retrieval.
- **Tuần 2:** Thêm reranking/citation, benchmark latency và chi phí.
- **Tuần 3:** Mở rộng test set, chạy RAGAS regression, tối ưu failure có faithfulness thấp.

## 4. Nếu làm lại

- Xây dựng benchmark cho policy bị thay thế ngay từ đầu.
- Thử metadata-aware retrieval, query rewriting bảo toàn điều kiện và verifier cho phép tính.

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1–5) |
|---|---:|
| Hiểu bài giảng | 4 |
| Code quality | 4 |
| Teamwork | 4 |
| Problem solving | 4 |
