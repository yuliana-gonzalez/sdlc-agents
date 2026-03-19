# Pipeline Review Report — [Pipeline Name]

**Date:** [YYYY-MM-DD]
**Agent:** ds-pipeline-review-agent
**Pipeline Type:** [Airflow DAG | Prefect flow | Python script | Jupyter notebook | SQL | dbt]
**Pipeline Purpose:** [ingestion | feature engineering | training | inference | evaluation | monitoring]
**Environment:** [local | cloud | Spark | containerised]
**Reviewer:** ds-pipeline-review-agent

---

## 1. Pipeline Stage Map

| Stage | Purpose | Input | Output | External Dependencies |
|-------|---------|-------|--------|-----------------------|
| | | | | |

---

## 2. Health Score

| Dimension | Weight | Score | Weighted Score |
|-----------|--------|-------|---------------|
| Leakage-Free | 25% | PASS (1.0) / WARN (0.5) / FAIL (0.0) | |
| Reproducibility | 20% | | |
| Schema Validation | 15% | | |
| Error Handling | 15% | | |
| Idempotency | 10% | | |
| Observability | 10% | | |
| Secrets Safety | 5% | | |
| **Overall Health Score** | 100% | | **__%** |

**Verdict:** [PASS (≥ 80%, zero CRITICAL) / WARN (60–79% or any HIGH) / FAIL (< 60% or any CRITICAL)]

---

## 3. Findings

All issues found during the audit:

| # | Dimension | Severity | Location | Issue | Remediation |
|---|-----------|----------|----------|-------|-------------|
| 1 | | CRITICAL / HIGH / MEDIUM / LOW | [file:line or stage name] | | |
| 2 | | | | | |

**Summary:**
- CRITICAL issues: ___
- HIGH issues: ___
- MEDIUM issues: ___
- LOW issues: ___

---

## 4. Leakage Audit Detail

**Overall leakage status:** [CLEAN / LEAKAGE DETECTED]

| # | Leakage Type | Location | Description | Evidence | Fix |
|---|-------------|----------|-------------|---------|-----|
| | Target / Temporal / Preprocessing / Join / Entity | | | | |

---

## 5. Reproducibility Detail

| Check | Status | Notes |
|-------|--------|-------|
| Random seeds set | PASS / FAIL | |
| Train/test split seeded | PASS / FAIL | |
| Dependencies pinned | PASS / FAIL | |
| Data versioned / checksummed | PASS / FAIL | |
| Environment spec committed | PASS / FAIL | |

---

## 6. Schema Validation Detail

| Stage | Input Validated | Output Validated | Null Strategy Explicit | Drift Handling |
|-------|----------------|-----------------|----------------------|----------------|
| | PASS / FAIL | PASS / FAIL | PASS / FAIL | PASS / WARN / FAIL |

---

## 7. Remediation Plan

*(For WARN and FAIL verdicts — ordered by priority)*

### CRITICAL Issues (Must fix before any deployment)

| # | Issue | Location | Fix | Owner |
|---|-------|----------|-----|-------|
| | | | | |

### HIGH Issues (Must fix before production promotion)

| # | Issue | Location | Fix | Owner |
|---|-------|----------|-----|-------|
| | | | | |

### MEDIUM Issues (Fix in next sprint)

| # | Issue | Location | Fix |
|---|-------|----------|-----|
| | | | |

---

## 8. Notes

<!-- Architecture-specific observations, follow-up questions, or context for the pipeline owner -->
