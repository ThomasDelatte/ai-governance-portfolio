# Fairness Assessment — Credit Scoring Decision-Support Model

## 1. Assessment Summary

| Item | Description |
|---|---|
| Project | Responsible AI Governance for Credit Scoring |
| Dataset | Statlog German Credit Data |
| Model | Logistic regression classifier |
| Positive class | `bad_risk = 1` |
| Selected threshold | `0.25` |
| Assessment type | Portfolio simulation / governance review |
| Production-ready | No |

This assessment reviews fairness risks in a simulated credit scoring decision-support model. It focuses on subgroup outcomes, sensitive and proxy-sensitive variables, and governance controls required before any real use.

The results should be read as **risk signals**, not as evidence of discrimination. The test set contains only 300 observations, and several subgroup sizes are too small for robust conclusions.

---

## 2. Scope

### In Scope

- sensitive and proxy-sensitive feature review;
- subgroup bad-risk rates in the dataset;
- subgroup model performance at the selected threshold;
- high-risk flag rate differences;
- false positive and false negative rate differences;
- explainability signals linked to sensitive or proxy-sensitive features;
- governance mitigations.

### Out of Scope

- legal conclusion on discrimination;
- full equal treatment law assessment;
- production fairness validation;
- causal analysis;
- intersectional fairness analysis;
- customer-level adverse action review;
- reject inference;
- fairness testing on current bank data.

---

## 3. Dataset and Model Context

The dataset contains:

| Item | Value |
|---|---:|
| Observations | 1,000 |
| Input features | 20 |
| Good credit risks | 700 |
| Bad credit risks | 300 |
| Bad-risk rate | 30.0% |
| Test set size | 300 |
| Test bad-risk rate | 30.0% |

Dataset limitation: the dataset is small, historical, weakly documented, lacks timestamps, longitudinal repayment, affordability, income, complaint and override data, and includes sensitive or proxy-sensitive variables requiring careful treatment.

The selected model is a logistic regression classifier. At the selected threshold of `0.25`, the model flags **47.3%** of test applicants as bad risk / high risk.

---

## 4. Sensitive and Proxy-Sensitive Variables

| Variable | Risk type | Fairness concern |
|---|---|---|
| `personal_status_sex` | Sensitive / proxy-sensitive | Combines sex and marital or personal status |
| `age_years` | Sensitive / regulated depending on context | Age may require legal and proportionality review |
| `foreign_worker` | Sensitive / proxy-sensitive | May proxy nationality, migration status or ethnic origin |
| `housing` | Proxy-sensitive | May reflect socioeconomic status and wealth |
| `job` | Proxy-sensitive | May reflect socioeconomic status, education or labour market segmentation |
| `property` | Proxy-sensitive | Wealth/security proxy |
| `telephone` | Outdated socioeconomic proxy | Historical phone registration may reflect socioeconomic status |

For this portfolio project, these variables are used to identify fairness risks. In a real model, their use would require explicit legal basis, necessity assessment, proportionality review and approval.

---

## 5. Dataset-Level Fairness Signals

The dataset itself contains meaningful differences in observed bad-risk rates across groups.

Selected examples:

| Feature | Group | Count | Bad-risk rate |
|---|---|---:|---:|
| `age_band` | `<=25` | 190 | 42.1% |
| `age_band` | `36-50` | 299 | 23.7% |
| `property` | `unknown / no property` | 154 | 43.5% |
| `property` | `real estate` | 282 | 21.3% |
| `housing` | `for free` | 108 | 40.7% |
| `housing` | `own` | 713 | 26.1% |
| `personal_status_sex` | `male - divorced/separated` | 50 | 40.0% |
| `personal_status_sex` | `male - single` | 548 | 26.6% |

These differences may reflect real credit-risk patterns, historical lending practices, sample bias, socioeconomic effects, or proxy discrimination. The dataset does not contain enough context to distinguish these explanations.

Governance implication: fairness review cannot start only after model training. The input data already contains structural differences that affect downstream model behaviour.

---

## 6. Model Fairness Results

Fairness metrics were calculated on the 300-row test set at the selected threshold of `0.25`.

At this threshold:

| Metric | Value |
|---|---:|
| High-risk flag rate | 47.3% |
| Bad-risk precision | 51.4% |
| Bad-risk recall | 81.1% |
| False positive rate | 32.9% |
| False negative rate | 18.9% |

The threshold improves detection of bad-risk cases compared with the default `0.50` threshold, but increases the number of applicants flagged as high risk.

---

## 7. Subgroup Results

### Largest Observed Gaps

Across subgroup comparisons with minimum group size of 20:

| Metric | Largest gap | Feature | Groups |
|---|---:|---|---|
| High-risk flag rate | 0.372 | `property` | unknown / no property vs real estate |
| False positive rate | 0.362 | `property` | unknown / no property vs real estate |
| False negative rate | 0.258 | `property` | building society/life insurance vs unknown / no property |

The strongest fairness signal appears around `property`, a wealth/security proxy.

### Selected Subgroup Examples

| Feature | Group | Count | High-risk flag rate | False positive rate | False negative rate |
|---|---|---:|---:|---:|---:|
| `property` | unknown / no property | 43 | 72.1% | 59.3% | 6.3% |
| `property` | real estate | 83 | 34.9% | 23.1% | 22.2% |
| `housing` | rent | 56 | 62.5% | 44.4% | 5.0% |
| `housing` | own | 211 | 41.2% | 28.6% | 24.6% |
| `age_band` | 26-35 | 105 | 56.2% | 44.0% | 13.3% |
| `age_band` | 51-65 | 31 | 32.3% | 24.0% | 33.3% |
| `personal_status_sex` | female - divorced/separated/married | 101 | 55.4% | 36.4% | 8.6% |
| `personal_status_sex` | male - single | 162 | 39.5% | 26.9% | 25.6% |

