---
name: negative-binomial-mixed-model-overdispersion-estimation
description: Use when analyzing large-scale multi-subject droplet scRNA-seq/snRNA-seq
  count data (e.g., human frontal cortex snRNA-seq, PBMCs) where hierarchical overdispersion
  must be decomposed into between-subject and within-subject components per gene.
when_to_use_negative:
- Bulk RNA-seq or small-sample-size data with few subjects, where conventional transcriptomics
  methods are adequate and large-sample approximations do not hold
- Data where cells cannot be grouped by subject (no multi-subject structure), since
  the NBMM between-subject component is undefined
- Genes with estimated cell-level overdispersion > 100 (for ≥50 cells per subject)
  or subject-level overdispersion > 1 when testing subject-level variables — these
  should be removed or interpreted cautiously rather than modeled
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: nebula
  role: R package (v1.5.8) providing nebula, group_cell, nbresidual, and scToNeb to
    fit NBGMMs and extract overdispersion estimates and summary statistics
  repo: https://github.com/lhe17/nebula
- name: NEBULA-LN
  role: large-sample-approximation algorithm used as the fast default for fitting
    the NBGMM per gene
  repo: https://github.com/lhe17/nebula
- name: NEBULA-HL
  role: h-likelihood algorithm reserved for re-estimation on genes where NEBULA-LN
    is unreliable (CPS*phi_hat < 20, CPS < 30, or partial-refit branch)
  repo: https://github.com/lhe17/nebula
- name: R
  role: runtime environment (R-4.4.0+ required from nebula v1.5.7, with Rcpp-1.0.7)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:66c89bac3234b4da5059ae8bae81ec6b9f211e0ebfe69a82a4bcbccb4fa54e34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - NEBULA-HL
  - NEBULA-LN
  - R
  - nebula
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# negative-binomial-mixed-model-overdispersion-estimation

## Summary

Fit per-gene negative binomial gamma mixed models (NBGMM) with NEBULA to decompose overdispersion in multi-subject single-cell data into subject-level (σ²) and cell-level (φ⁻¹) components, choosing between the fast large-sample-approximation algorithm (NEBULA-LN) and the h-likelihood algorithm (NEBULA-HL).

## When to use

Use when analyzing large-scale multi-subject droplet scRNA-seq/snRNA-seq count data (e.g., human frontal cortex snRNA-seq, PBMCs) where hierarchical overdispersion must be decomposed into between-subject and within-subject components per gene.

## When NOT to use

- Bulk RNA-seq or small-sample-size data with few subjects, where conventional transcriptomics methods are adequate and large-sample approximations do not hold
- Data where cells cannot be grouped by subject (no multi-subject structure), since the NBMM between-subject component is undefined
- Genes with estimated cell-level overdispersion > 100 (for ≥50 cells per subject) or subject-level overdispersion > 1 when testing subject-level variables — these should be removed or interpreted cautiously rather than modeled

## Inputs

- genes-by-cells count matrix (matrix or sparse dgCMatrix, integer elements) e.g. sample_data$count
- per-cell subject ID vector (character or numeric), e.g. sample_data$sid
- design matrix of predictors built with model.matrix including intercept, e.g. ~X1+X2+cc
- per-cell scaling factor vector passed to offset
- cells-per-subject (CPS) value for the analyzed cell type

## Outputs

- per-gene summary statistics: logFC for each design column, standard errors, p-values (raw, unadjusted)
- per-gene subject-level overdispersion (σ²) and cell-level overdispersion (φ⁻¹) estimates
- per-gene convergence codes from the estimation algorithm for quality control
- optional covariance matrix of estimated log(FC) when covariance=TRUE
- marginal and conditional Pearson residuals via nbresidual

## How to apply

