---
name: cells-per-subject-qc-evaluation
description: Decision support for choosing between NEBULA-LN and NEBULA-HL per gene
  in multi-subject snRNA-seq/scRNA-seq data, when cells-per-subject counts (e.g.,
  CPS ~380–700) may make the large-sample LN approximation unreliable for some genes.
when_to_use_negative:
- The data are bulk RNA-seq or single-subject single-cell data with no hierarchical
  subject structure — the CPS-based dispatch assumes multi-subject droplet-based single-cell
  data with per-subject cell counts.
- Every gene will be fit with NEBULA-HL regardless (e.g., very small subjects-per-data-set
  designs with CPS < 30 genome-wide) — the dispatch rule is then trivial and NEBULA-LN
  estimates need not be computed first.
- The task is filtering low-expression genes (cpc=0.005 threshold) or post-fit QC
  of convergence codes rather than model-choice dispatch between NEBULA-LN and NEBULA-HL.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: nebula (R package v1.5.8)
  role: Fits the NBGMM per gene; provides NEBULA-LN for the fast large-sample-approximation
    branch and NEBULA-HL for the h-likelihood refit branches; forces method='HL' when
    CPS < 30
  repo: https://github.com/lhe17/nebula
- name: R (R-4.4.0)
  role: Runtime environment required by nebula v1.5.7+; dispatch logic and dispatch_table.csv
    generation are implemented in R
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:7c90140145ea3a360ecb4242abbdbd93fb666174da2a6ec062dc0e8176093703
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - NEBULA-HL
  - NEBULA-LN
  - R
  - R (R-4.4.0)
  - nebula (R package v1.5.8)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cells-per-subject-qc-evaluation

## Summary

A decision rule for evaluating cells-per-subject (CPS) adequacy before fitting NEBULA's negative binomial gamma mixed model, dispatching each gene between NEBULA-LN (fast large-sample approximation) and NEBULA-HL (h-likelihood) based on estimated overdispersion components. This preserves the paper's recommended strategy in which ~90% and ~80% of genes (CPC > 0.1%) are resolved by NEBULA-LN alone at CPS ~700 and ~380.

## When to use

Decision support for choosing between NEBULA-LN and NEBULA-HL per gene in multi-subject snRNA-seq/scRNA-seq data, when cells-per-subject counts (e.g., CPS ~380–700) may make the large-sample LN approximation unreliable for some genes.

## When NOT to use

- The data are bulk RNA-seq or single-subject single-cell data with no hierarchical subject structure — the CPS-based dispatch assumes multi-subject droplet-based single-cell data with per-subject cell counts.
- Every gene will be fit with NEBULA-HL regardless (e.g., very small subjects-per-data-set designs with CPS < 30 genome-wide) — the dispatch rule is then trivial and NEBULA-LN estimates need not be computed first.
- The task is filtering low-expression genes (cpc=0.005 threshold) or post-fit QC of convergence codes rather than model-choice dispatch between NEBULA-LN and NEBULA-HL.

## Inputs

- Per-gene NEBULA-LN fitted estimates (subject-level sigma2_hat, cell-level phi_hat, c_hat) from the overdispersion decomposition output
- Cells-per-subject (CPS) count for the analyzed cell type
- Gene expression count matrix grouped by subject IDs (group_cell output, dgCMatrix or matrix) used with nebula
- Indicator of whether the predictor of interest is subject-level or cell-level (sets kappa to 800 vs 200)

## Outputs

- Per-gene dispatch table (dispatch_table.csv) with columns: gene ID, selected method (NEBULA-LN, NEBULA-HL full refit, or NEBULA-HL partial refit), and dispatch branch
- Gene subset flagged for NEBULA-HL re-estimation of sigma2 only (phi_hat fixed) or of both overdispersion components

## How to apply

Load the per-gene NEBULA-LN fitted estimates — subject-level sigma2_hat, cell-level phi_hat, and c_hat — together with the cells-per-subject (CPS) value for the analyzed cell type. For each gene, first evaluate the full-refit branch: if CPS*phi_hat < 20 or CPS < 30, re-estimate both overdispersions with NEBULA-HL (consistent with nebula forcing method='HL' when CPS < 30). Otherwise, evaluate the partial-refit branch: if phi_hat < kappa*(1+c_hat^2)/CPS and sigma2_hat < 8*(1+c_hat^2)/(CPS*phi_hat), fix phi_hat and re-estimate only sigma2 with NEBULA-HL, using kappa=200 when testing cell-level predictors and kappa=800 (the default) when testing subject-level predictors. Assign all remaining genes to the NEBULA-LN estimates, since NEBULA-LN's large-sample approximation is adequate when per-subject cell counts and the estimated overdispersion structure support it, and writing a full h-likelihood refit for every gene would be needlessly slow on large droplet-based data sets.

## Related tools

- **nebula (R package v1.5.8)** (Fits the NBGMM per gene; provides NEBULA-LN for the fast large-sample-approximation branch and NEBULA-HL for the h-likelihood refit branches; forces method='HL' when CPS < 30) — https://github.com/lhe17/nebula
- **R (R-4.4.0)** (Runtime environment required by nebula v1.5.7+; dispatch logic and dispatch_table.csv generation are implemented in R)

## Examples

```
re = nebula(sample_data$count, sample_data$sid, pred=df, ncore=1)
```

## Evaluation signals

- Every gene in the dispatch table receives exactly one method assignment (NEBULA-LN, NEBULA-HL full refit, or NEBULA-HL partial refit) — no gene is unassigned or double-assigned.
- All genes with CPS < 30 are routed to NEBULA-HL, matching nebula's forcing of method='HL' when CPS < 30.
- The fraction of genes with CPC > 0.1% resolved by NEBULA-LN alone is consistent with the paper's reported rates (~90% for excitatory neurons at CPS ~700 and ~80% for oligodendrocytes at CPS ~380).
- kappa is set to 200 for cell-level predictors and 800 (the default) for subject-level predictors in the partial-refit branch condition phi_hat < kappa*(1+c_hat^2)/CPS.
- dispatch_table.csv contains the three required columns (gene ID, selected method, dispatch branch) with one row per analyzed gene.

## Limitations

- The reported NEBULA-LN-only resolution rates (~90% at CPS ~700, ~80% at CPS ~380) apply to genes with CPC > 0.1%; the co-expression analysis has little statistical power to identify low-expression genes, so the rule is not calibrated for very low-expression genes.
- NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion compared to a standard h-likelihood method, so genes with small phi_hat (poorly estimated cell-level overdispersion) must be diverted to the partial-refit branch.
- NEBULA-HL exhibits a downward bias in estimating the subject-level overdispersion, which is why it is reserved for genes where NEBULA-LN is unreliable rather than used universally.
- Downstream QC still applies: genes with cell-level overdispersion >100 should be removed for data sets with at least 50 cells per subject, and a convergence code ≤ -20 indicates non-convergence requiring cautious interpretation.

## Evidence

- [methods] NEBULA-LN uses an approximated likelihood based on the law of large numbers, and NEBULA-HL uses an h-likelihood.: "NEBULA-LN uses an approximated likelihood based on the law of large numbers, and NEBULA-HL uses an h-likelihood."
- [discussion] Compared to a standard h-likelihood method, we see that NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion: "Compared to a standard h-likelihood method, we see that NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion"
- [methods] genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject.: "genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject."
- [intro] we investigated the number of cells and subjects required to accurately estimate the subject-level and cell-level overdispersions: "we investigated the number of cells and subjects required to accurately estimate the subject-level and cell-level overdispersions"
