---
name: ds-evaluating-models
description: >
  Model evaluation framework skill. Selects appropriate metrics per problem type,
  validates the evaluation strategy for leakage and correctness, diagnoses
  overfitting and underfitting, and runs a structured bias and fairness audit.
  Produces a model evaluation report with a final verdict.
---

## When to Apply This Skill
- Evaluating a trained model before deployment
- Comparing multiple model candidates to select the best
- Diagnosing why a model performs well in training but poorly in production
- Auditing a model for bias, fairness, or protected attribute disparities
- Validating that the evaluation methodology itself is sound

---

Read `references/index.md` before executing any step.

---

## Step 1 — Problem & Model Context

Establish context before selecting any metric:

```
1. Problem type: binary classification | multi-class | regression |
                 ranking | time-series forecasting | clustering | anomaly detection
2. Model type and algorithm family
3. Business objective: what does a correct prediction enable? What does an error cost?
4. Evaluation dataset type: hold-out | k-fold | stratified k-fold | time-series split
5. Protected attributes (fairness-sensitive columns): list all identified
6. Class distribution (classification): confirm from EDA report if available
```

---

## Step 2 — Metric Selection

**Principle:** Metrics must be selected before inspecting results. Choosing metrics after
seeing results is a form of evaluation bias (p-hacking equivalent).

Use `references/metrics-by-problem-type.md` for the full decision table.

### Classification — Key Decision Points

| Scenario | Avoid | Use Instead |
|----------|-------|-------------|
| Imbalanced classes (> 5:1) | Accuracy | AUC-PR, F1, MCC |
| High cost of false negatives (cancer, fraud) | Precision | Recall, Sensitivity |
| High cost of false positives (spam filter) | Recall | Precision, Specificity |
| Multi-class with unequal class sizes | Micro-avg | Macro-avg F1 |
| Calibrated probability needed (risk scoring) | Threshold metrics | Brier Score, Log Loss |

### Regression — Key Decision Points

| Scenario | Avoid | Use Instead |
|----------|-------|-------------|
| Large outliers in target | RMSE (sensitive to outliers) | MAE or MAPE |
| Relative error more meaningful than absolute | RMSE | MAPE, SMAPE |
| Comparing across different scales | RMSE | R², MAPE |
| Heavy-tailed target distribution | Plain RMSE | Huber loss or quantile metrics |

---

## Step 3 — Validation Strategy Review

A sound metric on a flawed evaluation is meaningless. Verify the strategy first:

### Hold-Out (Train / Validation / Test Split)
```
PASS criteria:
- Split is stratified (classification) or preserves time ordering (time-series)
- Test set is used only once — not for hyperparameter tuning
- Validation set size is sufficient (≥ 10% of data; ≥ 500 rows for classification)
- No duplicate rows across splits
- No entity leakage (same user/entity in both train and test)
```

### K-Fold Cross-Validation
```
PASS criteria:
- Folds are stratified for classification tasks (sklearn StratifiedKFold)
- k ≥ 5 (5 or 10 recommended)
- Preprocessing fitted within each fold (no leakage from global fit)
- Final reported metric is mean ± std across folds
```

### Time-Series Validation
```
PASS criteria:
- Train period strictly precedes validation period (no future leakage)
- Walk-forward (expanding window) or rolling-window validation used
- No random shuffle before split
- Gap period between train and test if target has autocorrelation
```

---

## Step 4 — Overfitting & Underfitting Diagnosis

Compare train vs. validation metric:

| Pattern | Diagnosis | Recommended Actions |
|---------|-----------|-------------------|
| Train high, val low (large gap > 10%) | Overfitting | Regularise, reduce model complexity, add data, drop correlated features |
| Train low, val low (both poor) | Underfitting | Add features, increase model complexity, reduce regularisation |
| Train high, val very high (val > train) | Likely leakage | Audit validation strategy immediately |
| Train ≈ val (small gap) | Good fit | Proceed to fairness audit |

### Learning Curve Interpretation (if provided)
- High train error, high val error → underfitting; need more features or complexity
- Low train error, high val error → overfitting; need more data or regularisation
- Both converge at a low error level → good fit

### Variance Across Folds (k-fold)
- Std dev of fold metric > 5% of mean → high variance; model is unstable
- Recommendation: increase k, add more data, or reduce model complexity

---

## Step 5 — Bias & Fairness Audit

Refer to `references/bias-fairness.md` for criterion definitions.

### Identify Protected Attributes
Common examples: gender, race/ethnicity, age group, nationality, disability status, income bracket, geography.

### Compute Per-Group Metrics
For each protected attribute with ≥ 2 groups:

| Group | Accuracy | Precision | Recall | FPR | FNR |
|-------|----------|-----------|--------|-----|-----|
| Group A | | | | | |
| Group B | | | | | |
| Overall | | | | | |

### Apply Fairness Criteria

| Criterion | Definition | Threshold | Verdict |
|-----------|-----------|-----------|---------|
| Demographic Parity | P(ŷ=1 \| group A) ≈ P(ŷ=1 \| group B) | Ratio within [0.8, 1.25] (80% rule) | PASS / FAIL |
| Equalized Odds | FPR and FNR equal across groups | Absolute difference < 0.10 | PASS / FAIL |
| Predictive Parity | Precision equal across groups | Absolute difference < 0.10 | PASS / FAIL |
| Calibration | P(y=1 \| ŷ=p) equal across groups | Difference in Brier score < 0.05 | PASS / FAIL |

**Criterion selection guidance:**
- Lending / hiring / criminal justice → Equalized Odds (false negatives and FPR matter equally)
- Advertising / recommendations → Demographic Parity
- Medical diagnosis → Equalized Odds or Calibration (high stakes for FNR)

---

## Step 6 — Report Assembly

Use `assets/model-eval-report-template.md` as structure.

Include:
1. Problem type and business objective
2. Selected metrics with rationale
3. Validation strategy assessment (PASS / WARN / FAIL per criterion)
4. Performance table: train, validation, test metrics
5. Overfitting/underfitting diagnosis
6. Fairness audit per protected attribute
7. Final verdict: APPROVED / APPROVED_WITH_NOTES / REQUIRES_REVISION / BLOCKED
8. Recommended next steps

---

## Quality Checklist

Before emitting the model evaluation report:
- [ ] Metrics selected before inspecting results (and documented as such)
- [ ] Business objective alignment stated explicitly — not just ML metric targets
- [ ] Validation strategy assessed for leakage (train/test contamination, entity leakage)
- [ ] Train vs. validation gap reported and interpreted
- [ ] Fairness audit covers all identified protected attributes
- [ ] Fairness criteria chosen based on use case, not convenience
- [ ] Final verdict is one of four defined values
- [ ] Report uses the model evaluation template structure
