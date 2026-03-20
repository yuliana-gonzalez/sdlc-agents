---
name: ds-reviewing-notebooks
description: >
  Jupyter and Colab notebook quality review skill. Checks cell execution order,
  hidden state, hardcoded paths and credentials, reproducibility (seeds, pinned
  dependencies), documentation quality, and output cleanliness. Produces a
  notebook review report with scored dimensions and an actionable issue list.
context: fork
agent: Plan
---

## When to Apply This Skill
- Reviewing a notebook before it is shared, published, or committed to a repo
- Auditing a notebook that is used as a production or semi-production pipeline
- Onboarding a notebook from another team and assessing its quality
- Pre-commit hook or CI/CD check for notebook quality standards
- Reviewing a data science intern or junior contributor's analysis notebook

---

Read `references/index.md` before executing any step.

---

## Step 1 — Execution Order & Hidden State

The most critical notebook defect: cells run out of order, creating hidden state that
makes the notebook non-reproducible and misleading.

### Checks
```
1. Cell execution numbers: are they sequential (1, 2, 3, ...) with no gaps or out-of-order cells?
   - Out-of-order cell numbers → cells were run in non-linear order
   - Gaps in execution numbers → cells were skipped or deleted after running

2. Hidden state test (if runnable): Kernel → Restart → Run All
   - If this fails or produces different results → hidden state exists

3. Variable shadowing: same variable name defined in multiple cells (overwrites silently)

4. Implicit import dependencies: a cell uses a library imported 10 cells earlier
   (fragile; reorganise to group imports)
```

### Severity
- Notebook fails on Restart → Run All: **CRITICAL**
- Out-of-order cell execution numbers: **HIGH**
- Variable shadowing: **MEDIUM**
- Imports scattered throughout notebook: **LOW**

---

## Step 2 — Hardcoded Values

### Paths
```
Scan for:
- Absolute paths: /Users/john/data/, C:\Users\john\, /home/ubuntu/
- Hardcoded S3/GCS/Azure bucket paths with environment-specific names
- Relative paths that assume a specific working directory

Fix: use pathlib.Path(__file__).parent or os.path.join(os.getcwd(), ...)
     or externalise paths to a config cell at the top of the notebook
```

### Credentials & Secrets
```
Scan for:
- API keys, tokens, passwords in code cells or outputs
- Hardcoded database connection strings with passwords
- Environment variable values printed in outputs

Fix: use os.environ['KEY'] or a secrets manager; never commit actual values
```

### Magic Numbers
```
Flag numeric literals with no explanation:
- Thresholds (0.5, 0.8, 1000) without a comment explaining their origin
- Dates hardcoded as strings ('2023-01-01') without a variable name

Recommendation: assign to named constants with comments at the top of the notebook
```

---

## Step 3 — Reproducibility

### Seed Discipline
```
Check for:
1. train_test_split without random_state: CRITICAL
2. Any model initialisation without random_state / seed (RandomForest, XGBoost, etc.)
3. np.random, random, torch, tensorflow operations without prior seed setting
4. Missing: np.random.seed(), random.seed(), torch.manual_seed()

Fix template (place in a seed cell near the top):
    SEED = 42
    import random, numpy as np
    random.seed(SEED)
    np.random.seed(SEED)
    # If using PyTorch:
    import torch
    torch.manual_seed(SEED)
    # If using TensorFlow:
    import tensorflow as tf
    tf.random.set_seed(SEED)
```

### Dependency Declaration
```
Check for:
1. No requirements.txt, environment.yml, or pip install cells
2. pip install cells with unpinned versions: pip install pandas (not pandas==2.2.1)
3. Library imported but not in requirements file

Fix: add a requirements cell at the top with pinned versions, OR
     commit requirements.txt alongside the notebook
```

### Data Source Repeatability
```
Check for:
1. Data loaded from a local path without a URL or version reference
2. Data pulled from a live API without a snapshot or version
3. No record of when/how the data was obtained

Fix: document data source, version, and retrieval date in the notebook header
```

---

## Step 4 — Documentation Quality

### Notebook Structure
A well-documented notebook has these sections in order:
1. **Header cell** (Markdown): title, author, date, purpose, dependencies, data source
2. **Configuration cell** (Code): all constants, seeds, and paths in one place
3. **Imports cell** (Code): all library imports
4. **Data loading** section
5. **EDA / Analysis** sections
6. **Modelling / Processing** sections
7. **Results / Conclusions** section

### Markdown Cell Coverage
```
Check:
- Major sections separated by Markdown headers (## Section Name)
- Findings and interpretations documented in Markdown, not just in code comments
- Complex code cells preceded by a Markdown cell explaining intent
- Cells that produce important output have a Markdown cell interpreting the output
```

### Code Comments
```
Flag:
- Complex transformations with no explanation
- Magic number thresholds with no justification
- Non-obvious column manipulations
```

---

## Step 5 — Output Cleanliness

### Sensitive Output
```
Scan outputs for:
- Printed API keys, tokens, or passwords
- PII in printed dataframe rows (names, emails, SSNs, phone numbers)
- Full connection strings with passwords
- Internal URLs, IP addresses, or server names

Severity: CRITICAL — strip outputs before committing to version control
```

### Output Size
```
Flag:
- Cells that print > 50 rows of a dataframe without .head() or .tail()
- Cells that dump full model parameters or large JSON/dict outputs
- Images or plots with resolution so high they bloat the .ipynb file

Fix: use df.head(10), truncate large outputs, save large plots as files
```

### Stale Outputs
```
Flag:
- Cell outputs present when cell execution number is missing (output from deleted run)
- Cell outputs that contradict the current code (code was edited after running)

Fix: Restart → Run All before committing; or clear all outputs before committing
```

---

## Step 6 — Notebook Health Score

| Dimension | Weight | Score | Weighted |
|-----------|--------|-------|----------|
| Execution Order / No Hidden State | 25% | | |
| No Hardcoded Secrets | 20% | | |
| Reproducibility (seeds + deps) | 20% | | |
| Documentation Quality | 15% | | |
| No Sensitive Outputs | 15% | | |
| Output Cleanliness | 5% | | |
| **Total** | 100% | | **__%** |

**Verdict:**
- ≥ 80% and zero CRITICAL → **PASS**
- 60–79% or any HIGH → **WARN**
- < 60% or any CRITICAL → **FAIL**

---

## Step 7 — Report Assembly

Use `assets/notebook-review-template.md` as structure. Include:
1. Notebook metadata (title, purpose, cell count, execution date if visible)
2. Health score per dimension
3. All findings with severity and specific cell reference
4. Remediation steps for CRITICAL and HIGH findings

---

## Quality Checklist

Before emitting the notebook review report:
- [ ] Execution order check completed (cell numbers sequential or gap-free)
- [ ] All hardcoded paths and secrets identified
- [ ] Seed usage checked for every random operation
- [ ] Dependency declaration assessed
- [ ] Documentation coverage rated (header, section headers, code explanations)
- [ ] All outputs scanned for PII and credentials
- [ ] Health score computed from weighted dimensions
- [ ] Every CRITICAL and HIGH finding includes a specific cell reference
- [ ] Report uses the notebook review template structure
