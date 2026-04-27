# Model Card — Credit Scoring Decision-Support Model

## 1. Model Overview

| Item | Description |
|---|---|
| Project | Responsible AI Governance for Credit Scoring |
| Model | Logistic regression classifier |
| Use case | Credit risk decision support |
| Dataset | Statlog German Credit Data |
| Target | Predict whether an applicant is a bad credit risk |
| Positive class | `bad_risk = 1` |
| Selected threshold | `0.25` |
| Status | Portfolio simulation |
| Production-ready | No |

This model card documents a simulated credit scoring decision-support model. It is intended to show how model performance, fairness, explainability, threshold selection and governance controls can be documented in a regulated financial services context.

---

## 2. Intended Use

The model is designed to support credit officers in reviewing personal credit applications.

In the simulated scenario:

- the model estimates the probability that an applicant represents a bad credit risk;
- the output is used as one input in a broader credit assessment;
- final credit decisions remain human-led;
- decisions remain subject to credit policy, affordability checks, legal requirements and customer context;
- model outputs must not be used as the sole basis for approval or rejection.

### Out of Scope

The model must not be used for:

- fully automated credit decisions;
- real lending decisions;
- production deployment;
- regulatory submissions;
- customer-facing adverse action notices;
- affordability assessment;
- pricing or limit assignment;
- fraud detection;
- collections or debt recovery.

---

## 3. Dataset

| Item | Description |
|---|---|
| Source | UCI Machine Learning Repository |
| Dataset | Statlog German Credit Data |
| Observations | 1,000 |
| Input features | 20 |
| Target distribution | 700 good credit risks / 300 bad credit risks |
| Bad-risk rate | 30.0% |
| Missing values | None detected in the loaded file |
| Feature types | Mixed categorical and numerical variables |

The dataset includes credit history, checking account status, credit amount, duration, savings, employment, age, housing, job, personal status/sex and foreign worker status.

Main limitation: the dataset is small, historical, weakly documented, lacks timestamps, longitudinal repayment, affordability, income, complaint and override data, and includes sensitive or proxy-sensitive variables requiring careful treatment.

---

## 4. Sensitive and Proxy-Sensitive Variables

The dataset includes variables that require explicit governance treatment:

| Variable | Governance concern |
|---|---|
| `personal_status_sex` | Encodes sex and marital/personal status in a combined variable |
| `age_years` | Age can be legally sensitive or regulated depending on context |
| `foreign_worker` | May proxy nationality, migration status or ethnic origin |
| `housing` | May proxy socioeconomic status and wealth |
| `job` | May proxy socioeconomic status, education or labour market segmentation |
| `property` | Wealth/security proxy |
| `telephone` | Outdated socioeconomic proxy |

These variables are useful for fairness analysis in this portfolio project. Their use in a real credit model would require legal basis, necessity assessment, proportionality review and formal approval.

---

## 5. Model Development Summary

The notebook workflow used:

1. governance-oriented data exploration;
2. train/test split;
3. baseline model comparison;
4. selection of a candidate model;
5. cost-sensitive threshold analysis;
6. fairness and explainability review.

Train/test split:

| Split | Rows | Bad-risk rate |
|---|---:|---:|
| Train | 700 | 30.0% |
| Test | 300 | 30.0% |

Candidate model selection was based on highest ROC-AUC on the holdout test set. This is acceptable for the portfolio simulation, but would be insufficient for production model approval.

---

## 6. Model Performance

### Baseline Comparison at Threshold 0.50

| Model | ROC-AUC | Average precision | Brier score | Accuracy | Bad-risk precision | Bad-risk recall |
|---|---:|---:|---:|---:|---:|---:|
| Dummy stratified | 0.585 | 0.350 | 0.350 | 0.650 | 0.418 | 0.422 |
| Logistic regression | 0.801 | 0.645 | 0.157 | 0.783 | 0.676 | 0.533 |
| Random forest | 0.787 | 0.604 | 0.175 | 0.727 | 0.833 | 0.111 |

The logistic regression model was selected as the candidate model because it had the strongest ROC-AUC and Brier score among the tested baselines, while remaining relatively interpretable.

### Candidate Model at Default Threshold 0.50

| Metric | Value |
|---|---:|
| ROC-AUC | 0.801 |
| Average precision | 0.645 |
| Brier score | 0.157 |
| Accuracy | 0.783 |
| Bad-risk precision | 0.676 |
| Bad-risk recall | 0.533 |
| False positives | 23 |
| False negatives | 42 |

At the default threshold, the model misses 42 out of 90 bad-risk cases in the test set. This is a material issue in a credit-risk context.

---

## 7. Threshold Selection

The UCI dataset includes a cost-sensitive framing where misclassifying a bad credit risk as good is more costly than misclassifying a good credit risk as bad.

For this project:

- false positive cost = 1;
- false negative cost = 5.

The selected threshold is **0.25**, based on the lowest expected misclassification cost among tested thresholds.

