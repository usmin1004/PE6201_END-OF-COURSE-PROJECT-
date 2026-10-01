# Data Explainer

This file explains the data used for the Influencer Reply Triage project, how the labels were produced, and how the development and held-out splits were kept separate.

## Data purpose

The data represent replies from influencers invited to a cosmetics campaign. Each reply is labelled with one of five categories:

- `Interested`
- `Declined`
- `Needs Information`
- `Negotiation`
- `Unclear`

The data also record whether a reply contains a hidden condition and whether the expected action is Manual Review. Hidden conditions include material restrictions involving exclusivity, timing, payment, products, usage rights, location, or deliverables.

The datasets are synthetic because real influencer correspondence may contain private or commercially sensitive information. Synthetic data made it possible to publish the full evaluation evidence, but it cannot represent every pattern found in real campaigns.

## Development dataset

File: [`dev_60.csv`](dev_60.csv)

- 60 replies generated with ChatGPT using [`dev_generation_prompt.txt`](dev_generation_prompt.txt)
- 12 replies in each of the five categories
- 8 hidden-condition cases
- all 60 labels manually reviewed and marked `Approved` before evaluation
- used for baseline testing, pilot testing, error analysis, and prompt improvement

The development data were used to create Prompt v2. Performance on this split therefore measures development performance rather than final generalisation.

## Held-out dataset

File: [`heldout_40.csv`](heldout_40.csv)

- 40 replies independently generated with Claude using [`heldout_generation_prompt.txt`](heldout_generation_prompt.txt)
- 8 replies in each of the five categories
- 7 hidden-condition cases
- all 40 labels manually reviewed and marked `Approved` before the final run
- no duplicate IDs or duplicate reply texts
- committed before the frozen Prompt v2 evaluation

Claude was used for the held-out data because ChatGPT had generated the development data. This reduced dependence on one generator's phrasing. The held-out cases were not used to revise Prompt v2.

## Important columns

| Column | Meaning |
|---|---|
| `id` | Stable case identifier such as `D001` or `H001`. |
| `reply_text` | Influencer reply provided to the classifier. |
| `true_category` | Manually reviewed ground-truth category. |
| `has_hidden_condition` | Whether the reply contains a material condition that may be easy to miss. |
| `expected_manual_review` | Ground-truth decision for human escalation. |
| `condition_type` | Type of condition, when applicable. |
| `difficulty` | Intended difficulty of the synthetic case. |
| `tone` | Tone variation used during generation. |
| `length_bucket` | Approximate reply-length group. |
| `label_rationale` | Reason for the assigned ground-truth category. |
| `expected_requests` | Requests expected to be extracted from the reply. |
| `expected_conditions` | Material conditions expected to be extracted. |
| `generation_model` | Model used to generate the synthetic reply. |
| `split` | `dev` or `heldout`. |
| `human_review_status` | Manual label-review status. Only approved cases were evaluated. |

## Leakage controls

Only `reply_text` was sent to the classification model. The model did not receive `true_category`, hidden-condition labels, expected Manual Review decisions, rationales, or other evaluation metadata.

Prompt v2 was frozen before the held-out run. The held-out results were used for final evaluation and failure analysis, not for another prompt revision.

## Data limitations

- Both datasets are synthetic.
- The held-out set contains only 40 cases.
- Generation by two different models improves independence but does not reproduce real campaign distribution.
- Tone, language, cultural context, and contractual wording may be more varied in real messages.
- A production pilot should use consented, anonymised replies and continuing human audits.

## Related files

| File | Purpose |
|---|---|
| [`dev_generation_prompt.txt`](dev_generation_prompt.txt) | Reproducible prompt for the development dataset. |
| [`heldout_generation_prompt.txt`](heldout_generation_prompt.txt) | Reproducible prompt for the independent held-out dataset. |
| [`pilot_10_results.csv`](pilot_10_results.csv) | Initial 10-case LLM pilot evidence. |
| [`remaining_50_results.csv`](remaining_50_results.csv) | Results for the other 50 development cases. |
| [`dev_60_results.csv`](dev_60_results.csv) | Prompt v1 results for all 60 development cases. |
| [`prompt_v2_dev_60_results.csv`](prompt_v2_dev_60_results.csv) | Prompt v2 results for all 60 development cases. |
| [`heldout_40_predictions.csv`](heldout_40_predictions.csv) | Raw records from the single final held-out run. |
| [`heldout_40_results.csv`](heldout_40_results.csv) | Held-out predictions with scored evaluation columns. |
