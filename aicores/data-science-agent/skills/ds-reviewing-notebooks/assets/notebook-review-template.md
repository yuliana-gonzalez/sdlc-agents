# Notebook Review Report — [Notebook Name]

**Date:** [YYYY-MM-DD]
**Agent:** ds-pipeline-review-agent (notebook mode)
**Notebook:** [filename.ipynb]
**Notebook Purpose:** [as described in header cell, or inferred]
**Cell Count:** [total cells] ([code cells] code, [markdown cells] markdown)
**Last Execution Date:** [from cell metadata if available]

---

## 1. Health Score

| Dimension | Weight | Score | Weighted Score |
|-----------|--------|-------|---------------|
| Execution Order / No Hidden State | 25% | PASS (1.0) / WARN (0.5) / FAIL (0.0) | |
| No Hardcoded Secrets | 20% | | |
| Reproducibility (seeds + deps) | 20% | | |
| Documentation Quality | 15% | | |
| No Sensitive Outputs | 15% | | |
| Output Cleanliness | 5% | | |
| **Overall Health Score** | 100% | | **__%** |

**Verdict:** [APPROVED (≥ 80%, zero CRITICAL) / APPROVED_WITH_NOTES (60–79% or any HIGH) / REQUIRES_REVISION (< 60% or any CRITICAL)]

---

## 2. Execution Order Assessment

**Restart → Run All result:** [PASS — runs cleanly / FAIL — error on cell N / UNKNOWN — not tested]

| Check | Status | Notes |
|-------|--------|-------|
| Cell execution numbers sequential | PASS / FAIL | |
| No gaps in execution order | PASS / FAIL | |
| No variable shadowing detected | PASS / FAIL | |
| All imports in a single cell near top | PASS / WARN / FAIL | |

---

## 3. Hardcoded Values

| Type | Cell | Value (masked) | Severity | Fix |
|------|------|---------------|----------|-----|
| Absolute path | Cell N | `/Users/***` | HIGH | Use pathlib relative path |
| Hardcoded secret | | `***` | CRITICAL | Use os.environ |
| Magic number | | `0.85` | LOW | Add comment explaining origin |

---

## 4. Reproducibility

| Check | Status | Notes |
|-------|--------|-------|
| SEED constant defined | PASS / FAIL | |
| `np.random.seed(SEED)` called | PASS / FAIL | |
| `random.seed(SEED)` called | PASS / FAIL | |
| `train_test_split(random_state=SEED)` | PASS / FAIL | Cell N |
| Model initialisation with `random_state` | PASS / FAIL | |
| Dependencies pinned | PASS / FAIL | |
| Data source documented | PASS / FAIL | |

---

## 5. Documentation Quality

| Check | Status | Notes |
|-------|--------|-------|
| Header cell (title, author, date, purpose) | PASS / FAIL | |
| Configuration cell with all constants | PASS / FAIL | |
| Major sections have Markdown headers | PASS / FAIL | |
| Complex cells preceded by explanation | PASS / WARN / FAIL | |
| Results cells followed by interpretation | PASS / WARN / FAIL | |
| Conclusions / next steps section | PASS / FAIL | |

---

## 6. Output Issues

| Cell | Issue | Severity | Fix |
|------|-------|----------|-----|
| Cell N | PII in output | CRITICAL | Clear output; add `.head(5)` |
| Cell N | Full dataframe (1000 rows) | MEDIUM | Use `.head(10)` |
| Cell N | Stale output (code changed) | HIGH | Restart → Run All |

---

## 7. All Findings

| # | Dimension | Severity | Cell | Issue | Recommended Fix |
|---|-----------|----------|------|-------|----------------|
| 1 | | CRITICAL / HIGH / MEDIUM / LOW | Cell N | | |
| 2 | | | | | |

**Summary:**
- CRITICAL: ___
- HIGH: ___
- MEDIUM: ___
- LOW: ___

---

## 8. Remediation Steps

*(For APPROVED_WITH_NOTES and REQUIRES_REVISION — ordered by priority)*

1. [CRITICAL] Cell N: [specific action]
2. [HIGH] Cell N: [specific action]
3. [MEDIUM] [specific action]

---

## 9. Notes

<!-- Domain observations, context about the notebook's intended audience, or follow-up questions -->
