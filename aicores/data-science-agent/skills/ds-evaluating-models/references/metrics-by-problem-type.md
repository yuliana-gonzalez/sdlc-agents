# Metrics by Problem Type

> The authoritative reference for metric selection across all ML problem types.

---

## Binary Classification

### Metric Definitions

| Metric | Formula | Range | Interpretation |
|--------|---------|-------|----------------|
| Accuracy | (TP + TN) / N | [0, 1] | % of correct predictions; misleading on imbalanced data |
| Precision | TP / (TP + FP) | [0, 1] | Of all positive predictions, how many are correct |
| Recall (Sensitivity) | TP / (TP + FN) | [0, 1] | Of all actual positives, how many were found |
| Specificity | TN / (TN + FP) | [0, 1] | Of all actual negatives, how many were correctly rejected |
| F1 Score | 2 × (P × R) / (P + R) | [0, 1] | Harmonic mean of precision and recall |
| F-beta | (1+β²)(P × R) / (β²P + R) | [0, 1] | β > 1 weights recall more; β < 1 weights precision more |
| AUC-ROC | Area under ROC curve | [0.5, 1] | Rank-ordering quality; threshold-independent |
| AUC-PR | Area under Precision-Recall curve | [0, 1] | Better than AUC-ROC for imbalanced classes |
| MCC | (TP·TN - FP·FN) / √[(TP+FP)(TP+FN)(TN+FP)(TN+FN)] | [-1, 1] | Best single metric for imbalanced binary; 1 = perfect |
| Brier Score | mean((p̂ - y)²) | [0, 1] | Calibration quality; lower is better |
| Log Loss | -mean(y·log(p̂) + (1-y)·log(1-p̂)) | [0, ∞) | Penalises confident wrong predictions; lower is better |

### Metric Selection Table

| Class Balance | Business Priority | Recommended Primary | Recommended Secondary |
|--------------|-------------------|--------------------|-----------------------|
| Balanced (< 3:1) | Equal FP/FN cost | F1, AUC-ROC | Precision, Recall |
| Imbalanced (3–10:1) | Equal FP/FN cost | AUC-PR, F1 | MCC |
| Imbalanced (> 10:1) | Equal FP/FN cost | AUC-PR, MCC | F1 at optimal threshold |
| Any | FN very costly (disease, fraud) | Recall, F-beta (β > 1) | AUC-ROC |
| Any | FP very costly (spam, alert fatigue) | Precision, F-beta (β < 1) | AUC-ROC |
| Any | Probability calibration needed | Brier Score | Log Loss |

**Rule:** Accuracy should only be the primary metric when classes are balanced AND costs of FP and FN are equal.

---

## Multi-Class Classification

| Metric | Averaging | When to Use |
|--------|-----------|-------------|
| Accuracy | N/A | Balanced classes; uniform cost |
| Macro F1 | Unweighted mean across classes | Imbalanced classes; all classes equally important |
| Micro F1 | Aggregate TP/FP/FN before computing | Dominated by majority class; equal instance cost |
| Weighted F1 | Weighted by class support | Imbalanced classes; majority class matters more |
| Cohen's Kappa | N/A | Accounts for chance agreement; useful for inter-rater tasks |
| Confusion Matrix | N/A | Always produce; reveals per-class error patterns |

**Default recommendation:** Macro F1 + confusion matrix for most multi-class problems.

---

## Regression

| Metric | Formula | Sensitive To | When to Use |
|--------|---------|-------------|-------------|
| RMSE | √(mean((y - ŷ)²)) | Outliers (squares errors) | When large errors are especially costly |
| MAE | mean(\|y - ŷ\|) | Less sensitive to outliers | Robust baseline; interpretable in target units |
| R² | 1 - SS_res / SS_tot | Scale | Proportion of variance explained; use with RMSE |
| MAPE | mean(\|y - ŷ\| / \|y\|) × 100 | Zero values (undefined) | Relative error; comparable across scales |
| SMAPE | mean(2·\|y - ŷ\| / (\|y\| + \|ŷ\|)) × 100 | Bounded | More stable than MAPE near zero |
| Huber Loss | MAE for large errors, MSE for small | Configurable | Combines RMSE and MAE robustness |

### Regression Selection Table

| Scenario | Primary | Secondary |
|----------|---------|-----------|
| Target has outliers | MAE | MAPE |
| Large errors very costly | RMSE | R² |
| Need interpretable % error | MAPE | SMAPE |
| Comparing models across different targets | R² | MAPE |
| Imbalanced error cost by magnitude | Huber or quantile regression loss | — |

---

## Time-Series Forecasting

| Metric | Notes |
|--------|-------|
| MAPE | Most common; undefined when y = 0 |
| SMAPE | Handles near-zero values better than MAPE |
| MAE | Robust to outliers; in target units |
| RMSE | Penalises large errors; use when spikes matter |
| WAPE | Weighted MAPE; handles zero values |
| Coverage | % of actual values falling within prediction interval; target ≥ 90% |
| Interval Width | Average prediction interval width; narrower is better for same coverage |

**Special concern:** Always use time-aware validation — never random split on time-series data.

---

## Ranking

| Metric | Definition | Use When |
|--------|-----------|----------|
| NDCG@k | Normalised Discounted Cumulative Gain at k | Position matters; graded relevance |
| MAP | Mean Average Precision across queries | Binary relevance; recall-oriented |
| MRR | Mean Reciprocal Rank | Only the first relevant result matters |
| Precision@k | Precision in top-k results | Simple; binary relevance |

---

## Clustering (Unsupervised)

### Internal Metrics (No Ground Truth)

| Metric | Range | Better When |
|--------|-------|-------------|
| Silhouette Score | [-1, 1] | Higher (tighter clusters, larger separation) |
| Davies-Bouldin Index | [0, ∞) | Lower (compact, well-separated clusters) |
| Calinski-Harabasz Index | [0, ∞) | Higher (dense, well-separated clusters) |

### External Metrics (With Ground Truth)

| Metric | Range | Notes |
|--------|-------|-------|
| Adjusted Rand Index (ARI) | [-1, 1] | Corrects for chance; 1 = perfect |
| Normalized Mutual Information (NMI) | [0, 1] | Measures shared information |
| Fowlkes-Mallows Index | [0, 1] | Geometric mean of precision and recall for pairs |

---

## Anomaly Detection

| Metric | Notes |
|--------|-------|
| AUC-ROC | Good for threshold-free evaluation; misleading if anomalies are very rare |
| AUC-PR | Better than AUC-ROC when anomalies are rare (< 5%) |
| Precision@k | How many of the top-k flagged instances are true anomalies |
| F1 at threshold | Requires choosing a threshold; tune on validation set |

---

## Calibration

A model is well-calibrated when its predicted probability matches observed frequency.

**Test:** Plot calibration curve — if P(y=1 | ŷ=0.7) ≈ 0.70, the model is calibrated.

| Tool | Use |
|------|-----|
| Reliability diagram (calibration curve) | Visual check |
| Brier Score | Scalar summary; lower = better |
| Expected Calibration Error (ECE) | Mean absolute difference between predicted and actual |
| Platt Scaling | Post-hoc calibration for SVM, tree models |
| Isotonic Regression | Post-hoc calibration; more flexible than Platt |
| Temperature Scaling | Post-hoc calibration for neural networks |
