# Bias & Fairness Framework

> Definitions, criteria, and remediation strategies for ML model fairness audits.

---

## Why Fairness Audits Are Mandatory

Models trained on historical data can perpetuate, amplify, or introduce discrimination
against protected groups — even when protected attributes are excluded from features
(proxy discrimination via correlated variables). A fairness audit must be completed
before any model is deployed in a high-stakes decision context.

**High-stakes contexts requiring mandatory fairness audit:**
- Lending, credit scoring, or financial services
- Hiring or employee evaluation
- Healthcare diagnosis or triage
- Criminal justice or recidivism
- Housing or rental applications
- Insurance pricing
- Educational admissions or placement

---

## Step 1 — Identify Protected Attributes

Common protected attributes (legal protections vary by jurisdiction):

| Attribute | Common Categories |
|-----------|-----------------|
| Race / ethnicity | Must be identified from context; often a proxy issue |
| Gender / sex | Binary or multi-category |
| Age | Often grouped: < 25, 25–40, 40–60, > 60 |
| Disability status | Binary or multi-category |
| National origin / citizenship | Binary or multi-category |
| Religion | Multi-category |
| Pregnancy / parental status | Binary |
| Socioeconomic status | Proxy: zip code, income bracket, education |

**Proxy discrimination:** Even if protected attributes are excluded, correlated variables
(zip code ↔ race, name ↔ gender) can encode group membership. Audit for proxies too.

---

## Step 2 — Compute Per-Group Metrics

For each protected attribute with 2+ groups, compute:

| Metric | Group A | Group B | Difference |
|--------|---------|---------|------------|
| Selection rate (P(ŷ=1)) | | | |
| Accuracy | | | |
| Precision | | | |
| Recall | | | |
| FPR (False Positive Rate) | | | |
| FNR (False Negative Rate) | | | |
| AUC-ROC | | | |

Require a **minimum group size of 30 instances** to report group-level metrics reliably.
Flag smaller groups but do not report unreliable per-group metrics.

---

## Step 3 — Apply Fairness Criteria

No single fairness criterion is universally correct. Choose based on use case domain.

### Demographic Parity (Statistical Parity)
**Definition:** P(ŷ=1 | group A) = P(ŷ=1 | group B)
**Interpretation:** The model predicts the positive outcome at equal rates across groups.
**80% Rule (Four-Fifths Rule):** Adverse impact exists if selection rate for a group is < 80% of the highest-rate group.
```
Pass: min_group_rate / max_group_rate ≥ 0.80
```
**Use when:** Fair representation in outcomes matters (advertising, content recommendations).
**Limitation:** Does not account for actual qualification rates that may differ across groups.

---

### Equalized Odds
**Definition:** FPR and FNR are equal across groups.
```
P(ŷ=1 | y=0, group A) = P(ŷ=1 | y=0, group B)  [equal FPR]
P(ŷ=0 | y=1, group A) = P(ŷ=0 | y=1, group B)  [equal FNR / equal recall]
```
**Pass threshold:** Absolute difference < 0.10 per metric.
**Use when:** Both false positives and false negatives have group-asymmetric consequences.
**Examples:** Criminal risk scoring (FPR = wrongful prediction; FNR = missed risk), medical screening.

---

### Equal Opportunity (Relaxed Equalized Odds)
**Definition:** Recall (TPR) is equal across groups — only the FNR is constrained.
```
P(ŷ=1 | y=1, group A) = P(ŷ=1 | y=1, group B)
```
**Pass threshold:** Absolute difference in recall < 0.10.
**Use when:** Missing a qualified candidate is the primary harm (hiring, loan approval for creditworthy applicants).

---

### Predictive Parity (Calibration by Group)
**Definition:** Precision is equal across groups.
```
P(y=1 | ŷ=1, group A) = P(y=1 | ŷ=1, group B)
```
**Pass threshold:** Absolute difference in precision < 0.10.
**Use when:** The model score is used directly as a probability (risk scoring, credit scoring).
**Note:** Predictive parity and equalized odds are mutually incompatible when base rates differ (Chouldechova's theorem).

---

### Criterion Selection by Domain

| Domain | Recommended Criterion | Rationale |
|--------|----------------------|-----------|
| Hiring | Equal Opportunity | Missing qualified candidates is the key harm |
| Lending / credit | Equalized Odds or Predictive Parity | Balance FPR and FNR; scores used as probabilities |
| Healthcare screening | Equalized Odds | Missed diagnoses (FNR) and over-treatment (FPR) both matter |
| Criminal justice | Equalized Odds | Both wrongful detention (FPR) and missed risk (FNR) matter |
| Advertising | Demographic Parity | Equal representation in targeting |
| Content recommendation | Demographic Parity | Equal exposure across groups |

---

## Step 4 — Remediation Strategies

When a fairness criterion fails, apply one or more of the following:

### Pre-Processing (Fix the Data)
- **Re-weighting:** Assign higher sample weights to underrepresented groups
- **Resampling:** Oversample minority group; undersample majority group
- **Disparate impact remover:** Transform features to reduce correlation with protected attributes
- **Data augmentation:** Collect more representative data for underrepresented groups

### In-Processing (Fix the Model)
- **Fairness constraints:** Add fairness penalty term to the loss function
- **Adversarial debiasing:** Train a classifier to maximise accuracy while minimising ability to predict protected attribute
- **Fairlearn / AI Fairness 360:** Libraries providing fairness-constrained estimators

### Post-Processing (Fix the Outputs)
- **Threshold adjustment:** Set different classification thresholds per group to equalise FPR/FNR
- **Calibration per group:** Apply Platt scaling or isotonic regression separately per group
- **Reject option classification:** Route borderline predictions (near threshold) for human review

---

## Regulatory Context

| Regulation | Jurisdiction | Key Requirement |
|-----------|-------------|----------------|
| Equal Credit Opportunity Act (ECOA) | USA | Non-discrimination in credit decisions |
| Fair Housing Act | USA | Non-discrimination in housing |
| EEOC Guidelines | USA | Adverse impact ≤ 80% rule in hiring |
| EU AI Act (2024) | EU | High-risk AI systems require fairness assessment |
| GDPR Article 22 | EU | Right to explanation for automated decisions |
| UK Equality Act 2010 | UK | Protected characteristics in automated decisions |

---

## Fairness Audit Verdict

| Verdict | Criteria |
|---------|----------|
| PASS | All applicable criteria met; no group difference > threshold |
| PASS_WITH_NOTES | Minor disparities (< 5% below threshold); document and monitor |
| FAIL | One or more criteria fail threshold; remediation required before deployment |
| BLOCKED | Difference > 20% on any criterion; deployment requires legal review |
