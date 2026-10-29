---
name: de-concordance-auc-evaluation
description: Use when you have paired scRNA-seq and bulk RNA-seq counts for the same
  sample/condition and need to quantify whether imputation improves DE gene concordance
  with bulk, comparing multiple imputation methods.
when_to_use_negative:
- No paired bulk RNA-seq data exist for the same biological system, so no bulk DE
  reference set can be built
- The goal is to evaluate imputation quality for clustering, trajectory, or pseudotime
  rather than differential expression consistency
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: DESeq2
  role: Run on bulk counts to define the top-200 bulk DE reference set and on raw/imputed
    scRNA-seq matrices to call DE genes at each FDR threshold
- name: Bfimpute
  role: Impute the scRNA-seq counts using cell labels or 7-cluster spectral clustering;
    produces the best DE-concordance AUC (0.670)
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Impute counts with recommended defaults as a comparator in the AUC benchmark
    (0.665)
- name: SAVER
  role: Impute counts with recommended defaults as a comparator in the AUC benchmark
    (0.624)
- name: VIPER
  role: Impute counts with recommended defaults as a comparator in the AUC benchmark
    (0.639)
- name: DrImpute
  role: Impute counts with recommended defaults as a comparator in the AUC benchmark
    (0.657)
- name: MAGIC
  role: Impute counts with recommended defaults as a comparator in the AUC benchmark
    (0.668)
- name: Spectrum
  role: Spectral clustering into 7 clusters when Bfimpute is run without cell type
    labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:b0c3de5a715f42ad845927b3f5f61b6803dfc7ff68c5fc068ca1b05ba9f5352a
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# de-concordance-auc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scRNA-seq imputation methods by measuring how well differential expression calls agree with bulk RNA-seq DE genes, summarized as an AUC across FDR significance thresholds. Applied to the Chu et al. (2016) dataset to compare Bfimpute, scImpute, SAVER, VIPER, DrImpute, and MAGIC.

## When to use

Use when you have paired scRNA-seq and bulk RNA-seq counts for the same sample/condition and need to quantify whether imputation improves DE gene concordance with bulk, comparing multiple imputation methods.

## When NOT to use

- No paired bulk RNA-seq data exist for the same biological system, so no bulk DE reference set can be built
- The goal is to evaluate imputation quality for clustering, trajectory, or pseudotime rather than differential expression consistency

## Inputs

- sc_qc.csv — QC'd Chu et al. (2016) scRNA-seq count matrix (genes x cells)
- bulk_qc.csv — bulk RNA-seq count matrix for the same sample
- cell_type.csv — cell-type label matrix for the 7 Chu cell types
- Imputed expression matrices from Bfimpute, scImpute, SAVER, VIPER, DrImpute, and MAGIC

## Outputs

- de_concordance_auc.csv — AUC table with one concordance-AUC value per condition (raw and six imputed matrices)
- Top-200 bulk DE gene reference set from DESeq2
- scRNA-seq DE call sets per method across the FDR threshold sweep

## How to apply

Download the QC'd Chu et al. (2016) files (sc_qc.csv, bulk_qc.csv, cell_type.csv covering 7 cell types) and run DESeq2 on the bulk counts to select the top 200 bulk DE genes as the reference set. Impute the scRNA-seq counts with Bfimpute using cell labels (ccluster = "labeled", label = cell_type, ncores = 5) or spectral clustering with 7 clusters (c(7,"Spectrum")), letting Bfimpute apply CPM normalization and log10 transform with bias 1.01 (normalized = FALSE). Impute the same counts with scImpute, SAVER, VIPER, DrImpute, and MAGIC at their recommended defaults. For the raw matrix and each imputed matrix, rerun DESeq2 and sweep a range of FDR significance thresholds, computing at each threshold the concordance of scRNA-seq DE calls with the top-200 bulk reference. Integrate concordance over the threshold sweep to obtain one AUC per condition; Bfimpute should rank highest (AUC 0.670 vs raw 0.568, MAGIC 0.668, scImpute 0.665, DrImpute 0.657, VIPER 0.639, SAVER 0.624).

## Related tools

- **DESeq2** (Run on bulk counts to define the top-200 bulk DE reference set and on raw/imputed scRNA-seq matrices to call DE genes at each FDR threshold)
- **Bfimpute** (Impute the scRNA-seq counts using cell labels or 7-cluster spectral clustering; produces the best DE-concordance AUC (0.670)) — https://github.com/maiziezhoulab/Bfimpute
- **scImpute** (Impute counts with recommended defaults as a comparator in the AUC benchmark (0.665))
- **SAVER** (Impute counts with recommended defaults as a comparator in the AUC benchmark (0.624))
- **VIPER** (Impute counts with recommended defaults as a comparator in the AUC benchmark (0.639))
- **DrImpute** (Impute counts with recommended defaults as a comparator in the AUC benchmark (0.657))
- **MAGIC** (Impute counts with recommended defaults as a comparator in the AUC benchmark (0.668))
- **Spectrum** (Spectral clustering into 7 clusters when Bfimpute is run without cell type labels)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- Raw (unimputed) counts should yield the lowest concordance AUC (~0.568); every imputation method should raise the AUC above this baseline
- Bfimpute should produce the highest AUC (0.670), closely followed by MAGIC (0.668) and scImpute (0.665)
- The AUC table must contain exactly seven conditions: raw plus the six imputed methods
- The bulk reference set must consistently contain exactly the top 200 DESeq2 genes across all comparisons
- de_concordance_auc.csv is produced with AUC values integrated over the FDR threshold sweep, not a single-threshold concordance

## Limitations

- No single best imputation method could be identified across criteria; performance varies with dataset, protocol, and downstream analysis
- Generalizability of the ranking beyond the benchmarked datasets is unconfirmed
- Imputation may increase false positives in downstream DE analysis; parameters may need tuning to maximize true positives or minimize false positives

## Evidence

- [other] All imputation methods improved DE concordance with bulk data, with Bfimpute generating DE genes most consistent with the bulk data (AUC: raw 0.568; Bfimpute 0.670; scImpute 0.665; SAVER 0.624; VIPER 0.639; DrImpute 0.657; MAGIC 0.668).: [evidence span withheld — anchor status: not_found]
