# EDA Report — [Dataset Name]

**Date:** [YYYY-MM-DD]
**Analyst / Agent:** ds-eda-agent
**Dataset Source:** [file path, table name, or description]
**Target Variable:** [column name or "None — unsupervised"]
**Problem Type:** [classification | regression | clustering | exploratory]

---

## 1. Dataset Overview

| Property | Value |
|----------|-------|
| Rows | |
| Columns | |
| Numeric columns | |
| Categorical columns | |
| Datetime columns | |
| Boolean columns | |
| Text / free-form columns | |
| Identifier columns (excluded from analysis) | |
| PII columns detected | |

### Column Summary

| Column | Type | Cardinality | Missing % | Notes |
|--------|------|-------------|-----------|-------|
| | | | | |

---

## 2. Data Quality Score

**Overall Score:** `___%`  | **Grade:** [A / B / C / F]

| Dimension | Score | Issues Found |
|-----------|-------|-------------|
| Completeness | % | |
| Type Validity | % | |
| Range & Constraints | % | |
| Uniqueness | % | |
| Consistency | % | |

---

## 3. Data Quality Issues

All issues requiring action before modeling:

| Severity | Column | Issue | Count / % | Recommended Action |
|----------|--------|-------|-----------|-------------------|
| CRITICAL | | | | |
| HIGH | | | | |
| MEDIUM | | | | |
| LOW | | | | |

---

## 4. Distribution Highlights

### Skewed Numeric Columns

| Column | Skewness | Direction | Recommended Transform |
|--------|----------|-----------|----------------------|
| | | | |

### High-Cardinality Categorical Columns

| Column | Unique Values | Top Category % | Rare Categories (< 1%) |
|--------|--------------|---------------|------------------------|
| | | | |

### Datetime Columns

| Column | Min Date | Max Date | Gaps Detected |
|--------|----------|----------|---------------|
| | | | |

---

## 5. Class Imbalance (Classification Only)

| Class | Label | Count | % |
|-------|-------|-------|---|
| | | | |

**Imbalance Ratio:** [X : 1]
**Severity:** [CRITICAL / HIGH / ACCEPTABLE]
**Recommended Metrics:** [AUC-PR / F1 / AUC-ROC — specify why]
**Recommended Strategy:** [SMOTE / class weights / undersampling / none]

---

## 6. Correlation Findings

### Top Correlated Feature Pairs (Multicollinearity Risk)

| Feature A | Feature B | Method | Coefficient | Risk Level |
|-----------|-----------|--------|-------------|------------|
| | | Pearson / Cramér's V | | HIGH / MEDIUM |

### Top Predictive Features (vs. Target)

| Feature | Method | Coefficient | Direction |
|---------|--------|-------------|-----------|
| | | | positive / negative |

---

## 7. Outlier Summary

| Column | IQR Outliers | Z-Score Outliers | Classification | Recommended Action |
|--------|-------------|-----------------|---------------|-------------------|
| | | | data_error / genuine_extreme | remove / cap / keep |

---

## 8. Visualisation Recommendations

Priority-ordered list of plots that will provide the most insight:

| Priority | Plot Type | Columns | Insight Expected |
|----------|-----------|---------|-----------------|
| 1 | | | |
| 2 | | | |
| 3 | | | |

---

## 9. Recommended Next Steps

Actions to take before feature engineering, in priority order:

1. [CRITICAL] — [Action]
2. [HIGH] — [Action]
3. [MEDIUM] — [Action]
4. [INFO] — [Action]

---

## 10. Analyst Notes

<!-- Free-form observations, domain-specific context, or questions for the data engineer -->
