---
name: negative-binomial-mixed-model-fitting
description: Use when analyzing droplet-based multi-subject scRNA-seq/snRNA-seq count
  matrices (e.g., thousands of cells across dozens of subjects) where per-gene tests
  must account for hierarchical overdispersion, or when glmer.nb-style NB mixed models
  are too slow for genome-scale testing (~16,000 genes).
when_to_use_negative:
- Input is already aggregated to pseudo-bulk/bulk per-subject profiles where subject-level
  random effects are not needed — a standard bulk negative binomial model suffices
- Data are single-subject or lack a multi-subject hierarchical structure, so the subject/cell
  overdispersion decomposition of the NBMM is unnecessary
- The gene set consists of very lowly expressed genes (counts per cell far below the
  cpc=0.005 cutoff), which the co-expression analysis has little statistical power
  to test
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: nebula (R package v1.5.8)
  role: Primary R package providing nebula (NBGMM fitting), group_cell (reorder cells
    by subject), scToNeb (Seurat/SCE input), and nbresidual (Pearson residuals)
  repo: https://github.com/lhe17/nebula
- name: R (R-4.4.0)
  role: Runtime environment; nebula v1.5.7+ requires R-4.4.0+
  repo: https://github.com/lhe17/nebula
- name: devtools
  role: Install the nebula package from GitHub via install_github('lhe17/nebula')
  repo: https://github.com/lhe17/nebula
- name: Rcpp 1.0.7
  role: Compilation dependency required for installing nebula
- name: Rfast
  role: Dependency installed during nebula installation; requires GSL in the environment
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  - task_006
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:ca560a7d08b8fa3ab2cf6d9daea414954a61bd679c51f34d8cc8722cbfe19b39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - NEBULA
  - NEBULA-HL
  - NEBULA-LN
  - R
  - R (R-4.4.0)
  - Rcpp
  - Rcpp 1.0.7
  - Rfast
  - devtools
  - nebula (R package v1.5.8)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# negative-binomial-mixed-model-fitting

## Summary

Fit negative binomial gamma mixed models (NBGMM) per gene to multi-subject single-cell count data using NEBULA, which decomposes overdispersion into subject-level (sigma^2) and cell-level (phi^-1) components, enabling fast differential expression, marker-gene, and cell-level co-expression testing across tens of thousands of cells.

## When to use

Use when analyzing droplet-based multi-subject scRNA-seq/snRNA-seq count matrices (e.g., thousands of cells across dozens of subjects) where per-gene tests must account for hierarchical overdispersion, or when glmer.nb-style NB mixed models are too slow for genome-scale testing (~16,000 genes).

## When NOT to use

- Input is already aggregated to pseudo-bulk/bulk per-subject profiles where subject-level random effects are not needed — a standard bulk negative binomial model suffices
- Data are single-subject or lack a multi-subject hierarchical structure, so the subject/cell overdispersion decomposition of the NBMM is unnecessary
- The gene set consists of very lowly expressed genes (counts per cell far below the cpc=0.005 cutoff), which the co-expression analysis has little statistical power to test

## Inputs

- Raw integer count matrix (genes x cells; matrix or sparse dgCMatrix, same format as Seurat)
- Per-cell subject ID vector (character or numeric, length = number of cells)
- Per-cell predictor data frame used to build the design matrix (e.g., case/control, cell-level covariates like total features, ribosomal-gene percentage)
- Per-cell library-size scaling factors for the offset argument
- Seurat or SingleCellExperiment objects (via scToNeb)

## Outputs

- Per-gene summary statistics: logFC, standard errors, raw p-values (not multiple-testing adjusted), subject-level overdispersion sigma^2, cell-level overdispersion phi^-1, and convergence codes
- Covariance matrix of estimated log(FC) when covariance=TRUE, for testing contrasts
- Marginal and conditional Pearson residuals via nbresidual
- Elapsed wall-clock runtime (e.g., ~40 min for ~16,000 genes in 34,000 excitatory neurons vs ~67 h for glmer.nb)

## How to apply

