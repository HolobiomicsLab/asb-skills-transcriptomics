---
name: cell-type-subset-cohort-analysis
description: Use when analyzing droplet-based multi-subject single-cell count data
  where the goal is to test a cell-level predictor (like APOE expression) or subject-level
  genotype group within one annotated cell type, while accounting for hierarchical
  subject structure.
when_to_use_negative:
- Data are single-subject or non-hierarchical — NEBULA's subject-level overdispersion
  component requires multiple subjects grouped by sid.
- The variable of interest is subject-level and the subject count is very small —
  testing is highly sensitive to subject-level variance component estimation and random-effects
  distribution assumptions.
- Genes are very lowly expressed — the co-expression analysis has little statistical
  power to identify low-expression genes, and no observed correlation does not mean
  genes are not co-expressed.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: nebula
  role: R package (v1.5.8) providing nebula() to fit the NBGMM per gene, group_cell()
    to reorder cells by subject ID, and scToNeb() to extract data from Seurat/SingleCellExperiment
    objects
  repo: https://github.com/lhe17/nebula
- name: NEBULA-LN
  role: The large-sample approximation algorithm used to fit the negative binomial
    gamma mixed model per gene; specify covariance=TRUE for contrast testing
  repo: https://github.com/lhe17/nebula
- name: devtools
  role: Used to install the nebula package from GitHub via install_github("lhe17/nebula")
- name: R (R-4.4.0+)
  role: Runtime environment; nebula >=1.5.7 requires R-4.4.0+ and installation requires
    Rcpp-1.0.7
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:c834c6264575c16f80cd3db505f5dd12e5f0daee311c9117db596a5eed1416c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - NEBULA-LN
  - R
  - R (R-4.4.0+)
  - devtools
  - nebula
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cell-type-subset-cohort-analysis

## Summary

Performs cell-level differential or co-expression analysis within a defined cell type (e.g., microglia) across subject genotype groups (e.g., APOE isoform groups e2e3/e3e3/e4+) by fitting a negative binomial gamma mixed model (NBGMM) with NEBULA-LN, which decomposes overdispersion into subject-level and cell-level components for large-scale multi-subject single-cell data.

## When to use

Use when analyzing droplet-based multi-subject single-cell count data where the goal is to test a cell-level predictor (like APOE expression) or subject-level genotype group within one annotated cell type, while accounting for hierarchical subject structure.

## When NOT to use

- Data are single-subject or non-hierarchical — NEBULA's subject-level overdispersion component requires multiple subjects grouped by sid.
- The variable of interest is subject-level and the subject count is very small — testing is highly sensitive to subject-level variance component estimation and random-effects distribution assumptions.
- Genes are very lowly expressed — the co-expression analysis has little statistical power to identify low-expression genes, and no observed correlation does not mean genes are not co-expressed.

## Inputs

- Droplet-based multi-subject single-cell count matrix (M genes x N cells, integer counts, dgCMatrix or matrix; e.g., Synapse syn21261143)
- Subject metadata with genotype annotations (e.g., Synapse syn3157322 APOE isoform assignment)
- Cell-type annotations used to subset cells (e.g., microglia)
- Vector of subject IDs of length N (one per cell)
- Predictor data frame for building the design matrix via model.matrix

## Outputs

- Per-gene NBGMM summary statistics: log(FC), standard errors, raw p-values, subject-level overdispersion (sigma^2) and cell-level overdispersion (phi^-1)
- Covariance matrix of estimated log(FC) when covariance=TRUE (for contrast testing)
- QC-filtered gene list (convergence code <= -20 and cell-level overdispersion >100 removed)
- Comparison table of log(FC) and p-values for target genes (e.g., CST3, C1QA, C1QB, C1QC, TREM2, CLU, ITM2B) across genotype groups, saved as e.g. apoe_isoform_coexpression.csv

## How to apply

