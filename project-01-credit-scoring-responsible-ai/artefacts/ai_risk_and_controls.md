# AI Risk and Controls — Credit Risk Decision-Support Model

## 1. Purpose

This document maps the main AI risks identified during model development, threshold testing, fairness review and explainability analysis to practical controls.

The objective is not to claim that the model is compliant or production-ready. The objective is to show how technical findings can be translated into governance actions.

---

## 2. Model Context

| Item | Description |
|---|---|
| Use case | Credit risk decision support |
| Candidate model | `logistic_regression_C1_calibrated_sigmoid` |
| Model type | Calibrated logistic regression |
| Dataset | Statlog German Credit Dataset |
| Positive class | Bad credit risk |
| Selected threshold | `0.16` |
| Model role | Decision-support only |
| Recommended status | Controlled testing only |

The model should not be used for fully automated credit rejection.

---

## 3. Main Risk Summary

| Risk area | Risk level | Summary |
|---|---|---|
| Customer impact | High | Selected threshold flags 71.5% of applicants as high risk |
| Threshold governance | High | Cost-sensitive threshold reduces false negatives but creates many false positives |
| Fairness | High | Age group and foreign worker status trigger review findings |
| Proxy-sensitive features | Medium / High | `foreign_worker` appears among top model drivers |
| Explainability | Medium | Main drivers are mostly credit-relevant, but require business and proxy review |
| Overreliance | High | Users may treat the model score as a decision |
| Data limitations | High | German Credit is small, dated and not representative of a modern banking population |
| Monitoring | Medium | No production monitoring is implemented in this portfolio version |

---

## 4. Risk-Control Matrix

| Risk | Evidence | Impact | Risk level | Required control | Residual risk |
|---|---|---|---|---|---|
| Excessive high-risk flagging | Selected threshold `0.16` flags 71.5% of applicants as high risk | High customer friction, large manual review workload, potential exclusion risk | High | Review threshold before any pilot; consider risk bands instead of binary decisioning | Medium / High |
| False positive burden | False positives increase from 16 at threshold `0.50` to 90 at threshold `0.16` | Good-risk applicants may face delay, stricter review or adverse treatment | High | Human review for all adverse or borderline outcomes; no automated rejection | Medium |
| Missed bad-risk applicants at default threshold | False negatives are 34 at threshold `0.50` | Credit risk exposure if default threshold is used | Medium | Use cost-sensitive analysis to inform threshold review; monitor false negatives | Medium |
| Threshold selected on simplified cost assumption | False negative cost = 5, false positive cost = 1 | Threshold may not reflect real risk appetite, profitability or customer impact | High | Validate cost assumptions with credit risk, business and compliance stakeholders | Medium |
| Age-group disparity | Age group triggers fairness review findings, especially at selected threshold | Potential unequal impact across age groups | High | Monitor age-group outcomes and error rates; review age-related model drivers | Medium / High |
| Foreign-worker-status disparity | Foreign worker status triggers review findings; subgroup size is small | Potential nationality or migration-status proxy concern | High | Review use of `foreign_worker`; validate on larger data; involve legal/compliance | Medium / High |
| Small subgroup instability | Foreign worker subgroup has limited sample support | Fairness metrics may be unstable or misleading | Medium | Require larger validation sample before conclusions; report confidence limitations | Medium |
| Proxy-sensitive feature use | `foreign_worker` appears among top global model drivers | Possible indirect discrimination or difficult-to-justify feature use | Medium / High | Feature governance review; test model with and without proxy-sensitive features | Medium |
| Explainability limits | Permutation importance gives global, not individual, explanations | Explanations may be insufficient for adverse action reasoning | Medium | Develop reason-code logic and local explanation review before any pilot | Medium |
| Overreliance by users | Model outputs a risk score that may be mistaken for a decision | Credit officers may defer too heavily to model output | High | Position as decision-support only; train users; require manual override process | Medium |
| Lack of contestability | No appeal or challenge process implemented | Applicants may be unable to challenge adverse outcomes influenced by model | High | Define appeal / escalation route before operational use | Medium |
| Dataset representativeness | German Credit is small and dated | Findings may not generalise to modern banking data | High | Treat as portfolio case only; require real-data validation before deployment | High |
| No out-of-time validation | Model was tested using random train/validation/test split | Temporal stability is unknown | Medium | Perform out-of-time validation on future or historical periods | Medium |
| No production monitoring | Monitoring is conceptual only | Drift or fairness degradation would not be detected | Medium | Define monitoring metrics, thresholds and escalation process | Medium |
| Legal and regulatory uncertainty | Fairness metrics are not legal conclusions | Risk of overclaiming compliance | High | Legal/compliance review required; avoid claims of compliance or fairness certification | Medium |

