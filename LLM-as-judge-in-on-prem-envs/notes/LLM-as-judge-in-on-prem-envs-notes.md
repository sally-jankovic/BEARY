# LLM-as-judge-in-on-prem-envs — Notes

<!-- Notes for the LLM-as-judge-in-on-prem-envs topic. Organized by question, not source. -->
<!-- See .windsurf/skills/internet-research/SKILL.MD for research workflow and guidelines. -->
<!-- Citations are stored in whitepaper/LLM-as-judge-in-on-prem-envs-references.md -->

## General Understanding

### Q1: What is LLM-as-judge and what are the primary use cases and evaluation paradigms?

**Definition:** LLM-as-a-Judge is an evaluation methodology where an LLM assesses the quality of outputs produced by another LLM application [1][2]. It emerged after GPT-4's release as a scalable alternative to costly human evaluation for open-ended text outputs [3].

**Why it works:** Evaluating/classifying content is simpler than generating it. The judge LLM performs a focused task (e.g., "Is this response relevant?") rather than the complex multi-variable generation task [1]. It activates different capabilities via a separate evaluation prompt.

**Primary Evaluation Paradigms:**
- **Pairwise comparison:** Judge compares two responses and selects the better one. Useful for model/prompt comparison during development. GPT-4 achieves 80%+ agreement with human preferences [1][3].
- **Pointwise scoring (direct assessment):** Judge scores a single response on criteria (e.g., 1-5 Likert scale). More scalable but less stable than pairwise [3].
- **Reference-guided scoring:** Judge given a reference answer alongside the response to help scoring [3].

**Key Use Cases:**
- Development: Compare models/prompts, regression testing [1]
- Production monitoring: Continuous quality/safety assessment [1]
- Benchmarks and leaderboards: MT-Bench, Chatbot Arena, AlpacaEval [4]
- Truthfulness/hallucination detection: TruthfulQA, SelfCheckGPT [4]
- Safety and alignment: Constitutional AI, RLAIF [4]
- Domain-specific: Education, code review, recommendation explanations [4]

**Evaluation Outputs:**
- Binary pass/fail decisions
- Likert-style scores on dimensions (relevance, coherence, fluency, safety)
- Pairwise preferences
- Natural language critiques/explanations alongside scores [4]

### Q2: What are the current best practices and frameworks for implementing LLM-as-judge systems?

**7 Key Best Practices (Monte Carlo Data) [5]:**

1. **Few-shot prompting:** Provide 1+ examples of good/bad outputs. Research shows 1-shot often performs better than more examples [5].

2. **Step decomposition:** Break big subjective decisions into smaller criteria and reasoning steps [5].

3. **Criteria decomposition:** Each evaluation should monitor a single criterion—don't combine relevancy and clarity in one prompt [5].

4. **Evaluation template (grading rubric):** G-Eval approach—provide scoring scale with clear descriptions of what each score means. "Scores that are floats are not great. LLM-as-judge does better with categorical integer scoring with clear explanations" [5].

5. **Constrain to structured outputs:** Use JSON format to remove ambiguity and enable standardized evaluation [5].

6. **Provide explanations (Chain-of-Thought):** Have judge explain reasoning before giving score. Improves consistency and helps human understanding of alerts [5].

7. **Score smoothing:** Reduce random fluctuations in raw scores. Re-run evaluations when "soft failures" occur [5].

**Prompting Best Practices (Arize) [6]:**
- Include explicit criteria in prompts
- Request chain-of-thought reasoning before final score
- Request structured outputs (JSON, bullet points)
- Randomize candidate order and evaluate both permutations to reduce position bias

**G-Eval Framework [7]:**
- Uses CoT prompting to stabilize LLM judges
- Generates evaluation steps from criteria, then uses form-filling paradigm for scoring
- State-of-the-art LLMs like GPT-4 align with human judgment up to 85% when used correctly [7]

**DAG (Decision Trees) [7]:**
- For deterministic evaluation, construct decision trees where nodes are LLM judges and edges represent decisions
- Useful when clear criteria exist (e.g., format correctness)

**Key Frameworks:**
- **DeepEval:** Open-source, runs locally, supports G-Eval, hallucination detection, answer relevancy [7]
- **Langfuse:** LLM-as-a-Judge evaluation with tracing
- **Arize Phoenix:** Pre-built evaluators, observability
- **Opik:** Prompt evaluation with experiment management

### Q3: What are the known limitations, biases, and failure modes of LLM-as-judge approaches?

**Presentation-Related Biases [8][9]:**
- **Position bias:** Judges favor responses based on position in prompt (e.g., first response in pairwise). Mitigation: swap positions and verify consistency, use PORTIA alignment-based approach [8].
- **Verbosity bias:** Judges prefer longer responses regardless of quality. Mitigation: persuasive debating techniques, CALM framework, contrastive judgments (Con-J) [8].

