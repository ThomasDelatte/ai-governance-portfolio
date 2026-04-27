# Project 01 — Responsible AI Governance for Credit Scoring

## Overview

This project is a Responsible AI and model governance case study for a credit scoring model using the **Statlog German Credit Data** dataset.

The goal is not to build the best possible model. The goal is to show how a predictive model can be reviewed from a governance perspective: fairness, explainability, thresholds, human oversight, monitoring, and documentation.

The project is framed as a **simulated European banking use case** and is intended as a portfolio project for Responsible AI, AI Governance, model risk, and EU AI Act implementation roles.

---

## Simulated Use Case

A European bank is exploring a machine learning model to support personal credit risk assessment.

The model is used as a **decision-support tool** for credit officers. It does not make fully automated credit decisions.

In this scenario:

- the model estimates credit risk based on applicant information;
- credit officers use the score as one input among others;
- final decisions remain human-led;
- decisions remain subject to credit policy, affordability checks, legal requirements, and customer context;
- adverse decisions require documented reasoning and appropriate human review.

This is a simulated scenario, not a production deployment.

---

## Dataset

**Dataset:** Statlog German Credit Data  
**Source:** UCI Machine Learning Repository  
**DOI:** https://doi.org/10.24432/C5NC77  
**Size:** 1,000 observations  
**Features:** 20 input variables  
**Target:** Good / bad credit risk classification  

The dataset contains a mix of categorical and numerical variables, including credit history, credit amount, duration, savings, employment, age, housing, job, personal status/sex, and foreign worker status.

The dataset also includes a cost-sensitive framing: misclassifying a bad credit risk as good is more costly than misclassifying a good credit risk as bad.

---

## Dataset Limitations

The dataset is useful for a public governance simulation, but it is not representative of modern credit decisioning. 

These limitations are treated as governance findings, not ignored.

---

## Methodology

### 1. Data Review

The first notebook reviews the dataset from a governance and modelling perspective.

It identifies:

- target distribution;
- key variable types;
- sensitive and proxy-sensitive attributes;
- basic data quality issues;
- limitations relevant to model governance.

### 2. Model Development

The second notebook trains baseline credit risk classifiers.

The focus is on producing a model suitable for governance review, not on maximizing predictive performance.

The analysis includes:

- preprocessing;
- baseline model training;
- performance evaluation;
- confusion matrix analysis;
- cost-sensitive interpretation of errors.

### 3. Responsible AI Review

The third notebook reviews the model from a Responsible AI perspective.

It covers:

- subgroup performance;
- fairness metrics;
- explainability;
- feature importance;
- threshold trade-offs;
- implications for human oversight and monitoring.

---

## Governance Artefacts

| File | Description |
|---|---|
| [`artefacts/model-card.md`](artefacts/model-card.md) | Documents the model purpose, intended use, dataset, performance, limitations, governance controls, and residual risks. |
| [`artefacts/fairness-assessment.md`](artefacts/fairness-assessment.md) | Reviews fairness risks, subgroup performance, sensitive/proxy-sensitive variables, and mitigation options. |
| [`artefacts/explainability-report.md`](artefacts/explainability-report.md) | Explains model behaviour, key drivers, explanation limits, and how explanations should be used by credit officers. |
| [`artefacts/threshold-decision-memo.md`](artefacts/threshold-decision-memo.md) | Documents threshold trade-offs, cost-sensitive errors, business impact, customer impact, and recommended decision rules. |
| [`artefacts/post-deployment-monitoring-plan.md`](artefacts/post-deployment-monitoring-plan.md) | Defines monitoring requirements for data drift, performance drift, fairness, overrides, complaints, and governance review. |

---

## What This Project Demonstrates

This project demonstrates the ability to:

- connect model outputs to governance decisions;
- document fairness and explainability risks;
- reason about threshold selection in a cost-sensitive setting;
- design human oversight controls;
- define practical monitoring requirements;
- produce governance artefacts suitable for a regulated financial services context.
