# Notebook Best Practices

> Quality standards for Jupyter and Colab notebooks in data science projects.

---

## 1. Canonical Notebook Structure

Every notebook should follow this cell order from top to bottom:

```
Cell 1 [Markdown] — Header
  # Notebook Title
  **Author:** [name]
  **Date:** [YYYY-MM-DD]
  **Purpose:** [one-paragraph description of what this notebook does]
  **Data source:** [where the data comes from, version, retrieval date]
  **Dependencies:** [link to requirements.txt or list key libraries + versions]

Cell 2 [Code] — Configuration (all constants in one place)
  SEED = 42
  DATA_PATH = Path("../data/raw/dataset_v2.parquet")
  TRAIN_SIZE = 0.8
  TARGET_COL = "churn"
  MODEL_OUTPUT_PATH = Path("../models/churn_v1.pkl")

Cell 3 [Code] — Imports (all imports in one cell)
  import random, os
  from pathlib import Path
  import numpy as np
  import pandas as pd
  import matplotlib.pyplot as plt
  ...

Cell 4 [Code] — Seed Setting
  random.seed(SEED)
  np.random.seed(SEED)

[Analysis cells — grouped by section with Markdown headers]
...

Final cell [Markdown] — Conclusions
  ## Key Findings
  - Finding 1
  ## Recommended Next Steps
  - Action 1
```

---

## 2. Execution Order Discipline

### Rule: Notebooks must run top-to-bottom with zero errors.

**The test:** `Kernel → Restart → Run All` must complete without errors and produce the same results as the last interactive run.

### Anti-patterns that break top-to-bottom execution:

| Anti-pattern | Why it breaks | Fix |
|-------------|--------------|-----|
| Cell run order: 1, 2, 5, 3, 4 | Cell 5 depends on state from 3 and 4 not yet run | Reorganise cells to match logical flow |
| Variable defined in a cell run interactively but not saved | Next run: NameError | Move definition to the appropriate cell in sequence |
| `del variable` in a middle cell, used again later | NameError on re-run | Remove the delete or restructure |
| Import at cell 15 that cell 3 depends on | ImportError on re-run | Move all imports to the imports cell |

### Checking execution numbers
Look at cell execution counts `[N]` in the left margin:
- Sequential with no gaps: good
- Out of order or gaps: cells were run non-linearly → hidden state risk

---

## 3. Seed Management

Set seeds **before any random operation** and **in a dedicated cell** near the top.

```python
# Minimum seed cell
SEED = 42
import random
import numpy as np
random.seed(SEED)
np.random.seed(SEED)
```

### Operations that require seeds

| Operation | Library | Seed Parameter |
|-----------|---------|---------------|
| `train_test_split` | sklearn | `random_state=SEED` |
| `RandomForestClassifier` | sklearn | `random_state=SEED` |
| `XGBClassifier` | xgboost | `seed=SEED` or `random_state=SEED` |
| `KMeans` | sklearn | `random_state=SEED` |
| `SMOTE` | imbalanced-learn | `random_state=SEED` |
| `torch` operations | PyTorch | `torch.manual_seed(SEED)` |
| `tf` operations | TensorFlow | `tf.random.set_seed(SEED)` |
| `np.random` | numpy | `np.random.seed(SEED)` |
| `random` module | Python | `random.seed(SEED)` |

**Rule:** If it involves randomness, it needs a seed. No exceptions.

---

## 4. Dependency Management

### Option A — pip install cell (minimum)
```python
# Cell at top of notebook — clearly labelled
# Run this cell once to install dependencies
# !pip install pandas==2.2.1 scikit-learn==1.4.2 lightgbm==4.3.0 matplotlib==3.8.4
```
Comment out the install cell after initial setup to avoid reinstalling on every run.

### Option B — requirements.txt (preferred for shared notebooks)
Commit a `requirements.txt` alongside the notebook:
```
pandas==2.2.1
scikit-learn==1.4.2
lightgbm==4.3.0
matplotlib==3.8.4
seaborn==0.13.2
```

