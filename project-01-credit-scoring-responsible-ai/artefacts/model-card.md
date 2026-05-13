# Model Card — Credit Risk Decision-Support Model

## 1. Model Overview

| Item | Description |
|---|---|
| Model name | Credit risk decision-support model |
| Model type | Calibrated logistic regression |
| Candidate model | `logistic_regression_C1_calibrated_sigmoid` |
| Use case | Credit risk decision support for loan application review |
| Dataset | Statlog German Credit Dataset |
| Positive class | Bad credit risk |
| Selected threshold | `0.16` |
| Threshold basis | Cost-sensitive threshold selected on validation set |
| Intended users | Credit risk, model governance, Responsible AI, compliance and business review teams |
| Status | Portfolio case study — not production-ready |

---

## 2. Intended Use

The model is designed as a **decision-support tool** for a simulated credit risk review process.

It predicts whether an applicant is likely to represent a higher credit risk. The model output is a probability score for `bad credit risk`.

The model is intended to support questions such as:

- which applications may require closer review;
- how different thresholds change business and customer impact;
- whether model outcomes differ across relevant subgroups;
- what controls would be needed before any operational use.

The model is **not** intended to make fully automated credit approval or rejection decisions.

---

## 3. Non-Intended Use

This model should not be used for:

- fully automated credit rejection;
- real lending decisions;
- customer-facing credit scoring;
- legal or regulatory compliance conclusions;
- production deployment;
- individual creditworthiness assessment;
- benchmarking modern credit risk systems.

The dataset is small, dated and simplified. The model is useful for Responsible AI governance demonstration, not for commercial credit scoring.

---

## 4. Dataset

The project uses the **Statlog German Credit Dataset**.

The target variable is converted as follows:

| Original outcome | Model target |
|---|---:|
| Good credit risk | `0` |
| Bad credit risk | `1` |

The following subgroup attributes are used for Responsible AI testing:

| Attribute | Source / derivation | Use |
|---|---|---|
| Gender group | Derived from `personal_status` | Fairness testing |
| Age group | Derived from `age` | Fairness testing |
| Foreign worker status | Dataset feature | Fairness and proxy-sensitive feature review |

These subgroup attributes are used for testing and governance review. They are not treated as a complete legal discrimination assessment.

---

## 5. Model Development Approach

A small set of candidate models was tested:

- logistic regression variants;
- random forest variants;
- histogram gradient boosting.

The final candidate model is a calibrated logistic regression.

This was selected because it offered a good balance between:

- predictive performance;
- interpretability;
- governance reviewability;
- ease of explanation;
- suitability for a Responsible AI case study.

The model was developed using a train / validation / test split:

| Split | Purpose |
|---|---|
| Training set | Fit candidate models |
| Validation set | Compare models and select threshold |
| Test set | Final held-out evaluation |

The selected operating threshold was chosen on the validation set, then evaluated on the held-out test set.

---

## 6. Performance Summary

Held-out test set results:

| Metric | Value |
|---|---:|
| ROC AUC | 0.780 |
| Average precision | 0.612 |
| Brier score | 0.165 |

Interpretation:

- The model has reasonable ranking ability for a simple baseline.
- Probability quality is acceptable for a portfolio case study, but not sufficient for production use without further validation.
- The model should be treated as a candidate for governance review, not as a deployable credit model.

---

## 7. Threshold Selection

The model predicts the probability of bad credit risk.

Two thresholds were reviewed:

| Threshold | Purpose |
|---:|---|
| `0.50` | Default classification threshold |
| `0.16` | Cost-sensitive threshold selected on validation set |

The threshold was selected using the German Credit asymmetric cost assumption:

| Error type | Interpretation | Assumed cost |
|---|---|---:|
| False negative | Bad credit risk classified as good | 5 |
| False positive | Good credit risk classified as bad | 1 |

This makes threshold selection a governance-relevant decision, not just a technical parameter.

---

## 8. Threshold Impact

Held-out test set comparison:

| Threshold | High-risk flag rate | False positives | False negatives | Interpretation |
|---:|---:|---:|---:|---|
| `0.50` | 21.0% | 16 | 34 | Conservative flagging, but many bad-risk applicants missed |
| `0.16` | 71.5% | 90 | 7 | Better bad-risk detection, but high customer-friction risk |

The selected threshold substantially reduces missed bad-risk applicants. However, it also flags a very large share of applicants as high risk.

This is the main model governance finding.

The selected threshold should **not** be treated as a deployment recommendation without further review.

---

## 9. Fairness and Subgroup Review

Fairness testing was performed on:

- gender group;
- age group;
- foreign worker status.

The fairness review found:

