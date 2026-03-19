# Statistical Tests Guide

> Reference for selecting and interpreting statistical methods used during EDA.

---

## Correlation Methods

### Pearson Correlation (r)
**Use when:** Both variables are continuous and approximately normally distributed.
**Range:** -1 to +1
**Interpretation:**
- `|r| < 0.3` → weak / negligible
- `0.3 ≤ |r| < 0.6` → moderate
- `0.6 ≤ |r| < 0.85` → strong
- `|r| ≥ 0.85` → multicollinearity risk — flag for removal or PCA

**Assumption check:** Plot a scatter to confirm linearity before trusting Pearson.
**When to avoid:** Skewed distributions, ordinal data, non-linear relationships.

---

### Spearman Rank Correlation (ρ)
**Use when:** Variables are ordinal, or numeric but non-normally distributed.
**Robustness:** Less sensitive to outliers than Pearson.
**Interpretation:** Same ranges as Pearson, but measures monotonic (not linear) association.

---

### Cramér's V
**Use when:** Both variables are categorical (nominal).
**Range:** 0 (no association) to 1 (perfect association)
**Interpretation:**
- `V < 0.1` → negligible
- `0.1 ≤ V < 0.3` → weak
- `0.3 ≤ V < 0.5` → moderate
- `V ≥ 0.5` → strong

**Formula note:** Cramér's V corrects chi-squared for sample size and table dimensions.

---

### Point-Biserial Correlation
**Use when:** One variable is continuous, the other is binary (0/1).
**Use case:** Measuring numeric feature correlation with a binary target variable.
**Interpretation:** Same ranges as Pearson; mathematically equivalent for binary groups.

---

## Normality Tests

Use normality tests before deciding between parametric and non-parametric methods.

### Shapiro-Wilk
**Best for:** Small to medium samples (n < 5,000).
**Null hypothesis:** Data is normally distributed.
**Decision rule:** p < 0.05 → reject normality; use non-parametric methods.
**Limitation:** Very sensitive to large samples — even trivial deviations become significant.

### D'Agostino–Pearson
**Best for:** Medium to large samples (n > 50).
**Tests:** Skewness and kurtosis jointly.
**More robust** than Shapiro-Wilk for larger datasets.

### Kolmogorov-Smirnov (KS Test)
**Best for:** Comparing a sample to a reference distribution, or two samples to each other.
**Use case in EDA:** Check if production data matches training distribution (distribution shift detection).

---

## Outlier Detection Methods

### IQR Method (Tukey Fences)
```
Q1 = 25th percentile
Q3 = 75th percentile
IQR = Q3 - Q1
Lower fence = Q1 - 1.5 × IQR
Upper fence = Q3 + 1.5 × IQR
```
**When to use:** Symmetric or mildly skewed distributions; robust to non-normality.
**Limitation:** May flag too many outliers in heavy-tailed distributions.

### Z-Score Method
```
z = (x - mean) / std
Flag: |z| > 3
```
**When to use:** Approximately normal distributions.
**Limitation:** Sensitive to the outliers it is trying to detect (non-robust).
**Alternative:** Modified Z-score using median and MAD (median absolute deviation) for robustness.

### Isolation Forest
**When to use:** High-dimensional data; multivariate outlier detection.
**Output:** Anomaly score per row; threshold at contamination rate (typically 0.05–0.10).
**Use in EDA:** Recommended when univariate methods are insufficient.

---

## Group Difference Tests

### t-Test (Independent Samples)
**Use when:** Comparing means of a continuous variable between two groups.
**Assumptions:** Normality, equal variance (Levene's test first).
**When assumptions fail:** Use Mann-Whitney U instead.

### Mann-Whitney U (Wilcoxon Rank-Sum)
**Use when:** Non-normal distributions or ordinal outcome; two-group comparison.
**Tests:** Whether one group's values tend to be larger than the other's.
**Non-parametric alternative** to the t-test.

### ANOVA (One-Way)
**Use when:** Comparing means across 3+ groups; normality assumed.
**Post-hoc:** Tukey HSD for pairwise comparisons after a significant ANOVA result.

### Kruskal-Wallis
**Use when:** 3+ group comparison without normality assumption.
**Non-parametric alternative** to ANOVA.
**Post-hoc:** Dunn's test with Bonferroni correction.

---

## Multiple Comparison Correction

When running many hypothesis tests simultaneously (e.g., testing all features against the target), control for false discovery:

### Bonferroni Correction
**Method:** Divide alpha by number of tests: `α_adjusted = α / n_tests`
**When to use:** Small number of tests (< 20); conservative; minimises false positives.
**Trade-off:** Increases false negatives as n_tests grows.

### Benjamini-Hochberg (FDR)
**Method:** Controls false discovery rate rather than family-wise error rate.
**When to use:** Large number of tests (genomics, wide-format data); less conservative.
**Preferred** for EDA feature screening where some false positives are acceptable.

---

## Skewness & Transform Guide

| Skewness | Distribution Shape | Recommended Transform |
|----------|-------------------|----------------------|
| 0.5–1.0 | Mild right skew | Square root: `√x` |
| 1.0–2.0 | Moderate right skew | Log: `log(x+1)` |
| > 2.0 | Severe right skew | Box-Cox or log |
| < -1.0 | Left skew | Reflect then log: `log(max(x)+1 - x)` |
| Near 0 | Symmetric | No transform needed |

**Note:** Always apply transforms to train set only; fit parameters on train, apply to test.