Interpretation: some groups are more likely to be flagged as high risk and may experience more false positives. This matters because false positives can lead to additional scrutiny, delays or potential exclusion from credit.

---

## 8. Small Subgroup Warning

Several subgroup results are unstable because group sizes are below 20 observations.

| Feature | Group | Count |
|---|---|---:|
| `age_band` | 65+ | 8 |
| `foreign_worker` | no | 9 |
| `job` | unemployed / unskilled non-resident | 10 |
| `personal_status_sex` | male - divorced/separated | 12 |

These results should not be used for firm conclusions. They should be treated as indicators for further testing on larger and more representative data.

---

## 9. Explainability Signals Relevant to Fairness

Top permutation-importance features:

| Rank | Feature |
|---:|---|
| 1 | `checking_account_status` |
| 2 | `credit_history` |
| 3 | `duration_months` |
| 4 | `purpose` |
| 5 | `credit_amount` |

Sensitive or proxy-sensitive features with positive permutation importance:

| Feature | Rank | Importance mean |
|---|---:|---:|
| `housing` | 7 | 0.0088 |
| `personal_status_sex` | 10 | 0.0076 |
| `property` | 11 | 0.0074 |
| `age_years` | 12 | 0.0041 |
| `foreign_worker` | 13 | 0.0038 |
| `telephone` | 14 | 0.0021 |
| `job` | 16 | 0.0003 |

Governance implication: the model does not rely only on traditional credit-risk features. Several sensitive or proxy-sensitive variables contribute to model behaviour and require explicit review.

---

## 10. Fairness Risk Assessment

| Risk | Assessment | Rationale |
|---|---|---|
| Direct use of sensitive variables | High | `personal_status_sex`, `age_years` and `foreign_worker` appear in the dataset |
| Proxy discrimination | High | `property`, `housing`, `job` and `telephone` may encode socioeconomic status |
| Differential false positives | Medium / High | Largest observed FPR gap is 0.362 for `property` |
| Differential false negatives | Medium | Largest observed FNR gap is 0.258 for `property` |
| Statistical reliability | Low / Medium | Test set is small and several groups have low counts |
| Explainability risk | Medium | Important variables include proxy-sensitive features |
| Customer harm risk | Medium / High | False positives may create unnecessary scrutiny or exclusion |
| Business risk | Medium | False negatives remain possible even at selected threshold |

Overall fairness risk rating: **Medium / High**, mainly due to proxy-sensitive variables, subgroup gaps and limited data reliability.

---

## 11. Mitigation Options

| Mitigation | Purpose | Recommended? |
|---|---|---|
| Exclude direct sensitive variables from model training | Reduce direct reliance on sensitive attributes | Yes, test as challenger |
| Keep sensitive variables only for fairness monitoring | Enable bias detection without direct decision use | Yes |
| Review proxy-sensitive variables | Assess necessity and proportionality | Yes |
| Compare model with and without `property`, `housing`, `personal_status_sex`, `foreign_worker` | Measure performance / fairness trade-off | Yes |
| Add human review for high-risk flags | Reduce automatic exclusion risk | Yes |
| Require reasoned override logging | Monitor whether humans correct or amplify model bias | Yes |
| Monitor subgroup error rates over time | Detect emerging differential impact | Yes |
| Use fairness constraints automatically | Not recommended at this stage | Needs stronger data and policy basis |

---

## 12. Human Oversight Requirements

Fairness controls should be embedded in the operating model.

Minimum requirements:

- no automatic rejection based only on model output;
- credit officer review for high-risk flags;
- documented reasons for final decisions;
- override logging with structured reason codes;
- second-line review for borderline or high-impact cases;
- periodic fairness review by Model Risk / Compliance / Credit Risk;
- clear policy on which variables may be used for prediction versus monitoring;
- escalation if subgroup outcomes breach agreed thresholds.

---

## 13. Monitoring Requirements

If used in a controlled pilot, fairness monitoring should include:

| Metric | Segment |
|---|---|
| High-risk flag rate | age band, personal status/sex, foreign worker status, housing, job, property |
| False positive rate | same segments, subject to sufficient sample size |
| False negative rate | same segments, subject to sufficient sample size |
| Approval / rejection rate | same segments |
| Override rate | same segments |
| Complaint / appeal rate | same segments |
| Feature drift | top drivers and sensitive/proxy-sensitive variables |

Suggested cadence:

| Review | Frequency |
|---|---|
| Operational monitoring | Monthly |
| Fairness review | Quarterly |
| Model risk review | At least annually |
| Trigger-based review | After major data, policy, product or macroeconomic changes |

Because subgroup sizes can be small, monitoring should combine statistical thresholds with expert review.

---

## 14. Fairness Assessment Conclusion

The model shows useful predictive performance, but fairness risk remains material.

Main findings:

- the dataset includes direct sensitive and proxy-sensitive variables;
- subgroup bad-risk rates differ materially in the data;
- the selected threshold flags nearly half of test applicants as high risk;
- the largest observed fairness gaps relate to `property`, a wealth/security proxy;
- several sensitive or proxy-sensitive features have positive model importance;
- subgroup analysis is limited by small sample sizes.

Conclusion: **not approved for production use**.

For a real deployment, this model would require stronger data, independent validation, legal review of feature use, challenger models excluding sensitive variables, documented human oversight, and ongoing fairness monitoring.
