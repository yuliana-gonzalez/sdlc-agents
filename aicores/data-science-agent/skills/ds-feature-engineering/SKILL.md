---
name: ds-feature-engineering
description: >
  Feature engineering strategy and review skill. Covers leakage detection,
  encoding strategy selection per column type and cardinality, scaling selection
  per distribution and algorithm, feature selection using filter, wrapper, and
  embedded methods. Produces a feature catalog and transformation pipeline.
context: fork
agent: Plan
---

## When to Apply This Skill
- Preparing features before training any supervised or unsupervised model
- Reviewing an existing feature pipeline for leakage or correctness
- Selecting the best feature subset from a large candidate set
- Deciding which encoding or scaling strategy to apply per column

---

Read `references/index.md` before executing any step.

---

## Step 1 — Leakage Detection (Always First)

Leakage is a CRITICAL defect. Check all of the following before any transformation:

### Target Leakage
A feature is derived from or directly encodes information about the target variable
in a way that would not be available at inference time.

**Detection patterns:**
- Feature is computed using the label (e.g., "is_churned" used to create "days_since_churn")
- Feature is a proxy for the label with near-perfect correlation (|r| > 0.95)
- Feature is only populated after the target event occurs

**Test:** Remove the feature and check if model performance drops to chance → if yes, it was leakage.

### Temporal Leakage (Time-Series)
Future information visible during training due to incorrect window definitions.

**Detection patterns:**
- Aggregation windows include the current or future timestep
- Lag features use negative lags (look-ahead)
- Train/test split does not respect time ordering (random split on time-series data)

### Preprocessing Leakage
Statistics computed on the full dataset (including test) are used to transform training data.

**Detection patterns:**
- `scaler.fit(X)` called on `X_full` instead of `X_train`
- Imputation mean/median computed on full dataset
- Encoding target statistics computed without cross-validation fold isolation

**Fix:** Always fit preprocessing on train split only. Apply (transform only) to validation and test.

### Train/Test Contamination
- Duplicate rows appear in both train and test splits
- Rows from the same entity (user, household) appear in both splits (entity leakage)

---

## Step 2 — Encoding Strategy

For each categorical column, use this decision table:

| Cardinality | Ordinality | Recommendation | Notes |
|-------------|-----------|---------------|-------|
| Binary (2 values) | Any | Map to 0/1 | Explicit mapping preferred over label encoder |
| Low (3–10) | Unordered (nominal) | One-hot encoding | Drop first to avoid dummy trap |
| Low (3–10) | Ordered (ordinal) | Ordinal encoding with explicit order | Define order explicitly; do not infer |
| Medium (11–50) | Unordered | Target encoding with k-fold CV | Prevents leakage; apply only on train folds |
| Medium (11–50) | Unordered, tree model | Frequency encoding | Count or proportion of each category |
| High (> 50) | Unordered | Hashing trick or embedding | Hashing: fast, lossy; embedding: needs deep model |
| High (> 50) | Unordered, interpretability required | Frequency encoding + group rare categories | Group values < 1% into "Other" |
| Datetime | — | Decompose to components | year, month, day_of_week, hour; cyclical encoding for periodic |

### Cyclical Encoding for Periodic Features
Apply to: hour of day, day of week, month of year.

```
feature_sin = sin(2π × value / period)
feature_cos = cos(2π × value / period)
```

Example: hour of day → `hour_sin = sin(2π × hour / 24)`, `hour_cos = cos(2π × hour / 24)`

---

## Step 3 — Scaling Strategy

For each numeric column, use this decision table:

| Distribution | Outliers Present | Algorithm Type | Recommendation |
|-------------|-----------------|---------------|---------------|
| Normal / near-normal | Few | Linear, SVM, KNN | StandardScaler (zero mean, unit variance) |
| Normal | Few | Neural network with bounded activations | MinMaxScaler to [0, 1] |
| Skewed | Few | Linear, distance-based | Log transform then StandardScaler |
| Any | Many | Linear, SVM, KNN | RobustScaler (median and IQR-based) |
| Any | Any | Tree-based (RF, XGBoost, LightGBM) | No scaling needed — trees are invariant to monotonic transforms |
| Bounded domain [0,1] | Few | Logistic regression | MinMaxScaler |

**Order of operations:** Apply transforms in this order:
1. Log/Box-Cox transform (if skewed)
2. Clip outliers (if using RobustScaler approach)
3. Scale (StandardScaler / MinMaxScaler / RobustScaler)

**Critical:** Fit scaler on train split. Call `.transform()` on validation and test — never `.fit_transform()`.

---

## Step 4 — Feature Selection

### Phase 4a — Remove Uninformative Features
```
1. Zero-variance features: constant columns → drop
2. Near-zero-variance: > 95% same value → drop or flag
3. Duplicate columns (identical values): keep one, drop duplicates
4. High-cardinality identifiers (ID columns): drop; they cause overfitting, not information
```

### Phase 4b — Remove Redundant Features (Multicollinearity)
```
1. Compute pairwise Pearson correlation on numeric features
2. For each pair with |r| > 0.90:
   a. Keep the feature with higher absolute correlation with the target
   b. Drop the other
3. Run VIF (Variance Inflation Factor) for linear models: VIF > 10 → remove
```

### Phase 4c — Select by Relevance

Choose method based on dataset size and algorithm:

| Method | When to Use | Pros | Cons |
|--------|------------|------|------|
| Pearson / mutual information filter | Large datasets, fast screening | Fast, scalable | Ignores feature interactions |
| Chi-squared (for classification) | Categorical features vs. categorical target | Fast, no model needed | Assumes independence |
| Recursive Feature Elimination (RFE) | Small–medium datasets (< 10k features) | Considers interactions | Slow on large sets |
| L1 (Lasso) regularization | Linear models; interpretability needed | Automatic sparse selection | Requires hyperparameter tuning |
| Tree feature importance (RF, XGBoost) | Large datasets; non-linear models | Captures interactions | Can overfit; use permutation importance for reliability |
| Permutation importance | Any trained model | Model-agnostic | Slower than built-in importance |

**Recommended default for tabular data:**
1. Remove zero-variance and duplicates
2. Remove multicollinear pairs (|r| > 0.90)
3. Run LightGBM or RandomForest feature importance on train set
4. Select top-k features + manually inspect dropped features with high domain relevance

---

## Step 5 — Feature Catalog Assembly

Use `assets/feature-engineering-report-template.md` as structure.

For each feature, populate one row:

| Feature Name | Original Type | Transformation | Encoding/Scaling | Leakage Status | Keep / Drop | Rationale |
|-------------|--------------|---------------|-----------------|---------------|-------------|-----------|

Include:
- **Transformation pipeline pseudocode** — ordered steps (must be leakage-safe)
- **leakage_report.md** — if any leakage was detected (one issue per row with fix)

---

## Quality Checklist

Before emitting the feature catalog:
- [ ] Leakage detection completed before any encoding or scaling decisions
- [ ] Every categorical column has an encoding choice with cardinality-based rationale
- [ ] Every numeric column has a scaling choice with distribution and algorithm rationale
- [ ] Scaler/encoder fit is on train split only (documented in pipeline pseudocode)
- [ ] At least one feature selection method was applied and documented
- [ ] Multicollinear pairs (|r| > 0.90) are identified and resolved
- [ ] Constant and duplicate columns are removed
- [ ] Cyclical encoding applied to all periodic datetime features
- [ ] Feature catalog uses the template structure
