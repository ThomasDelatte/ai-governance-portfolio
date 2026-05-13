# Fairness Assessment — Credit Risk Decision-Support Model

## 1. Purpose

This document summarises the fairness review of the credit risk decision-support model.

The objective is to assess whether model outcomes and error patterns differ materially across relevant subgroups, and whether these differences create Responsible AI risks requiring controls before any operational use.

This is a Responsible AI assessment for a portfolio case study. It is not a legal discrimination assessment.

---

## 2. Model and Thresholds Reviewed

| Item | Description |
|---|---|
| Candidate model | `logistic_regression_C1_calibrated_sigmoid` |
| Model type | Calibrated logistic regression |
| Use case | Credit risk decision support |
| Positive class | Bad credit risk |
| Default threshold | `0.50` |
| Selected threshold | `0.16` |
| Threshold basis | Cost-sensitive threshold selected on validation set |
| Final evaluation | Held-out test set |

The selected threshold uses the German Credit asymmetric cost assumption:

| Error type | Interpretation | Assumed cost |
|---|---|---:|
| False negative | Bad credit risk classified as good | 5 |
| False positive | Good credit risk classified as bad | 1 |

---

## 3. Fairness Testing Setup

The model predicts whether an applicant is a higher credit risk.

For this assessment:

| Concept | Definition |
|---|---|
| Positive model class | Bad credit risk |
| Favourable outcome | Applicant is not flagged as high risk |
| Unfavourable outcome | Applicant is flagged as high risk |
| High-risk flag | Model prediction equals `1` |
| Review threshold 1 | Default threshold `0.50` |
| Review threshold 2 | Cost-sensitive threshold `0.16` |

This distinction matters because the positive class is not favourable. A positive prediction means the applicant is flagged as higher risk.

---

## 4. Subgroups Tested

Fairness testing focused on attributes available or derivable from the dataset.

| Attribute | Source / derivation | Reason for inclusion |
|---|---|---|
| Gender group | Derived from `personal_status` | Detect potential gender-related differences in outcomes |
| Age group | Derived from `age` | Detect age-related outcome and error-rate differences |
| Foreign worker status | Dataset feature | Detect potential nationality / migration-status proxy concerns |

These attributes are imperfect and dataset-specific. They are used as fairness testing dimensions, not as definitive protected-class determinations.

---

## 5. Metrics Used

The fairness review used both outcome and error metrics.

| Metric | Meaning |
|---|---|
| High-risk flag rate | Share of applicants flagged as bad credit risk |
| Favourable outcome rate | Share of applicants not flagged as high risk |
| False positive rate | Share of good-risk applicants incorrectly flagged as high risk |
| False negative rate | Share of bad-risk applicants missed by the model |
| Recall for bad risk | Share of bad-risk applicants correctly flagged |
| Disparate impact screen | Ratio between lowest and highest favourable outcome rates |

The disparate impact ratio is used as a screening metric. It is not treated as a legal compliance conclusion.

---

## 6. Overall Threshold Impact

Before reviewing subgroups, the overall operating point must be understood.

| Threshold | High-risk flag rate | False positives | False negatives | Main interpretation |
|---:|---:|---:|---:|---|
| `0.50` | 21.0% | 16 | 34 | Conservative flagging, but many bad-risk applicants missed |
| `0.16` | 71.5% | 90 | 7 | Better bad-risk detection, but high customer-friction risk |

The selected threshold substantially reduces missed bad-risk applicants. However, it also flags most applicants as high risk.

This is the central fairness and customer-impact issue. Even if subgroup disparities were limited, a 71.5% high-risk flag rate would require operational review before use.

---

## 7. Fairness Findings Summary

The fairness finding screen identified the following issues:

| Finding | Severity |
|---|---|
| Cost-sensitive threshold creates high flagging rate | High |
| Age-group disparity at default threshold | Medium |
| Foreign-worker-status disparity at default threshold | Medium |
| Age-group disparity at selected threshold | High |
| Foreign-worker-status disparity at selected threshold | High |
| Main model drivers require business and proxy review | Medium |

Main assessment:

```text
The model should not be used for automated rejection. The selected threshold creates high customer-friction risk and fairness review triggers for age group and foreign worker status.
```

---

## 8. Gender Group Assessment

Gender group did not trigger a material fairness finding in this test sample.

Interpretation:

- No major gender-related disparity was detected by the rule-based screen.
- This does not prove that the model is fair with respect to gender.
- The dataset is small and dated.
- Gender is derived from `personal_status`, which is an imperfect representation.
- Gender outcomes should still be monitored if the model were tested further.

Assessment:

| Attribute | Finding | Severity |
|---|---|---|
| Gender group | No material review trigger in this test sample | Low |

Recommended control:

```text
Continue monitoring gender-related outcomes and error rates in any future pilot or expanded validation.
```

---

## 9. Age Group Assessment

Age group triggered fairness review findings.

At the default threshold, age group was already identified as a medium-severity review area. At the selected threshold, age group became a high-severity review area.

Main concern:

```text
The selected threshold changes the distribution of high-risk flags across age groups enough to require governance review.
```

Potential reasons this matters:

- age can correlate with credit history length;
- age can correlate with employment stability;
- age can correlate with income patterns;
- age-related treatment may be legally or policy-sensitive in credit contexts.

Assessment:

| Attribute | Finding | Severity |
|---|---|---|
| Age group | Disparity trigger at default threshold | Medium |
| Age group | Disparity trigger at selected threshold | High |

Recommended controls:

- review high-risk flag rates and error rates by age group;
- avoid using the model for automated rejection;
- review whether age or age-correlated variables are acceptable under applicable policy;
- monitor age-group outcomes in any future pilot;
- consider threshold adjustment or risk-band design if age disparities remain material.

---

## 10. Foreign Worker Status Assessment

Foreign worker status triggered fairness review findings at both thresholds, with high severity at the selected threshold.

Main concern:

```text
Foreign worker status creates a high-priority review trigger, but the evidence is unstable because the subgroup size is small.
```

This finding should be interpreted carefully.

What can be concluded:

- there is a material review signal;
- foreign worker status is potentially proxy-sensitive;
- the feature appears among the top global model drivers;
- the selected threshold may amplify outcome differences.

What cannot be concluded:

- this is not proof of unlawful discrimination;
- the subgroup sample is too small for a definitive conclusion;
- further validation on larger and more representative data would be required.

Assessment:

| Attribute | Finding | Severity |
|---|---|---|
| Foreign worker status | Disparity trigger at default threshold | Medium |
| Foreign worker status | Disparity trigger at selected threshold | High |
| Foreign worker status | Appears among top model drivers | Medium |

Recommended controls:

- review whether `foreign_worker` should be used in the model;
- test model performance and fairness with and without this feature;
- validate findings on a larger dataset;
- monitor high-risk flag rate and false positive rate for this subgroup;
- involve legal/compliance review before any operational use.

---

## 11. Explainability and Proxy Review

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

- most top drivers are broadly credit-relevant;
- `checking_status`, `credit_history`, `credit_amount`, `duration` and `savings_status` are plausible credit-risk indicators;
- `foreign_worker` is proxy-sensitive and should not be accepted without further review;
- even credit-relevant variables can create indirect subgroup disparities.

Important point:

```text
Removing protected attributes is not sufficient by itself. Other variables may act as proxies, so the assessment focuses on observed subgroup outcomes and error patterns.
```

---

## 12. Mitigation Options Considered

No algorithmic mitigation was applied automatically in V1.

This is intentional. The assessment follows a diagnosis-first approach:

```text
measure subgroup outcomes
      ↓
identify material disparities
      ↓
assess operational and legal relevance
      ↓
select proportionate controls or mitigation
      ↓
re-test model behaviour
```

Mitigation options considered:

| Option | Potential use |
|---|---|
| Threshold adjustment | Reduce excessive high-risk flagging or subgroup disparity |
| Risk bands | Replace binary thresholding with low / medium / high risk bands |
| Reweighing / sample weighting | Address stable and confirmed subgroup disadvantage |
| Feature review or exclusion | Remove or restrict sensitive/proxy-sensitive variables |
| Post-processing constraints | Reduce error-rate disparities across groups |
| Human review | Prevent direct adverse impact from model output |
| Monitoring | Detect fairness drift or subgroup performance degradation |

For this version, the preferred mitigation is not a new algorithm. The recommended approach is controlled use, human review, threshold review and subgroup monitoring.

---

## 13. Recommended Controls

| Control | Purpose |
|---|---|
| Decision-support only | Prevent fully automated adverse decisions |
| No automated rejection | Avoid direct customer harm from unvalidated model output |
| Human review | Ensure credit officers can override or challenge model output |
| Threshold review | Assess whether threshold `0.16` is operationally and ethically acceptable |
| Subgroup monitoring | Track high-risk flag rates and error rates over time |
| Feature governance | Review `foreign_worker`, age-related variables and proxy-sensitive features |
| Reason-code review | Ensure explanations are appropriate and understandable |
| Larger validation sample | Reduce instability in subgroup estimates |
| Periodic reassessment | Re-test fairness after data, policy or model changes |

---

## 14. Residual Fairness Risks

Even with controls, several residual risks remain.

| Residual risk | Explanation |
|---|---|
| High flagging rate | The selected threshold flags 71.5% of applicants as high risk |
| Small subgroup instability | Foreign worker status has limited sample support |
| Proxy bias | Features may encode sensitive or socioeconomic characteristics |
| Age-related disparity | Age group triggered review findings |
| Threshold sensitivity | Fairness findings may change materially at different thresholds |
| Dataset limitation | German Credit is not representative of a modern banking population |
| Legal uncertainty | Statistical fairness metrics do not establish legal compliance |

---

## 15. Fairness Assessment Decision

Recommended fairness decision:

```text
Do not approve for automated credit rejection.
Approve only for further controlled testing, subject to fairness controls.
```

Rationale:

- gender did not trigger a material finding in this test sample;
- age group triggered review findings, especially at the selected threshold;
- foreign worker status triggered high-priority review findings;
- the selected threshold creates a very high overall flagging rate;
- proxy-sensitive feature concerns require further review;
- the dataset is too limited for a production-grade fairness conclusion.

---

## 16. Final Conclusion

The model is not rejected as a technical baseline, but it is not fairness-ready for operational use.

The main fairness issue is the interaction between threshold choice and subgroup impact. The selected threshold improves bad-risk detection but creates high customer-friction risk and review triggers for age group and foreign worker status.

Final position:

```text
The model may proceed to further controlled testing only.
It should not be used for fully automated rejection.
Further threshold review, subgroup monitoring and proxy-sensitive feature review are required.
```
