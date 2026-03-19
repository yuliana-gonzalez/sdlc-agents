# Data Science Agent — AI Core

> **AI Core** | Data Science & ML Engineering | From raw data to validated, production-ready models and pipelines.

The **Data Science Agent** accelerates every phase of the data science lifecycle: exploratory data analysis, feature engineering, model evaluation, pipeline review, and notebook quality audits. It enforces best practices from the ground up — reproducibility, statistical rigor, bias awareness, and production readiness.

---

## Architecture Overview

```
data-science-agent/
├── agents/
│   ├── ds-eda-agent.md               ← Exploratory Data Analysis: profiling, distributions, correlations
│   ├── ds-feature-eng-agent.md       ← Feature Engineering: encoding, scaling, selection, leakage detection
│   ├── ds-model-eval-agent.md        ← Model Evaluation: metrics, validation strategy, bias & fairness
│   └── ds-pipeline-review-agent.md   ← Data Pipeline Review: quality, leakage, reproducibility
└── skills/
    ├── ds-conducting-eda/             ← Atomic: data profiling, distributions, correlations, anomalies
    ├── ds-feature-engineering/        ← Atomic: encoding, scaling, feature selection, leakage checks
    ├── ds-evaluating-models/          ← Atomic: metrics selection, validation strategy, bias detection
    ├── ds-reviewing-pipelines/        ← Atomic: pipeline audit, leakage, reproducibility, schema drift
    └── ds-reviewing-notebooks/        ← Atomic: notebook quality, reproducibility, documentation standards
```

---

## Core Principles (Enforced Across All Agents and Skills)

1. **Reproducibility first** — every analysis must be reproducible with fixed seeds and pinned dependencies
2. **Data before models** — understand the data deeply before selecting or tuning models
3. **Leakage vigilance** — train/test contamination is treated as a critical defect, not a warning
4. **Metric-first evaluation** — choose evaluation metrics before training, never after seeing results
5. **Bias awareness** — all models are evaluated for demographic and distributional bias
6. **Statistical honesty** — report confidence intervals and uncertainty; avoid p-hacking
7. **Schema-driven pipelines** — pipelines validate schemas at every stage, not just at ingestion
8. **Notebook discipline** — notebooks are documentation artifacts; cells must run top-to-bottom cleanly

---

## Agents

### `ds-eda-agent`

**Description:** Guides a structured exploratory data analysis session. Produces a reproducible EDA report covering data quality, distributions, correlations, outliers, and visualisation recommendations.

**Triggers:** providing a dataset, CSV, dataframe, or database table and asking for exploration, profiling, or analysis

**Skills loaded:** `ds-conducting-eda`

**Examples:**
```
"Run an EDA on this customer churn dataset"
"Profile this CSV and flag data quality issues"
"Explore correlations between features in this dataframe"
"I have a new dataset — what should I know before modeling?"
```

---

### `ds-feature-eng-agent`

**Description:** Reviews or designs a feature engineering strategy. Detects data leakage, recommends encoding and scaling approaches, and produces a feature catalog with transformation rationale.

**Triggers:** feature selection, encoding choices, scaling decisions, leakage suspicion, or pre-modeling preparation

**Skills loaded:** `ds-feature-engineering`

**Examples:**
```
"Review my feature engineering pipeline for leakage"
"What encoding should I use for these categorical columns?"
"Select the most predictive features from this set of 80 candidates"
"My model is overfitting — help me simplify the feature set"
```

---

### `ds-model-eval-agent`

**Description:** Evaluates a trained model or compares multiple models. Selects appropriate metrics for the problem type, validates the evaluation strategy, detects overfitting, and checks for bias and fairness issues.

**Triggers:** model evaluation, metrics discussion, comparing models, fairness audit, overfitting/underfitting diagnosis

**Skills loaded:** `ds-evaluating-models`

