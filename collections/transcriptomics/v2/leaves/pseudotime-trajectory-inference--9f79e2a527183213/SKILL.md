---
name: pseudotime-trajectory-inference
description: Use when you have single-cell counts with known developmental/staged
  time labels (e.g., Chu GSE75748, Scialdone GSE74994, Petropoulos E-MTAB-3929) and
  need to test whether imputation recovers true temporal cell order.
when_to_use_negative:
- The dataset has no external time/stage labels or biological ordering against which
  POS and Kendall's correlation can be computed
- The goal is clustering, DE analysis, or visualization rather than temporal ordering
  — the article notes prior imputation methods improve these analyses differently
- 'You need a single universally best imputation method: the article''s discussion
  warns ''There is no consensus about which imputation method is the best'' and performance
  varies by dataset and downstream analysis'
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bfimpute
  role: Bayesian factorization imputation of the raw counts before trajectory inference;
    uses spectral clustering (Spectrum/specc) of cells and CPM + log10 transform with
    bias 1.01
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Competing imputation method run on the same matrices to benchmark pseudotime
    improvement
- name: DrImpute
  role: Competing imputation method that averages imputation from multiple clustering
    results
- name: TSCAN
  role: Trajectory/pseudotime ordering tool applied to raw and imputed matrices
- name: Monocle
  role: Trajectory/pseudotime ordering tool applied to raw and imputed matrices
- name: Spectrum
  role: Spectral clustering of cells grouping cell types before Bfimpute imputation
    (e.g., 7 clusters for Chu data)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:93eb296d619e8dee56d102b6ea4dd3ad387b5c913b1dc14bd536b83acbddbd45
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  repo_url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - Monocle
  - Monocle3
  - R
  - Rtsne
  - SCORPIUS
  - Seurat
  - Spectrum
  - TSCAN
  - mclust
  - scImpute
  - scran
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- trajectory-pseudotime-inference
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pseudotime-trajectory-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark whether imputation improves pseudotime ordering of developmental scRNA-seq cells by inferring trajectories with TSCAN and Monocle on raw versus imputed matrices (Bfimpute, scImpute, DrImpute) and scoring orderings against true developmental time with POS and Kendall's rank correlation.

## When to use

Use when you have single-cell counts with known developmental/staged time labels (e.g., Chu GSE75748, Scialdone GSE74994, Petropoulos E-MTAB-3929) and need to test whether imputation recovers true temporal cell order.

## When NOT to use

- The dataset has no external time/stage labels or biological ordering against which POS and Kendall's correlation can be computed
- The goal is clustering, DE analysis, or visualization rather than temporal ordering — the article notes prior imputation methods improve these analyses differently
- You need a single universally best imputation method: the article's discussion warns 'There is no consensus about which imputation method is the best' and performance varies by dataset and downstream analysis

## Inputs

- QC-filtered scRNA-seq count matrix (genes x cells), e.g., Chu GSE75748 sc_qc.csv
- Known developmental time / stage labels for each cell (true ordering)
- Optional cell type label file (cell_type.csv) or bulk RNA-seq matrix (bulk_qc.csv) for Bfimpute side information

## Outputs

- Pseudotime orderings of cells from TSCAN and Monocle on raw and each imputed matrix
- POS (percent of overlap) scores per method/tool/dataset
- Kendall's rank correlation scores per method/tool/dataset
- Score table comparing 3 datasets x 2 trajectory tools x 4 imputation methods

## How to apply

Run Bfimpute on the raw QC-filtered count matrix with spectral clustering (e.g., ccluster = c(7, "Spectrum") for the 7-cell-type Chu data), letting it perform CPM normalization and log10 transform with bias 1.01, using defaults D=32, totalepoch=300, burnin=200, threshold=0.5, ncores=5. Impute the same matrices with competing methods scImpute and DrImpute, so each dataset yields 4 expression matrices (raw + 3 imputed). Infer pseudotime orderings on every matrix with two trajectory tools, TSCAN and Monocle. Score each ordering against the known developmental ordering using POS (percent of overlap) and Kendall's rank correlation; e.g., on Chu with TSCAN, raw data scores POS 0.610 / Kendall 0.317 while Bfimpute imputation improves this to POS 1.000 / Kendall 0.902. Assemble the datasets x tools x methods score table and check reproduced values against published Table 1 to verify correct application.

## Related tools

- **Bfimpute** (Bayesian factorization imputation of the raw counts before trajectory inference; uses spectral clustering (Spectrum/specc) of cells and CPM + log10 transform with bias 1.01) — https://github.com/maiziezhoulab/Bfimpute
- **scImpute** (Competing imputation method run on the same matrices to benchmark pseudotime improvement)
- **DrImpute** (Competing imputation method that averages imputation from multiple clustering results)
- **TSCAN** (Trajectory/pseudotime ordering tool applied to raw and imputed matrices)
- **Monocle** (Trajectory/pseudotime ordering tool applied to raw and imputed matrices)
- **Spectrum** (Spectral clustering of cells grouping cell types before Bfimpute imputation (e.g., 7 clusters for Chu data))

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- On the Chu dataset with TSCAN, raw counts give POS 0.610 and Kendall 0.317; Bfimpute imputation should improve these to 1.000 and 0.902
- Reproduced scores for all 3 datasets (Chu, Scialdone, Petropoulos) x 2 tools (TSCAN, Monocle) x 4 methods match published Table 1 values
- Bfimpute's POS and Kendall scores exceed those of raw data, scImpute, and DrImpute across the benchmarked datasets
- Imputed matrices are generated under documented Bfimpute defaults: D=32, totalepoch=300, burnin=200, threshold=0.5, ncores=5, with CPM + log10 (bias 1.01) normalization applied

## Limitations

- The article cautions that 'it is premature to conclude that Bfimpute will achieve better performance across all datasets and improve downstream analysis' — generalizability beyond Chu, Scialdone, and Petropoulos is unconfirmed
- Imputation may increase false positives in downstream analyses; users must tune parameters depending on the application
- There is no consensus about which imputation method is the best; performance varies across criteria, protocols, datasets, and analyses

## Evidence