---

## 5. Control Design by Lifecycle Stage

### 5.1 Pre-Deployment Controls

| Control | Purpose | Owner |
|---|---|---|
| Use-case classification | Confirm whether the system is decision-support or automated decisioning | AI governance / legal |
| Data suitability review | Assess dataset quality, representativeness and limitations | Data science / model risk |
| Feature governance review | Review sensitive and proxy-sensitive variables | Data science / compliance |
| Threshold review | Assess whether threshold `0.16` is acceptable | Credit risk / business / governance |
| Fairness review | Review subgroup outcomes and error rates | Responsible AI / model risk |
| Explainability review | Check whether drivers are understandable and justifiable | Data science / business |
| Human oversight design | Define how credit officers use and challenge the score | Business / model governance |
| Documentation review | Complete model card, testing summary and fairness assessment | Model owner / governance |

---

### 5.2 Deployment Controls

| Control | Purpose | Owner |
|---|---|---|
| Decision-support only | Prevent the score from becoming an automated rejection rule | Business owner |
| No automated rejection | Avoid direct adverse decisions without human review | Credit policy / compliance |
| Manual review for adverse cases | Reduce customer harm from false positives | Credit operations |
| Reason-code support | Provide understandable explanation for review decisions | Data science / business |
| Manual override | Allow credit officers to depart from model output | Credit operations |
| Audit trail | Record model score, threshold, user action and override reason | IT / model governance |
| User guidance | Explain model limitations and appropriate use | Model owner |

---

### 5.3 Post-Deployment / Pilot Controls

| Control | Purpose | Owner |
|---|---|---|
| Performance monitoring | Track ROC AUC, precision, recall, false positives and false negatives | Data science |
| Calibration monitoring | Check whether predicted risks remain meaningful | Data science / model risk |
| Subgroup monitoring | Track outcomes and error rates by gender, age group and foreign worker status | Responsible AI / model risk |
| Threshold monitoring | Assess whether the operating threshold remains appropriate | Credit risk / business |
| Drift monitoring | Detect changes in input data or score distributions | Data science / MLOps |
| Incident escalation | Define what happens if fairness or performance thresholds are breached | Governance / risk |
| Periodic reassessment | Re-test model performance, fairness and explainability | Model risk / Responsible AI |
| Model retirement criteria | Define when the model should be suspended or replaced | Model owner / governance |

---

## 6. Risk Appetite and Tolerance Considerations

The selected threshold reduces assumed credit-risk cost, but it creates high customer-friction risk.

A real institution would need to define tolerance levels for:

| Dimension | Example question |
|---|---|
| Credit risk | How many bad-risk applicants can be missed? |
| Customer impact | How many good-risk applicants can be flagged for additional review? |
| Fairness | What subgroup disparity triggers remediation? |
| Operations | What high-risk flag rate can credit officers realistically review? |
| Explainability | What level of explanation is required for adverse or borderline cases? |
| Compliance | Which features require legal or policy approval before use? |

For this case study, the selected threshold `0.16` should be treated as a review trigger, not as an approved operating point.

---

## 7. Monitoring Metrics

If the model were moved to a controlled pilot, the following metrics should be monitored.