| Threshold | Accuracy | Bad-risk precision | Bad-risk recall | False positives | False negatives | High-risk flag rate | Expected cost |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0.50 | 0.783 | 0.676 | 0.533 | 23 | 42 | 23.7% | 233 |
| 0.25 | 0.713 | 0.514 | 0.811 | 69 | 17 | 47.3% | 154 |

The 0.25 threshold reduces false negatives from 42 to 17, but increases false positives from 23 to 69. It also increases the high-risk/manual-review flag rate from 23.7% to 47.3%.

This threshold should be treated as a governance decision, not a purely technical optimization.

---

## 8. Fairness Review Summary

Fairness analysis was performed at the selected threshold of 0.25.

The model flags 47.3% of test applicants as bad risk / high risk.

Largest observed gaps across eligible subgroup comparisons:

| Metric | Largest observed gap | Feature | Groups |
|---|---:|---|---|
| High-risk flag rate | 0.372 | `property` | unknown / no property vs real estate |
| False positive rate | 0.362 | `property` | unknown / no property vs real estate |
| False negative rate | 0.258 | `property` | building society/life insurance vs unknown / no property |

Important caveat: fairness metrics are calculated on a 300-row test set and should be treated as risk signals, not definitive evidence of discriminatory behaviour. Several subgroup results are unstable because subgroup sizes are below 20 observations.

---

## 9. Explainability Summary

Permutation importance identified the following top model drivers:

| Rank | Feature | Interpretation |
|---:|---|---|
| 1 | `checking_account_status` | Liquidity / financial situation proxy |
| 2 | `credit_history` | Past credit behaviour |
| 3 | `duration_months` | Loan duration |
| 4 | `purpose` | Loan purpose |
| 5 | `credit_amount` | Credit exposure |

Sensitive or proxy-sensitive variables with positive permutation importance include:

- `housing`;
- `personal_status_sex`;
- `property`;
- `age_years`;
- `foreign_worker`.

This does not automatically make the model invalid, but it creates a clear governance issue: these variables require explicit policy treatment before any real-world use.

Logistic regression coefficients were also reviewed. They provide useful transparency, but they should not be interpreted causally or used directly as customer-facing reasons without business, legal and compliance review.

---

## 10. Human Oversight Requirements

The model should only be used with structured human oversight.

Minimum requirements:

| Control | Requirement |
|---|---|
| Human decision-maker | A credit officer remains responsible for the final decision |
| No automatic rejection | A high-risk flag cannot automatically reject an application |
| Override process | Credit officers can override the model, with reasons logged |
| Explanation use | Explanations support review but do not replace credit policy |
| Sensitive variables | Use must be reviewed by Compliance / Legal / Model Risk |
| Adverse decisions | Final decision rationale must be based on approved credit policy, not only model score |
| Escalation | Borderline and high-impact cases should be escalated for second-line review |

---

## 11. Monitoring Requirements

If this model were used in a controlled pilot, monitoring should include:

| Area | Indicators |
|---|---|
| Performance | ROC-AUC, average precision, Brier score, confusion matrix |
| Threshold stability | High-risk flag rate at selected threshold |
| Fairness | Subgroup high-risk flag rate, false positive rate, false negative rate |
| Data drift | Distribution changes in top model drivers |
| Explainability | Stability of top feature importance rankings |
| Human oversight | Override rates, manual review outcomes, policy exceptions |
| Customer impact | Complaints, appeals, adverse decision patterns |

Monitoring should combine quantitative thresholds with expert review because the dataset is small and subgroup metrics can be unstable.

---

## 12. Key Risks and Limitations

| Risk | Description | Mitigation |
|---|---|---|
| Dataset age | Historical dataset may not reflect modern credit practices | Treat as portfolio simulation only |
| Small sample size | Fairness and subgroup metrics are unstable | Use results as risk signals, not conclusions |
| Sensitive/proxy variables | Several variables may raise fairness or legal concerns | Require explicit policy and legal review |
| Threshold impact | Lower threshold reduces false negatives but increases customer friction | Require governance approval of threshold |
| Limited monitoring evidence | No timestamps, overrides, complaints or longitudinal outcomes | Monitoring plan remains scenario-based |
| Explainability limits | Coefficients and permutation importance are not causal explanations | Use explanations as internal decision support only |
| Regulatory incompleteness | Analysis is not a full conformity or compliance assessment | Document as portfolio simulation |

---

## 13. Approval Position

For this simulated project, the model would be classified as:

**Not approved for production use.**

Potentially acceptable only as:

- an educational Responsible AI case study;
- a controlled internal prototype;
- a decision-support pilot subject to additional validation;
- an example for model governance documentation.

Before any real deployment, the following would be required:

- legal basis and use-case classification;
- independent model validation;
- representative and current training data;
- full fairness assessment;
- calibration review;
- production monitoring design;
- human oversight procedure;
- customer communication review;
- model risk approval.

---

## 14. References

This model card structure is aligned with common model card practice: intended use, data, evaluation, ethical considerations, limitations and recommendations. It also reflects practical risk-management concepts from NIST AI RMF and high-impact AI governance expectations under the EU AI Act.
