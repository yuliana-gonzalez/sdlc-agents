# Encoding & Scaling Guide

> Mechanics, trade-offs, and leakage risks for every common transformation method.

---

## Encoding Methods

### One-Hot Encoding
**Use for:** Nominal categoricals with low cardinality (≤ 10 unique values).
**Output:** k-1 binary columns (drop_first=True to avoid dummy variable trap).
**Pitfalls:**
- High cardinality → combinatorial explosion; switch to target or frequency encoding
- New categories at inference time not seen during training → requires handle_unknown strategy
- `handle_unknown='ignore'` (sklearn) → silently encodes as all-zeros; acceptable default

**Leakage risk:** None, if fit on train set only.

---

### Ordinal Encoding
**Use for:** Ordered categoricals (e.g., Low/Medium/High, Cold/Warm/Hot).
**Output:** Single integer column with explicit order.
**Critical:** Always define the order explicitly — do not rely on alphabetical inference.

```python
from sklearn.preprocessing import OrdinalEncoder
enc = OrdinalEncoder(categories=[['low', 'medium', 'high']])
```

**Pitfalls:** Applying to nominal data implies false ordering (e.g., red=0, blue=1, green=2 is wrong).

---

### Target Encoding (Mean Encoding)
**Use for:** Nominal categoricals with medium–high cardinality in supervised tasks.
**Method:** Replace each category with the mean of the target variable for that category.
**Leakage risk: HIGH** — must use cross-validation fold isolation to prevent target leakage.

```python
# Correct: compute target mean within each CV fold on train data only
# Incorrect: compute target mean on the full dataset before splitting
```

**Smoothing:** Apply additive smoothing to handle rare categories:
```
encoded = (count × mean + global_mean × alpha) / (count + alpha)
```
Where `alpha` is a smoothing parameter (typically 10–50).

---

### Frequency Encoding
**Use for:** High-cardinality nominal categoricals when target encoding is too risky.
**Method:** Replace each category with its frequency (count or proportion) in the training set.
**Leakage risk:** Low — computed from feature distribution only, not the target.
**Pitfall:** Two different categories with the same frequency get identical encoded values.

---

### Hashing Encoding (Feature Hashing)
**Use for:** Very high cardinality (> 1,000 unique values) or unknown cardinality at training time.
**Method:** Apply a hash function to map categories to a fixed-size vector.
**Trade-off:** Hash collisions can merge distinct categories — some information loss.
**When to use:** Online learning, streaming pipelines, or NLP feature engineering.

---

### Embedding Encoding
**Use for:** High-cardinality categoricals in deep learning contexts.
**Method:** Learn a dense vector representation as part of the neural network.
**Dimensionality rule of thumb:** `min(50, ceil(n_categories / 2))`
**Requires:** Deep learning framework (PyTorch, TensorFlow/Keras).

---

### Cyclical Encoding (Periodic Features)
**Use for:** Features with natural periodicity — hour, day of week, month, angle.
**Method:** Encode as sine and cosine pair to preserve cyclic distance.

```python
import numpy as np
df['hour_sin'] = np.sin(2 * np.pi * df['hour'] / 24)
df['hour_cos'] = np.cos(2 * np.pi * df['hour'] / 24)

df['month_sin'] = np.sin(2 * np.pi * df['month'] / 12)
df['month_cos'] = np.cos(2 * np.pi * df['month'] / 12)
```

**Why:** Simple label encoding (0–23 for hours) makes hour 23 appear far from hour 0,
which is incorrect — they are adjacent.

---

## Scaling Methods

### StandardScaler
**Formula:** `z = (x - mean) / std`
**Use when:** Normally distributed features; linear models, SVM, KNN, PCA, neural nets.
**Output:** Mean = 0, Std = 1 (approximately).
**Pitfall:** Sensitive to outliers — outliers pull mean and std, compressing non-outlier values.

### MinMaxScaler
**Formula:** `x' = (x - min) / (max - min)`  →  Output range [0, 1]
**Use when:** Bounded input range; neural networks with sigmoid/tanh activations.
**Pitfall:** Sensitive to outliers — a single outlier compresses all other values near 0.

### RobustScaler
**Formula:** `x' = (x - median) / IQR`
**Use when:** Data contains significant outliers; robust to non-normality.
**Output:** Not guaranteed to be in [0, 1]; centered at 0 but unbounded.
**Preferred over StandardScaler** when outliers cannot be removed.

### MaxAbsScaler
**Formula:** `x' = x / max(|x|)`  →  Output range [-1, 1]
**Use when:** Sparse data (preserves sparsity); already centered near 0.

---

## Distribution Transforms (Pre-Scaling)

Apply before scaling when the distribution is skewed:

### Log Transform
```python
df['col_log'] = np.log1p(df['col'])  # log(x + 1) handles zeros
```
**Use when:** Right-skewed, strictly positive values (counts, prices, revenue).
**Reduces:** Right skew and compresses extreme high values.

### Box-Cox Transform
**Use when:** Strictly positive values; automatically finds the best lambda.
**Limitation:** Requires all values > 0; not suitable for data with zeros.

### Yeo-Johnson Transform
**Use when:** Data includes zero or negative values; generalises Box-Cox.
**Recommended default** when automation is preferred over manual log transform.

---

## Leakage Risk Summary

| Method | Leakage Risk | Fit On | Transform On |
|--------|-------------|--------|-------------|
| One-hot encoding | None | Train | Train + Val + Test |
| Ordinal encoding | None | Train | Train + Val + Test |
| Target encoding | HIGH | Train folds only (CV) | Val + Test |
| Frequency encoding | Low | Train | Train + Val + Test |
| StandardScaler | None | Train | Train + Val + Test |
| MinMaxScaler | None | Train | Train + Val + Test |
| RobustScaler | None | Train | Train + Val + Test |
| Log/Box-Cox/Yeo-Johnson | None | N/A (deterministic) | All splits |

**Rule:** If a method has a `.fit()` step, it MUST be fit on training data only.
