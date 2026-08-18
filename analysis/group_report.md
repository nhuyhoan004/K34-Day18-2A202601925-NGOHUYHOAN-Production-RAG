# Group Report — Lab 18: Production RAG

**Nhóm:** Cá nhân

**Ngày:** 2026-08-18

## Thành viên & Phân công

| Tên | Module | Hoàn thành | Tests pass |
|---|---|---|---:|
| Ngô Huy Hoan | M1: Chunking | ✅ | 13/13 |
| Ngô Huy Hoan | M2: Hybrid Search | ✅ | 5/5 |
| Ngô Huy Hoan | M3: Reranking | ✅ | 5/5 |
| Ngô Huy Hoan | M4: Evaluation | ✅ | 4/4 |
| Ngô Huy Hoan | M5: Enrichment | ✅ | 10/10 |

**Tổng:** 37/37 tests pass.

## Kết quả RAGAS

| Metric | Naive | Production | Δ |
|---|---:|---:|---:|
| Faithfulness | 0.8375 | 0.6817 | -0.1558 |
| Answer Relevancy | 0.7222 | 0.7322 | +0.0100 |
| Context Precision | 0.9250 | 0.9208 | -0.0042 |
| Context Recall | 0.9250 | 0.8417 | -0.0833 |

## Key Findings

1. **Biggest improvement:** Answer relevancy tăng nhẹ (+0.0100), cho thấy hybrid retrieval/enrichment giúp câu trả lời bám câu hỏi hơn.
2. **Biggest challenge:** Faithfulness giảm 0.1558; các policy có version cũ/mới và điều kiện ngoại lệ cần retrieval có metadata.
3. **Surprise finding:** Precision gần như giữ nguyên nhưng recall giảm; thêm kỹ thuật retrieval không tự động bảo đảm lấy đủ facts cho câu hỏi tổng hợp.

## Presentation Notes

1. Production tăng relevancy nhưng cần tối ưu faithfulness trước release.
2. M2 kết hợp BM25, dense và RRF; M3 giảm context nhiễu bằng reranking.
3. Case study: 90/120 ngày đổi mật khẩu cho thấy lỗi versioning nằm ở retrieval trước khi LLM sinh answer.
4. Next step: filter policy bị thay thế, thêm citation, benchmark multi-hop và ngoại lệ.