**Social-Related Biases [9]:**
- **Authority bias:** Judges favor responses with fake references/citations [9]
- **Bandwagon-effect bias:** Judges influenced by stated "popular opinion" in prompts [9]
- **Self-enhancement bias:** Judges favor outputs from their own model family (e.g., GPT-4 rates GPT-4 outputs higher) [3][4]

**Content-Related Biases [9]:**
- **Sentiment bias:** Judges may penalize responses with negative/angry tone even if technically correct [9]
- **Refinement-aware bias:** Judges give higher scores when told a response was "refined" [9]
- **Fallacy oversight:** Ignoring logical errors in reasoning [9]

**Cognitive-Related Biases [8]:**
- Narcissistic evaluators: Judges inflate scores for architecturally similar models [4]

**Inherent Weaknesses [4][8]:**
- Agreement with humans drops in specialized domains (medicine, law, low-resource languages)
- Judges miss subtle but critical errors while over-emphasizing surface fluency
- Difficulty grading responses to questions the judge itself struggles to answer (complex reasoning, math)
- Judges can be misled by incorrect information in context
- Sensitivity to prompt wording and domain shift
- Can inherit and amplify societal stereotypes

**Adversarial Vulnerabilities [4]:**
- LLMs can be fooled into labeling irrelevant documents as relevant
- Carefully crafted prompts can manipulate judge decisions

**Goodhart Effects [4]:**
- Optimizing against a fixed LLM judge can cause learned behavior to diverge from actual human preferences
- Training directly on judge labels may reduce reliability as proxy for true quality

**Practical Limitations [1][4]:**
- Not instant—slower than rule-based checks
- Cost at scale with large proprietary models
- Internal decision process difficult to interpret/audit
- Requires effort to set up and maintain

### Q4: Can smaller language models (SLMs) serve as effective judges, and under what conditions?

**The Gap Between SLMs and LLMs [10]:**
- JudgeBoard research reveals significant performance gap between SLMs and LLMs in isolated judging tasks
- However, multi-agent SLM systems can potentially match or exceed LLM performance through collaborative deliberation [10]

**Prometheus: The Leading Open-Source Judge Model [11][12]:**
- **Prometheus 2 (7B):** Lighter version requiring only 16GB VRAM, suitable for consumer GPUs. Achieves ~80% of 8x7B model's performance. Outperforms Llama-2-70B, on par with Mixtral-8x7B [11].
- **Prometheus 2 (8x7B):** State-of-the-art open evaluator. Pearson correlation 0.6-0.7 with GPT-4-1106 on direct assessment. 72-85% agreement with human judgments on pairwise ranking [11].
- **M-Prometheus (3B, 7B, 14B):** Latest iteration (2025), outperforms previous open judges on multilingual benchmarks. 7B and 14B surpass Prometheus 2 on RewardBench [11].

**Key Advantages of Fine-tuned SLM Judges [12]:**
- Can match or exceed GPT-4 evaluation quality when fine-tuned on domain-specific data
- Prometheus achieves 0.897 Pearson correlation with human evaluators (comparable to GPT-4)
- More critical and targeted feedback than proprietary models (less neutral/abstract)
- Humans prefer Prometheus feedback over GPT-4 in 58.67% of cases [12]

**Training Recipe for SLM Judges [12]:**
1. Solidify evaluation criteria with detailed descriptions
2. Prepare high-quality human evaluation data (1K-100K examples for fine-tuning)
3. Can use synthetic data (Prometheus trained almost entirely on GPT-4-generated data)
4. Focus on high-quality rationales—teach effective feedback formats
5. Use reference answers to address knowledge gaps in base model
6. Train with SFT, generate feedback before score

**Multi-Agent Approaches for SLMs [8][10]:**
- **MAJ (Multi-Agent Judging):** Multiple interacting SLMs with distinct reasoning profiles approximate LLM-level accuracy through collaborative deliberation [10]
- **PoLL:** Panel of smaller models as judges through max voting and average pooling—reduces intra-model bias [8]
- **LLM Jury:** Multiple smaller judges in parallel with majority/weighted voting—more robust to model-specific quirks [4]

**On-Prem Considerations:**
- Prometheus 2 7B needs ~16GB VRAM—runnable on single consumer GPU
- Open-source models avoid API costs and data privacy concerns
- Can be fine-tuned on domain-specific criteria
- Trade-off: Less flexible than proprietary models for arbitrary evaluation formats [12]

### Summary

