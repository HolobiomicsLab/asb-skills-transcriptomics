---
name: multi-sample-scrna-seq-data-preparation
description: Use when preparing multi-sample scRNA-seq expression, pseudotime, and
  sample-covariate data for differential pseudotime (XDE/TDE) testing, or when a Monocle2
  trajectory must be retrofitted for covariate-associated differential expression
  tests.
when_to_use_negative:
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories,
  so this skill cannot seamlessly plug those reconstructed trajectories into topology
  or trajectory-inference modules
- Single-sample scRNA-seq data without sample-level covariates or replicates — Lamian's
  cross-sample variability accounting and the group covariate require multiple samples
  per condition
- Systematic comparison of sample harmonization/embedding methods, which the article
  explicitly states is beyond the study's scope
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: Provides the example dataset expdata and the differential pseudotime framework
    (permutation-based likelihood ratio testing) against which the monocle2TrajTestCorr
    benchmark is compared
  repo: https://github.com/Winnie09/Lamian
- name: Monocle
  role: Trajectory/pseudotime method whose GAM is tailored into monocle2TrajTestCorr
    to enable XDE testing via null vs. full nested model comparison
- name: TSCAN
  role: Dependency installed from GitHub alongside Lamian; TSCAN is used as the default
    method to construct pseudotime in the Lamian workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:c093d06b391c4c43ff95a7275de12ec1aae8ce600ceae4c2ce60941469adeb71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Monocle
  - TSCAN
  - scRNA-seq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-sample-scrna-seq-data-preparation

## Summary

A preparation and benchmarking workflow for multi-sample scRNA-seq differential pseudotime analysis, including the Monocle2 adaptation monocle2TrajTestCorr that tests group-specific intercept and trend differences along pseudotime for XDE detection as a baseline against Lamian.

## When to use

Use when preparing multi-sample scRNA-seq expression, pseudotime, and sample-covariate data for differential pseudotime (XDE/TDE) testing, or when a Monocle2 trajectory must be retrofitted for covariate-associated differential expression tests.

## When NOT to use

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories, so this skill cannot seamlessly plug those reconstructed trajectories into topology or trajectory-inference modules
- Single-sample scRNA-seq data without sample-level covariates or replicates — Lamian's cross-sample variability accounting and the group covariate require multiple samples per condition
- Systematic comparison of sample harmonization/embedding methods, which the article explicitly states is beyond the study's scope

## Inputs

- Gene-by-cell expression matrix (e.g., Lamian expdata: 100 genes x 1000 cells)
- Per-cell pseudotime values along the inferred trajectory
- Per-cell sample annotation (cellanno) mapping cells to biological samples
- Design matrix with intercept in column 1 and sample-group covariate in column 2 (testvar = 2)

## Outputs

- monocle2_xde_results.csv with per-gene likelihood-ratio statistic, p-value, and FDR for each of the 100 genes
- XDE gene set defined by FDR.overall < 0.05

## How to apply

Load the Lamian example dataset expdata (100 genes, 1000 cells) and build the design matrix with the intercept in column 1 and the sample-group covariate in column 2 (testvar = 2). For each gene, fit the null Monocle2-style generalized additive model g(E(Y)) = beta0 + f2(pseudotime) using a spline on pseudotime, then fit the full model g(E(Y)) = beta0 + f1(group) + f2(pseudotime) + f2(pseudotime)*f1(group) adding the group factor, pseudotime spline, and their interaction. Compute the likelihood-ratio statistic between the nested models and assess significance via permutation-based likelihood ratio testing with permuiter >= 100 for real applications. Convert p-values to FDR and call XDE genes at FDR.overall < 0.05, saving per-gene statistics, p-values, and FDR to monocle2_xde_results.csv. Rationale: Monocle2 does not natively support XDE, so the model is tailored so group-specific intercept (mean shift) and group-by-pseudotime trend differences can be tested along pseudotime; upstream sample filtering (e.g., HCA-BM cells with <5000 reads, <1000 expressed genes, or >10% mitochondrial reads removed) ensures clean inputs.

## Related tools

- **Lamian** (Provides the example dataset expdata and the differential pseudotime framework (permutation-based likelihood ratio testing) against which the monocle2TrajTestCorr benchmark is compared) — https://github.com/Winnie09/Lamian
- **Monocle** (Trajectory/pseudotime method whose GAM is tailored into monocle2TrajTestCorr to enable XDE testing via null vs. full nested model comparison)
- **TSCAN** (Dependency installed from GitHub alongside Lamian; TSCAN is used as the default method to construct pseudotime in the Lamian workflow)

## Examples

```
devtools::install_github("zji90/TSCAN"); devtools::install_github("Winnie09/Lamian"); library(Lamian); data(expdata)
```

## Evaluation signals

- Output table monocle2_xde_results.csv contains a likelihood-ratio statistic, p-value, and FDR for each of the 100 genes in expdata
- XDE gene set is exactly the rows with FDR.overall < 0.05
- Permutation-based likelihood ratio test used permuiter >= 100 (recommended for real applications)
- Design matrix check: intercept occupies column 1 and the sample-group covariate occupies column 2 (testvar = 2)
- Null model contains only intercept + pseudotime spline; full model contains group factor, pseudotime spline, and their interaction — verify nesting before computing LRT

## Limitations

- Monocle2 does not natively support XDE; the adapted function is a tailored workaround, and Lamian modules 1 and 2 remain non-interoperable with Monocle2/Monocle3/slingshot trajectories (deferred to future work)
- Phenopath-style linear-assumption issues do not apply here, but non-linear arbitrary differences between conditions require the spline interaction term to be specified correctly
- The workflow is demonstrated on a small example dataset (100 genes, 1000 cells); real applications need permuiter >= 100, increasing compute cost

## Evidence

- [methods] we will use an example dataset **expdata** of 100 genes and 1000 cells to demonstration the workflow: "we will use an example dataset **expdata** of 100 genes and 1000 cells to demonstration the workflow"
- [discussion] permutation-based likelihood ratio test: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] low-quality cell filters: "Cells with fewer than 5000 reads, fewer than 1,000 expressed genes (i.e. genes with nonzero read count), or more than 10% of reads mapped to the mitochondrial genome were deemed as low quality and"
- [readme] README: Lamian framework capabilities: "this framework allows one to (1) construct cellular pseudotemporal trajectory, evaluate the uncertainty of the trajectory branching structure"
- [readme] README: multi-sample motivation: "As scRNA-seq studies are increasingly performed on multiple patient samples, comparing gene expression dynamics across samples has been emerging as a new demand"
