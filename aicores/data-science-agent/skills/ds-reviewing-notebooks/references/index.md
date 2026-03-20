# Data Science Agent — Notebook Review Reference Index

> Read this file first before applying any notebook review skill step.

## Required Reading Order

### 1. notebook-best-practices.md — The Standards
`./notebook-best-practices.md`

Defines quality standards for Jupyter and Colab notebooks:
- Canonical notebook structure (header, config, imports, analysis, conclusions)
- Cell execution order discipline and hidden state prevention
- Seed management and reproducibility requirements
- Dependency declaration and environment specification
- Output hygiene: what to clear, what to keep, what is never acceptable
- Version control integration: what to commit, what to .gitignore
- Notebook-as-pipeline requirements for production or semi-production use
- Common notebook anti-patterns with fixes

---

## Who Reads This
Every notebook review skill step loads this index as its first action.
ds-pipeline-review-agent loads it conditionally — only when the pipeline under
review is a Jupyter or Colab notebook (Phase 6 of the pipeline review workflow).