**Examples:**
```
"Evaluate this classification model — is accuracy the right metric here?"
"Compare these three models and tell me which to ship"
"My model has 95% accuracy but performs poorly in production — why?"
"Run a fairness audit on this loan approval model"
```

---

### `ds-pipeline-review-agent`

**Description:** Audits a data pipeline or ML pipeline for correctness, reproducibility, leakage, schema drift, and production readiness. Produces a scored pipeline health report.

**Triggers:** reviewing a pipeline before production deployment, suspecting data issues, diagnosing pipeline failures, pre-release pipeline audit

**Skills loaded:** `ds-reviewing-pipelines`, `ds-reviewing-notebooks`

**Examples:**
```
"Review this Airflow pipeline before we deploy to production"
"Audit our feature pipeline for train/test leakage"
"Our pipeline breaks on schema changes — review it"
"This Jupyter notebook is our pipeline — review it for reproducibility"
```

---

## Skills

### `ds-conducting-eda` *(Atomic)*

**Description:** Structured exploratory data analysis. Covers data shape, type inference, missing value analysis, distribution profiling, correlation detection, outlier identification, and class imbalance detection.

**Output:** `eda_report.md` + visualisation recommendations

---

### `ds-feature-engineering` *(Atomic)*

**Description:** Feature engineering strategy and review. Covers encoding (ordinal, one-hot, target, embeddings), scaling (standard, min-max, robust), feature selection (filter, wrapper, embedded methods), and data leakage detection.

**Output:** `feature_catalog.md` + `leakage_report.md`

---

### `ds-evaluating-models` *(Atomic)*

**Description:** Model evaluation framework. Selects metrics by problem type, validates the evaluation strategy (hold-out, k-fold, time-series split), detects overfitting/underfitting, and runs a bias/fairness check.

**Output:** `model_eval_report.md`

---

### `ds-reviewing-pipelines` *(Atomic)*

**Description:** Data and ML pipeline audit. Checks for schema validation, data leakage, reproducibility (seeds, versioning), idempotency, error handling, and production readiness.

**Output:** `pipeline_review_report.md`

---

### `ds-reviewing-notebooks` *(Atomic)*

**Description:** Jupyter/Colab notebook quality review. Checks cell execution order, hardcoded paths/credentials, reproducibility (seeds, pinned dependencies), documentation quality, and output cleanliness.

**Output:** `notebook_review_report.md`

---

## Supported Inputs

- Datasets: CSV, Parquet, JSON, SQL query results, Pandas/Polars dataframes
- Model artifacts: sklearn pipelines, model cards, evaluation reports (JSON, YAML, Markdown)
- Pipelines: Airflow DAGs, Prefect flows, Python scripts, Jupyter notebooks
- Feature stores: feature catalogs, transformation configs
- Requirements: problem statements, KPI definitions, fairness constraints

## Supported Outputs

- EDA reports (Markdown)
- Feature catalogs with transformation rationale
- Model evaluation reports with metric tables and bias summaries
- Pipeline health reports with scored dimensions
- Notebook quality reviews with actionable issue lists

---

## KPIs This AI Core Drives

| Metric | Description |
|--------|-------------|
| Data Quality Score | % of columns passing null, type, and range checks |
| Leakage Detection Rate | Pipeline leakage issues caught before production |
| Model Fairness Coverage | % of protected attributes evaluated for bias |
| Notebook Reproducibility Rate | % of notebooks that run top-to-bottom cleanly |
| Feature Engineering Efficiency | Features selected vs. total candidates (signal-to-noise ratio) |

---

## Installation

### Install the full AI Core (all agents)
```bash
npx aicores add https://github.com/wizeline/sdlc-agents/tree/main/aicores/data-science-agent
```

### Install a single agent
```bash
npx subagents add https://github.com/wizeline/sdlc-agents/tree/main/aicores/data-science-agent/agents/ds-eda-agent
```

### Install a single skill
```bash
npx skills add https://github.com/wizeline/sdlc-agents/tree/main/aicores/data-science-agent/skills/ds-conducting-eda
```
