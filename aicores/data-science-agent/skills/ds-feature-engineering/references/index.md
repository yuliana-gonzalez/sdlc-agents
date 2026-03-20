# Data Science Agent — Feature Engineering Reference Index

> Read this file first before applying any feature engineering skill step.

## Required Reading Order

### 1. encoding-scaling-guide.md — The Transformations
`./encoding-scaling-guide.md`

Covers the mechanics and trade-offs of every encoding and scaling method:
- One-hot, ordinal, target, frequency, hashing, and embedding encoding
- StandardScaler, MinMaxScaler, RobustScaler, MaxAbsScaler
- Log, Box-Cox, Yeo-Johnson transforms for skewed distributions
- Cyclical encoding for periodic features
- Implementation pitfalls and leakage risks per method

### 2. feature-selection-guide.md — The Selection Methods
`./feature-selection-guide.md`

Covers feature selection methods with selection criteria:
- Filter methods: correlation, mutual information, chi-squared, ANOVA F-score
- Wrapper methods: RFE, forward/backward selection, RFECV
- Embedded methods: L1/Lasso, tree importance, permutation importance
- Multicollinearity: VIF analysis and pair-based elimination
- Dimensionality reduction: PCA, UMAP — when to use instead of selection

---

## Who Reads This
Every feature engineering skill step and every ds-feature-engineering-agent invocation
loads this index as its first action. Reading order is fixed — do not skip.
