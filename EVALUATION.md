# Evaluation Explainer

This file explains the evaluation sequence, the meaning of each metric, the evidence files, and the limitations of the final results.

## Evaluation principle

The project uses development results for prompt improvement and an independently generated held-out set for final evaluation. Prompt v2 was frozen before the held-out run. Only `reply_text` was sent to the model, while labels and evaluation metadata remained hidden.

The final evidence is the single frozen Prompt v2 run on 40 held-out replies.

## Target metrics

| Metric | Target | Final held-out result |
|---|---:|---:|
| Category accuracy | At least 80% | **95.0% (38/40)** |
| Hidden-condition escalation | 100% | **100% (7/7)** |
| Valid structured JSON | 100% | **97.5% (39/40)** |
| Human decision authority | 100%; no autonomous messages or contract decisions | **100% by design** |

Latency and cost were measured as operational metrics. They did not receive artificial pass thresholds after the experiment.

## Evaluation sequence

### 1 Majority-class baseline

The development set is balanced, so predicting one category for every reply produces 20.0% accuracy. This is the honest minimum-performance reference.

### 2 Keyword-rule baseline

A transparent keyword classifier reached 68.3% development accuracy. It handled explicit phrases but struggled with paraphrases, mixed intent, and restrictions appearing late in a reply.

### 3 Ten-case pilot

The initial pilot used 10 development cases before the remaining batch was run.

- Category accuracy: 90.0%
- Manual Review accuracy: 100.0%
- Valid JSON: 100.0%
- Hidden conditions sent to Manual Review: 1/1
- Evidence: [`pilot_10_results.csv`](pilot_10_results.csv)

### 4 Prompt v1 development evaluation

Prompt v1 was evaluated across all 60 development cases.

- Category accuracy: 96.7%
- Manual Review accuracy: 98.3%
- Valid JSON: 100.0%
- Hidden conditions sent to Manual Review: 8/8
- Evidence: [`dev_60_results.csv`](dev_60_results.csv)

The category errors involved conflicting intent and an irrelevant question. These failures informed the Prompt v2 rules.

### 5 Prompt v2 development evaluation

Prompt v2 clarified the decision order, limited Needs Information to campaign-relevant questions, and required conflicting or irrelevant intent to be classified as Unclear and manually reviewed.

- Category accuracy: 100.0%
- Manual Review accuracy: 100.0%
- Evidence: [`prompt_v2_dev_60_results.csv`](prompt_v2_dev_60_results.csv)
- Case-level comparison: [`prompt_v1_v2_comparison.csv`](prompt_v1_v2_comparison.csv)

The 100% result is a development result and is not the final reported generalisation performance.

### 6 Frozen Prompt v2 held-out evaluation

Prompt v2 was frozen and evaluated once on the 40 independently generated held-out replies.

| Metric | Result |
|---|---:|
| Correct categories | 38/40 |
| Category accuracy | **95.0%** |
| Manual Review accuracy | **97.5%** |
| Valid JSON rate | **97.5% (39/40)** |
| Hidden-condition cases sent to Manual Review | **7/7** |
| Replies sent to Manual Review | **15/40 (37.5%)** |
| Category errors caught by Manual Review | **1/2** |
| Average latency | **0.803 seconds** |
| Actual cost for 40 replies | **USD 0.004424** |
| Projected cost for 50 replies | **USD 0.005530** |

Evidence:

- raw run records: [`heldout_40_predictions.csv`](heldout_40_predictions.csv)
- scored results: [`heldout_40_results.csv`](heldout_40_results.csv)

## Metric definitions

| Metric | Calculation or interpretation |
|---|---|
| Category accuracy | Correct category predictions divided by all evaluated replies. |
| Manual Review accuracy | Correct Manual Review decisions divided by all evaluated replies. |
| Valid JSON rate | Replies producing output that passed the strict JSON validity check divided by all replies. |
| Hidden-condition escalation | Hidden-condition cases with `predicted_manual_review = True` divided by all hidden-condition cases. |
| Review rate | Replies sent to Manual Review divided by all evaluated replies. |
| Error capture | Incorrect category predictions that were nevertheless sent to Manual Review divided by all category errors. |
| Average latency | Mean measured model-call latency across evaluated replies. |
| Model cost | Token counts multiplied by the configured OpenRouter input and output prices. |

## Failure analysis

### H006

- Ground truth: `Negotiation`
- Prediction: `Unclear`
- Manual Review: `True`

The model response failed strict JSON validation and triggered the safe fallback. The category was wrong, but the case was still escalated to Hana. This is a safe failure from the business-process perspective.

### H031

- Ground truth: `Unclear`
- Prediction: `Declined`
- Manual Review: `False`

The reply contained conflicting signals but was treated as a simple decline. Because the system did not request review, this was the main silent failure. It shows why the prototype should remain supervised.

## Evaluation critique

- Forty held-out cases provide useful evidence but are not enough to establish production reliability.
- Both datasets are synthetic, so measured performance may be optimistic relative to real messages.
- The project quantified category and Manual Review performance, but it did not assign a separate extraction-accuracy score to requests and conditions.
- Manual Review caught one of two category errors, so escalation reduces risk but does not remove it.
- Prompt v2 reached 100% on development data, but only the 95.0% held-out result should be treated as final performance.

## Recommended next evaluation

A small supervised pilot should use consented, anonymised real replies. The pilot evaluation should audit both escalated and non-escalated samples, monitor invalid JSON, measure request-and-condition extraction accuracy, and specifically test conflicting acceptance and refusal language.
