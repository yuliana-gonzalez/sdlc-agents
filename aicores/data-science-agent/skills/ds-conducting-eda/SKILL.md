---
name: ds-conducting-eda
description: >
  Structured exploratory data analysis skill. Guides data profiling, distribution
  analysis, correlation detection, outlier identification, and class imbalance checks.
  Produces a reproducible EDA report with actionable findings and visualisation
  recommendations.
context: fork
agent: Plan
---

## When to Apply This Skill
- Starting any new ML or analytics project with an unfamiliar dataset
- Validating data quality before feature engineering or model training
- Investigating unexpected model behaviour by tracing it back to data issues
- Onboarding a new dataset into an existing pipeline

---

Read `references/index.md` before executing any step.

---

## Step 1 — Dataset Intake & Type Inference

Before any analysis, establish the dataset's shape and column semantics:

```
1. Record: n_rows × n_columns
2. For each column, infer:
   - Type: continuous numeric | discrete numeric | nominal categorical |
            ordinal categorical | datetime | boolean | free text | ID/key
   - Unique value count (cardinality)
   - Example values (3–5 non-null samples)
3. Flag columns that appear to be identifiers (high cardinality, no analytical value)
4. Flag potential PII: names, emails, phone numbers, SSNs, addresses
   → document presence; do NOT reproduce PII values in the report
5. Ask user to confirm target variable before proceeding to distribution analysis
```

---

## Step 2 — Missing Value Analysis

Produce a missing value table for every column:

| Column | Type | Missing Count | Missing % | Recommendation |
|--------|------|--------------|-----------|----------------|
| age | numeric | 412 | 8.2% | Median imputation or flag-and-fill |
| income | numeric | 2,100 | 42.0% | Drop column or model-based imputation |
| gender | categorical | 0 | 0.0% | No action |

**Severity thresholds:**
- `> 40%` missing → CRITICAL; recommend dropping or treating as a separate indicator
- `5–40%` missing → HIGH; recommend imputation strategy
- `1–5%` missing → MEDIUM; recommend simple imputation
- `< 1%` missing → LOW; can drop rows safely

**Patterns to detect:**
- Missing not at random (MNAR): missingness correlated with another column → flag
- Monotone missingness in time-series: data stops at a date → flag ingestion issue

---

## Step 3 — Distribution Analysis

### Numeric Columns

For each numeric column, report:

| Statistic | Value |
|-----------|-------|
| Mean | |
| Median | |
| Std Dev | |
| Min | |
| Max | |
| Skewness | |
| Kurtosis | |
| % Zeros | |

**Interpretation rules:**
- `|skewness| > 1.0` → flag as highly skewed; recommend log1p or Box-Cox transform
- `kurtosis > 3` → heavy tails; flag for outlier sensitivity
- `% zeros > 50%` → consider zero-inflation treatment or log(x+1) transform

### Categorical Columns

For each categorical column, report:
- Total unique values
- Top 5 most frequent values and their %
- % of rows in the "other" bucket (all non-top-5 values)
- Rare categories: values appearing in < 1% of rows → recommend grouping into "Other"

### Datetime Columns

- Date range: min → max
- Gap detection: flag missing dates/periods if a regular frequency is expected
- Seasonal components: note if strong weekly/monthly patterns are visible from value counts

---

## Step 4 — Class Imbalance (Classification Tasks Only)

When a target variable is identified and the task is classification:

| Class | Count | % |
|-------|-------|---|
| 0 (negative) | | |
| 1 (positive) | | |

**Imbalance thresholds:**
- Ratio `> 5:1` → HIGH; recommend oversampling (SMOTE), undersampling, or class weights
- Ratio `> 20:1` → CRITICAL; accuracy is misleading; require AUC-PR as primary metric
- Report which metrics to avoid (accuracy) and which to prefer (F1, AUC-ROC, AUC-PR)

---

## Step 5 — Correlation Analysis

### Numeric Features

Compute pairwise Pearson correlations. Present the top 10 highest absolute correlations:

| Feature A | Feature B | Pearson r | Risk |
|-----------|-----------|-----------|------|
| age | tenure | 0.91 | Multicollinearity |
| income | spend | 0.73 | High correlation |

- `|r| > 0.85` → flag as multicollinearity risk; recommend keeping one or using PCA
- `|r| > 0.5 with target` → flag as a strong predictor (positive signal)

### Categorical Features

Use Cramér's V for pairs of categorical columns. Report top 5 pairs.

### Mixed (Numeric × Binary Target)

Use point-biserial correlation. Report top 5 predictive numeric features.

---

## Step 6 — Outlier Identification

For each numeric column, apply both methods and reconcile:

**IQR Method:**
```
lower_bound = Q1 - 1.5 × IQR
upper_bound = Q3 + 1.5 × IQR
Flag values outside [lower_bound, upper_bound]
```

**Z-Score Method:**
```
Flag values where |z| > 3
```

For each flagged column, classify each outlier:
- `data_error`: physically impossible value (e.g., age = 450, negative price)
- `genuine_extreme`: rare but plausible (e.g., a billionaire in income data)

Recommend action per column:
- `data_error` → remove or correct
- `genuine_extreme`, tree model → keep
- `genuine_extreme`, linear/distance model → winsorize at 1st/99th percentile

---

## Step 7 — Visualisation Recommendations

Do not generate plots — recommend the most informative ones:

| Finding | Recommended Plot |
|---------|-----------------|
| Skewed numeric distribution | Histogram + log-scale version |
| Outliers in numeric column | Box plot |
| Class imbalance | Bar chart of target value counts |
| Numeric correlation | Heatmap (Pearson) |
| Categorical vs. target | Grouped bar chart or stacked %  |
| Time-series pattern | Line chart with date on x-axis |
| Feature vs. target (numeric) | Scatter plot with regression line |

---

## Step 8 — Report Assembly

Use `assets/eda-report-template.md` as the output structure. Populate:

1. **Dataset Overview** — shape, column summary table
2. **Data Quality Issues** — table of all findings with severity
3. **Distribution Highlights** — top skewed / unusual columns
4. **Correlation Findings** — multicollinearity risks and top predictors
5. **Outlier Summary** — count and recommended actions
6. **Class Imbalance** — if applicable
7. **Visualisation Recommendations** — prioritised plot list
8. **Recommended Next Steps** — ordered actions before feature engineering

---

## Quality Checklist

Before emitting the final EDA report:
- [ ] Every column has a type inference and cardinality count
- [ ] Missing value analysis is complete for all columns
- [ ] Numeric distributions report skewness for each column
- [ ] Outlier analysis uses both IQR and Z-score methods
- [ ] Correlation analysis covers numeric, categorical, and mixed pairs
- [ ] Class imbalance section present if target variable is categorical
- [ ] No PII values reproduced verbatim in the report
- [ ] Visualisation recommendations are specific (not generic "make plots")
- [ ] Report uses the EDA template structure
