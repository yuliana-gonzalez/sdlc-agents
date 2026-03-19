# Data Quality Checklist

> Standards every dataset must meet before feature engineering or model training begins.

---

## 1. Completeness

| Threshold | Severity | Action |
|-----------|----------|--------|
| > 40% missing in any column | CRITICAL | Drop column or treat as binary indicator |
| > 40% missing in target variable | CRITICAL | Halt — label quality is compromised |
| 20–40% missing | HIGH | Model-based imputation or explicit missingness feature |
| 5–20% missing | MEDIUM | Median/mode imputation; add missingness indicator flag |
| 1–5% missing | LOW | Simple imputation; dropping rows is acceptable |
| < 1% missing | INFO | Drop rows or fill; no structural concern |

**Missing not at random (MNAR):** If missingness is correlated with the value itself
(e.g., high-income respondents skip the income field), imputation will introduce bias.
Flag and surface to the data scientist before proceeding.

---

## 2. Type Validity

Check that every column's data type matches its domain semantics:

| Column Domain | Expected Type | Common Violation |
|--------------|--------------|-----------------|
| Age | Positive integer or float | Stored as string; negative values |
| Price / Revenue | Non-negative float | Negative values; commas in string |
| Date / Timestamp | datetime64 | Stored as object; mixed formats |
| Identifier (ID) | Integer or UUID string | Numeric ID accidentally treated as feature |
| Proportion / Rate | Float in [0.0, 1.0] | Values > 1 or < 0 |
| Count | Non-negative integer | Negative counts; float type |
| Boolean flag | 0/1 or True/False | Mixed: "yes"/"no", "Y"/"N", "1"/"0" as strings |

**Action:** Cast columns to correct types before any analysis. Log all coercions.

---

## 3. Range & Constraint Checks

Validate domain-specific constraints per column. Common examples:

| Column | Constraint | Flag if violated |
|--------|-----------|-----------------|
| age | 0 ≤ age ≤ 120 | Negative or > 120 |
| probability | 0.0 ≤ p ≤ 1.0 | Outside [0, 1] |
| revenue | revenue ≥ 0 | Negative (unless credits are valid) |
| latitude | -90 ≤ lat ≤ 90 | Outside range |
| longitude | -180 ≤ lon ≤ 180 | Outside range |
| percentage | 0 ≤ pct ≤ 100 | Negative or > 100 |

---

## 4. Uniqueness

| Column Type | Uniqueness Requirement |
|-------------|----------------------|
| Primary key / row ID | 100% unique — duplicates are critical defects |
| Target variable row | Check for duplicate (feature_key, target) pairs — label leakage risk |
| Transaction ID | 100% unique |
| Email / phone | Near-unique (minor duplicates may be valid) |

**Duplicate row check:**
- Exact duplicates (all columns match): remove, log count
- Near-duplicates (fuzzy match on key columns): flag for human review

---

## 5. Consistency (Cross-Column Constraints)

Check logical relationships between columns:

| Rule | Example | Action if violated |
|------|---------|-------------------|
| Start ≤ End | `signup_date ≤ churn_date` | Flag as data error |
| Parent ≥ Child | `total_spend ≥ product_spend` | Flag; check aggregation logic |
| Conditional presence | If `status = "shipped"`, `ship_date` must be non-null | Flag MNAR |
| Referential integrity | `customer_id` must exist in the customers table | Flag orphan records |

---

## 6. Timeliness

| Use Case | Maximum Acceptable Data Age |
|----------|---------------------------|
| Real-time fraud detection | < 24 hours |
| Daily operational models | < 7 days |
| Monthly reporting | < 35 days |
| Historical analysis | Dataset date range must match stated scope |

Flag if the most recent record is older than the threshold for the use case.

---

## 7. PII Detection Patterns

Detect the following patterns and flag their presence — do NOT reproduce values in reports:

| PII Type | Detection Pattern |
|----------|-----------------|
| Email address | Column name contains "email" or values match `*@*.* ` |
| Phone number | Values match 10–15 digit patterns with optional separators |
| Social Security Number | Values match `\d{3}-\d{2}-\d{4}` |
| Credit card number | Values match 13–19 digit sequences (Luhn check) |
| Full name | Column name is "name", "full_name", "first_name", "last_name" |
| Address | Column name contains "address", "street", "zip", "postal" |
| Date of birth | Column name contains "dob", "birth_date", "birthdate" |
| IP address | Values match IPv4 or IPv6 patterns |

**Required action on PII detection:**
1. Flag column name and PII type in the EDA report
2. Do not display sample values from PII columns
3. Recommend anonymisation, pseudonymisation, or removal based on use case
4. Note if PII could be reconstructed from combinations of non-PII columns

---

## 8. Data Quality Score (Summary)

Compute an overall data quality score before handing off to feature engineering:

```
score = (
    completeness_score × 0.30
  + type_validity_score × 0.20
  + range_check_score  × 0.20
  + uniqueness_score   × 0.15
  + consistency_score  × 0.15
)
```

| Score | Grade | Action |
|-------|-------|--------|
| ≥ 90% | A — Excellent | Proceed to feature engineering |
| 75–89% | B — Acceptable | Address HIGH issues before modeling |
| 60–74% | C — Marginal | Significant cleaning required; model results may be unreliable |
| < 60% | F — Unacceptable | Halt; return dataset to data engineering for remediation |