Subset cells to the annotated cell type (e.g., microglia) and assign each subject to genotype groups (e.g., APOE e2e3, e3e3, e4+) from the subject metadata. Reorder cells so that cells of the same subject are consecutive using group_cell, then build the design matrix with model.matrix including the intercept, placing APOE expression as the cell-level predictor and covariates alongside. Set library sizes as the offset vector and fit the NBGMM per gene with the nebula function using the NEBULA-LN algorithm (covariance=TRUE to enable contrast testing). Apply QC by filtering genes with cpc=0.005, dropping genes with convergence code <= -20, and removing genes with cell-level overdispersion >100, then extract log(FC) and p-values for the genes of interest (e.g., CST3, C1QA/C1QB/C1QC, TREM2, CLU, ITM2B) and compile into a comparison table across isoform groups.

## Related tools

- **nebula** (R package (v1.5.8) providing nebula() to fit the NBGMM per gene, group_cell() to reorder cells by subject ID, and scToNeb() to extract data from Seurat/SingleCellExperiment objects) — https://github.com/lhe17/nebula
- **NEBULA-LN** (The large-sample approximation algorithm used to fit the negative binomial gamma mixed model per gene; specify covariance=TRUE for contrast testing) — https://github.com/lhe17/nebula
- **devtools** (Used to install the nebula package from GitHub via install_github("lhe17/nebula"))
- **R (R-4.4.0+)** (Runtime environment; nebula >=1.5.7 requires R-4.4.0+ and installation requires Rcpp-1.0.7)

## Examples

```
library(nebula); df = model.matrix(~APOE+covariates, data=pred); re = nebula(count_matrix, sid, pred=df, offset=lib_sizes, covariance=TRUE, cpc=0.005, ncore=4)
```

## Evaluation signals

- Verify that cells of the same subject appear consecutively in the count matrix after group_cell (nebula requires grouped subject IDs).
- Confirm the design matrix contains the intercept term and has unique variable names per column (model.matrix requirement).
- Check the convergence code per gene: results with code <= -20 indicate non-convergence and must be dropped or interpreted with caution.
- Verify genes with estimated cell-level overdispersion >100 (for >=50 cells per subject) were removed from the results.
- The final table should reproduce the expected pattern: strong APOE co-expression with CST3 and C1QA/C1QB/C1QC in the APOE e2e3 microglia population but not with TREM2, with substantial differences among isoform groups (matching Supplementary Data 4/5).

## Limitations

- NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion compared to a standard h-likelihood method.
- Genes with an estimated cell-level overdispersion >100 should be removed for data sets with at least 50 cells per subject.
- If the variable of interest is subject-level, genes with very large subject-level overdispersion (>1) should be removed or interpreted cautiously.
- Misspecification of the random-effects distribution may lead to inflated false-positive rates when testing a subject-level predictor with few subjects.
- The co-expression analysis has little statistical power to identify low-expression genes; absence of observed correlation does not mean two genes are not co-expressed.

## Evidence

- [abstract] Cell-level expression of APOE correlated with other genetic risk factors (CLU, CST3, TREM2, C1q, ITM2B) in a cell-type-specific pattern and isoform-dependent manner in microglia: "we found that the cell-level expression of APOE correlated with that of other genetic risk factors (including CLU, CST3, TREM2, C1q, and ITM2B) in a cell-type-specific pattern and an"
- [readme] group_cell requirement: "The cells in the count matrix need to be grouped by the subjects (that is, the cells of the same subject should be placed consecutively) before using as the input to the function"
- [readme] design matrix intercept requirement: "To build the design matrix, we can use the function model.matrix`` The intercept term must be included in the design matrix."
- [methods] cell-level overdispersion QC threshold: "genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject."
- [methods] convergence QC: "Generally, a convergence code $≤$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
- [methods] contrast testing via covariance: "The first step is to tell nebula to output the covariance matrix of the estimated log(FC). This can be done by specifying covariance=TRUE in nebula."
- [discussion] NEBULA-LN efficiency trade-off: "Compared to a standard h-likelihood method, we see that NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion"
