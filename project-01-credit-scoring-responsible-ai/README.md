# Responsible AI Governance for Credit Risk Decision Support

This repository is a compact Responsible AI / AI Governance case study for a credit risk decision-support model.

It is not a Kaggle-style modelling project. The objective is to show how a credit risk model can be developed, tested, documented and challenged from a Responsible AI and model governance perspective.

The project is based on the Statlog German Credit Dataset and is framed as a simulated European banking use case. The model is treated as a decision-support tool, not as a fully automated credit approval or rejection system.

---

## Project Objective

The objective is to demonstrate how technical machine learning evidence can be translated into governance-relevant artefacts.

The project covers:

- model development and performance evaluation;
- cost-sensitive threshold selection;
- calibration and probability quality;
- fairness and subgroup analysis;
- explainability review;
- AI risk and control documentation;
- testing evidence for Responsible AI review.

The intended audience is hiring managers and reviewers for roles in:

- Responsible AI Engineering;
- AI Governance;
- AI Risk Management;
- Model Governance;
- AI Assurance;
- AI Compliance Engineering.

---

## Scenario

A European financial institution wants to assess whether a credit risk model could support loan application review.

The model predicts whether an applicant is likely to represent a higher credit risk. The output is a risk score that may support human credit officers, subject to credit policy, affordability checks, human review and documented reasoning.

This case study assumes that the model would not be used for fully automated rejection without additional legal, compliance, operational and human oversight controls.

---

## Audit Scope

| Component | Scope |
|---|---|
| Use case | Credit risk decision-support |
| Dataset | Statlog German Credit Dataset |
| Model role | Decision-support, not fully automated decisioning |
| Target | Good / bad credit risk |
| Sensitive or subgroup attributes tested | Gender, age group, foreign worker status |
| Performance metrics | ROC AUC, average precision, Brier score, accuracy, precision, recall, confusion matrix |
| Business metric | Cost-sensitive error analysis using the German Credit cost matrix |
| Fairness metrics | Selection rate, disparate impact, false positive rate gap, false negative rate gap, equal opportunity difference |
| Explainability | Feature importance and/or SHAP analysis |
| Governance outputs | Model card, fairness assessment, AI risk-control matrix, testing summary |

---

## Repository Structure

~~~text
project-01-credit-scoring-responsible-ai/
│
├── README.md
│
├── notebooks/
│   ├── 01_model_development_and_thresholding.ipynb
│   └── 02_fairness_explainability_and_testing.ipynb
│
├── artefacts/
│   ├── model_card.md
│   ├── fairness_assessment.md
│   ├── ai_risk_and_controls.md
│   └── testing_summary.md
│
├── outputs/
│   ├── figures/
│   └── tables/
│
└── requirements.txt
~~~

---

## Main Deliverables

| Deliverable | Purpose |
|---|---|
| `01_model_development_and_thresholding.ipynb` | Develop baseline credit risk models, evaluate performance, assess calibration and select a cost-sensitive threshold |
| `02_fairness_explainability_and_testing.ipynb` | Test subgroup outcomes, fairness metrics, explainability and residual Responsible AI risks |
| `model_card.md` | Summarise model purpose, intended use, limitations, performance and governance constraints |
| `fairness_assessment.md` | Document subgroup analysis, fairness findings, limitations and mitigation options |
| `ai_risk_and_controls.md` | Map key AI risks to practical controls and residual risk levels |
| `testing_summary.md` | Provide a concise testing and validation summary for governance review |

---

## Current Status

| Area | Status |
|---|---|
| Repository structure | In progress |
| Notebook 01 — model development and thresholding | In progress |
| Notebook 02 — fairness, explainability and testing | Not started |
| Model card | Placeholder |
| Fairness assessment | Placeholder |
| AI risk and controls matrix | Placeholder |
| Testing summary | Placeholder |

---

## Methodological Approach

The project follows a simple Responsible AI assurance workflow:

~~~text
Data understanding
      ↓
Baseline model development
      ↓
Cost-sensitive threshold selection
      ↓
Fairness and subgroup testing
      ↓
Explainability review
      ↓
Risk-control documentation
      ↓
Conditional governance recommendation
~~~

The focus is not on maximizing predictive performance. The focus is on producing evidence that can support responsible model review.

---

## Key Governance Questions

The project is structured around the following questions:

1. Is the model performance sufficient for decision-support use?
2. Are the predicted probabilities sufficiently calibrated?
3. How does threshold selection affect business cost and customer impact?
4. Do model outcomes differ materially across relevant subgroups?
5. Are explanations sufficiently reliable for internal review and customer-facing reasoning?
6. What controls would be needed before deployment?
7. What residual risks would remain after mitigation?

