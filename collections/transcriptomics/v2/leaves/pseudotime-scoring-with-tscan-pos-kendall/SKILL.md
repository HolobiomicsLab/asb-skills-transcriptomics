---
name: pseudotime-scoring-with-tscan-pos-kendall
description: Use when comparing scRNA-seq imputation methods by how well each imputed
  matrix recovers cell-to-cell trajectory order — e.g., scoring imputed 10x data with
  TSCAN POS and Kendall rank correlation to rank Bfimpute vs. competitors.
when_to_use_negative:
- The dataset has no trajectory/developmental structure (static, unrelated cell types),
  so pseudotime recovery is not a meaningful accuracy criterion
- You need a single per-gene expression accuracy estimate rather than recovery of
  cell-to-cell relationships — prior imputation methods were noted to be less effective
  at recovering cell-to-cell relationships, and pseudotime scoring does not measure
  point-estimate gene accuracy
- Only one matrix is available with no competing imputation methods to rank
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TSCAN
  role: Pseudotime inference engine whose POS and Kendall rank correlation scores
    quantify how well each imputed matrix recovers trajectory order
- name: Bfimpute
  role: Primary imputation method whose output imputed matrices are scored with TSCAN;
    run in labeled mode with D=32, totalepoch=300, burnin=200, ncores=5
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Competitor imputation method run in labeled mode before TSCAN scoring
- name: DrImpute
  role: Competitor imputation method; fails on the 62,893-cell Vladoiu dataset
- name: SAVER
  role: Competitor imputation method run with defaults before TSCAN scoring
- name: MAGIC
  role: Competitor imputation method; fastest in runtime ordering (MAGIC < PBLR <
    Bfimpute < scImpute < SAVER)
- name: VIPER
  role: Competitor imputation method; fails on the 62,893-cell Vladoiu dataset
- name: PBLR
  role: Competitor imputation method run in labeled mode; second fastest after MAGIC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:02bb1cef52662f41b07524a9753260cf0daf582b418a4ddb8bcf99223d11652d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - MAGIC
  - PBLR
  - SAVER
  - TSCAN
  - VIPER
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pseudotime-scoring-with-tscan-pos-kendall

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scRNA-seq imputation quality by running TSCAN pseudotime inference on each imputed count matrix and recording POS and Kendall's rank correlation scores, as done in the Bfimpute benchmark on the 62,893-cell Vladoiu mouse cerebellum dataset (91.9% dropout).

## When to use

Use when comparing scRNA-seq imputation methods by how well each imputed matrix recovers cell-to-cell trajectory order — e.g., scoring imputed 10x data with TSCAN POS and Kendall rank correlation to rank Bfimpute vs. competitors.

## When NOT to use

- The dataset has no trajectory/developmental structure (static, unrelated cell types), so pseudotime recovery is not a meaningful accuracy criterion
- You need a single per-gene expression accuracy estimate rather than recovery of cell-to-cell relationships — prior imputation methods were noted to be less effective at recovering cell-to-cell relationships, and pseudotime scoring does not measure point-estimate gene accuracy
- Only one matrix is available with no competing imputation methods to rank

## Inputs

- Raw 10x scRNA-seq count matrix (genes x cells), e.g. Vladoiu mouse cerebellum GSE118068 (62,893 cells, 91.9% dropout)
- Cell type labels for labeled-mode imputation
- Imputed expression matrices from each method (Bfimpute, scImpute, DrImpute, SAVER, MAGIC, VIPER, PBLR)

## Outputs

- TSCAN POS score per imputed matrix
- TSCAN Kendall's rank correlation score per imputed matrix
- Four clustering metrics per imputed matrix
- Runtime table per method
- Score/runtime tables matching Figure S6, saved as vladoiu_imputation_benchmark_results.csv

## How to apply

First obtain imputed matrices for every method under comparison: for Bfimpute, impute the raw count matrix with cell labels supplied via ccluster='labeled'/label and defaults D=32, totalepoch=300, burnin=200, sn_max=10, sn_init=1, threshold=0.5, ncores=5, and run competitors (scImpute, DrImpute, SAVER, PBLR, MAGIC, VIPER) with their defaults. Run TSCAN pseudotime inference on each imputed matrix and record POS and Kendall's rank correlation scores. In parallel, compute the four clustering metrics on the same imputed matrices and record wall-clock runtime per method. In the Vladoiu benchmark, Bfimpute attained the best TSCAN POS/Kendall and clustering scores, and the runtime ordering reproduced MAGIC < PBLR < Bfimpute < scImpute < SAVER, with DrImpute and VIPER failing on the 62,893-cell dataset. Assemble scores and runtimes into tables matching Figure S6 and save as vladoiu_imputation_benchmark_results.csv.

## Related tools

- **TSCAN** (Pseudotime inference engine whose POS and Kendall rank correlation scores quantify how well each imputed matrix recovers trajectory order)
- **Bfimpute** (Primary imputation method whose output imputed matrices are scored with TSCAN; run in labeled mode with D=32, totalepoch=300, burnin=200, ncores=5) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Competitor imputation method run in labeled mode before TSCAN scoring)
- **DrImpute** (Competitor imputation method; fails on the 62,893-cell Vladoiu dataset)
- **SAVER** (Competitor imputation method run with defaults before TSCAN scoring)
- **MAGIC** (Competitor imputation method; fastest in runtime ordering (MAGIC < PBLR < Bfimpute < scImpute < SAVER))
- **VIPER** (Competitor imputation method; fails on the 62,893-cell Vladoiu dataset)
- **PBLR** (Competitor imputation method run in labeled mode; second fastest after MAGIC)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- Bfimpute-imputed matrix attains the best TSCAN POS and Kendall's rank correlation scores among all compared imputation methods
- Reproduced score tables match Figure S6 of the Bfimpute paper
- Runtime ordering reproduces MAGIC < PBLR < Bfimpute < scImpute < SAVER
- DrImpute and VIPER are expected to fail (error/exit) on the 62,893-cell Vladoiu dataset
- Bfimpute also achieves the best clustering accuracy among compared methods on the same imputed matrices

## Limitations

- DrImpute and VIPER fail on extremely large datasets (62,893 cells), so the comparison may be incomplete for those methods
- No single best imputation method could be identified overall; performance varies across evaluation criteria, datasets, and downstream analyses, and pseudotime scoring is only one axis
- Imputation can increase false positives in downstream analysis; parameters must be tuned depending on the application
- Generalizability beyond benchmarked datasets is unconfirmed — it is premature to conclude Bfimpute will always win on pseudotime scores

## Evidence

- [other] Score each imputed matrix with TSCAN pseudotime inference, recording POS and Kendall correlation scores, and compute the four clustering metrics on the imputed data.: [evidence span withheld — anchor status: not_found]
- [results] Bfimpute improved t-SNE visualization and achieved the best pseudotime performance (POS and Kendall's rank correlation) and the best clustering accuracy among the compared imputation methods.: [evidence span withheld — anchor status: not_found]
- [supplementary] the reproduced tables match Figure S6, with Bfimpute attaining the best TSCAN POS/Kendall and clustering scores and the runtime ordering reproducing MAGIC < PBLR < Bfimpute < scImpute < SAVER: [evidence span withheld — anchor status: not_found]