| Metric | Purpose | Suggested review trigger |
|---|---|---|
| High-risk flag rate | Detect excessive customer friction | Material increase from approved baseline |
| False positive rate | Detect burden on good-risk applicants | Increase beyond approved tolerance |
| False negative rate | Detect missed bad-risk applicants | Increase beyond approved tolerance |
| ROC AUC | Track ranking performance | Material degradation |
| Brier score | Track probability quality | Material degradation |
| Score distribution | Detect drift in applicant risk profile | Distribution shift |
| Subgroup high-risk flag rate | Detect unequal outcome patterns | Material subgroup gap |
| Subgroup false positive rate | Detect unequal burden on good-risk applicants | Material subgroup gap |
| Subgroup false negative rate | Detect unequal missed-risk patterns | Material subgroup gap |
| Override rate | Detect user disagreement with model | High or increasing override rate |
| Appeal / complaint rate | Detect customer harm or contestability issues | Increase above baseline |

Exact thresholds should be defined by the model owner, credit risk, compliance and Responsible AI stakeholders before pilot use.

---

## 8. Incident and Escalation Triggers

Potential escalation triggers:

| Trigger | Example response |
|---|---|
| High-risk flag rate exceeds approved tolerance | Review threshold and pause model expansion |
| Subgroup disparity exceeds approved tolerance | Conduct fairness review and remediation analysis |
| False positive rate increases materially | Review customer impact and manual review workload |
| Calibration degrades | Recalibrate model or suspend risk-score use |
| Proxy-sensitive feature concern emerges | Review feature use with legal/compliance |
| User overreliance is detected | Retrain users and adjust workflow controls |
| Customer complaints increase | Review explanations, appeal process and adverse outcomes |

---

## 9. Controls Mapped to Responsible AI Objectives

| Responsible AI objective | Controls |
|---|---|
| Fairness | Subgroup monitoring, threshold review, feature governance, fairness reassessment |
| Explainability | Permutation importance review, reason-code design, local explanation review |
| Accountability | Model card, testing summary, audit trail, named model owner |
| Human oversight | Decision-support only, manual review, override process |
| Contestability | Appeal route, documented reasons, escalation process |
| Robustness | Performance monitoring, calibration monitoring, drift monitoring |
| Transparency | Documentation, user guidance, clear model-use boundaries |
| Risk management | Risk-control matrix, residual risk review, periodic reassessment |

---

## 10. Residual Risk Assessment

| Residual risk | Residual level | Reason |
|---|---|---|
| Customer-friction risk | Medium / High | Threshold remains aggressive even with human review |
| Age-group disparity | Medium / High | Material review trigger remains unresolved |
| Foreign-worker-status concern | Medium / High | Proxy-sensitive and small-sample issue remains unresolved |
| Proxy bias | Medium | Feature review can reduce but not eliminate proxy effects |
| Overreliance | Medium | Human review reduces risk but does not remove automation bias |
| Dataset limitation | High | Public dataset cannot validate real-world deployment suitability |
| Legal uncertainty | Medium / High | Statistical testing does not establish legal compliance |

---

## 11. Recommended Governance Decision

Recommended decision:

```text
Approve for controlled testing only.
Do not approve for production deployment.
Do not approve for fully automated rejection.
```

Rationale:

- The model has reasonable baseline technical performance.
- The selected threshold improves bad-risk detection.
- The selected threshold creates high customer-friction and false-positive burden.
- Age group and foreign worker status trigger fairness review findings.
- `foreign_worker` appears among top model drivers and requires proxy-sensitive review.
- Dataset limitations prevent production-grade conclusions.

---

## 12. Next Governance Actions

Recommended next actions before any controlled pilot:

1. Review whether threshold `0.16` is operationally acceptable.
2. Consider risk bands instead of a single binary threshold.
3. Review use of `foreign_worker` and other proxy-sensitive variables.
4. Validate fairness findings on a larger and more representative dataset.
5. Define human review and override procedures.
6. Define monitoring metrics and escalation thresholds.
7. Prepare reason-code logic for adverse or borderline cases.
8. Document final residual risk acceptance by the appropriate governance owner.

---

## 13. Final Position

The main risk is not that the model fails technically. The main risk is that an analytically cost-sensitive threshold could create unacceptable operational, fairness and customer-impact consequences.

The model should therefore remain a controlled testing candidate until threshold governance, subgroup monitoring, proxy-sensitive feature review and human oversight controls are defined.
