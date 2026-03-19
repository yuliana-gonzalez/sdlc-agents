---
name: ds-reviewing-pipelines
description: >
  Data and ML pipeline audit skill. Reviews pipelines for data leakage, schema
  validation, reproducibility, idempotency, error handling, observability, and
  production readiness. Scores each dimension and produces a pipeline health
  report with prioritised remediation steps.
---

## When to Apply This Skill
- Auditing a pipeline before production deployment
- Diagnosing intermittent pipeline failures or data quality regressions
- Reviewing a pipeline after a schema change breaks downstream models
- Assessing whether a notebook-based pipeline is production-ready
- CI/CD gate before promoting a pipeline to a production environment

---

Read `references/index.md` before executing any step.

---

## Step 1 — Pipeline Map

Before auditing, establish the full pipeline structure:

```
For each stage, document:
1. Stage name and purpose (ingestion | transformation | feature engineering |
   training | inference | evaluation | monitoring)
2. Input: source, format, schema, expected update frequency
3. Output: destination, format, schema
4. Dependencies: upstream stages and external systems
5. Runtime environment: Python version, framework, compute (local / cloud / Spark)
```

Produce a stage map:

| Stage | Purpose | Input | Output | External Dependencies |
|-------|---------|-------|--------|-----------------------|
| | | | | |

---

## Step 2 — Leakage Audit

The highest-severity dimension. Leakage findings are always CRITICAL.

### Target Leakage
- Features computed using the label or any information available only after the target event
- **Test:** Remove suspicious feature → if performance drops to chance, it was leakage

### Temporal Leakage
- Future timesteps visible in aggregation windows or lag calculations
- **Check:** Are all aggregation windows anchored to t-1 (not t or t+1)?
- **Check:** Is the train/test split time-ordered (not random shuffle)?

### Preprocessing Leakage
- Scalers, encoders, imputers, or normalisation statistics fit on the full dataset
- **Check:** Every `.fit()` call must be on training data only
- **Check:** `train_test_split()` called BEFORE any preprocessing step

### Join/Aggregation Leakage
- Joining training data to a table that contains future information
- **Check:** All join keys filter by event timestamp or training window

### Sampling Leakage
- The same entity (user, session, household) appears in both train and test
- **Check:** Split is stratified by entity ID, not by row

---

## Step 3 — Reproducibility Review

A pipeline is reproducible if re-running it on the same input produces identical output.

| Check | Pass Criteria | Status |
|-------|--------------|--------|
| Random seeds | All random operations seeded: `np.random.seed()`, `random.seed()`, `tf.random.set_seed()`, `torch.manual_seed()` | PASS / FAIL |
| Train/test split seed | `train_test_split(random_state=N)` or equivalent | PASS / FAIL |
| Dependency pinning | `requirements.txt` or `pyproject.toml` with exact versions | PASS / FAIL |
| Data versioning | Input data is versioned, checksummed, or identified by timestamp | PASS / FAIL |
| Deterministic ops | `torch.use_deterministic_algorithms(True)` or GPU non-determinism documented | PASS / FAIL |
| Container / environment | Docker image or conda env spec is committed alongside pipeline code | PASS / FAIL |

---

## Step 4 — Schema & Data Validation

Pipelines must validate data at every stage boundary, not just at ingestion.

| Check | Pass Criteria | Status |
|-------|--------------|--------|
| Input schema validation | Column names, types, and presence validated before processing begins | PASS / FAIL |
| Null handling | Explicit strategy per column (not silent drop or fill-with-zero) | PASS / FAIL |
| Type casting | Defensive casting with error on unexpected values (not silent coercion) | PASS / FAIL |
| Range constraints | Critical columns validated against expected ranges | PASS / FAIL |
| Cardinality checks | Categorical columns validated against expected value sets | PASS / FAIL |
| Schema drift handling | Pipeline fails loudly on unexpected columns or type changes (not silent continue) | PASS / FAIL |
| Output schema validation | Output schema validated after each transformation stage | PASS / FAIL |

**Recommended libraries:** `pandera`, `great_expectations`, `pydantic`, `cerberus`

---

## Step 5 — Error Handling & Observability

| Check | Pass Criteria | Status |
|-------|--------------|--------|
| Explicit exception handling | Failures caught and raise with context, not swallowed silently | PASS / FAIL |
| Failure logging | All exceptions logged with stage name, timestamp, input summary | PASS / FAIL |
| Alerting on failure | Pipeline failure triggers alert (PagerDuty, Slack, email) | PASS / WARN / FAIL |
| Data statistics logged | Row count, null %, key metric distributions logged per stage | PASS / FAIL |
| Data drift monitoring | Input feature distributions compared to training baseline | PASS / WARN / FAIL |
| Model performance monitoring | Prediction distributions, score histograms logged at inference | PASS / WARN / FAIL |

---

## Step 6 — Production Readiness

| Check | Pass Criteria | Status |
|-------|--------------|--------|
| Idempotency | Re-running produces same result without side effects or duplicate writes | PASS / FAIL |
| Secrets management | No hardcoded credentials, API keys, or connection strings in code | PASS / FAIL |
| Configuration externalised | Environment-specific config in env vars or config files (not hardcoded) | PASS / FAIL |
| Scalability assessment | Pipeline estimated to handle 10× current volume; bottlenecks documented | PASS / WARN / FAIL |
| Rollback plan | Previous model version can be restored within 15 minutes | PASS / WARN / FAIL |
| Documentation | Pipeline purpose, inputs, outputs, and dependencies documented | PASS / WARN / FAIL |

---

## Step 7 — Health Score Computation

Score each dimension as PASS (1.0), WARN (0.5), or FAIL (0.0). Compute weighted health score:

| Dimension | Weight | Score (1.0 / 0.5 / 0.0) | Weighted |
|-----------|--------|--------------------------|----------|
| Leakage-Free | 25% | | |
| Reproducibility | 20% | | |
| Schema Validation | 15% | | |
| Error Handling | 15% | | |
| Idempotency | 10% | | |
| Observability | 10% | | |
| Secrets Safety | 5% | | |
| **Total** | 100% | | **__%** |

**Overall verdict:**
- ≥ 80% and zero CRITICAL → **PASS**
- 60–79% or any HIGH → **WARN**
- < 60% or any CRITICAL → **FAIL**

---

## Step 8 — Report Assembly

Use `assets/pipeline-review-template.md` as structure. Include:
1. Pipeline map (stage table)
2. Dimension scores and overall health score
3. All findings with severity: CRITICAL / HIGH / MEDIUM / LOW
4. Specific remediation for each CRITICAL and HIGH finding
5. Final verdict

---

## Quality Checklist

Before emitting the pipeline review report:
- [ ] Full pipeline stage map documented before auditing
- [ ] Leakage audit completed first and covers all four leakage types
- [ ] Reproducibility checked for every random operation in the pipeline
- [ ] Schema validation presence confirmed at each stage boundary
- [ ] Health score computed from weighted dimension scores
- [ ] Every CRITICAL and HIGH finding has a specific, actionable remediation step
- [ ] Overall verdict matches health score and CRITICAL/HIGH count
- [ ] Report uses the pipeline review template structure
