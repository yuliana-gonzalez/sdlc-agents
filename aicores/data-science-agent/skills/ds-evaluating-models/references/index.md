# Data Science Agent — Model Evaluation Reference Index

> Read this file first before applying any model evaluation skill step.

## Required Reading Order

### 1. metrics-by-problem-type.md — The Metrics
`./metrics-by-problem-type.md`

The authoritative metric selection guide for every ML problem type:
- Binary classification: accuracy, precision, recall, F1, AUC-ROC, AUC-PR, MCC, Brier Score
- Multi-class classification: macro/micro/weighted F1, confusion matrix, Cohen's kappa
- Regression: RMSE, MAE, R², MAPE, SMAPE, Huber loss
- Ranking: NDCG@k, MAP, MRR, Precision@k
- Clustering: Silhouette, Davies-Bouldin, Calinski-Harabasz, Adjusted Rand Index
- Anomaly detection: AUC-ROC, Precision@k, F1 at threshold
- Time-series forecasting: MAPE, SMAPE, MAE, RMSE, coverage (for prediction intervals)

### 2. bias-fairness.md — The Fairness Framework
`./bias-fairness.md`

Defines fairness criteria and auditing procedures:
- Demographic parity, equalized odds, predictive parity, calibration
- The 80% rule (four-fifths rule) for disparate impact
- Per-group metric computation procedures
- Criterion selection guidance by use case domain
- Remediation strategies: re-weighting, re-sampling, post-processing threshold adjustment
- Regulatory context: EEOC guidelines, EU AI Act, FCRA, ECOA

---

## Who Reads This
Every model evaluation skill step and every ds-model-eval-agent invocation
loads this index as its first action. Reading order is fixed — do not skip.
