---
name: cell-type-scaling-benchmark-analysis
description: Use when evaluating whether a transcriptomic perturbation score (e.g.,
  number of DEGs, scDist distance) scales with the number of cells per cell type in
  single-cell RNA-seq data, or when benchmarking speed of differential-state methods.
when_to_use_negative:
- Data are raw counts not yet normalized — scDist cannot currently be applied directly
  to raw counts and is sensitive to sub-optimal normalization
- Cell types are over-clustered or annotation is unreliable — double dipping of expression
  data for annotation and distance estimation causes over-clustering and downward
  bias in estimated distances
- The question is differential abundance (changing cell proportions) rather than differential
  state — DA-seq, Milo, or Meld are the appropriate tools, not this perturbation-distance
  benchmark
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Primary perturbation-distance estimator run at each subsample size via linear
    mixed models on PC scores
  repo: https://github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed-effects model used inside scDist
- name: lmerTest
  role: Provides Satterthwaite degrees-of-freedom approximation for the mixed-model
    tests
- name: Seurat
  role: scTransform Pearson-residual normalization of the single-cell counts before
    benchmarking
- name: ggplot2
  role: Renders the scaling, ranking-preservation, and timing benchmark panels as
    PNG files
- name: data.table
  role: Tabulates and combines benchmark results across subsample sizes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:55e163040caa39398790dbf21a4694d8ba1a4baea21abf7e7ef8a77baedabddd
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cell-type-scaling-benchmark-analysis

## Summary

A benchmarking skill that resamples cells per cell type over a controlled grid to test whether a perturbation metric (e.g., nDEG vs. the scDist mixed-model distance) is confounded by cell count, and quantifies speed differences between methods.

## When to use

Use when evaluating whether a transcriptomic perturbation score (e.g., number of DEGs, scDist distance) scales with the number of cells per cell type in single-cell RNA-seq data, or when benchmarking speed of differential-state methods.

## When NOT to use

- Data are raw counts not yet normalized — scDist cannot currently be applied directly to raw counts and is sensitive to sub-optimal normalization
- Cell types are over-clustered or annotation is unreliable — double dipping of expression data for annotation and distance estimation causes over-clustering and downward bias in estimated distances
- The question is differential abundance (changing cell proportions) rather than differential state — DA-seq, Milo, or Meld are the appropriate tools, not this perturbation-distance benchmark

## Inputs

- Normalized single-cell expression matrix (e.g., scTransform Pearson residuals) for a condition-vs-control dataset
- Cell-level metadata with condition, patient, and cluster annotations (e.g., COVID-19 dataset, CD14 monocytes)
- Grid of subsample sizes (cells per cell type, e.g., 100 to 10,000) with patient-level grouping preserved

## Outputs

- downsampling_benchmark_results.csv with scDist distance, nDEG count, per-method cell-type rankings, and runtimes across subsample sizes
- PNG benchmark panels (scaling, ranking-preservation, timing) rendered with ggplot2, cf. Figs. S5a-c, 4A, B, S4

## How to apply

Load a single-cell expression matrix with metadata (condition, patient, cluster) and normalize counts with scTransform Pearson residuals, as recommended by the scDist authors. Subsample a chosen population (e.g., COVID-19 CD14 monocytes) at a grid of cells-per-cell-type sizes spanning 100 to 10,000, preserving patient-level grouping. At each size, run scDist with fixed.effects = condition, random.effects = patient, clusters = cell type, fitting per-PC linear mixed models with lme4 and Satterthwaite degrees of freedom from lmerTest (K = 20 or 50 PCs per the fixed heuristic); in parallel, compute nDEG via pseudobulk aggregation and gene-wise differential testing. Tabulate distances, DEG counts, per-method cell-type rankings, and wall-clock runtimes, then render scaling, ranking-preservation, and timing panels with ggplot2. Rationale: the paper's downsampling experiment shows nDEG inflates with cell count while scDist's distance remains approximately constant and runs over 60x faster.

## Related tools

- **scDist** (Primary perturbation-distance estimator run at each subsample size via linear mixed models on PC scores) — https://github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed-effects model used inside scDist)
- **lmerTest** (Provides Satterthwaite degrees-of-freedom approximation for the mixed-model tests)
- **Seurat** (scTransform Pearson-residual normalization of the single-cell counts before benchmarking)
- **ggplot2** (Renders the scaling, ranking-preservation, and timing benchmark panels as PNG files)
- **data.table** (Tabulates and combines benchmark results across subsample sizes)

## Examples

```
out <- scDist(sub$Y, sub$meta.data, fixed.effects = "condition", random.effects = "patient", clusters = "cluster")
```

## Evaluation signals

- nDEG count increases monotonically with cells per cell type across the 100–10,000 range while the scDist distance estimate stays approximately constant
- scDist cell-type rankings are preserved at small subsample sizes while nDEG rankings change
- scDist runtime is at least 60x shorter than the gene-wise nDEG approach
- downsampling_benchmark_results.csv contains a row per subsample size with distance, nDEG, ranks, and runtime columns
- scDist results data frame reports per-cell-type distance with 95% CI (low/upper) and p.val columns as in the README demo

## Limitations

- No adaptive procedure for selecting the number of PCs K; current choice is a fixed heuristic (20 or 50), and results are sensitive to this choice
- Performance could be affected if input data were normalized by a sub-optimal approach other than Pearson residuals; scDist cannot currently be applied directly to raw counts
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests (estimation is less affected)
- Double dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances; no cluster-free extension exists yet
- Cell types not present in more than 10% of samples should be excluded from analysis

## Evidence

- [readme] downsampling validation of metric stability: "scDist is an R package that estimates the distance between cell populations in high-dimensional gene-expression space."
- [readme] mixed-model correction for replicates: "When there are multiple replicates (i.e., patients), `scDist` uses linear-mixed effects models to correct for sample-to-sample variability."
- [discussion] per-PC model fitting rationale: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] degrees-of-freedom approximation: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [discussion] normalization recommendation: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [discussion] PC-number sensitivity: "Another point of sensitivity for scDist is the choice of the number of principal components used to estimate the distance."
- [methods] concrete scDist invocation from methods: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