LLM-as-a-Judge has emerged as the dominant paradigm for scalable LLM evaluation, achieving 80%+ agreement with human preferences when properly implemented. Three main evaluation modes exist: pairwise comparison (most reliable), pointwise scoring (most scalable), and reference-guided scoring.

**Key themes across sources:**
- **Prompt engineering is critical:** Chain-of-thought, structured outputs, clear rubrics, and few-shot examples significantly improve reliability [5][6][7].
- **Biases are well-documented but manageable:** Position, verbosity, and self-enhancement biases can be mitigated through position swapping, criteria decomposition, and multi-judge approaches [3][8][9].
- **SLMs can be effective judges:** Prometheus 2 (7B) achieves near-GPT-4 performance with 16GB VRAM, making on-prem deployment feasible [11][12]. Multi-agent SLM systems can match LLM performance [10].
- **Fine-tuning enables domain specialization:** Custom judge models outperform general-purpose LLMs on specific criteria [12].

**Contradictions/tensions:**
- Proprietary models offer flexibility but lack transparency; open models require training for new formats [12]
- Single powerful judge vs. ensemble of smaller judges—both approaches have merit [4][8]
- More examples in few-shot prompting doesn't always improve performance [5]

---

## Deeper Dive

### Subtopic 1: On-Prem Implementation Considerations

#### Q1: Infrastructure Requirements and Deployment Options

**Inference Framework Comparison [14][15]:**

| Framework | Best For | Key Features |
|-----------|----------|--------------|
| **vLLM** | Production, high concurrency | PagedAttention, continuous batching, 793 TPS peak vs Ollama's 41 TPS [14] |
| **Ollama** | Development, single-user | Simple CLI, Docker-like UX, offline by default [15] |

**vLLM advantages for production [14][15]:**
- Significantly higher throughput: handles hundreds of concurrent users
- Lower P99 latency (80ms vs 673ms at peak) 
- Dynamic batching keeps GPU utilization high
- Tensor parallelism for multi-GPU scaling

**Ollama advantages for development [15]:**
- Minimal setup, intuitive for beginners
- Works offline by default—good for air-gapped testing
- Abstracts away CUDA configuration complexity

**Hardware Requirements:**
- Prometheus 2 (7B): ~16GB VRAM—single consumer GPU (RTX 4090) [11]
- Prometheus 2 (8x7B): ~70GB+ VRAM—requires A100 or multi-GPU [11]
- 4-bit quantized 70B model: fits in 35GB VRAM (single A100) [16]

#### Q2: Balancing Quality vs. Computational Cost

**Top Cost Optimization Strategies [16]:**

1. **Prompt compression:** Reduce verbose prompts by 79% while maintaining quality. Tools like LLMLingua compress up to 20x [16].

2. **Semantic/token caching:** Cache computed key-value pairs for repeated prefixes. Anthropic's caching delivers 90% cost reduction on cached tokens [16].

3. **Model routing:** Route simple tasks to smaller models (GPT-4o-mini at 1/16th cost), reserve large models for complex reasoning [16].

4. **Batch inference:** vLLM's continuous batching improves throughput. OpenAI Batch API offers 50% discount for 24-hour turnaround [16].

5. **Quantization:** 4-bit quantization cuts memory 50-75% with minimal quality loss. Use bitsandbytes or GPTQ [16].

6. **Fine-tuning vs few-shot:** If few-shot adds 500 tokens/request at 100K requests/month = 50M tokens just for examples. Fine-tuning recovers cost in 2-3 months [16].

7. **Distillation:** Train 7B model on GPT-4 outputs—achieve 90% quality at fraction of cost, zero per-token cost in production [16].

**For on-prem LLM-as-judge specifically:**
- Use Prometheus 2 (7B) for routine evaluations—16GB VRAM, consumer GPU
- Reserve 8x7B or jury approaches for high-stakes decisions
- Batch evaluations during off-peak hours
- Cache evaluation results for identical inputs

### Subtopic 2: Practical Bias Mitigation Strategies

#### Q1: Position and Verbosity Bias Mitigation

**Position Bias (~40% GPT-4 inconsistency) [17][18]:**
- **Swap technique:** Evaluate both (A,B) and (B,A) orderings; only count consistent wins
- **Score-based:** Average scores across both orderings
- **Comparison-based:** Mark as tie if judgments inconsistent after swap
- **PORTIA approach:** Segment answers, align similar content, merge into single prompt for balanced comparison [8]

**Verbosity Bias (~15% score inflation) [17][18]:**
- Use 1-4 integer scales instead of continuous scores
- Explicitly reward conciseness in rubric
- CALM framework: controlled perturbations to assess verbosity impact [9]
- Contrastive judgments (Con-J): train on rationale pairs, not scalar scores [8]

