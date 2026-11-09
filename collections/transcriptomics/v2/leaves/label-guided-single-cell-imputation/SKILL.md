---
name: label-guided-single-cell-imputation
description: Use when an scRNA-seq count matrix has pervasive dropouts and cell-type
  labels are available, and the goal is to improve downstream DE analysis concordance
  against a bulk RNA-seq reference.
when_to_use_negative:
- The count matrix is already normalized/log-transformed — you must pass normalized
  = TRUE or the default CPM + log10 (bias 1.01) transform will be applied a second
  time.
- Cell-type labels are unavailable and you cannot supply a clustering function — although
  c(7,"Spectrum") is a documented fallback, label-guided imputation itself requires
  labels.
- No bulk RNA-seq or other reference exists to judge whether imputation improved downstream
  DE analysis.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bfimpute
  role: Performs the label-guided Bayesian factorization imputation, reconstructing
    latent gene and cell matrices within each given cell-type group
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: DESeq2
  role: Identifies the top-200 bulk DE reference genes and reruns DE on each imputed
    matrix for the concordance AUC benchmark
- name: Spectrum
  role: Spectral clustering fallback (c(7,"Spectrum")) to group cells when labels
    are not supplied
- name: scImpute
  role: Comparator imputation method run at default settings in the DE-concordance
    benchmark
- name: SAVER
  role: Comparator imputation method run at default settings in the DE-concordance
    benchmark
- name: VIPER
  role: Comparator imputation method run at default settings in the DE-concordance
    benchmark
- name: DrImpute
  role: Comparator imputation method run at default settings in the DE-concordance
    benchmark
- name: MAGIC
  role: Comparator imputation method run at default settings in the DE-concordance
    benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:52df0707c526a39fe057a6aaba55c334c51546b656b37c43d60c121da0e1f21b
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
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# label-guided-single-cell-imputation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Imputes dropout events in a scRNA-seq count matrix using Bfimpute's Bayesian factorization with user-provided cell type labels (ccluster='labeled') instead of clustering, then verifies the imputation by improved differential-expression concordance with bulk RNA-seq (AUC lift from 0.568 raw to 0.670 on the Chu et al. 2016 dataset).

## When to use

Use when an scRNA-seq count matrix has pervasive dropouts and cell-type labels are available, and the goal is to improve downstream DE analysis concordance against a bulk RNA-seq reference.

## When NOT to use

- The count matrix is already normalized/log-transformed — you must pass normalized = TRUE or the default CPM + log10 (bias 1.01) transform will be applied a second time.
- Cell-type labels are unavailable and you cannot supply a clustering function — although c(7,"Spectrum") is a documented fallback, label-guided imputation itself requires labels.
- No bulk RNA-seq or other reference exists to judge whether imputation improved downstream DE analysis.

## Inputs

- scRNA-seq raw count matrix (sc_qc.csv, genes x cells)
- bulk RNA-seq count matrix (bulk_qc.csv)
- cell-type label matrix/vector (cell_type.csv, 7 cell types)
- Number of cell types / cluster count (7 for the Chu dataset)

## Outputs

- Imputed gene expression count matrix (R_calculate) from Bfimpute
- Six-method imputed matrices (scImpute, SAVER, VIPER, DrImpute, MAGIC, Bfimpute) for benchmarking
- DE concordance AUC table (de_concordance_auc.csv): raw 0.568; Bfimpute 0.670; scImpute 0.665; SAVER 0.624; VIPER 0.639; DrImpute 0.657; MAGIC 0.668

## How to apply

Load the QC'd Chu et al. (2016) inputs (sc_qc.csv counts, bulk_qc.csv bulk counts, cell_type.csv labels, 7 cell types) with genes as rows and cells as columns. Call Bfimpute with ccluster = "labeled" and label = cell_type (ncores = 5) so that imputation happens within each given cell-type group rather than via spectral clustering (c(7,"Spectrum") is the fallback when labels are absent); keep normalized = FALSE so Bfimpute applies CPM plus log10 transform with bias 1.01. Optionally pass bulk data as gene side information via S_G = t(bulk). For validation, impute the same counts with scImpute, SAVER, VIPER, DrImpute, and MAGIC at default settings, run DESeq2 on the bulk counts to define the top-200 bulk DE reference set, rerun DESeq2 on each raw/imputed matrix, and sweep FDR significance thresholds to compute the concordance AUC between scRNA-seq DE calls and the bulk reference.

## Related tools

- **Bfimpute** (Performs the label-guided Bayesian factorization imputation, reconstructing latent gene and cell matrices within each given cell-type group) — https://github.com/maiziezhoulab/Bfimpute
- **DESeq2** (Identifies the top-200 bulk DE reference genes and reruns DE on each imputed matrix for the concordance AUC benchmark)
- **Spectrum** (Spectral clustering fallback (c(7,"Spectrum")) to group cells when labels are not supplied)
- **scImpute** (Comparator imputation method run at default settings in the DE-concordance benchmark)
- **SAVER** (Comparator imputation method run at default settings in the DE-concordance benchmark)
- **VIPER** (Comparator imputation method run at default settings in the DE-concordance benchmark)
- **DrImpute** (Comparator imputation method run at default settings in the DE-concordance benchmark)
- **MAGIC** (Comparator imputation method run at default settings in the DE-concordance benchmark)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- The seven-condition AUC table should reproduce raw 0.568; Bfimpute 0.670; scImpute 0.665; SAVER 0.624; VIPER 0.639; DrImpute 0.657; MAGIC 0.668, with every imputation method above the raw baseline and Bfimpute highest.
- The imputed matrix returned as R_calculate should preserve the genes x cells dimensions of sc_qc.csv.
- With normalized = FALSE, the pipeline should have applied CPM normalization and a log10 transform with bias 1.01 before factorization.
- Bulk DE reference set should contain exactly the top 200 DESeq2 genes from bulk_qc.csv.
- Output files (csv/txt/rds or 'all') should appear under out_dir ('./Bfimpute/') when out_type is not 'none'.

## Limitations

- No single best imputation method exists overall — performance varies across datasets, protocols, and downstream analyses, so Bfimpute's superiority is not guaranteed beyond the benchmarked data.
- Depending on the application, imputation may increase false positives in downstream analysis; parameters should be tuned to maximize true positives or minimize false positives.
- Larger D (default 32), totalepoch (default 300), and burnin (default 200) improve accuracy at the cost of runtime; hyperparameters are fixed at mu_0=0, beta_0=2, nu_0=D, W_0=I.

## Evidence
