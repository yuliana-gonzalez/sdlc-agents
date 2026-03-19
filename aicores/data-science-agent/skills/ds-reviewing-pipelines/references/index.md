# Data Science Agent — Pipeline Review Reference Index

> Read this file first before applying any pipeline review skill step.

## Required Reading Order

### 1. data-leakage-guide.md — The Leakage Patterns
`./data-leakage-guide.md`

The authoritative reference for all forms of data leakage in ML pipelines:
- Target leakage: feature encodes the label directly or indirectly
- Temporal leakage: future data visible in training features
- Preprocessing leakage: statistics computed before train/test split
- Join/aggregation leakage: future data introduced via table joins
- Sampling/entity leakage: same entity in train and test
- Detection tests for each leakage type
- Fix patterns with code examples

### 2. pipeline-best-practices.md — The Standards
`./pipeline-best-practices.md`

Defines production-readiness standards for data and ML pipelines:
- Reproducibility requirements: seeds, pinned dependencies, data versioning
- Schema validation patterns: pandera, great_expectations, pydantic
- Error handling and observability standards
- Idempotency patterns
- Secrets management
- Scalability assessment checklist
- Recommended pipeline frameworks by use case

---

## Who Reads This
Every pipeline review skill step and every ds-pipeline-review-agent invocation
loads this index as its first action. Reading order is fixed — do not skip.