---

## Cost-Sensitive Thresholding

The German Credit Dataset includes an asymmetric cost framing.

For this case study, the following assumption is used:

| Error type | Interpretation | Assumed cost |
|---|---|---:|
| False negative | Bad credit risk classified as good | 5 |
| False positive | Good credit risk classified as bad | 1 |

The selected threshold is therefore not treated as a purely technical parameter. It is a governance-relevant decision that affects:

- credit risk exposure;
- customer exclusion risk;
- fairness outcomes;
- human review workload;
- business risk appetite.

The selected threshold will be documented in the model card and testing summary after Notebook 01 is completed.

---

## Fairness Testing Scope

Fairness testing will focus on subgroup outcomes using attributes available or derivable from the dataset:

| Attribute | Rationale |
|---|---|
| Gender | Relevant for detecting potential disparate treatment or impact in credit outcomes |
| Age group | Relevant because age can influence credit history, employment stability and financial access |
| Foreign worker status | Relevant as a potential proxy for nationality, migration status or financial exclusion risk |

These attributes are used for Responsible AI testing in a simulated portfolio context. They are not presented as a complete legal discrimination assessment.

The project also assumes that removing protected attributes from the model is not sufficient by itself. Other variables may act as proxies, so the assessment focuses on actual subgroup outcomes and error patterns.

---

## Responsible AI Positioning

This project treats Responsible AI as an operational discipline, not as a set of abstract principles.

The analysis links technical ML work to governance controls:

| Technical issue | Governance relevance |
|---|---|
| Model performance | Determines whether the model is reliable enough for decision support |
| Calibration | Determines whether risk scores can be meaningfully interpreted |
| Threshold choice | Determines the balance between credit risk and customer exclusion |
| Fairness metrics | Identifies potential unequal impact across groups |
| Explainability | Supports internal review, adverse action reasoning and auditability |
| Monitoring | Defines how performance and fairness should be tracked after deployment |
| Human oversight | Reduces the risk of inappropriate reliance on model outputs |

---

## Planned Artefact Logic

The artefacts are intentionally limited to a small number of high-signal documents.

| Artefact | Main question answered |
|---|---|
| Model card | What is the model, what is it for, how well does it perform, and what are its limitations? |
| Fairness assessment | Are there material subgroup disparities and what do they imply? |
| AI risk and controls | What are the main AI risks and what controls would reduce them? |
| Testing summary | Is the model ready for limited use, further testing or rejection? |

---

## Preliminary Governance Recommendation

The final recommendation will be completed after the notebooks are executed.

Expected recommendation format:

~~~text
Conditionally approve / do not approve / approve for testing only
~~~

Likely deployment conditions to assess:

- use as decision-support only;
- human review for adverse or borderline cases;
- documented reason codes;
- subgroup performance monitoring;
- periodic fairness reassessment;
- threshold review after material drift;
- no fully automated rejection without additional controls.

---

## Out of Scope for V1

To keep the project focused and portfolio-ready, the following are out of scope for the first version:

- full production MLOps implementation;
- full EU AI Act conformity assessment;
- complete ISO/IEC 42001 management system mapping;
- full NIST AI RMF control mapping;
- RACI matrix and committee operating model;
- AutoML benchmark;
- deep learning models;
- full bias mitigation benchmark;
- production monitoring dashboard;
- real customer data.

These may be added later if they improve the project’s relevance for specific roles.

---

## Limitations

This is a simulated governance case study based on a public dataset.

Main limitations:

- the German Credit Dataset is small and dated;
- the dataset does not represent a real modern banking portfolio;
- protected attributes are limited and imperfect;
- fairness analysis is illustrative, not a legal conclusion;
- operational controls are proposed conceptually, not implemented in production;
- results should not be interpreted as a deployable credit scoring system.

The value of the project is in the Responsible AI workflow, not in the commercial validity of the model.

---

## References and Inspiration

This project is informed by public work on:

- credit scoring fairness analysis using the German Credit Dataset;
- cost-sensitive evaluation of credit risk models;
- fairness metrics such as disparate impact and equal opportunity;
- explainability methods such as SHAP and feature importance;
- Responsible AI governance artefacts such as model cards, risk assessments and testing summaries;
- AI governance frameworks including the EU AI Act, ISO/IEC 42001 and the NIST AI Risk Management Framework.

Detailed references will be added as the artefacts are completed.

---

## Disclaimer

This repository is a personal portfolio project.

It is not legal advice, financial advice, regulatory advice or a production-ready model governance package. The scenario, controls and recommendations are simulated for educational and professional demonstration purposes.
