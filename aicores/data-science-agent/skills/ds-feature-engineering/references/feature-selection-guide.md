# Feature Selection Guide

> Methods for selecting the most predictive, non-redundant feature subset.

---

## When to Apply Feature Selection

Feature selection is beneficial when:
- Dataset has > 20 features (reduces overfitting risk)
- Training is slow (reduces compute cost)
- Model interpretability is required (fewer features → clearer explanations)
- Multicollinearity is suspected (correlated features destabilise linear models)

Feature selection is NOT needed when:
- Using regularised models (L1/L2 handle multicollinearity internally)
- Dataset has < 20 features and no obvious redundancy
- Dimensionality reduction (PCA/UMAP) is preferred over feature selection

---

## Step 0 — Remove Obviously Uninformative Features (Always First)

Before any statistical selection:
1. **Zero-variance features:** Constant value across all rows → drop unconditionally
2. **Near-zero-variance:** > 95% same value → drop or flag (use `VarianceThreshold` in sklearn)
3. **Exact duplicates:** Columns with identical values → keep one, drop the rest
4. **Identifiers:** High-cardinality string/integer IDs with no semantic meaning → drop
5. **Free-text columns:** Handle separately (NLP pipeline) — do not include in tabular selection

---

## Step 1 — Multicollinearity Removal

### Pairwise Correlation (Numeric Features)
```
1. Compute n×n Pearson correlation matrix
2. For every pair (i, j) where |r| > 0.90:
   a. Compute |correlation with target| for both i and j
   b. Drop the feature with lower target correlation
   c. If no target exists (unsupervised): drop the feature with higher mean correlation to others
```

### Variance Inflation Factor (VIF) — Linear Models
```
VIF_i = 1 / (1 - R²_i)
Where R²_i = coefficient of determination when feature i is regressed on all other features
```
- VIF < 5 → acceptable
- VIF 5–10 → investigate
- VIF > 10 → remove feature

**Use VIF for linear regression and logistic regression.** Tree models are robust to multicollinearity.

---

## Step 2 — Filter Methods (Fast Screening)

Apply filter methods to rank features by relevance to the target. Use as a first pass before wrapper or embedded methods.

### Pearson Correlation (Numeric → Numeric Target)
- Compute |correlation| with target
- Rank features descending
- Remove features with |r| < 0.01 (effectively zero predictive value)

### Mutual Information (Any Feature Type → Any Target)
- Measures information shared between feature and target (non-linear)
- `sklearn.feature_selection.mutual_info_classif` / `mutual_info_regression`
- Scale-invariant; handles non-linear relationships
- **Preferred over Pearson** when non-linear relationships are expected

### Chi-Squared Test (Categorical → Categorical Target)
- Tests independence between feature and target
- Higher chi-squared statistic → stronger association
- `sklearn.feature_selection.chi2`
- **Requires non-negative features** — apply frequency encoding first

### ANOVA F-Score (Numeric → Categorical Target)
- Tests whether means of a numeric feature differ significantly across target classes
- `sklearn.feature_selection.f_classif`
- Assumes normality; use mutual information for non-normal distributions

---

## Step 3 — Wrapper Methods (Interaction-Aware, Slower)

Use wrapper methods when filter methods leave too many features or interactions matter.

### Recursive Feature Elimination (RFE)
```
1. Train model on all features
2. Rank features by importance
3. Remove the least important feature
4. Repeat until n_features_to_select remain
```
- `sklearn.feature_selection.RFE`
- Use with: linear models (uses coefficients), tree models (uses feature importance)
- **RFECV:** Cross-validated RFE — automatically selects optimal feature count

**Computational cost:** O(n_features × training_cost) — expensive for large feature sets.

### Forward Selection
```
1. Start with no features
2. Add the feature that most improves CV score
3. Repeat until adding features no longer improves score
```
**Use when:** Feature set is small–medium (< 50 features).

### Backward Elimination
```
1. Start with all features
2. Remove the feature whose removal least hurts CV score
3. Repeat until removing features starts hurting score
```
**Use when:** Feature set is medium (< 100 features); baseline model already trained.

---

## Step 4 — Embedded Methods (Best Default for Large Feature Sets)

Embedded methods perform selection as part of model training — most efficient option.

### L1 Regularisation (Lasso)
- Forces coefficients of irrelevant features to exactly zero
- `sklearn.linear_model.Lasso`, `LogisticRegression(penalty='l1')`
- **Best for:** Linear models; produces sparse, interpretable feature sets
- Tune `alpha` (Lasso) or `C` (LogisticRegression) via cross-validation

### Tree-Based Feature Importance
```python
from sklearn.ensemble import RandomForestClassifier
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
importances = model.feature_importances_
```
**Warning:** Built-in importance is biased toward high-cardinality features. Use permutation importance for reliability.

### Permutation Importance (Model-Agnostic)
```python
from sklearn.inspection import permutation_importance
result = permutation_importance(model, X_val, y_val, n_repeats=10, random_state=42)
```
- Measures how much shuffling each feature decreases model performance
- **More reliable** than built-in tree importance
- Works on any trained model
- Compute on **validation set**, not training set, to avoid overfitting bias

---

## Dimensionality Reduction (Alternative to Selection)

Use instead of selection when:
- Many features are correlated but each contributes some information
- Interpretability of individual features is not required
- Input to neural networks where dense representations are preferred

### PCA (Principal Component Analysis)
- Produces orthogonal components that maximise variance
- **Loses feature interpretability** — components are linear combinations
- Fit on train set only; transform all splits
- Rule of thumb: retain components explaining 95% of variance

### UMAP
- Non-linear dimensionality reduction; better preserves local structure than t-SNE
- **Use for:** Visualisation (2D/3D); feature preprocessing for clustering
- Less suitable for supervised downstream tasks than PCA

---

## Selection Decision Summary

| Scenario | Recommended Method |
|----------|-------------------|
| Fast screening, linear relationships | Pearson correlation filter |
| Fast screening, non-linear | Mutual information |
| Small dataset (< 1000 rows), interactions matter | RFE with cross-validation |
| Linear model, interpretability required | Lasso (L1 regularisation) |
| Tree model, large dataset | Permutation importance |
| Very high-dimensional (> 1000 features) | Variance threshold → Mutual info → L1 / tree importance |
| Unsupervised, many correlated features | PCA |
