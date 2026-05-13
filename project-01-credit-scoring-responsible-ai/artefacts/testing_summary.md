# Testing Summary — Credit Risk Decision-Support Model

## 1. Purpose

This testing summary consolidates the main technical and Responsible AI testing results for the credit risk decision-support model.

The objective is to determine whether the candidate model is suitable for:

```text
controlled testing / pilot use / further review
```

not whether it is ready for production deployment.

---

## 2. Model Under Review

| Item | Description |
|---|---|
| Candidate model | `logistic_regression_C1_calibrated_sigmoid` |
| Model type | Calibrated logistic regression |
| Use case | Credit risk decision support |
| Dataset | Statlog German Credit Dataset |
| Positive class | Bad credit risk |
| Selected threshold | `0.16` |
| Threshold basis | Cost-sensitive threshold selected on validation set |
| Final evaluation | Held-out test set |

---

## 3. Testing Scope

The following tests were performed:

| Test area | Completed | Purpose |
|---|---:|---|
| Model performance | Yes | Assess predictive quality |
| Probability calibration | Yes | Assess whether scores can be interpreted as risk scores |
| Cost-sensitive thresholding | Yes | Assess business impact of threshold choice |
| Subgroup fairness testing | Yes | Identify material differences across tested groups |
| Error-rate analysis | Yes | Compare false positive and false negative rates |
| Explainability review | Yes | Identify main model drivers and proxy-sensitive features |
| Mitigation review | Conceptual | Identify possible controls and mitigation options |

---

## 4. Data Split

The model was developed using a train / validation / test split.

| Split | Purpose |
|---|---|
| Training set | Fit model candidates |
| Validation set | Select candidate model and threshold |
| Test set | Final held-out evaluation |

This avoids selecting the operating threshold directly on the final test set.

---

## 5. Performance Test Results

Held-out test set performance:

| Metric | Value |
|---|---:|
| ROC AUC | 0.780 |
| Average precision | 0.612 |
| Brier score | 0.165 |

Interpretation:

- The model shows reasonable baseline ranking performance.
- Probability quality is acceptable for a portfolio case study.
- The results are not sufficient for production deployment without further validation.
- The model is suitable for Responsible AI testing and governance review.

---

## 6. Threshold Testing

Two thresholds were reviewed:

| Threshold | Purpose |
|---:|---|
| `0.50` | Default threshold |
| `0.16` | Cost-sensitive threshold selected on validation set |

The selected threshold uses the German Credit asymmetric cost assumption:

| Error type | Interpretation | Assumed cost |
|---|---|---:|
| False negative | Bad credit risk classified as good | 5 |
| False positive | Good credit risk classified as bad | 1 |

---

## 7. Threshold Impact on Test Set

| Threshold | High-risk flag rate | False positives | False negatives | Total assumed cost |
|---:|---:|---:|---:|---:|
| `0.50` | 21.0% | 16 | 34 | 186 |
| `0.16` | 71.5% | 90 | 7 | 125 |

Interpretation:

- The selected threshold reduces false negatives from `34` to `7`.
- It also increases false positives from `16` to `90`.
- The high-risk flag rate increases from `21.0%` to `71.5%`.
- The threshold reduces assumed business cost but creates high customer-friction and operational workload risk.

Testing conclusion:

```text
The selected threshold is analytically justified under the assumed cost matrix, but it is not automatically suitable for operational deployment.
```

---

## 8. Fairness Testing Setup

Fairness testing was performed using the following definitions:

| Concept | Definition |
|---|---|
| Positive model class | Bad credit risk |
| Favourable outcome | Applicant is not flagged as high risk |
| Unfavourable outcome | Applicant is flagged as high risk |
| Tested thresholds | `0.50` and `0.16` |

Tested subgroup attributes:

| Attribute | Rationale |
|---|---|
| Gender group | Detect potential gender-related outcome differences |
| Age group | Detect age-related differences in credit risk flagging and error rates |
| Foreign worker status | Detect potential nationality / migration-status proxy concerns |

These tests are Responsible AI screening tests, not legal discrimination conclusions.

---

## 9. Fairness Test Results

The fairness finding screen identified the following issues:

| Finding | Severity |
|---|---|
| Cost-sensitive threshold creates high flagging rate | High |
| Age-group disparity at default threshold | Medium |
| Foreign-worker-status disparity at default threshold | Medium |
| Age-group disparity at selected threshold | High |
| Foreign-worker-status disparity at selected threshold | High |
| Main model drivers require business and proxy review | Medium |

Interpretation:

- Gender did not trigger a material finding in this test sample.
- Age group triggered review findings, especially under the selected threshold.
- Foreign worker status triggered high-priority review findings.
- The foreign-worker-status finding should be interpreted carefully because subgroup size is small.
- The selected threshold increases fairness and customer-impact concerns.

