---
name: fdr-threshold-sweep-analysis
description: Use when comparing imputation methods (Bfimpute, scImpute, SAVER, VIPER,
  DrImpute, MAGIC) on a paired scRNA-seq/bulk dataset by measuring how imputation
  changes agreement between single-cell and bulk DE gene sets.
when_to_use_negative:
- No matched bulk RNA-seq data are available to define a reference DE gene set
- The goal is clustering, visualization, or pseudotime benchmarking rather than differential-expression
  concordance — the article benchmarks these separately
- The dataset is simulated data without a bulk ground truth for the DE comparison
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Bfimpute
  role: Imputation method under benchmark; its imputed matrix yields the best DE-concordance
    AUC (0.670) when DESeq2 calls are compared against the bulk top-200 reference
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparison imputation method run at recommended defaults before the DE-threshold
    sweep (AUC 0.665)
- name: SAVER
  role: Comparison imputation method run at recommended defaults before the DE-threshold
    sweep (AUC 0.624)
- name: VIPER
  role: Comparison imputation method run at recommended defaults before the DE-threshold
    sweep (AUC 0.639)
- name: DrImpute
  role: Comparison imputation method run at recommended defaults before the DE-threshold
    sweep (AUC 0.657)
- name: MAGIC
  role: Comparison imputation method run at recommended defaults before the DE-threshold
    sweep (AUC 0.668)
- name: DESeq2
  role: Differential expression engine used on both bulk counts (to build the top-200
    reference) and each raw/imputed scRNA-seq matrix
- name: Spectrum
  role: Spectral clustering option inside Bfimpute when cell labels are not used (ccluster
    = c(7,"Spectrum"))
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:4697375c508ae768e3b31cc56da19a7d0b1eff32f9dc6214e7970da4ff4d693c
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
  - DESeq2
  - DrImpute
  - MAGIC
  - SAVER
  - Spectrum
  - VIPER
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# fdr-threshold-sweep-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scRNA-seq imputation methods by differential-expression concordance with bulk RNA-seq: sweep FDR significance thresholds and compute the AUC of overlap between single-cell DE calls and a top-200 bulk DESeq2 reference gene set.

## When to use

Use when comparing imputation methods (Bfimpute, scImpute, SAVER, VIPER, DrImpute, MAGIC) on a paired scRNA-seq/bulk dataset by measuring how imputation changes agreement between single-cell and bulk DE gene sets.

## When NOT to use

- No matched bulk RNA-seq data are available to define a reference DE gene set
- The goal is clustering, visualization, or pseudotime benchmarking rather than differential-expression concordance — the article benchmarks these separately
- The dataset is simulated data without a bulk ground truth for the DE comparison

## Inputs

- sc_qc.csv: QC'd scRNA-seq gene-by-cell count matrix (Chu et al. 2016)
- bulk_qc.csv: bulk RNA-seq count matrix
- cell_type.csv: cell-type label matrix for the 7 cell types
- Imputed count matrices from Bfimpute, scImpute, SAVER, VIPER, DrImpute, and MAGIC

## Outputs

- de_concordance_auc.csv: seven-condition AUC table (raw 0.568; Bfimpute 0.670; scImpute 0.665; SAVER 0.624; VIPER 0.639; DrImpute 0.657; MAGIC 0.668)
- Per-threshold DE gene call sets for each raw/imputed matrix
- Top-200 bulk DESeq2 reference gene set

## How to apply

Load the QC'd Chu et al. (2016) matrices (sc_qc.csv counts, bulk_qc.csv bulk counts, cell_type.csv labels for 7 cell types). Run DESeq2 on the bulk counts to define the top 200 bulk differentially expressed genes as the fixed reference set, then impute the scRNA-seq matrix with each method — Bfimpute can use given cell labels (ccluster = "labeled", label = cell_type, ncores = 5) or spectral clustering with 7 clusters, applying CPM normalization and log10 transform with bias 1.01 (normalized = FALSE), while scImpute, SAVER, VIPER, DrImpute, and MAGIC run with recommended defaults. Rerun DESeq2 on the raw and each of the six imputed matrices, then sweep across FDR significance thresholds to call scRNA-seq DE genes at each cutoff and compute the AUC of concordance with the top-200 bulk reference. In this analysis all imputation methods improved concordance over raw data (raw AUC 0.568), with Bfimpute generating the DE genes most consistent with bulk (AUC 0.670 vs. scImpute 0.665, MAGIC 0.668, DrImpute 0.657, VIPER 0.639, SAVER 0.624). Assemble the seven-condition AUC table and save it as de_concordance_auc.csv.

## Related tools

- **Bfimpute** (Imputation method under benchmark; its imputed matrix yields the best DE-concordance AUC (0.670) when DESeq2 calls are compared against the bulk top-200 reference) — https://github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparison imputation method run at recommended defaults before the DE-threshold sweep (AUC 0.665))
- **SAVER** (Comparison imputation method run at recommended defaults before the DE-threshold sweep (AUC 0.624))
- **VIPER** (Comparison imputation method run at recommended defaults before the DE-threshold sweep (AUC 0.639))
- **DrImpute** (Comparison imputation method run at recommended defaults before the DE-threshold sweep (AUC 0.657))
- **MAGIC** (Comparison imputation method run at recommended defaults before the DE-threshold sweep (AUC 0.668))
- **DESeq2** (Differential expression engine used on both bulk counts (to build the top-200 reference) and each raw/imputed scRNA-seq matrix)
- **Spectrum** (Spectral clustering option inside Bfimpute when cell labels are not used (ccluster = c(7,"Spectrum")))

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- The raw (unimputed) scRNA-seq DE-concordance AUC should be near 0.568 on the Chu et al. dataset; substantially lower values suggest the top-200 bulk reference or DESeq2 calls were built incorrectly
- All six imputed matrices should yield AUC above the raw baseline if imputation improved concordance, as reported for every method in this benchmark
- The output de_concordance_auc.csv must contain exactly seven rows (raw plus six imputation methods) with AUC values
- The bulk DESeq2 step must return exactly the top 200 differentially expressed genes as the fixed reference set used for every threshold-sweep comparison

## Limitations

- There is no consensus about which imputation method is the best; performance varies across evaluation criteria, experimental protocols, datasets, and downstream analyses
- Imputation can increase false positives in downstream analysis; depending on the application users may tune parameters to maximize true positives or minimize false positives
- It is premature to conclude that Bfimpute will achieve better performance across all datasets and improve downstream analysis

## Evidence
