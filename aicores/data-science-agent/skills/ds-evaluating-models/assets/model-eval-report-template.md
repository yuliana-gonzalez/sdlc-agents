# Model Evaluation Report — [Model Name / Experiment ID]

**Date:** [YYYY-MM-DD]
**Agent:** ds-model-eval-agent
**Problem Type:** [binary classification | multi-class | regression | ranking | forecasting | clustering | anomaly detection]
**Algorithm:** [e.g., XGBoost, LightGBM, Logistic Regression, LSTM]
**Dataset:** [name / version / date range]
**Feature Engineering Version:** [link to feature_catalog.md if available]

---

## 1. Business Objective Alignment

**Business objective:** [What real-world outcome does a correct prediction enable?]
**Cost of false positive:** [e.g., unnecessary outreach, wasted budget]
**Cost of false negative:** [e.g., missed fraud, undetected disease]
**Decision:** [How will model output be used — binary decision, ranking, probability score?]

---

## 2. Selected Metrics

Metrics selected **before** inspecting results:

| Metric | Type | Rationale |
|--------|------|-----------|
| [Primary] | | Why this metric for this problem |
| [Secondary] | | |
| [Calibration — if applicable] | Brier Score / Log Loss | |

---

## 3. Validation Strategy Assessment

| Criterion | Method Used | Status | Notes |
|-----------|------------|--------|-------|
| Split type | hold-out / k-fold / time-series | PASS / WARN / FAIL | |
| Stratification | yes / no | PASS / FAIL | |
| No future leakage | confirmed / not verified | PASS / FAIL | |
| Test set untouched | yes / no | PASS / FAIL | |
| Entity isolation | yes / no / N/A | PASS / FAIL | |
| Preprocessing leakage | no leakage / leakage detected | PASS / FAIL | |

**Validation Strategy Overall:** [PASS / WARN / FAIL]

---

## 4. Performance Results

### Summary Table

| Split | [Primary Metric] | [Secondary Metric] | [Other] |
|-------|-----------------|-------------------|---------|
| Train | | | |
| Validation | | | |
| Test | | | |

### Train vs. Validation Gap

**Gap:** [Metric value difference]
**Diagnosis:** [Good fit / Overfitting / Underfitting / Possible leakage]
**Recommended action:** [none / regularise / add data / investigate leakage]

### Cross-Validation Results (if applicable)

| Fold | [Primary Metric] |
|------|-----------------|
| 1 | |
| 2 | |
| 3 | |
| 4 | |
| 5 | |
| **Mean ± Std** | **__ ± __** |

---

## 5. Fairness Audit

**Protected attributes evaluated:** [list all]

### Per-Group Metrics

**Attribute:** [e.g., Gender]

| Group | Selection Rate | Accuracy | Precision | Recall | FPR | FNR |
|-------|--------------|----------|-----------|--------|-----|-----|
| | | | | | | |
| | | | | | | |
| Overall | | | | | | |

### Fairness Criteria Results

| Criterion | Threshold | Result | Verdict |
|-----------|----------|--------|---------|
| Demographic Parity | Ratio ≥ 0.80 | | PASS / FAIL |
| Equalized Odds (FPR) | Diff < 0.10 | | PASS / FAIL |
| Equalized Odds (FNR) | Diff < 0.10 | | PASS / FAIL |
| Predictive Parity | Diff < 0.10 | | PASS / FAIL |

**Fairness Overall:** [PASS / PASS_WITH_NOTES / FAIL / BLOCKED]

---

## 6. Model Comparison (Multi-Model Evaluation)

*(Complete only when comparing multiple candidates)*

| Model | [Primary Metric] | [Secondary] | Training Time | Inference Time | Fairness | Verdict |
|-------|-----------------|-------------|--------------|---------------|----------|---------|
| | | | | | | |
| | | | | | | |

**Recommended model:** [name] — [one-line rationale]

---

## 7. Final Verdict

**Verdict:** [APPROVED / APPROVED_WITH_NOTES / REQUIRES_REVISION / BLOCKED]

| Dimension | Status | Notes |
|-----------|--------|-------|
| Performance vs. target | PASS / FAIL | Target: [value] — Achieved: [value] |
| Validation strategy | PASS / WARN / FAIL | |
| Overfitting | PASS / WARN / FAIL | |
| Fairness | PASS / FAIL | |

---

## 8. Required Actions

*(Complete for REQUIRES_REVISION or BLOCKED verdicts)*

| Priority | Issue | Severity | Recommended Fix |
|----------|-------|----------|----------------|
| 1 | | CRITICAL / HIGH | |
| 2 | | MEDIUM | |

---

## 9. Notes

<!-- Domain-specific observations, comparison to baseline, or questions for the data scientist -->