Testing conclusion:

```text
The model should not proceed to operational use without subgroup monitoring, threshold review and human oversight.
```

---

## 10. Explainability Review

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

- Most top features are broadly credit-relevant.
- `foreign_worker` appears among the top drivers and should be treated as proxy-sensitive.
- Top model drivers should be reviewed with credit risk, legal/compliance and business stakeholders.
- Explainability evidence supports further review, not deployment approval.

---

## 11. Main Testing Findings

| Finding | Evidence | Risk | Severity |
|---|---|---|---|
| Aggressive threshold | Threshold `0.16` flags 71.5% of applicants as high risk | Customer friction and manual review burden | High |
| Reduced missed bad-risk cases | False negatives fall from 34 to 7 | Positive from credit-risk perspective, but creates trade-off | Medium |
| Increased false positives | False positives rise from 16 to 90 | Good-risk applicants may face adverse treatment or delay | High |
| Age-group disparity | Age group triggers review findings under selected threshold | Potential unequal impact | High |
| Foreign-worker-status disparity | Foreign worker status triggers high-priority review findings | Proxy-sensitive fairness concern | High |
| Proxy-sensitive feature | `foreign_worker` appears among top model drivers | Legal/compliance and fairness review needed | Medium |
| Dataset limitation | German Credit is small and dated | Limited external validity | High |

---

## 12. Mitigation Review

No algorithmic mitigation was applied automatically in this version.

This is intentional. The testing approach follows a diagnosis-first sequence:

```text
measure model performance
      ↓
select candidate threshold
      ↓
test subgroup outcomes
      ↓
identify material risks
      ↓
select proportionate controls or mitigation
```

Potential mitigation options:

| Option | Use if |
|---|---|
| Threshold adjustment | The selected threshold is too aggressive operationally |
| Risk bands | A binary threshold is too crude for decision-support use |
| Reweighing / sample weighting | Subgroup disadvantage is confirmed and stable |
| Feature review or exclusion | Sensitive or proxy-sensitive features drive outcomes |
| Post-processing constraints | Error rates differ materially across groups |
| Human review | Model output may create adverse customer impact |
| Monitoring | Model proceeds to controlled pilot |

For this V1, the preferred mitigation is not a new algorithm. The preferred controls are:

- decision-support only;
- no fully automated rejection;
- human review for adverse or borderline cases;
- threshold review before approval;
- subgroup monitoring;
- proxy-sensitive feature review.

---

## 13. Recommended Testing Decision

Recommended decision:

```text
Approve for controlled testing only.
Do not approve for production deployment.
Do not approve for fully automated rejection.
```

Rationale:

- The model has reasonable baseline performance.
- The selected threshold reduces missed bad-risk applicants.
- The selected threshold creates a very high flagging rate.
- Subgroup testing identifies age-group and foreign-worker-status review triggers.
- Explainability review identifies proxy-sensitive feature concerns.
- Dataset limitations prevent strong real-world conclusions.

---

## 14. Conditions Before Any Pilot

Before any controlled pilot, the following conditions should be met:

| Condition | Purpose |
|---|---|
| Human review for adverse or borderline cases | Prevent overreliance on model output |
| Threshold review | Assess whether threshold `0.16` is operationally acceptable |
| Subgroup monitoring plan | Track fairness and error rates over time |
| Feature governance review | Assess `foreign_worker`, age-related variables and proxy-sensitive features |
| Reason-code review | Ensure explanations are understandable and appropriate |
| Manual override process | Allow credit officers to challenge model output |
| Escalation procedure | Define what happens if subgroup disparities worsen |
| Periodic reassessment | Re-test performance, calibration and fairness |

---

## 15. Residual Risks

Even after controls, the following residual risks remain:

| Residual risk | Explanation |
|---|---|
| Dataset representativeness | German Credit does not reflect a modern European banking population |
| Small subgroup instability | Some subgroup metrics are based on very small samples |
| Proxy bias | Features may indirectly encode sensitive characteristics |
| Threshold instability | The selected threshold may not generalise to new data |
| Overreliance | Users may treat model output as a decision rather than support |
| Explainability limits | Global feature importance does not fully explain individual outcomes |
| Legal uncertainty | Fairness metrics do not equal legal compliance |

---

## 16. Final Conclusion

The candidate model passes basic technical testing as a baseline Responsible AI case study.

However, it does not pass as a production-ready credit decision model.

The most important testing finding is that the cost-sensitive threshold improves bad-risk detection but creates material governance concerns:

- high customer-friction risk;
- high false positive burden;
- age-group disparity triggers;
- foreign-worker-status review triggers;
- proxy-sensitive feature concerns.

The appropriate next step is controlled review, not deployment.

```text
Final testing position: approve for controlled testing only, subject to human oversight, threshold review, subgroup monitoring and proxy-sensitive feature review.
```
