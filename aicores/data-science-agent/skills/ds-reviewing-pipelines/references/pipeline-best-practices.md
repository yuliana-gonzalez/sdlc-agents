# Pipeline Best Practices

> Production-readiness standards for data and ML pipelines.

---

## 1. Reproducibility

A pipeline is reproducible if re-running it on the same input always produces identical output.

### Seed Management
Set all random seeds explicitly and early:
```python
import random, numpy as np, torch, tensorflow as tf

SEED = 42
random.seed(SEED)
np.random.seed(SEED)
torch.manual_seed(SEED)
tf.random.set_seed(SEED)

# For GPU non-determinism in PyTorch:
torch.use_deterministic_algorithms(True)
torch.backends.cudnn.benchmark = False
```

**Rule:** Every random operation must have a fixed seed. "I'll add it later" = not reproducible.

### Dependency Pinning
Pin exact versions — not ranges — in production pipelines:

```
# requirements.txt — acceptable
scikit-learn==1.4.2
pandas==2.2.1
lightgbm==4.3.0

# pyproject.toml — preferred
[tool.poetry.dependencies]
scikit-learn = "1.4.2"
pandas = "2.2.1"
```

**Rule:** `>=` and `^` version specifiers are acceptable for development, not for production.

### Data Versioning
- Use checksums (MD5/SHA256) to verify input data integrity before processing
- Store training data version alongside model artifacts
- Recommended tools: DVC, Delta Lake, LakeFS, or simply checksum + timestamp in run metadata

---

## 2. Schema Validation

Validate schema at every stage boundary — not just at ingestion.

### pandera (Python — type-safe, declarative)
```python
import pandera as pa

schema = pa.DataFrameSchema({
    "age":    pa.Column(int, pa.Check.between(0, 120), nullable=False),
    "income": pa.Column(float, pa.Check.ge(0), nullable=True),
    "gender": pa.Column(str, pa.Check.isin(["M", "F", "Other"])),
})

validated_df = schema.validate(df)  # raises SchemaError on violation
```

### great_expectations (Enterprise-grade, CI/CD integrated)
```python
# Define expectations
expect_column_values_to_not_be_null("customer_id")
expect_column_values_to_be_between("age", min_value=0, max_value=120)
expect_column_values_to_be_in_set("status", ["active", "inactive", "churned"])
```

### Fail loudly on schema drift
```python
expected_columns = {"customer_id", "age", "income", "gender", "churn"}
actual_columns = set(df.columns)
unexpected = actual_columns - expected_columns
missing = expected_columns - actual_columns
assert not unexpected, f"Unexpected columns: {unexpected}"
assert not missing, f"Missing columns: {missing}"
```

---

## 3. Error Handling

### Anti-patterns (never do these)
```python
# WRONG — swallows all errors silently
try:
    result = process(data)
except:
    pass

# WRONG — catches too broadly, loses context
try:
    result = transform(df)
except Exception:
    result = None
```

### Correct patterns
```python
# CORRECT — specific exception, log with context, re-raise
import logging
logger = logging.getLogger(__name__)

try:
    result = transform(df)
except ValueError as e:
    logger.error("Transform failed at stage=feature_engineering: %s", e,
                 extra={"input_shape": df.shape, "stage": "feature_engineering"})
    raise
```

### Pipeline-level error contract
Every stage must:
1. Validate its inputs before processing
2. Log start, end, row counts, and any anomalies at INFO level
3. Log all exceptions at ERROR level with stage context
4. Raise (not swallow) exceptions to allow upstream failure detection

---

## 4. Idempotency

An idempotent pipeline produces the same result when run multiple times and does not
create side effects on re-run (no duplicate rows, no double-charged accounts).

### Patterns for idempotency
```python
# Pattern 1: Delete-then-insert (overwrite)
# In SQL
DELETE FROM feature_table WHERE run_date = '{run_date}';
INSERT INTO feature_table SELECT * FROM staging WHERE run_date = '{run_date}';

# Pattern 2: Upsert (merge)
# In pandas
df_existing = load_existing()
df_new = compute_features()
df_result = df_existing[~df_existing['id'].isin(df_new['id'])]
df_result = pd.concat([df_result, df_new])
save(df_result)

# Pattern 3: Write to versioned partition
df.to_parquet(f"s3://bucket/features/run_date={run_date}/features.parquet")
```

### Test for idempotency
```python
result_1 = pipeline.run(input_data)
result_2 = pipeline.run(input_data)
assert result_1.equals(result_2), "Pipeline is not idempotent"
```

---

## 5. Secrets Management

### Forbidden patterns
```python
# WRONG — hardcoded credentials
DB_PASSWORD = "my_super_secret_password"
API_KEY = "sk-abc123..."
conn = psycopg2.connect(host="prod-db.company.com", password="my_super_secret_password")
```

### Correct patterns
```python
# CORRECT — environment variables
import os
DB_PASSWORD = os.environ["DB_PASSWORD"]
API_KEY = os.environ["API_KEY"]

# CORRECT — secrets manager (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager)
import boto3
client = boto3.client('secretsmanager')
secret = client.get_secret_value(SecretId='prod/ml-pipeline/db-credentials')
```

**Checklist:**
- [ ] No credentials in source code files
- [ ] No credentials in Jupyter notebook outputs
- [ ] `.env` files are in `.gitignore`
- [ ] Secrets are injected via environment variables or a secrets manager
- [ ] Connection strings do not contain passwords in plain text

---

## 6. Observability

A production ML pipeline must log enough information to diagnose data drift, quality
regressions, and performance degradation without re-running the pipeline.

### Minimum logging per stage
```python
logger.info("Stage=ingestion started", extra={"run_id": run_id})
logger.info("Stage=ingestion completed", extra={
    "run_id": run_id,
    "rows_ingested": len(df),
    "null_pct": df.isnull().mean().to_dict(),
    "date_range": [df['date'].min(), df['date'].max()],
})
```

### Data drift monitoring
After each ingestion or preprocessing stage, compute and log:
- Feature mean / std for numeric columns
- Category distribution for categorical columns
- Compare to training baseline (stored at training time)
- Alert if any feature drifts by > 2 standard deviations from baseline

### Model performance monitoring (inference pipelines)
- Log prediction score distributions (mean, std, quantiles) per batch
- Track label distribution if ground truth is available with delay
- Alert on score distribution shift (PSI — Population Stability Index > 0.2)

---

## 7. Pipeline Framework Selection

| Use Case | Recommended Framework | Notes |
|----------|----------------------|-------|
| Simple batch pipelines (< 1M rows) | Python scripts + cron | Keep it simple |
| Complex DAG dependencies | Apache Airflow | De facto standard for data engineering |
| ML-specific orchestration | Prefect, Metaflow | Better ML-native features than Airflow |
| Real-time / streaming | Apache Kafka + Flink or Spark Streaming | High operational complexity |
| Feature stores | Feast, Tecton, Hopsworks | Only for teams with > 5 ML models in production |
| Experiment tracking | MLflow, Weights & Biases | Mandatory for reproducibility at scale |
| Data versioning | DVC, Delta Lake | Recommended when training data changes frequently |
