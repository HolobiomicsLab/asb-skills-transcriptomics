---
name: rank-preservation-analysis
description: Use when benchmarking whether a cell-type perturbation metric keeps stable
  rankings under varying cells-per-cell-type subsampling, or when demonstrating cell-count
  confounding of differential-expression-count metrics.
when_to_use_negative:
- Input data has not been normalized (scDist cannot be applied directly to raw counts;
  scTransform Pearson residual normalization is recommended)
- The goal is differential abundance of cell-type proportions rather than differential-state
  perturbation magnitude (use DA-seq, Milo, or Meld instead)
- Cells are heavily over-clustered or annotations are unreliable, since condition-specific
  effects can cause over-clustering and downward bias in estimated distances
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Computes the per-cell-type distance between condition means under varying
    subsample sizes using linear mixed models on PC scores
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed-effect models inside scDist
- name: lmerTest
  role: Provides Satterthwaite degrees of freedom for the mixed models
- name: Seurat
  role: Loads and handles the single-cell expression matrix and metadata
- name: scTransform
  role: Normalizes counts via Pearson residuals before subsampling and distance estimation
- name: ggplot2
  role: Renders the scaling, ranking-preservation, and timing figure panels
- name: data.table
  role: Tabulates benchmark results across subsample sizes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:b42dd4c8ffd51ebae716e14c87174dcf5b01e6b3ec7256aa24fee4476b41bca2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - data.table
  - ggplot2
  - lme4
  - lmerTest
  - scDist
  - scTransform
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# rank-preservation-analysis

## Summary

A benchmarking skill that tests whether a cell-type perturbation metric (e.g., scDist distance) preserves its cell-type rankings across subsampled data sizes, while contrasting it against a confounded baseline (nDEG counts vs. cells per cell type) on resampled COVID-19 single-cell data.

## When to use

Use when benchmarking whether a cell-type perturbation metric keeps stable rankings under varying cells-per-cell-type subsampling, or when demonstrating cell-count confounding of differential-expression-count metrics.

## When NOT to use

- Input data has not been normalized (scDist cannot be applied directly to raw counts; scTransform Pearson residual normalization is recommended)
- The goal is differential abundance of cell-type proportions rather than differential-state perturbation magnitude (use DA-seq, Milo, or Meld instead)
- Cells are heavily over-clustered or annotations are unreliable, since condition-specific effects can cause over-clustering and downward bias in estimated distances

## Inputs

- COVID-19 single-cell expression matrix (normalized counts, genes x cells)
- Cell-level metadata with condition, patient, and cluster/cell-type annotations
- Grid of cells-per-cell-type subsample sizes (100 to 10,000)

## Outputs

- downsampling_benchmark_results.csv (scDist distance, nDEG count, per-method cell-type rankings, runtimes per subsample size)
- PNG figure panels (ggplot2) showing scaling, ranking preservation, and timing (cf. Figs. S5a-c, 4A, B, S4)

## How to apply

Load the COVID-19 single-cell expression matrix with metadata (condition, patient, cluster), normalize with scTransform Pearson residuals, and subsample the CD14 monocyte population at a grid of cells-per-cell-type sizes from 100 to 10,000 while preserving patient-level grouping. At each size, run scDist with fixed.effects = condition, random.effects = patient, clusters = cell type, fitting per-PC linear mixed models with lme4 and Satterthwaite degrees of freedom from lmerTest (K = 20 or 50 PCs per the fixed heuristic), and in parallel run nDEG by pseudobulk aggregation followed by gene-wise differential testing to count significant DEGs. Tabulate the scDist distance, nDEG count, and per-method cell-type rankings across subsample sizes, and record wall-clock runtimes to verify the >60x speedup. Render scaling, ranking-preservation, and timing panels (cf. Figs. S5a-c, 4A, B, S4) with ggplot2 and write the combined table to downsampling_benchmark_results.csv. Correct application requires the nDEG count to increase monotonically with cell count while the scDist distance stays approximately constant, and scDist rankings to be preserved at small subsample sizes while nDEG rankings change.

## Related tools

- **scDist** (Computes the per-cell-type distance between condition means under varying subsample sizes using linear mixed models on PC scores) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed-effect models inside scDist)
- **lmerTest** (Provides Satterthwaite degrees of freedom for the mixed models)
- **Seurat** (Loads and handles the single-cell expression matrix and metadata)
- **scTransform** (Normalizes counts via Pearson residuals before subsampling and distance estimation)
- **ggplot2** (Renders the scaling, ranking-preservation, and timing figure panels)
- **data.table** (Tabulates benchmark results across subsample sizes)

## Examples

```
out <- scDist(Y_sub, meta_sub, fixed.effects = "condition", random.effects = "patient", clusters = "cluster")
```

## Evaluation signals

- nDEG count increases monotonically with cells per cell type across the 100-10,000 range while the scDist distance estimate stays approximately constant
- scDist cell-type rankings are preserved at small subsample sizes whereas nDEG-based rankings change across subsample sizes
- scDist wall-clock runtime is at least 60x shorter than the gene-wise nDEG pseudobulk approach
- downsampling_benchmark_results.csv contains distance, nDEG count, rank, and runtime columns for every grid size and cell type
- All three figure panels (scaling, ranking preservation, timing) are rendered as PNGs matching Figs. S5a-c, 4A, B, S4

## Limitations

- scDist is sensitive to the choice of the number of principal components K; the current choice is a fixed heuristic (20 or 50) with no adaptive selection procedure
- Performance could be affected if input data were normalized by a sub-optimal approach other than Pearson residuals
- Violations of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests, though estimation should be unaffected
- Significant clustering/annotation errors cause unavoidable bias; a cluster-free extension of scDist does not yet exist

## Evidence

- [results] nDEG confounding finding: [evidence span withheld — anchor status: not_found]
- [abstract] scDist mixed-model description: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally efficient approach for detecting transcriptomic differences"
- [discussion] per-PC mixed model fitting: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite df: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [readme] README demo invocation: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [discussion] recommend normalization with scTransform: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
