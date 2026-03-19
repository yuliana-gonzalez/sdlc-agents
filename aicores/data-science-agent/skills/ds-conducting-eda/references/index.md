# Data Science Agent — EDA Reference Index

> Read this file first before applying any EDA skill step or invoking the ds-eda-agent.

## Required Reading Order

### 1. statistical-tests-guide.md — The Methods
`./statistical-tests-guide.md`

Covers the statistical foundations behind EDA decisions:
- When to use Pearson vs. Spearman vs. Cramér's V vs. point-biserial
- Normality tests: Shapiro-Wilk, D'Agostino, Kolmogorov-Smirnov — when each applies
- Outlier detection methods: IQR, Z-score, isolation forest — trade-offs
- Hypothesis testing for group differences: t-test, Mann-Whitney U, ANOVA, Kruskal-Wallis
- Multiple comparison correction: Bonferroni, Benjamini-Hochberg

### 2. data-quality-checklist.md — The Standards
`./data-quality-checklist.md`

Defines the data quality bar every dataset must meet before modeling:
- Completeness thresholds per column type
- Type validity rules per domain (age, price, date, identifier)
- Consistency checks: cross-column constraints, referential integrity
- Uniqueness requirements for key columns
- Timeliness: acceptable data age per use case
- PII detection patterns and handling rules

---

## Who Reads This
Every EDA skill step and every ds-eda-agent invocation loads this index as its
first action. The index defines reading order — do not skip or reorder.
