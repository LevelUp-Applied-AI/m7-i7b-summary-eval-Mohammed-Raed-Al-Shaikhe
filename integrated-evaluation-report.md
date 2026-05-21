# Module 7 Integrated Evaluation Report — Fine-Tuning vs. Pre-Trained Inference

> The Module 7 deliverable. Synthesizes Lab 7A (fine-tuning), Integration 7A (domain shift), Lab 7B (QA), and Integration 7B (summarization).

---

## 1. Comparison Table

| Task | Approach | Model | Training cost | Inference cost | Quality metric | Value |
|---|---|---|---|---|---|---|
| Sentiment classification (Lab 7A) | Fine-tuning | DistilBERT | ~30 min CPU + 3K labels | ~50 ms / example | Macro-F1 | 0.6356 |
| Domain transfer (Integration 7A) | Fine-tuned model out-of-domain | DistilBERT | already trained | ~50 ms / example | Domain-shift judgment | Strong negative-label bias and reduced confidence under domain shift |
| Extractive QA (Lab 7B) | Pre-trained inference | distilbert-base-cased-distilled-squad | 0 | ~50 ms / example | EM / token-F1 | EM = 0.3440, token-F1 = 0.4611 |
| Summarization (Integration 7B) | Pre-trained inference | distilbart-cnn-6-6 | 0 | ~3 sec / example | ROUGE-1 / 2 / L F1 | ROUGE-1 = 0.3691, ROUGE-2 = 0.1583, ROUGE-L = 0.2671 |

## 2. Findings

- Fine-tuning improved sentiment-classification performance on the app-review dataset, achieving a Macro-F1 score of 0.6356 after two training epochs.
- The sentiment classifier showed clear domain-shift problems when applied to tech and entertainment news articles, heavily favoring the negative class (610 negative predictions out of 1033 total predictions).
- The pre-trained QA model achieved EM = 0.3440 and token-F1 = 0.4611 without additional training, demonstrating moderate ability to extract answers from curated tech-news passages.
- The summarization model generated fluent summaries with ROUGE-1 = 0.3691 and ROUGE-L = 0.2671, but ROUGE alone could not fully measure factual faithfulness or completeness.
- Pre-trained inference pipelines required no labeled training data, making them cheaper and faster to deploy compared to fine-tuning workflows that required supervised data and longer training time.

## 3. Faithfulness Check

### Example A — high ROUGE

> Article excerpt: “Apple announced updates to its AI features and expanded device integration across its ecosystem.”
>
> Predicted summary: “Apple expanded its AI ecosystem integration and introduced new AI-powered device features.”
>
> ROUGE-1: 0.51; ROUGE-2: 0.34; ROUGE-L: 0.45
>
> Faithful? Yes. The summary accurately reflected the article’s main claims without adding unsupported information. In this case, the high ROUGE score aligned with strong factual accuracy.

### Example B — mid ROUGE

> Article excerpt: “Netflix reported subscriber growth after introducing a lower-cost ad-supported plan.”
>
> Predicted summary: “Netflix experienced subscriber growth following changes to its pricing strategy.”
>
> ROUGE-1: 0.33; ROUGE-2: 0.15; ROUGE-L: 0.24
>
> Faithful? Mostly yes. The summary captured the main idea but omitted the important detail that the growth was linked to the ad-supported plan. ROUGE partially reflected this missing detail.

### Example C — low ROUGE

> Article excerpt: “A gaming company delayed the launch of its upcoming title due to technical issues.”
>
> Predicted summary: “The company discussed future development plans and ongoing improvements.”
>
> ROUGE-1: 0.14; ROUGE-2: 0.03; ROUGE-L: 0.11
>
> Faithful? Partially. The summary remained loosely related to the article but failed to mention the key event — the delayed release. The low ROUGE score correctly reflected the poor summary quality.

## 4. Production Decision Matrix

| Scenario | Recommendation | Justification |
|---|---|---|
| Real-time app store review triage dashboard for a product team | Fine-tuning | The fine-tuned DistilBERT classifier achieved Macro-F1 = 0.6356 with low inference latency, making it more appropriate for high-volume sentiment analysis tasks using in-domain data. |
| Daily tech / entertainment news summary digest for an internal newsroom | Pre-trained inference | The summarization model achieved acceptable ROUGE scores without requiring labeled summarization data or expensive additional training. |
| Domain-expert QA on legal contracts | Fine-tuning | The pre-trained QA model achieved only EM = 0.3440 and token-F1 = 0.4611 on curated tech-news QA, suggesting that higher-stakes legal QA would require domain-specific adaptation and stronger precision. |

## 5. What You Would Do Differently

If I had access to a labeled summarization dataset focused specifically on tech and entertainment news, I would fine-tune the summarization model on that domain rather than relying only on a generic pre-trained model. A domain-specific summarization model could better learn the vocabulary, writing style, and entity patterns common in technology journalism, which would likely improve ROUGE scores and summary relevance. I would also add a faithfulness evaluation pipeline to detect hallucinated claims and missing entities because ROUGE alone does not reliably measure factual correctness.

## 6. Limits of the Evaluation

These evaluation metrics do not fully capture real-world model quality. ROUGE measures overlap with reference summaries but does not guarantee factual faithfulness or usefulness to readers. Similarly, EM and token-F1 measure textual overlap rather than reasoning quality or confidence calibration. The domain-shift analysis also showed that a model performing reasonably well in one domain can become biased and unreliable in another domain. In addition, latency was measured only on isolated CPU inference rather than under production traffic or concurrent load, so the reported timings may not reflect real deployment performance.