| Finding | Severity |
|---|---|
| Cost-sensitive threshold creates high flagging rate | High |
| Age-group disparity at default threshold | Medium |
| Foreign-worker-status disparity at default threshold | Medium |
| Age-group disparity at selected threshold | High |
| Foreign-worker-status disparity at selected threshold | High |
| Main model drivers require business and proxy review | Medium |

Main interpretation:

- No major gender disparity was identified in this test sample.
- Age group triggered review findings, especially at the selected threshold.
- Foreign worker status triggered high-priority review findings, but subgroup size is small, so results should be interpreted cautiously.
- The selected threshold increases fairness and customer-impact concerns.

These findings are review triggers, not legal conclusions.

---

## 10. Explainability Summary

Global explainability was assessed using permutation importance.

Top model drivers:

| Feature | Importance |
|---|---:|
| `checking_status` | 0.0916 |
| `credit_history` | 0.0322 |
| `credit_amount` | 0.0307 |
| `installment_commitment` | 0.0195 |
| `duration` | 0.0179 |
| `purpose` | 0.0150 |
| `savings_status` | 0.0045 |
| `foreign_worker` | 0.0040 |

Interpretation:

- Most top drivers are broadly credit-relevant.
- `foreign_worker` appears among the top features and should be reviewed as a proxy-sensitive variable.
- Feature relevance should be reviewed with credit risk, legal/compliance and business stakeholders before any operational use.
- Explainability evidence supports further review, not deployment approval.

---

## 11. Main Risks

| Risk | Description | Severity |
|---|---|---|
| High customer-friction risk | The selected threshold flags 71.5% of applicants as high risk | High |
| False positive burden | Many good-risk applicants are incorrectly flagged at the selected threshold | High |
| Age-group disparity | Age-group outcomes differ materially under the selected threshold | High |
| Foreign-worker-status review trigger | Foreign worker status shows disparity signals, with small subgroup size | High |
| Proxy-sensitive feature use | `foreign_worker` appears among the top model drivers | Medium |
| Dataset limitation | German Credit is small, dated and not representative of a modern banking portfolio | High |
| Overreliance risk | Users could treat the model score as a decision rather than decision support | High |

---

## 12. Required Controls Before Any Operational Use

The model should not be used for automated rejection.

Minimum controls before any controlled pilot:

| Control | Purpose |
|---|---|
| Human review | Prevent direct adverse decisions based only on model output |
| Threshold review | Assess whether threshold `0.16` is operationally acceptable |
| Subgroup monitoring | Track outcomes and error rates by relevant groups |
| Feature review | Assess sensitive and proxy-sensitive variables |
| Reason-code review | Ensure explanations are meaningful and appropriate |
| Manual override process | Allow credit officers to challenge model output |
| Periodic reassessment | Re-test performance, calibration and fairness over time |
| Documentation | Maintain model card, testing summary and risk-control evidence |

---

## 13. Recommended Use Decision

Recommended status:

```text
Approve for controlled testing only.
Do not approve for fully automated rejection.
```

Rationale:

- The model has reasonable baseline performance.
- The selected threshold reduces missed bad-risk applicants.
- However, the selected threshold creates high customer-friction risk.
- Fairness review identifies age-group and foreign-worker-status review triggers.
- Proxy-sensitive features require further review.
- The dataset is not sufficient for production-grade validation.

The model is suitable for demonstrating Responsible AI review and governance controls, not for production use.

---

## 14. Limitations

Main limitations:

- The dataset is small and dated.
- The dataset does not represent a modern European banking portfolio.
- Subgroup attributes are limited and imperfect.
- Some subgroup results are unstable due to small sample size.
- The cost matrix is simplified.
- The model is trained on public data, not real institutional credit data.
- No out-of-time validation was performed.
- No production monitoring was implemented.
- Fairness findings are illustrative, not legal conclusions.

---

## 15. Open Questions

Before any real-world use, the following questions would need to be answered:

1. Is the cost assumption appropriate for the institution’s actual credit risk appetite?
2. Is threshold `0.16` operationally feasible given the high flagging rate?
3. Are age-related outcome differences acceptable under applicable policy and law?
4. Should `foreign_worker` be excluded, transformed or subject to stricter review?
5. Can credit officers receive usable explanations for adverse or borderline cases?
6. What monitoring thresholds should trigger model review or suspension?
7. How should applicants contest or challenge adverse outcomes influenced by the model?

---

## 16. Conclusion

The candidate model is a useful baseline for Responsible AI review.

It demonstrates how a simple credit risk model can be assessed beyond standard performance metrics. The key issue is not whether the model has acceptable ROC AUC. The key issue is how threshold choice changes customer impact, fairness risk and operational workload.

The model should proceed only as a controlled decision-support candidate, with human review, threshold governance, subgroup monitoring and proxy-sensitive feature review.