### Option C — environment.yml (conda environments)
```yaml
name: churn-analysis
channels: [conda-forge]
dependencies:
  - python=3.11
  - pandas=2.2.1
  - scikit-learn=1.4.2
```

**Never acceptable:** "just install the latest version" with no pinning.

---

## 5. Path Management

### Anti-patterns
```python
# WRONG — absolute path, machine-specific
df = pd.read_csv("/Users/john/Documents/projects/churn/data/raw.csv")

# WRONG — assumes specific working directory
df = pd.read_csv("../../data/raw.csv")  # breaks if notebook is opened from another directory
```

### Correct patterns
```python
# CORRECT — relative to notebook location
from pathlib import Path
NOTEBOOK_DIR = Path(__file__).parent if '__file__' in dir() else Path.cwd()
DATA_PATH = NOTEBOOK_DIR / "../../data/raw.csv"

# CORRECT — environment variable (preferred for shared notebooks)
import os
DATA_PATH = Path(os.environ.get("DATA_PATH", "data/raw.csv"))

# CORRECT — defined in configuration cell at top
DATA_PATH = Path("../data/raw/dataset.parquet")  # documented in header
```

---

## 6. Output Hygiene

### What to NEVER commit to version control
- Credentials, API keys, tokens (in any cell output)
- PII: names, emails, SSNs, phone numbers in printed dataframe rows
- Full dataframe outputs with > 50 rows
- Large binary outputs or base64-encoded images bloating the .ipynb file

### Pre-commit output cleaning
Option 1 — Clear manually: `Kernel → Restart → Clear All Outputs`
Option 2 — `nbstripout` (automated pre-commit hook):
```bash
pip install nbstripout
nbstripout --install  # installs as a git pre-commit hook
```

Option 3 — `nbconvert` to strip outputs:
```bash
jupyter nbconvert --ClearOutputPreprocessor.enabled=True --to notebook notebook.ipynb
```

### Output size limits
- Print no more than 10–20 rows of any dataframe: use `df.head(10)`
- For large outputs: save to a file and print the file path instead
- For plots: use `plt.savefig('output/figure.png')` and print the path in production pipelines

---

## 7. Documentation Standards

### Minimum documentation per section
- Each major section must start with a `##` Markdown header
- Complex transformations must have a preceding Markdown cell explaining the intent
- Results cells must have a following Markdown cell interpreting the output (not just raw numbers)
- Magic numbers must have inline comments: `threshold = 0.5  # balanced precision/recall at this threshold`

### Example: good vs. bad cell documentation

**Bad:**
```python
df['risk_score'] = df['age'] * 0.3 + df['income_scaled'] * 0.7
```

**Good:**
```markdown
## Risk Score Computation
Combining age and income into a single risk score using expert-defined weights
(0.3 and 0.7 respectively) from the domain team specification doc [link].
Higher score = higher risk.
```
```python
AGE_WEIGHT = 0.3     # expert-defined: younger customers correlate with higher risk
INCOME_WEIGHT = 0.7  # expert-defined: lower income is a stronger risk signal
df['risk_score'] = df['age_scaled'] * AGE_WEIGHT + df['income_scaled'] * INCOME_WEIGHT
```

---

## 8. Version Control Integration

### What to commit
- Notebook file (`.ipynb`) with outputs cleared (use nbstripout)
- `requirements.txt` or `environment.yml`
- Any data documentation or data dictionary referenced in the notebook

### What to .gitignore
```gitignore
# Jupyter artifacts
.ipynb_checkpoints/
*.pyc
__pycache__/

# Data files (store in DVC or cloud storage)
data/raw/
data/processed/

# Model artifacts
models/*.pkl
models/*.joblib
```

### Notebook as a production pipeline
If a notebook is used to run recurring jobs:
1. Convert to a Python script: `jupyter nbconvert --to script notebook.ipynb`
2. Or use Papermill for parameterised notebook execution:
   ```bash
   papermill notebook.ipynb output_notebook.ipynb -p run_date "2024-01-15"
   ```
3. Store execution outputs as versioned artifacts, not in the notebook file itself
