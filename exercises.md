# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Câu trả lời chủ động từ chối/ghi rõ không đủ evidence khi context thiếu hoặc lỗi thời; không đưa claim thực tế không có nguồn. | Có claim về chính sách, đơn hàng, hoàn tiền... không được context hỗ trợ (hallucination), dù câu trả lời nghe có vẻ hợp lý. | Log theo dõi case abstain và bổ sung/cập nhật corpus; với claim không có evidence thì block hoặc route human review. |
| Answer Relevance | Câu hỏi ngoài phạm vi hoặc không an toàn được chuyển hướng ngắn gọn, đúng chính sách nên overlap với câu hỏi có thể thấp. | Câu hỏi support trong phạm vi nhưng câu trả lời lạc đề hoặc không giải quyết ý định chính của user. | Kiểm tra intent routing/prompt; thêm các câu cùng intent vào golden set và sửa retrieval hoặc prompt. |
| Context Recall | Câu trả lời chỉ cần một phần thông tin và phần context thiếu là chi tiết không thiết yếu; hệ thống nói rõ phạm vi trả lời. | Retriever bỏ sót evidence cần cho đáp án hoặc policy bắt buộc, khiến model không thể trả lời đúng/đủ. | Bổ sung chunk, cải thiện indexing/query expansion và kiểm tra coverage theo từng loại câu hỏi. |
| Context Precision | Retriever lấy thêm vài chunk nhiễu nhưng evidence liên quan vẫn ở đầu danh sách và câu trả lời vẫn grounded. | Chunk đầu bảng xếp hạng chủ yếu không liên quan, làm tăng chi phí và dẫn model tới evidence sai. | Tuning ranking/reranking, lọc duplicate/noise và đặt giới hạn top-k hợp lý. |
| Completeness | User chỉ yêu cầu câu trả lời ngắn hoặc một trường thông tin; chi tiết bị lược bỏ không làm thay đổi quyết định của user. | Bỏ sót bước, điều kiện, ngoại lệ hoặc thông tin hành động bắt buộc trong câu trả lời support. | So khớp với expected answer, bổ sung rubric/prompt về các ý bắt buộc và thêm regression cases. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Dùng cùng một tập câu hỏi và các cặp đáp án A/B đã được human label; giữ nội dung, độ dài và rubric cố định. Condition 1: judge thấy A trước, B sau. Condition 2: đảo thứ tự để B trước, A sau. Phân ngẫu nhiên và cân bằng số mẫu ở hai condition, ẩn tên model/nguồn đáp án. So sánh tỷ lệ thắng và điểm của cùng một đáp án khi đứng trước với khi đứng sau (có thể dùng paired test/CI). Nếu một đáp án thắng đáng kể chỉ vì ở vị trí đầu, đó là position bias.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Rubric cần chấm trực tiếp correctness, evidence/faithfulness và mức độ đáp ứng yêu cầu, với mô tả điểm 1–5 rõ ràng. Ghi rõ “không cộng điểm cho chi tiết không cần thiết hoặc độ dài”, thưởng cho câu trả lời ngắn nhưng đủ, và trừ điểm cho lặp lại/lan man. Có thể yêu cầu judge đánh giá từng tiêu chí độc lập trước khi tổng hợp; nếu cần chấm độ dài, tách thành tiêu chí conciseness riêng thay vì để nó ảnh hưởng accuracy.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* Human labels là chuẩn tham chiếu cho định nghĩa “đúng”, “đủ” và mức độ chấp nhận được trong domain. Calibration cho biết judge có tương quan/độ đồng thuận với người chấm hay không, phát hiện thiên lệch hệ thống (quá dễ, quá nghiêm hoặc ưu tiên văn phong của chính nó), và giúp chọn prompt, rubric, threshold phù hợp. Cần lặp lại calibration khi domain, rubric hoặc model judge thay đổi để tránh quality gate bị lệch.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.80 | Claim không grounded có thể tạo thông tin sai về chính sách/đơn hàng; đặt ngưỡng cao hơn các metric còn lại. |
| Answer Relevance | 0.75 | Dưới mức này câu trả lời thường không xử lý đúng ý định chính, nhưng cho phép một ít biến thiên ở câu hỏi đa ý hoặc route an toàn. |
| Completeness | 0.75 | Bảo đảm phần lớn thông tin hành động/điều kiện cần thiết có mặt; các thiếu sót có tính hệ thống phải chặn release. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* Offline evaluation dùng golden dataset phiên bản hoá trong CI, trước merge/release, để phát hiện regression có thể tái lập và áp dụng các threshold quality gate. Online evaluation dùng sau deploy trên traffic thật (đã bảo vệ dữ liệu cá nhân), kết hợp telemetry, user feedback, sampled evaluation để theo dõi drift, latency, tỷ lệ escalation và những intent mới mà golden set chưa đại diện. Human review dùng cho case rủi ro cao hoặc mơ hồ (refund, policy, safety), mẫu thất bại/điểm sát ngưỡng, disagreement giữa judge và metric, và các đợt calibration/audit; kết quả review được đưa lại golden dataset.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