Group cells by subject IDs (using group_cell so cells of the same subject are consecutive) and build a design matrix with model.matrix, ensuring the intercept is included. Call nebula on the genes-by-cells integer/dgCMatrix count matrix, choosing NEBULA-LN (approximated likelihood via law of large numbers, faster, some asymptotic efficiency sacrificed for cell-level overdispersion) or NEBULA-HL (h-likelihood, more reliable for low cell counts but downward-biased for subject-level overdispersion). Apply the recommended dispatch strategy: use NEBULA-LN first, then refit with NEBULA-HL when CPS*φ_hat < 20 or CPS < 30 (full refit of both overdispersions), or fix φ_hat and re-estimate only σ² when φ_hat < kappa*(1+c_hat²)/CPS and σ²_hat < 8*(1+c_hat²)/(CPS*phi_hat), with kappa=200 for cell-level predictors and kappa=800 for subject-level predictors. Specify per-cell scaling factors via the offset argument and low-expression filtering via cpc (e.g., cpc=0.005, counts per cell < 0.5%). Check the returned per-gene convergence codes for QC (code ≤ -20 indicates non-convergence) and interpret those genes with caution.

## Related tools

- **nebula** (R package (v1.5.8) providing nebula, group_cell, nbresidual, and scToNeb to fit NBGMMs and extract overdispersion estimates and summary statistics) — https://github.com/lhe17/nebula
- **NEBULA-LN** (large-sample-approximation algorithm used as the fast default for fitting the NBGMM per gene) — https://github.com/lhe17/nebula
- **NEBULA-HL** (h-likelihood algorithm reserved for re-estimation on genes where NEBULA-LN is unreliable (CPS*phi_hat < 20, CPS < 30, or partial-refit branch)) — https://github.com/lhe17/nebula
- **R** (runtime environment (R-4.4.0+ required from nebula v1.5.7, with Rcpp-1.0.7))

## Examples

```
library(nebula); data(sample_data); df = model.matrix(~X1+X2+cc, data=sample_data$pred); re = nebula(sample_data$count, sample_data$sid, pred=df, ncore=1)
```

## Evaluation signals

- Every gene receives exactly one method assignment (NEBULA-LN alone, full NEBULA-HL refit, or partial σ²-only refit) in the dispatch table
- All genes with CPS < 30 are routed to NEBULA-HL, consistent with nebula forcing method='HL' when CPS < 30
- Fraction of genes resolved by NEBULA-LN alone matches the reported rates: ~90% for excitatory neurons (CPS ≈ 700) and ~80% for oligodendrocytes (CPS ≈ 380) among genes with CPC > 0.1%
- Convergence QC: no genes with convergence code ≤ -20 are interpreted without caution flagging
- Returned summary contains both overdispersion components (σ² and φ⁻¹) and raw p-values for every analyzed gene

## Limitations

- NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion compared to a standard h-likelihood method
- NEBULA-HL shows downward bias in estimating the subject-level overdispersion, in line with theoretical results
- Testing subject-level variables is highly sensitive to estimation of the subject-level variance component and random-effects distribution assumptions when the number of subjects is small; misspecification may inflate false positives
- Convergence code ≤ -20 indicates the algorithm does not converge well and results should be interpreted with caution
- Genes with estimated cell-level overdispersion > 100 (datasets with ≥50 cells/subject) or subject-level overdispersion > 1 (subject-level predictors) should be removed or interpreted cautiously

## Evidence

- [intro] Decomposition rationale: "Owing to the hierarchical nature of multi-subject scRNA-seq data, it is appropriate to decompose the overdispersion into the subject and cell levels."
- [methods] Algorithm choice: "In nebula, a user can choose one of the two algorithms to fit an NBGMM. NEBULA-LN uses an approximated likelihood based on the law of large numbers, and NEBULA-HL uses an h-likelihood."
- [methods] Convergence QC: "Generally, a convergence code $\leq$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
- [discussion] LN efficiency tradeoff: "Compared to a standard h-likelihood method, we see that NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion"
- [discussion] HL bias: "The downward bias of NEBULA-HL in estimating the subject-level overdispersion is in line with theoretical results"
- [intro] Subject-level sensitivity: "we found that testing a subject-level variable was highly sensitive to the estimation of the subject-level variance component and the assumption of the distribution of the random effects"
- [readme] Grouping requirement: "The cells in the count matrix need to be grouped by the subjects (that is, the cells of the same subject should be placed consecutively) before using as the input to the function"
- [methods] Overdispersion filter thresholds: "genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject."
