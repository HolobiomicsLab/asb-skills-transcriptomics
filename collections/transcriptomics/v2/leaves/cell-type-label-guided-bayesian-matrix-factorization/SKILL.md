---
name: cell-type-label-guided-bayesian-matrix-factorization
description: Use when scRNA-seq count matrices come with trusted cell type labels
  and the goal is dropout imputation that leverages those labels, e.g. benchmarking
  silhouette/Pearson clustering metrics against scImpute, DrImpute, PBLR.
when_to_use_negative:
- Cell type labels are unavailable or unreliable — use Bfimpute's unsupervised mode
  with ccluster = c(7, "Spectrum") or a custom clustering function instead of "labeled"
- The count matrix is already normalized — set normalized = TRUE (Bfimpute otherwise
  performs CPM and log10 transform with bias 1.01, and double-normalizing is undesirable)
- The goal is a universally best imputation method — the article's discussion notes
  'There is no consensus about which imputation method is the best' and generalizability
  beyond benchmarked datasets is unconfirmed
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Bfimpute
  role: Bayesian factorization imputation run in labeled mode (ccluster = "labeled")
    to impute dropouts within cell-type-defined groups
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: devtools
  role: Installs Bfimpute from GitHub via devtools::install_github
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparator imputation method run with default settings for labeled benchmark
    comparison
- name: DrImpute
  role: Comparator imputation method averaging imputation over multiple clustering
    results, run with defaults for benchmarking
- name: PBLR
  role: Comparator low-rank matrix-factorization imputation method run with defaults
    for benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:62d14903403e2d5dc443ad170bed1c1c6192d9352f20346d4b79e999cb7a3cda
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
  - PBLR
  - devtools
  - scImpute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-type-label-guided-bayesian-matrix-factorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run Bfimpute in labeled mode, using provided cell type labels to replace internal clustering so that imputation is performed within each known cell group, improving recovery of cell-to-cell relationships in QC'd scRNA-seq count matrices.

## When to use

Use when scRNA-seq count matrices come with trusted cell type labels and the goal is dropout imputation that leverages those labels, e.g. benchmarking silhouette/Pearson clustering metrics against scImpute, DrImpute, PBLR.

## When NOT to use

- Cell type labels are unavailable or unreliable — use Bfimpute's unsupervised mode with ccluster = c(7, "Spectrum") or a custom clustering function instead of "labeled"
- The count matrix is already normalized — set normalized = TRUE (Bfimpute otherwise performs CPM and log10 transform with bias 1.01, and double-normalizing is undesirable)
- The goal is a universally best imputation method — the article's discussion notes 'There is no consensus about which imputation method is the best' and generalizability beyond benchmarked datasets is unconfirmed

## Inputs

- QC'd scRNA-seq count matrix (genes x rows, cells x columns; CSV, e.g. sc_qc.csv)
- Cell type labels (vector, data.frame, or one-row/one-column matrix, e.g. cell_type.csv)
- Optional bulk expression matrix for gene side information (CSV, e.g. bulk_qc.csv)

## Outputs

- Imputed gene expression matrix (returned as R_calculate, and saved to ./Bfimpute/ as csv, txt, and rds by default)
- Optionally the latent G and C matrices when returnGC = TRUE
- Per-method benchmark summary tables of silhouette scores and same/different-cell-type Pearson correlations (CSV)

## How to apply

Install Bfimpute from GitHub with devtools::install_github("maiziezhoulab/Bfimpute") in R. Load the QC'd scRNA-seq count matrix (genes x cells, e.g. sc_qc.csv) and the cell type label file (cell_type.csv) with read.csv. Invoke Bfimpute with ccluster = "labeled" and label = cell_type so provided labels replace the spectral clustering (Spectrum/specc) step; leave normalized = FALSE so Bfimpute performs CPM and log10 transform with bias 1.01, and keep defaults totalepoch = 300, burnin = 200, D = 32, threshold = 0.5, ncores = 5. Bfimpute factorizes the count matrix into gene-specific and cell-specific latent matrices via full Bayesian inference with MCMC, imputing within each label-defined cell group to better capture cell-to-cell relationships. Optionally supply bulk-derived side information via S_G (e.g. S_G = t(bulk)) from bulk_qc.csv. For benchmarking, run scImpute, DrImpute, and PBLR with defaults on the same counts, then compute silhouette scores with true labels and same/different-cell-type Pearson correlations per imputed matrix.

## Related tools

- **Bfimpute** (Bayesian factorization imputation run in labeled mode (ccluster = "labeled") to impute dropouts within cell-type-defined groups) — https://github.com/maiziezhoulab/Bfimpute
- **devtools** (Installs Bfimpute from GitHub via devtools::install_github) — https://github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparator imputation method run with default settings for labeled benchmark comparison)
- **DrImpute** (Comparator imputation method averaging imputation over multiple clustering results, run with defaults for benchmarking)
- **PBLR** (Comparator low-rank matrix-factorization imputation method run with defaults for benchmarking)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- Bfimpute attains the best silhouette score (computed with true cell type labels) in all but one of the benchmarked datasets (Chu, Tang, Petropoulos, Zheng), matching reported Figure 6 / Table S1 results
- Same-cell-type Pearson correlations between cells exceed different-cell-type correlations after labeled-mode imputation, consistent with improved cell-to-cell relationship recovery
- The returned imputed matrix R_calculate has the same genes x cells dimensions as the input count matrix and is log10(CPM + 1.01)-scaled when normalized = FALSE
- Saved output files (csv, txt, rds by default) appear in ./Bfimpute/ with the configured out_name
- Comparator runs (scImpute, DrImpute, PBLR) on identical counts produce per-method summary rows so relative silhouette/correlation rankings can be aggregated and compared

## Limitations

- Performance gains are demonstrated only on benchmarked datasets; the discussion states it is premature to conclude Bfimpute will achieve better performance across all datasets
- Larger D, totalepoch, and burnin increase accuracy but cost more compute time; smaller values save time at an accuracy price
- Imputation can increase false positives in downstream analyses; users may need to constrain parameters (e.g. the dropout-probability threshold, default 0.5) per application
- No single best imputation method exists — performance varies across evaluation criteria, experimental protocols, datasets, and downstream analyses

## Evidence

- [results] labeled benchmark finding: [evidence span withheld — anchor status: not_found]