Load the raw integer count matrix (genes x cells, dgCMatrix) with per-cell subject IDs, reorder cells so those of the same subject are consecutive via group_cell (or use scToNeb to extract from Seurat/SingleCellExperiment), and build the design matrix with model.matrix, ensuring the intercept is included and columns have unique names. Run nebula with method='LN' (NEBULA-LN large-sample approximation; NEBULA-HL is automatically invoked when average counts per subject are <3), default kappa=800 for subject-level predictors, cpc=0.005 to filter low-expression genes, offset set to per-cell library size (e.g., colSums of the raw counts), and ncore set to available logical cores. The default NBGMM returns per-gene logFC, raw p-values, subject-level (sigma^2) and cell-level (phi^-1) overdispersions, and convergence codes; set covariance=TRUE to test contrasts via the covariance matrix of the estimated log(FC). Apply QC: interpret results with caution for convergence codes <= -20, remove genes with cell-level overdispersion >100 (for datasets with >=50 cells per subject), and for subject-level predictors remove or flag genes with subject-level overdispersion >1.

## Related tools

- **nebula (R package v1.5.8)** (Primary R package providing nebula (NBGMM fitting), group_cell (reorder cells by subject), scToNeb (Seurat/SCE input), and nbresidual (Pearson residuals)) — https://github.com/lhe17/nebula
- **R (R-4.4.0)** (Runtime environment; nebula v1.5.7+ requires R-4.4.0+) — https://github.com/lhe17/nebula
- **devtools** (Install the nebula package from GitHub via install_github('lhe17/nebula')) — https://github.com/lhe17/nebula
- **Rcpp 1.0.7** (Compilation dependency required for installing nebula)
- **Rfast** (Dependency installed during nebula installation; requires GSL in the environment)

## Examples

```
library(nebula); data(sample_data); df = model.matrix(~X1+X2+cc, data=sample_data$pred); re = nebula(sample_data$count, sample_data$sid, pred=df, cpc=0.005, ncore=1)
```

## Evaluation signals

- All retained genes have convergence codes > -20 (codes <= -20 indicate non-convergence and require cautious interpretation or removal)
- Genes with estimated cell-level overdispersion >100 are removed; genes with subject-level overdispersion >1 are removed or flagged when testing subject-level variables
- Runtime should be orders of magnitude faster than glmer.nb (e.g., ~40 minutes vs ~67 hours for ~16,000 genes)
- Design matrix includes an intercept and columns with unique variable names; cells are grouped consecutively by subject before calling nebula
- In false-positive simulations, empirical FPR stays near the nominal 0.05 level as subject-level overdispersion (sigma^2) increases from 0 to 0.1, unlike naive glm.nb

## Limitations

- NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion compared to a standard h-likelihood method
- NEBULA-HL shows a downward bias in estimating the subject-level overdispersion
- Misspecification of the random-effects distribution may inflate false-positive rates when testing a subject-level predictor with few subjects
- Co-expression analysis has little statistical power to identify low-expression genes; absence of a detected correlation does not mean two genes are not co-expressed
- Convergence codes <= -20 mean the algorithm did not converge well and results must be interpreted with caution
- Testing a subject-level variable is highly sensitive to estimation of the subject-level variance component and random-effects distribution assumptions when the number of subjects is small

## Evidence

- [readme] The cells in the count matrix need to be grouped by the subjects (that is, the cells of the same subject should be placed consecutively) before using as the input to the function nebula: "The cells in the count matrix need to be grouped by the subjects (that is, the cells of the same subject should be placed consecutively) before using as the input to the function nebula"
- [methods] Filtering out low-expressed genes can be specified by cpc=0.005 (i.e., counts per cell<0.5%).: "Filtering out low-expressed genes can be specified by cpc=0.005 (i.e., counts per cell<0.5%). The argument cpc is defined by the ratio between the total count of the gene and the number of cells."
- [methods] Generally, a convergence code <= -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases.: "Generally, a convergence code $\leq$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
- [methods] genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject.: "genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject."
- [methods] NEBULA-LN uses an approximated likelihood based on the law of large numbers, and NEBULA-HL uses an h-likelihood.: "In *nebula*, a user can choose one of the two algorithms to fit an NBGMM. NEBULA-LN uses an approximated likelihood based on the law of large numbers, and NEBULA-HL uses an h-likelihood."
- [methods] The installation also requires Rcpp-1.0.7 and has been tested on R-4.4.0.: "The installation also requires Rcpp-1.0.7 and has been tested on R-4.4.0."
