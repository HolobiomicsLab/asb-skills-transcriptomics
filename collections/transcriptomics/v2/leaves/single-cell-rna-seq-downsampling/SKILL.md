---
name: single-cell-rna-seq-downsampling
description: Use when evaluating whether a cell-type perturbation statistic (e.g.,
  nDEG from pseudobulk DE testing or scDist distance) remains stable under varying
  numbers of cells per cell type in scRNA-seq data.
when_to_use_negative:
- The goal is to compare cell-type abundance/proportion changes between conditions
  rather than transcriptomic perturbation magnitude (differential abundance tools
  like DA-seq, Milo, or Meld are the relevant class).
- The input data are raw counts — scDist's model assumes normalized expression (Pearson
  residuals recommended), and applying it directly to counts is deferred to a future
  version.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: 'Counts as the stable comparator statistic: estimates between-condition distance
    per cell type via mixed models on PC scores, largely invariant to cells-per-cell-type
    resampling and >60x faster than gene-wise nDEG'
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed models underlying scDist distances at each subsample
    size
  repo: github.com/phillipnicol/scDist
- name: lmerTest
  role: Provides Satterthwaite degrees-of-freedom approximation for the mixed-model
    inference
- name: Seurat
  role: Loads and handles the single-cell expression matrix and metadata during resampling
- name: ggplot2
  role: Renders the scaling, ranking-preservation, and timing panels (cf. Figs. S5a-c,
    4A, B, S4) as PNG files
- name: data.table
  role: Tabulates the combined benchmark results table
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:f4f54ed0176bf90ca7fa1ec9d52f1f62b63c53b865a5759c90436d5a67ba3e02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - data.table
  - ggplot2
  - irlba
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-rna-seq-downsampling

## Summary

A benchmarking skill that systematically resamples cells per cell type across a size grid (e.g., 100 to 10,000 cells) in single-cell RNA-seq data to expose methods whose statistics are confounded by cell count, contrasting nDEG (number of differentially expressed genes) with the scDist between-condition distance.

## When to use

Use when evaluating whether a cell-type perturbation statistic (e.g., nDEG from pseudobulk DE testing or scDist distance) remains stable under varying numbers of cells per cell type in scRNA-seq data.

## When NOT to use

- The goal is to compare cell-type abundance/proportion changes between conditions rather than transcriptomic perturbation magnitude (differential abundance tools like DA-seq, Milo, or Meld are the relevant class).
- The input data are raw counts — scDist's model assumes normalized expression (Pearson residuals recommended), and applying it directly to counts is deferred to a future version.

## Inputs

- Normalized single-cell RNA-seq expression matrix (e.g., scTransform Pearson residuals) for a condition-vs-control dataset (e.g., COVID-19)
- Cell-level metadata with condition, patient, and cluster/cell-type columns
- Grid of cells-per-cell-type subsample sizes (100 to 10,000)

## Outputs

- downsampling_benchmark_results.csv with per-size scDist distance, nDEG count, per-method cell-type rankings, and wall-clock runtimes
- ggplot2 PNG panels showing scaling of each statistic with cell count, ranking preservation, and method timing

## How to apply

Load a single-cell expression matrix with metadata (condition, patient, cluster), normalize counts with scTransform Pearson residuals, then subsample a target population (e.g., CD14 monocytes in a COVID-19 dataset) on a grid of cells-per-cell-type sizes spanning 100 to 10,000, preserving patient-level grouping. At each subsample size, run both the perturbation statistic under test and a comparator: pseudobulk aggregation followed by gene-wise differential testing to count significant DEGs (nDEG), and scDist with fixed.effects = condition, random.effects = patient, clusters = cell type, fitting per-PC linear mixed models with lme4 and Satterthwaite degrees of freedom from lmerTest (K = 20 or 50 PCs). Tabulate distances, nDEG counts, cell-type rankings, and wall-clock runtimes; write the combined table to a CSV (e.g., downsampling_benchmark_results.csv) and render scaling, ranking-preservation, and timing panels with ggplot2. The rationale is that a robust statistic should be approximately invariant to cell count, whereas count-dependent statistics will show monotonic inflation with cells per cell type.

## Related tools

- **scDist** (Counts as the stable comparator statistic: estimates between-condition distance per cell type via mixed models on PC scores, largely invariant to cells-per-cell-type resampling and >60x faster than gene-wise nDEG) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed models underlying scDist distances at each subsample size) — github.com/phillipnicol/scDist
- **lmerTest** (Provides Satterthwaite degrees-of-freedom approximation for the mixed-model inference)
- **Seurat** (Loads and handles the single-cell expression matrix and metadata during resampling)
- **ggplot2** (Renders the scaling, ranking-preservation, and timing panels (cf. Figs. S5a-c, 4A, B, S4) as PNG files)
- **data.table** (Tabulates the combined benchmark results table)

## Examples

```
out <- scDist(Y_sub, meta_sub, fixed.effects = "condition", random.effects = "patient", clusters = "celltype")
```

## Evaluation signals

- nDEG count increases monotonically with cells per cell type across the 100-10,000 cell range while the scDist distance estimate stays approximately constant
- scDist cell-type rankings are preserved at small subsample sizes while nDEG rankings change with resampling
- scDist wall-clock runtime is at least 60x shorter than the gene-wise nDEG approach
- Output CSV (downsampling_benchmark_results.csv) contains one row per subsample size with distance, nDEG count, rankings, and runtime columns
- Subsampling preserves patient-level grouping so no patient is split inconsistently across resampled sizes

## Limitations

- Results are sensitive to the choice of the number of principal components K used to estimate the distance; the current choice is a fixed heuristic (20 or 50) rather than an adaptive procedure
- The stable statistic assumes acceptable clustering/annotation; condition-specific effects can cause over-clustering and downward bias in estimated distances
- Performance could be affected if the input data were normalized by a sub-optimal approach other than scTransform Pearson residuals

## Evidence

- [results] nDEG confounded by cell count while scDist stable: [evidence span withheld — anchor status: not_found]
- [discussion] scDist uses mixed-effects models on PC scores: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] scTransform recommended for normalization: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [abstract] scDist runtime advantage: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally efficient approach for detecting transcriptomic differences"
- [readme] README demo invocation of scDist: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
