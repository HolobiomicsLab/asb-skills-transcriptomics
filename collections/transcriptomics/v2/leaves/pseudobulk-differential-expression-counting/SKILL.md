---
name: pseudobulk-differential-expression-counting
description: Use when benchmarking perturbation metrics for single-cell differential
  state analysis, or when you need a gene-count baseline (nDEG) to contrast against
  cell-count-confounded measures on datasets with variable cells per cell type.
when_to_use_negative:
- As a primary perturbation metric when cells per cell type vary substantially — nDEG
  is highly confounded by cell count in the 100–10,000 cell resampling of the COVID-19
  dataset
- When computational efficiency matters at scale — the gene-wise pseudobulk testing
  was over 60x slower than scDist's PC-based mixed-model distance
- When a statistically rigorous distance with per-cell-type uncertainty is needed
  rather than a raw count of significant genes
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scDist
  role: Mixed-model comparator that estimates between-condition distance per cell
    type, used to demonstrate nDEG's cell-count confounding (>60x faster)
  repo: https://github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed models underlying the scDist comparison
- name: lmerTest
  role: Provides Satterthwaite degrees of freedom for the per-PC mixed models
- name: Seurat
  role: Single-cell data handling and normalization workflow context
- name: ggplot2
  role: Renders scaling, ranking-preservation, and timing panels as PNG figures
- name: data.table
  role: Tabulates benchmark results across subsample sizes
- name: scTransform
  role: Normalizes counts via Pearson residuals before pseudobulk aggregation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:22fd262d1c69d1dd0cd0b9f2fd2d55099d1882225c09b0033febfd694966b45f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/phillipnicol/scDist
  tools:
  - Seurat
  - data.table
  - ggplot2
  - lme4
  - lmerTest
  - scDist
  - scTransform
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pseudobulk-differential-expression-counting

## Summary

A baseline transcriptomic-perturbation metric that aggregates single-cell counts per sample and cell type (pseudobulk), performs gene-wise differential testing, and counts significant DEGs (nDEG). The scDist paper shows this nDEG count is highly confounded by the number of cells per cell type (resampled 100–10,000 cells in COVID-19 data), making it a cautionary benchmark against model-based distances like scDist.

## When to use

Use when benchmarking perturbation metrics for single-cell differential state analysis, or when you need a gene-count baseline (nDEG) to contrast against cell-count-confounded measures on datasets with variable cells per cell type.

## When NOT to use

- As a primary perturbation metric when cells per cell type vary substantially — nDEG is highly confounded by cell count in the 100–10,000 cell resampling of the COVID-19 dataset
- When computational efficiency matters at scale — the gene-wise pseudobulk testing was over 60x slower than scDist's PC-based mixed-model distance
- When a statistically rigorous distance with per-cell-type uncertainty is needed rather than a raw count of significant genes

## Inputs

- Normalized single-cell expression matrix (genes x cells, e.g., scTransform Pearson residuals)
- Cell-level metadata with condition, patient, and cluster/cell-type annotations
- Subsample size grid (e.g., 100 to 10,000 cells per cell type) with patient-level grouping preserved

## Outputs

- Per-cell-type nDEG counts at each subsample size
- Combined benchmark table (scDist distance, nDEG count, method rankings, runtimes), e.g., downsampling_benchmark_results.csv
- Scaling, ranking-preservation, and timing plots (ggplot2 PNGs, cf. Figs. S5a-c, 4A, B, S4)

## How to apply

From a normalized single-cell matrix (e.g., scTransform Pearson residuals) with metadata for condition, patient, and cluster, aggregate counts by sample-by-cell-type to form pseudobulk profiles, then run gene-wise differential testing and count significant DEGs per cell type (nDEG). To expose the cell-count confounding, subsample one cell population (e.g., CD14 monocytes) across a grid of 100 to 10,000 cells per cell type while preserving patient-level grouping, and recompute nDEG at each size. Tabulate nDEG counts, cell-type rankings, and wall-clock runtimes alongside a mixed-model comparator (scDist with fixed.effects=condition, random.effects=patient, clusters=cell type) fitted per-PC with lme4 and Satterthwaite degrees of freedom from lmerTest. The expected demonstration outcome is that nDEG increases monotonically with cells per cell type while scDist's distance stays approximately constant, and that the gene-wise approach is over 60 times slower than scDist.

## Related tools

- **scDist** (Mixed-model comparator that estimates between-condition distance per cell type, used to demonstrate nDEG's cell-count confounding (>60x faster)) — https://github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed models underlying the scDist comparison)
- **lmerTest** (Provides Satterthwaite degrees of freedom for the per-PC mixed models)
- **Seurat** (Single-cell data handling and normalization workflow context)
- **ggplot2** (Renders scaling, ranking-preservation, and timing panels as PNG figures)
- **data.table** (Tabulates benchmark results across subsample sizes)
- **scTransform** (Normalizes counts via Pearson residuals before pseudobulk aggregation)

## Examples

```
out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response", random.effects="patient", clusters="clusters")
```

## Evaluation signals

- nDEG count increases monotonically with cells per cell type across the 100–10,000 cell subsampling grid
- scDist distance stays approximately constant over the same resampling range, isolating the confounding in nDEG
- scDist cell-type rankings are preserved at small subsample sizes while nDEG rankings change with sample size
- Recorded wall-clock runtime of scDist is at least 60x shorter than the gene-wise nDEG approach
- Combined results table (downsampling_benchmark_results.csv) contains scDist distance, nDEG count, per-method rankings, and runtimes for all cell types at each size

## Limitations

- nDEG is highly confounded by the number of cells per cell type, so rankings derived from it change as sample size changes
- scDist results are sensitive to the fixed choice of number of principal components (K = 20 or 50 per heuristic; no adaptive selection exists)
- scDist requires normalized (e.g., Pearson-residual) input and cannot be applied directly to raw counts; over-clustering can bias estimated distances downward

## Evidence

- [results] nDEG confounding finding: [evidence span withheld — anchor status: not_found]
- [results] Speed comparison: [evidence span withheld — anchor status: not_found]
- [methods] nDEG workflow definition: [evidence span withheld — anchor status: not_found]
- [discussion] scDist model fitting: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite df: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [discussion] Normalization recommendation: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [readme] README usage: "To run `scDist` you will need a normalized expression matrix and
metadata assigning each cell to a condition and cluster."