**Self-Enhancement Bias (5-7% boost) [17]:**
- Use different model families as judges than the model being evaluated
- If evaluating Llama outputs, use GPT or Claude as judge

**Authority Bias [9]:**
- Include hallucination examples in prompt
- Instruct explicit claim verification
- Don't trust citations without verification

#### Q2: Ensemble/Jury Approaches with Limited Models

**LLM Jury Concept [19][20]:**
- Multiple LLM judges independently score, then aggregate via voting
- Research shows diverse panel of smaller models outperforms single large judge [19]
- Reduces bias 30-40% but costs 3-5x more [17]

**Aggregation Methods [19]:**
- **Max pooling:** Binary classification
- **Average/median pooling:** Rating scales
- **Majority voting:** Multi-class classification
- **Stacking:** Open-ended evaluations

**Practical Implementation [17][19]:**
- Run 3-5 models (e.g., GPT-4, Claude, Llama-3) with majority vote for critical evaluations
- Reserve jury approach for high-stakes decisions only due to cost
- Even with limited model variety, running same model with different prompts/temperatures can help

**Limitations of Juries [19]:**
- Managing multiple models adds complexity
- Smaller models may share similar biases if trained on similar data
- Cost savings vs single large model have narrowed as token prices dropped

### Subtopic 3: Purpose-Specific (On-Prem Rigor)

#### Q1: Enterprise Evaluation Criteria and Rubrics

**Designing Effective Rubrics [5][21]:**
- Use categorical integer scales (1-5) with clear descriptions per score
- Avoid float scores—LLMs perform better with discrete categories
- Decompose complex criteria into single-criterion evaluations
- Include explicit examples of each score level

**Grading Notes Approach (Databricks) [21]:**
- Annotate short "grading note" per question describing desired answer attributes
- Not comprehensive steps—just "spot-check" key solution ingredients
- Allows ambiguity where needed
- With grading notes: alignment with human judges increased to 96.3% (from baseline) [21]

**Enterprise-Specific Criteria:**
- **Factual accuracy:** Does response contain verifiable facts?
- **Policy adherence:** Does response follow organizational guidelines?
- **Completeness:** Are all required elements present?
- **Safety:** Does response avoid harmful content?
- **Format compliance:** Does output match required structure?

#### Q2: Validation and Calibration with Limited Resources

**Meta-Evaluation Metrics [22]:**
- **Percentage agreement:** Proportion of samples where LLM and human agree
- **Cohen's Kappa:** Agreement accounting for chance
- **Spearman correlation:** For continuous scores
- **Precision/recall:** Treating as classification problem

**Bootstrapping with Limited Labels [17][21]:**
- Start with 30-100 human-annotated examples for initial calibration
- Use these to measure judge agreement before scaling
- Iteratively refine rubrics based on disagreement patterns
- "Criteria drift"—manually grading outputs helps refine expectations [1]

**Practical Calibration Process:**
1. Create small benchmark (50-100 examples) with human labels
2. Run LLM judge on benchmark, measure agreement
3. Analyze disagreements—adjust rubric or prompt
4. Re-run until acceptable agreement (target: 80%+)
5. Periodically re-calibrate as judge model or domain changes

**Cost-Benefit at Scale [17]:**
- At 10,000 monthly evaluations, LLM judges save $50,000-100,000 vs human review
- Maintain 80% agreement—acceptable for screening, not final decisions
- Sample 1-5% of traffic for human spot-checks
- Use LLM judges for screening, humans for high-stakes final review

### Summary

The Deeper Dive research reveals practical considerations for implementing LLM-as-judge in resource-constrained on-prem environments:

**Infrastructure:** vLLM is the clear choice for production serving (20x throughput vs Ollama), while Ollama works for development. Prometheus 2 (7B) at 16GB VRAM is the sweet spot for on-prem judge deployment—achieves near-GPT-4 quality on consumer hardware [14][15][11].

**Cost optimization:** Prompt compression, caching, model routing, and quantization can reduce costs 50-90%. Fine-tuning or distillation eliminates per-token costs entirely for high-volume use cases [16].

**Bias mitigation:** Position swap technique (evaluate both orderings) and integer scoring scales are low-effort, high-impact. LLM juries reduce bias 30-40% but reserve for critical decisions due to 3-5x cost [17][18][19].

**Validation:** Start with 50-100 human-labeled examples for calibration. Target 80% agreement for screening use cases. Use grading notes (per-question guidance) to boost alignment to 96%+ [21][22].

**Key insight for on-prem:** The combination of Prometheus 2 (7B) + vLLM + careful prompt engineering can achieve production-quality evaluation at a fraction of the cost of API-based approaches, with full data privacy and no external dependencies.

