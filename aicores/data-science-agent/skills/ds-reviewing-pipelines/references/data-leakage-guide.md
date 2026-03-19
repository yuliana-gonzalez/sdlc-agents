# Data Leakage Guide

> Detection tests and fix patterns for every type of data leakage in ML pipelines.

---

## What Is Data Leakage?

Data leakage occurs when information from outside the training window — including
the target variable or future data — is used to train or evaluate a model.
Leaked models appear to perform well but fail catastrophically in production.

**Leakage is always a CRITICAL defect. No model with confirmed leakage should be deployed.**

---

## Type 1 — Target Leakage

### Definition
A feature encodes information about the target variable that would not be available
at inference time (when the prediction is made).

### Examples
- `days_since_churn` as a feature in a churn prediction model
- `claim_paid` as a feature in a fraud prediction model (fraud is determined after the claim)
- `discharge_diagnosis` as a feature in a hospital readmission model (diagnosis finalised after discharge)

### Detection Tests
1. **Correlation test:** Feature with |correlation| > 0.95 with target → likely leakage
2. **Ablation test:** Remove feature → if model drops to near-chance performance, it was encoding the target
3. **Temporal audit:** Does this feature's value change after the target event occurs?
4. **Business logic review:** Could this value be known at prediction time in production?

### Fix
Remove the feature. If domain knowledge indicates the feature is valid, document the
business logic clearly and add a test that confirms the feature is populated at prediction time.

---

## Type 2 — Temporal Leakage (Time-Series)

### Definition
Future information is visible in training data due to incorrect time window definitions,
look-ahead bias, or incorrect train/test splits.

### Examples
- Rolling 7-day average computed at time t uses values from t+1 through t+6
- `next_month_revenue` used as a feature for predicting current month behaviour
- Random train/test split on a time-series dataset (future rows end up in train)

### Detection Tests
1. **Window audit:** For every aggregation or lag feature, verify the window is anchored at t-1 or earlier
2. **Split audit:** Confirm `min(test_date) > max(train_date)` — no overlap
3. **Date ordering:** Sort data by timestamp and confirm no train rows appear after test rows

### Fix
- Redefine aggregation windows to use only data up to `t-1`
- Replace random split with time-based split: `df[df['date'] < split_date]` for train
- Add a gap period between train end and test start to prevent autocorrelation bleed

---

## Type 3 — Preprocessing Leakage

### Definition
Preprocessing statistics (means, standard deviations, quantiles, target encoding values,
imputation values) are computed on the full dataset — including the test set — before splitting.

### Examples
```python
# WRONG — leakage
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)                    # fit on full dataset
X_train, X_test = train_test_split(X_scaled, ...)     # split after scaling

# CORRECT — no leakage
X_train, X_test = train_test_split(X, ...)            # split first
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)        # fit on train only
X_test_scaled = scaler.transform(X_test)              # transform only
```

### Detection Tests
1. **Code audit:** Search for `.fit_transform(X)` or `.fit(X)` on the full dataset before splitting
2. **Pipeline structure:** The `train_test_split` call must appear before any `.fit()` call
3. **Target encoding audit:** Target means computed inside CV folds only (not on full dataset)

### Fix
Always: `split → fit on train → transform train and test`

Use sklearn Pipelines to enforce this automatically:
```python
from sklearn.pipeline import Pipeline
pipe = Pipeline([('scaler', StandardScaler()), ('model', LogisticRegression())])
pipe.fit(X_train, y_train)       # scaler fitted on X_train only
pipe.predict(X_test)             # scaler applied (transform only) to X_test
```

---

## Type 4 — Join / Aggregation Leakage

### Definition
A table join introduces future information because the joined table is not filtered
to the training observation window.

### Examples
- Training on customer behaviour at time t, then joining to a customer lifetime value table
  that includes post-t transactions
- Joining to a "current status" table when the current status reflects the future outcome

### Detection Tests
1. **Join key audit:** Every join must have a timestamp filter: `ON a.customer_id = b.customer_id AND b.date <= a.event_date`
2. **Table freshness:** Verify the joined table's maximum date is within the training window
3. **Feature bleed test:** Check if joined features have anomalously high correlation with the target

### Fix
Add explicit date filters to every join:
```sql
-- WRONG
SELECT a.*, b.total_spend
FROM events a
LEFT JOIN customer_summary b ON a.customer_id = b.customer_id

-- CORRECT
SELECT a.*, b.total_spend
FROM events a
LEFT JOIN customer_summary b
  ON a.customer_id = b.customer_id
  AND b.snapshot_date <= a.event_date
```

---

## Type 5 — Sampling / Entity Leakage

### Definition
The same entity (user, household, device, account) appears in both train and test,
causing the model to "memorise" entity-level patterns from train that also appear in test.

### Examples
- A user makes 100 transactions; 80 in train and 20 in test (random row split, not entity split)
- Household members split across train/test (related individuals share features)
- Session-level split when a user has sessions in both train and test

### Detection Tests
1. **Entity ID overlap:** `set(train_ids) & set(test_ids)` must be empty
2. **Duplicate entity check:** If the same entity appears in both, measure the % overlap
3. **Performance anomaly:** Model performs well on all test entities that appeared in train; poorly on new entities

### Fix
Split by entity ID, not by row:
```python
entity_ids = df['customer_id'].unique()
train_ids, test_ids = train_test_split(entity_ids, test_size=0.2, random_state=42)
df_train = df[df['customer_id'].isin(train_ids)]
df_test  = df[df['customer_id'].isin(test_ids)]
```

---

## Leakage Severity and Response

| Leakage Type | Severity | Response |
|-------------|----------|----------|
| Target leakage (confirmed) | CRITICAL | Block deployment; remove feature; retrain |
| Temporal leakage (confirmed) | CRITICAL | Block deployment; fix window; retrain |
| Preprocessing leakage (confirmed) | CRITICAL | Fix pipeline; retrain; re-evaluate |
| Join/aggregation leakage | CRITICAL | Fix join filter; retrain |
| Entity leakage | HIGH | Fix split strategy; retrain; may not invalidate model if overlap < 5% |
| Suspected leakage (unconfirmed) | HIGH | Block until confirmed or cleared |

---

## Automated Leakage Checks (CI/CD Integration)

Add these checks to your CI/CD pipeline to catch leakage automatically:

```python
# Check 1: No entity overlap between splits
assert len(set(X_train.index) & set(X_test.index)) == 0, "Entity leakage detected"

# Check 2: Test dates > train dates (time-series)
assert X_test['date'].min() > X_train['date'].max(), "Temporal leakage: test dates overlap train"

# Check 3: Scaler not fit on test data (check using scaler.mean_ before/after transform)
# This requires architecture-level enforcement via sklearn Pipeline

# Check 4: Feature correlation with target below leakage threshold
high_corr = correlations[correlations['abs_corr_with_target'] > 0.95]
assert len(high_corr) == 0, f"Possible target leakage: {high_corr['feature'].tolist()}"
```
