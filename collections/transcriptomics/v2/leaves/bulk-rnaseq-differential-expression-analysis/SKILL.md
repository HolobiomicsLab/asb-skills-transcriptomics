---
name: bulk-rnaseq-differential-expression-analysis
description: Use when you have matched scRNA-seq and bulk RNA-seq counts for the same
  biological system and want to quantify whether imputation improves differential
  expression concordance with bulk.
when_to_use_negative:
- No matched bulk RNA-seq data exist for the same system, so no bulk DE reference
  set can be built
- The goal is to evaluate imputation quality for clustering, visualization, or pseudotime
  inference rather than differential expression
- Cells are unlabeled and the number of clusters is unknown, preventing the labeled
  or c(7, "Spectrum") Bfimpute configurations used here
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Runs differential expression on the bulk RNA-seq counts (top 200 reference
    genes) and is re-run on the raw and imputed scRNA-seq matrices for concordance
    testing
- name: Bfimpute
  role: Imputes scRNA-seq dropouts using cell-type labels or spectral clustering (7
    clusters), with CPM + log10(bias 1.01) normalization, producing the best DE-concordance
    AUC (0.670)
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Imputes scRNA-seq counts at recommended defaults as one of the comparison
    methods (AUC 0.665)
- name: SAVER
  role: Imputes scRNA-seq counts at recommended defaults as one of the comparison
    methods (AUC 0.624)
- name: VIPER
  role: Imputes scRNA-seq counts at recommended defaults as one of the comparison
    methods (AUC 0.639)
- name: DrImpute
  role: Imputes scRNA-seq counts at recommended defaults as one of the comparison
    methods (AUC 0.657)
- name: MAGIC
  role: Imputes scRNA-seq counts at recommended defaults as one of the comparison
    methods (AUC 0.668)
- name: Spectrum
  role: Provides spectral clustering into 7 clusters when Bfimpute runs without provided
    cell labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:23f647333708abcd3f26f9b33b7223cac0ae86a41eece0b601ecca6f06d6f078
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
  - DESeq2
  - DrImpute
  - MAGIC
  - SAVER
  - Spectrum
  - VIPER
  - scImpute
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# bulk-rnaseq-differential-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark imputation of scRNA-seq dropout events by measuring how well DE calls agree with a bulk RNA-seq reference: DESeq2 is run on bulk counts to pick the top 200 bulk DE genes, then the raw and six imputed scRNA-seq matrices are re-run through DESeq2 and concordance is summarized as an AUC over FDR thresholds.

## When to use

Use when you have matched scRNA-seq and bulk RNA-seq counts for the same biological system and want to quantify whether imputation improves differential expression concordance with bulk.

## When NOT to use

- No matched bulk RNA-seq data exist for the same system, so no bulk DE reference set can be built
- The goal is to evaluate imputation quality for clustering, visualization, or pseudotime inference rather than differential expression
- Cells are unlabeled and the number of clusters is unknown, preventing the labeled or c(7, "Spectrum") Bfimpute configurations used here

## Inputs

- sc_qc.csv: QC'd Chu et al. (2016) scRNA-seq count matrix (genes x cells)
- bulk_qc.csv: matched bulk RNA-seq count matrix
- cell_type.csv: cell-type label matrix for 7 cell types
- top 200 bulk DE genes from DESeq2 as the reference gene set

## Outputs

- DE gene lists per condition (raw + 6 imputed matrices) from DESeq2
- de_concordance_auc.csv: seven-condition AUC table (raw 0.568; Bfimpute 0.670; scImpute 0.665; SAVER 0.624; VIPER 0.639; DrImpute 0.657; MAGIC 0.668)

## How to apply

Load the QC'd Chu et al. (2016) files sc_qc.csv (scRNA-seq counts), bulk_qc.csv (bulk counts), and cell_type.csv (7 cell types). Run DESeq2 on the bulk counts to define the top 200 bulk DE genes as the reference set. Impute the scRNA-seq matrix with Bfimpute using ccluster = 'labeled' with label = cell_type (or spectral clustering c(7,"Spectrum")), letting Bfimpute apply CPM normalization and log10 transform with bias 1.01 (normalized = FALSE, ncores = 5), and with scImpute, SAVER, VIPER, DrImpute, and MAGIC at their recommended defaults. Re-run DESeq2 on the raw matrix and each imputed matrix, sweep FDR significance thresholds, and compute the AUC of concordance between scRNA-seq DE calls and the top-200 bulk reference. Rationale: dropouts in the single-cell matrix obscure DE signal, so agreement with bulk data serves as an external ground-truth proxy for DE fidelity.

## Related tools

- **DESeq2** (Runs differential expression on the bulk RNA-seq counts (top 200 reference genes) and is re-run on the raw and imputed scRNA-seq matrices for concordance testing)
- **Bfimpute** (Imputes scRNA-seq dropouts using cell-type labels or spectral clustering (7 clusters), with CPM + log10(bias 1.01) normalization, producing the best DE-concordance AUC (0.670)) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Imputes scRNA-seq counts at recommended defaults as one of the comparison methods (AUC 0.665))
- **SAVER** (Imputes scRNA-seq counts at recommended defaults as one of the comparison methods (AUC 0.624))
- **VIPER** (Imputes scRNA-seq counts at recommended defaults as one of the comparison methods (AUC 0.639))
- **DrImpute** (Imputes scRNA-seq counts at recommended defaults as one of the comparison methods (AUC 0.657))
- **MAGIC** (Imputes scRNA-seq counts at recommended defaults as one of the comparison methods (AUC 0.668))
- **Spectrum** (Provides spectral clustering into 7 clusters when Bfimpute runs without provided cell labels)

## Examples

```
counts = read.csv(paste0(data_dir, "sc_qc.csv"), row.names = 1, header = T); counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- The seven-condition AUC table matches the expected values: raw 0.568; Bfimpute 0.670; scImpute 0.665; SAVER 0.624; VIPER 0.639; DrImpute 0.657; MAGIC 0.668
- Every imputation method's AUC exceeds the raw-matrix AUC of 0.568, confirming that all imputations improved bulk concordance
- The bulk DE reference set contains exactly the top 200 genes ranked by DESeq2 on bulk_qc.csv
- Bfimpute was run with normalized = FALSE so the output reflects CPM normalization and log10 transform with bias 1.01, not pre-normalized input
- de_concordance_auc.csv exists with one row per condition (7 rows) and an AUC column over the FDR threshold sweep

## Limitations

- There is no consensus about which imputation method is the best; performance varies across evaluation criteria, datasets, and downstream analyses
- It is premature to conclude Bfimpute will achieve better performance across all datasets and improve downstream analysis
- Depending on the application, users may need to constrain parameters to maximize true positives or minimize false positives in downstream analysis

## Evidence

- [results] AUC results for DE concordance: [evidence span withheld — anchor status: not_found]
- [methods] Loading the matched data matrices: "you can load the cell types, bulk and scRNAseq counts matrices as followed"
- [readme] Bfimpute normalization behavior: "If `FALSE`, Bfimpute will perform CPM and log10 transform with bias 1.01. If `TRUE`, Bfimpute will not perform CPM or log10 transform."
- [readme] Chu et al. (2016) dataset source: "In this section we will run the dataset from [Chu. et al (2016)](https://link.springer.com/article/10.1186/s13059-016-1033-x) and the version with QC can be downloaded"
- [intro] Motivation for improving DE analysis: "it is important to design new imputation methods to further improve the analysis of cell-to-cell relationships or DE that takes into account cell variability"
