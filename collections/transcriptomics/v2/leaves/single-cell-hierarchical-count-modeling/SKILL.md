---
name: single-cell-hierarchical-count-modeling
description: Use when a sparse droplet-based scRNA-seq/snRNA-seq count matrix (genes
  x cells, integer counts) has multi-subject structure (e.
when_to_use_negative:
- Data are bulk RNA-seq with small sample sizes — NEBULA is designed for large-scale
  multi-subject single-cell hierarchies, and existing bulk tools suffice
- Counts are already normalized/non-integer values (nebula requires integer count
  matrices)
- Genes are lowly expressed or have estimated cell-level overdispersion > 100 — co-expression
  analysis has little statistical power for low-expression genes and results should
  be removed or interpreted cautiously
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: nebula
  role: Core R package (v1.5.8) providing nebula, group_cell, nbresidual, and scToNeb
    for fitting NBGMMs to grouped single-cell counts
  repo: https://github.com/lhe17/nebula
- name: NEBULA-LN
  role: Fast large-sample approximation algorithm for NBGMM fitting, used as the first-pass
    fit resolving ~90% (CPS~700) to ~80% (CPS~380) of genes
  repo: https://github.com/lhe17/nebula
- name: NEBULA-HL
  role: H-likelihood refit algorithm reserved for genes where NEBULA-LN is unreliable
    (CPS < 30 or small overdispersion estimates)
  repo: https://github.com/lhe17/nebula
- name: R
  role: Runtime environment; nebula requires R-4.4.0+ from version 1.5.7 and Rcpp-1.0.7
- name: devtools
  role: Used to install nebula from GitHub (install_github("lhe17/nebula"))
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:340f4a9a7de4c51b72b01e39d14a94acc9fc9e3c07d64255560d3e65cc8bac69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - NEBULA-HL
  - NEBULA-LN
  - R
  - devtools
  - nebula
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-hierarchical-count-modeling

## Summary

Fit gene-wise negative binomial gamma mixed models (NBGMM) to multi-subject single-cell counts, decomposing overdispersion into subject-level (sigma^2) and cell-level (phi^-1) components using NEBULA-LN or NEBULA-HL. This enables differential and co-expression testing on droplet-based scRNA-seq data with thousands of cells grouped into tens of subjects.

## When to use

Use when a sparse droplet-based scRNA-seq/snRNA-seq count matrix (genes x cells, integer counts) has multi-subject structure (e.g. 30 subjects) and you need gene-level association testing (marker genes, treatment effects, eQTL-like predictors) or cell-level co-expression while accounting for between-subject vs within-subject overdispersion.

## When NOT to use

- Data are bulk RNA-seq with small sample sizes — NEBULA is designed for large-scale multi-subject single-cell hierarchies, and existing bulk tools suffice
- Counts are already normalized/non-integer values (nebula requires integer count matrices)
- Genes are lowly expressed or have estimated cell-level overdispersion > 100 — co-expression analysis has little statistical power for low-expression genes and results should be removed or interpreted cautiously

## Inputs

- Gene x cell integer count matrix (matrix or dgCMatrix, as in Seurat)
- Per-cell subject ID vector (character or numeric, length N)
- Predictor data frame converted to a design matrix via model.matrix (intercept required)
- Per-cell scaling factor offsets (length-N positive vector, e.g. library size normalization)
- Seurat or SingleCellExperiment object (via scToNeb) or the packaged sample_data

## Outputs

- Per-gene summary table with logFC estimates, standard errors, raw p-values, and convergence codes
- Per-gene subject-level overdispersion (sigma^2) and cell-level overdispersion (phi^-1) estimates
- Optionally, covariance matrix of estimated logFC (covariance=TRUE) for testing contrasts
- Marginal and conditional Pearson residuals via nbresidual

## How to apply

Cells must be consecutively grouped by subject IDs using group_cell before fitting, since the count matrix passed to nebula must be an M genes by N cells integer matrix (dense or dgCMatrix). Build the design matrix with model.matrix including an intercept and unique column names, and supply per-cell scaling factors via the offset argument (a length-N vector of positive values). Choose NEBULA-LN (large-sample analytical approximation, orders of magnitude faster) or NEBULA-HL (h-likelihood); under the recommended strategy for fixed-effect testing, fit NEBULA-LN first and route genes with CPS*phi_hat < 20, CPS < 30, or small sigma2_hat/phi_hat ratios to NEBULA-HL re-estimation (kappa=200 for cell-level predictors, kappa=800 for subject-level predictors). Check convergence codes per gene — values <= -20 indicate non-convergence — and filter low-expression genes with cpc=0.005 (counts per cell < 0.5%). Validate the fit by confirming the summary contains logFC, p-values, and both sigma^2 and phi^-1 overdispersion estimates, and remove genes with cell-level overdispersion > 100 (for >= 50 cells/subject) or subject-level overdispersion > 1 when testing subject-level variables.

## Related tools

- **nebula** (Core R package (v1.5.8) providing nebula, group_cell, nbresidual, and scToNeb for fitting NBGMMs to grouped single-cell counts) — https://github.com/lhe17/nebula
- **NEBULA-LN** (Fast large-sample approximation algorithm for NBGMM fitting, used as the first-pass fit resolving ~90% (CPS~700) to ~80% (CPS~380) of genes) — https://github.com/lhe17/nebula
- **NEBULA-HL** (H-likelihood refit algorithm reserved for genes where NEBULA-LN is unreliable (CPS < 30 or small overdispersion estimates)) — https://github.com/lhe17/nebula
- **R** (Runtime environment; nebula requires R-4.4.0+ from version 1.5.7 and Rcpp-1.0.7)
- **devtools** (Used to install nebula from GitHub (install_github("lhe17/nebula")))

## Examples

```
re = nebula(sample_data$count, sample_data$sid, pred=df, ncore=1)
```

## Evaluation signals

- Every gene receives a convergence code from nebula; codes <= -20 flag non-convergence and require cautious interpretation or removal
- With the LN-first dispatch strategy, the fraction of genes (with CPC > 0.1%) resolved by NEBULA-LN alone should match reported rates: ~90% at CPS ~700 (excitatory neurons) and ~80% at CPS ~380 (oligodendrocytes)
- Reported summary must contain both overdispersion components (sigma^2 and phi^-1) per gene alongside logFC and p-values
- Genes with estimated cell-level overdispersion > 100 (datasets with >= 50 cells/subject) and subject-level overdispersion > 1 (subject-level predictors) are excluded from downstream calls
- Low-expression filtering with cpc=0.005 must remove counts-per-cell < 0.5% genes before fitting

## Limitations

- NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion compared to a standard h-likelihood method, while NEBULA-HL has downward bias in estimating the subject-level overdispersion
- Misspecification of the random-effects distribution may inflate false-positive rates when testing a subject-level predictor with few subjects
- Co-expression analysis has little statistical power to identify low-expression genes, and no observed correlation does not imply two genes are not co-expressed
- Installation requires Rcpp-1.0.7 and R-4.4.0+ (v1.5.7+); Rfast requires GSL, and Mac users need the LLVM compiler for source compilation
- Applicability to other single-cell modalities such as scATAC-seq is expected but not yet demonstrated

## Evidence

- [intro] decompose overdispersion into subject and cell levels: "Owing to the hierarchical nature of multi-subject scRNA-seq data, it is appropriate to decompose the overdispersion into the subject and cell levels."
- [readme] two fitting algorithms: "In *nebula*, a user can choose one of the two algorithms to fit an NBGMM. NEBULA-LN uses an approximated likelihood based on the law of large numbers, and NEBULA-HL uses an h-likelihood."
- [readme] grouping by subjects: "The cells in the count matrix need to be grouped by the subjects (that is, the cells of the same subject should be placed consecutively) before using as the input to the function nebula"
- [methods] overdispersion QC cutoffs: "genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject."
- [methods] convergence QC: "Generally, a convergence code $≤$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
