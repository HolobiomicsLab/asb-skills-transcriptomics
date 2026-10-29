---
name: silhouette-based-clustering-evaluation
description: Use when comparing imputed scRNA-seq expression matrices (e.g., Bfimpute
  vs scImpute/DrImpute/PBLR outputs) on datasets with known cell type labels and you
  need silhouette-based clustering quality metrics.
when_to_use_negative:
- Datasets lack reliable true cell type labels and clustering must be inferred instead
  (Bfimpute would need ccluster = c(7,"Spectrum") or a custom clustering function
  rather than labeled mode)
- The goal is only imputation itself, not benchmarking clustering/cell-to-cell relationship
  recovery across methods
- Input counts are already normalized — set normalized=TRUE, otherwise CPM plus log10
  bias 1.01 will be re-applied inappropriately
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bfimpute
  role: Bayesian factorization imputation run in labeled mode (ccluster = "labeled")
    to produce the imputed matrix evaluated by silhouette scores
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparator imputation method run with default settings on the same count matrix
    before silhouette evaluation
- name: DrImpute
  role: Comparator imputation method run with default settings on the same count matrix
    before silhouette evaluation
- name: PBLR
  role: Comparator low-rank imputation method run with default settings on the same
    count matrix before silhouette evaluation
- name: devtools
  role: Used to install Bfimpute from GitHub via devtools::install_github("maiziezhoulab/Bfimpute")
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:ada71fd10577c5e8b448e05aa97a130114ae761683dadf6f16c2e73733bce13a
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
  - PBLR
  - devtools
  - scImpute
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# silhouette-based-clustering-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scRNA-seq imputation methods by computing silhouette scores against true cell type labels and same/different-cell-type Pearson correlations on each imputed expression matrix, as done for Bfimpute vs scImpute, DrImpute, PBLR, ALRA, and mcImpute.

## When to use

Use when comparing imputed scRNA-seq expression matrices (e.g., Bfimpute vs scImpute/DrImpute/PBLR outputs) on datasets with known cell type labels and you need silhouette-based clustering quality metrics.

## When NOT to use

- Datasets lack reliable true cell type labels and clustering must be inferred instead (Bfimpute would need ccluster = c(7,"Spectrum") or a custom clustering function rather than labeled mode)
- The goal is only imputation itself, not benchmarking clustering/cell-to-cell relationship recovery across methods
- Input counts are already normalized — set normalized=TRUE, otherwise CPM plus log10 bias 1.01 will be re-applied inappropriately

## Inputs

- QC'd scRNA-seq count matrix (genes x cells CSV, e.g., sc_qc.csv)
- Cell type label file (cell_type.csv, vector/data.frame/matrix with one row or column)
- Imputed expression matrices from Bfimpute, scImpute, DrImpute, PBLR, ALRA, mcImpute
- Optional bulk RNA-seq matrix (bulk_qc.csv) for S_G gene side information

## Outputs

- Per-method silhouette scores computed with true cell type labels
- Same/different-cell-type Pearson correlation distributions between cells
- Summary CSV tables of silhouette and correlation values (Figure 6 boxplot / Table S1 style)

## How to apply

Install Bfimpute from GitHub (devtools::install_github) and load the QC'd count matrix (sc_qc.csv), optional bulk matrix (bulk_qc.csv), and cell type labels (cell_type.csv) with read.csv. Run Bfimpute in labeled mode via Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5) with defaults (totalepoch 300, burnin 200, D 32, threshold 0.5); with normalized=FALSE it performs CPM and log10 transform with bias 1.01. Run each comparator (scImpute, DrImpute, PBLR, ALRA, mcImpute) on the same count matrix with default settings. For every imputed matrix, compute silhouette scores using the true cell type labels plus same/different-cell-type Pearson correlations between cells. Aggregate per-method silhouette and correlation values into summary CSV tables mirroring the Figure 6 boxplot data and Table S1 metrics.

## Related tools

- **Bfimpute** (Bayesian factorization imputation run in labeled mode (ccluster = "labeled") to produce the imputed matrix evaluated by silhouette scores) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparator imputation method run with default settings on the same count matrix before silhouette evaluation)
- **DrImpute** (Comparator imputation method run with default settings on the same count matrix before silhouette evaluation)
- **PBLR** (Comparator low-rank imputation method run with default settings on the same count matrix before silhouette evaluation)
- **devtools** (Used to install Bfimpute from GitHub via devtools::install_github("maiziezhoulab/Bfimpute"))

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- Bfimpute attains the best silhouette score in all but one of the evaluated benchmark sets (Chu, Tang, Petropoulos, Zheng), matching reported results
- Same-cell-type Pearson correlations exceed different-cell-type correlations on the Bfimpute-imputed matrix
- Each imputed matrix has identical gene x cell dimensions to the QC'd input sc_qc.csv matrix
- Summary CSV tables contain one silhouette/correlation entry per method per dataset, suitable for Figure 6-style boxplots
- Bfimpute run with normalized=FALSE shows evidence of CPM and log10 transform with bias 1.01 being applied

## Limitations

- No single best imputation method could be identified; performance varies across evaluation criteria, experimental protocols, datasets, and downstream analyses
- It is premature to conclude Bfimpute will achieve better performance across all datasets — generalizability beyond the benchmarked datasets is unconfirmed
- Depending on the application, parameters may need constraining to maximize true positives or minimize false positives

## Evidence

- [other] benchmark finding: [evidence span withheld — anchor status: not_found]
