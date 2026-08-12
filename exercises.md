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
| Faithfulness | Một vài câu trả lời sáng tạo hoặc câu hỏi còn thiếu dữ kiện khiến context chưa đủ để kiểm chứng; câu trả lời phải nói rõ giới hạn và không khẳng định quá mức. | Thường xuyên có claim không được hỗ trợ bởi context, đặc biệt với giá, thanh toán, bảo hành, bảo mật hoặc chính sách. | Kiểm tra từng claim và nguồn truy xuất; cải thiện grounding/retrieval. Block release nếu lỗi tạo thông tin sai hoặc liên quan an toàn/chính sách. |
| Answer Relevance | Câu hỏi mơ hồ hoặc người dùng chỉ hỏi một phần nhỏ nên hệ thống cần hỏi lại để làm rõ. | Câu trả lời lạc đề, không giải quyết intent chính, hoặc buộc người dùng hỏi lại trong các luồng hỗ trợ quan trọng. | Phân tích intent, query rewrite và mẫu thất bại; block nếu mức thấp xảy ra phổ biến trên các intent chính. |
| Context Recall | Gold answer chứa chi tiết không cần cho câu hỏi hiện tại, hoặc bộ truy xuất nhỏ có chủ ý và câu trả lời vẫn đúng trong phạm vi hẹp. | Context bỏ sót điều kiện, ngoại lệ, thời hạn hoặc bước bắt buộc của chính sách, khiến câu trả lời có thể sai dù phần còn lại đúng. | Bổ sung/chia nhỏ tài liệu, điều chỉnh top-k và kiểm tra coverage theo intent; block với các policy-critical cases. |
| Context Precision | Có một số chunk nhiễu trong top-k nhưng chunk liên quan vẫn đứng sớm và câu trả lời bám đúng nguồn. | Chunk không liên quan đứng trước hoặc chiếm đa số, làm model dựa vào thông tin sai/lẫn chính sách. | Cải thiện search, metadata filter và reranking; theo dõi precision ở các rank đầu và block nếu retrieval dẫn đến hallucination. |
| Completeness | Câu hỏi chỉ yêu cầu một phần thông tin, hoặc câu trả lời ngắn nhưng đã nêu rõ phạm vi và hướng dẫn người dùng hỏi tiếp. | Bỏ sót bước thực hiện, điều kiện eligibility, cảnh báo hoặc thông tin cần thiết để người dùng hoàn thành yêu cầu. | So sánh với checklist/gold answer, bổ sung coverage và test edge cases; block nếu thiếu thông tin bắt buộc. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:*

Tạo một tập câu hỏi và các cặp answer A/B có chất lượng tương đương (độ đúng, độ dài và format được cân bằng), sau đó giữ nguyên prompt và rubric của judge. Chạy ít nhất hai condition:

1. Condition 1: hiển thị A trước, B sau.
2. Condition 2: hoán đổi thứ tự, hiển thị B trước, A sau.

Randomize thứ tự trên nhiều câu hỏi và chạy lặp với nhiều seed. Ghi lại winner/score của từng answer ở cả hai condition. Nếu một answer có điểm hoặc xác suất thắng cao hơn đáng kể chỉ vì đứng trước, đó là evidence của position bias. Có thể bổ sung condition không có nhãn A/B và đối chiếu với human labels để loại trừ khác biệt chất lượng thật.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:*

Thiết kế rubric theo các tiêu chí độc lập, có trọng số và điểm tối đa rõ ràng: correctness, coverage, faithfulness/grounding và khả năng thực hiện yêu cầu. Quy định rằng câu trả lời ngắn nhưng đầy đủ được điểm bằng hoặc cao hơn câu trả lời dài; không chấm độ dài như một proxy cho chất lượng. Chỉ thưởng cho thông tin liên quan, đồng thời trừ điểm cho lan man, lặp ý, claim không cần thiết hoặc không được hỗ trợ. Nên dùng các answer có độ dài khác nhau trong calibration để kiểm tra rubric thực sự length-neutral.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:*

Human labels là mốc chuẩn độc lập để đo agreement, phát hiện judge chấm lệch có hệ thống và hiệu chỉnh mapping từ score sang pass/fail. Calibration cũng cho biết ngưỡng nào tương ứng với chất lượng mà con người chấp nhận, nhất là với lỗi policy hoặc safety mà điểm trung bình có thể che khuất. Sau khi triển khai, dùng một tập human-labeled cố định để theo dõi drift và re-calibrate khi model, prompt hoặc domain thay đổi.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.85 | Đây là hard gate vì claim không grounded có thể tạo thông tin sai về đơn hàng, thanh toán hoặc chính sách. Chỉ cần một critical slice dưới ngưỡng cũng phải block. |
| Answer Relevance | 0.80 | Bảo đảm hệ thống trả lời đúng intent chính thay vì chỉ chứa các từ khóa liên quan; dưới ngưỡng sẽ làm tăng số lượt hỏi lại và escalation. |
| Completeness | 0.80 | Đảm bảo không bỏ sót bước, điều kiện và cảnh báo cần thiết để người dùng thực hiện đúng yêu cầu. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:*

Offline evaluation dùng trước mỗi pull request/release trên golden set và regression set: nhanh, lặp lại được, phù hợp để block deployment. Online evaluation dùng sau khi deploy, thường qua canary hoặc A/B, để đo dữ liệu và hành vi người dùng thật như feedback, escalation, latency và drift; nên có rollback guard. Human review dùng cho các case điểm thấp hoặc confidence thấp, câu hỏi mới/ngoài domain, thay đổi policy, và các luồng nhạy cảm như thanh toán, bảo mật, hoàn tiền hoặc khi cần tạo nhãn chuẩn cho calibration. Kết hợp cả ba: offline là gate, online là monitoring, human review là kiểm tra chất lượng sâu và xử lý ngoại lệ.

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
