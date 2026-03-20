# Feature Engineering Report — [Dataset / Project Name]

**Date:** [YYYY-MM-DD]
**Agent:** ds-feature-engineering-agent
**Problem Type:** [binary classification | multi-class | regression | clustering | time-series]
**Target Variable:** [column name or "None"]
**Algorithm Family:** [tree-based | linear | distance-based | neural network | mixed]

---

## 1. Leakage Audit Summary

**Overall Leakage Status:** [CLEAN / LEAKAGE DETECTED]

| # | Issue Type | Column(s) | Description | Severity | Fix |
|---|-----------|-----------|-------------|----------|-----|
| | Target leakage / Temporal / Preprocessing / Contamination | | | CRITICAL | |

> If LEAKAGE DETECTED: pipeline is blocked until all CRITICAL issues are resolved.

---

## 2. Feature Catalog

One row per feature in the final feature set:

| Feature Name | Original Type | Cardinality | Transformation | Encoding / Scaling | Leakage | Decision | Rationale |
|-------------|--------------|-------------|---------------|-------------------|---------|----------|-----------|
| | numeric / categorical / datetime / boolean | | log / none / cyclical | StandardScaler / one-hot / target-enc / none | CLEAN / FLAG | Keep / Drop / Monitor | |

**Summary:**
- Total input features: ___
- Features kept: ___
- Features dropped: ___
- Features to monitor: ___

---

## 3. Multicollinearity Report

Features removed due to high correlation (|r| > 0.90):

| Feature Removed | Correlated With | Correlation | Reason Kept Other |
|----------------|----------------|-------------|------------------|
| | | | Higher target correlation |

---

## 4. Feature Importance Ranking

(Populate after initial model fit or using filter method scores)

| Rank | Feature | Method | Score | Keep / Drop |
|------|---------|--------|-------|-------------|
| 1 | | Permutation / Mutual Info / Lasso | | Keep |

---

## 5. Transformation Pipeline

Ordered steps for the full preprocessing pipeline (leakage-safe):

```
Step 1 — Split data into train / validation / test BEFORE any fitting

Step 2 — Imputation (fit on train only)
  - [column]: [strategy — median / mode / model-based]

Step 3 — Encoding (fit on train only)
  - [column]: [method — one-hot / ordinal / target-enc / frequency]

Step 4 — Scaling (fit on train only)
  - [column]: [scaler — StandardScaler / RobustScaler / MinMaxScaler]

Step 5 — Feature Selection
  - Method: [filter / RFE / Lasso / permutation importance]
  - Final feature count: ___

Step 6 — Apply fitted pipeline to validation and test sets (transform only)
```

---

## 6. Leakage Report

*(Only produced if leakage was detected)*

| # | Leakage Type | Column | Description | Severity | Remediation |
|---|-------------|--------|-------------|----------|-------------|
| | Target / Temporal / Preprocessing / Contamination | | | CRITICAL | |

---

## 7. Recommended Next Steps

1. [Fix leakage issues above before training]
2. [Validate transformation pipeline on a held-out sample]
3. [Re-run EDA on engineered features to confirm distribution improvements]
4. [Pass feature_catalog.md to ds-model-eval-agent after training]

---

## 8. Notes

<!-- Domain-specific context, edge cases, or decisions that required human judgement -->
