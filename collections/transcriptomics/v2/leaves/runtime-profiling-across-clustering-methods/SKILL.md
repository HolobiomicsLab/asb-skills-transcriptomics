---
name: runtime-profiling-across-clustering-methods
description: Use when comparing clustering runtimes and resource costs of single-cell
  methods on identical inputs — e.g. FFC vs SC3/Seurat on Splatter-simulated data
  or FFC vs pegasus/scanpy on the Mouse Cell Atlas — while also recording ARI/purity
  quality.
when_to_use_negative:
- Datasets where the distance matrix cannot be approximated by KNN — for >1 million
  data points the runtime/memory profiling itself becomes dominated by the distance-matrix
  cost the article flags as computationally expensive
- Datasets containing only one cluster, where Monte Carlo internal validation and
  hence the full FFC pipeline being timed cannot produce meaningful posterior estimates
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: forest-fire-clustering
  role: Method under benchmark; run FFC.preprocess then FFC.fit(fire_temp=100) per
    dataset and time it
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: Splatter
  role: R package to generate 40 simulated scRNA-seq datasets for the paired FFC vs
    SC3/Seurat runtime benchmark
- name: SC3
  role: R baseline clustered with default recommended settings and timed against FFC
- name: Seurat
  role: R baseline clustered with default recommended settings and timed against FFC
- name: pegasus
  role: Baseline via 'pg.infer_cell_types' timed and compared to FFC on MCA runtime
    and memory
- name: scanpy
  role: Preprocessing (sc.pp.neighbors, zheng17 recipe) producing the identical inputs
    each method is timed on
- name: scikit-learn (sklearn)
  role: adjusted_rand_score to pair each runtime measurement with clustering quality
- name: numba
  role: Numerical acceleration that makes FFC's measured runtime competitive
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:60a74f7e7486270c09d93352ee45b0791d8ba440fcf839cab29245192646dbb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - R
  - SC3
  - Seurat
  - Splatter
  - forest-fire-clustering
  - numba
  - numpy
  - pegasus
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# runtime-profiling-across-clustering-methods

## Summary

Benchmark wall-clock runtime (and peak memory) of Forest Fire Clustering against SC3, Seurat, pegasus, and scanpy-based baselines on the same simulated Splatter datasets and the Mouse Cell Atlas, to substantiate claims that FFC clusters faster than state-of-the-art methods at comparable quality.

## When to use

Use when comparing clustering runtimes and resource costs of single-cell methods on identical inputs — e.g. FFC vs SC3/Seurat on Splatter-simulated data or FFC vs pegasus/scanpy on the Mouse Cell Atlas — while also recording ARI/purity quality.

## When NOT to use

- Datasets where the distance matrix cannot be approximated by KNN — for >1 million data points the runtime/memory profiling itself becomes dominated by the distance-matrix cost the article flags as computationally expensive
- Datasets containing only one cluster, where Monte Carlo internal validation and hence the full FFC pipeline being timed cannot produce meaningful posterior estimates

## Inputs

- Preprocessed scRNA-seq expression matrix [sample x feature] (e.g. scanpy zheng17, 5000 HVGs, 30 PCs)
- Splatter-simulated datasets (20 discrete 5-cell-type and 20 trajectory datasets)
- Mouse Cell Atlas (MCA) gene expression count matrix
- Ground-truth labels for quality-vs-runtime paired comparison

## Outputs

- Per-method wall-clock runtime (and peak memory) per dataset
- benchmark_comparison_results.csv with ARI, purity, significance statistics, and runtimes
- mca_benchmark_results.csv with ARI (unfiltered and PEP<0.1-filtered), runtime, and memory metrics

## How to apply

Run each candidate clustering method on the same preprocessed matrices so timing is directly comparable: FFC via FFC().preprocess(X) then fit(fire_temp=100) on [sample x feature] inputs after scanpy 'zheng17' preprocessing (top 5000 highly variable genes, first 30 principal components); baselines such as SC3 and Seurat in R with default recommended settings, and pegasus via 'pg.infer_cell_types'. Record per-dataset wall-clock runtime (and peak memory on large datasets like MCA) alongside quality metrics (ARI via sklearn.metrics.adjusted_rand_score and purity) against ground-truth labels, then write ARI, purity, significance statistics, and runtime to a results CSV (e.g. benchmark_comparison_results.csv or mca_benchmark_results.csv). Interpret runtime jointly with quality: the correct conclusion pattern is that FFC is faster than state-of-the-art baselines while generating similar-quality clusters.

## Related tools

- **forest-fire-clustering** (Method under benchmark; run FFC.preprocess then FFC.fit(fire_temp=100) per dataset and time it) — https://github.com/gersteinlab/forest-fire-clustering
- **Splatter** (R package to generate 40 simulated scRNA-seq datasets for the paired FFC vs SC3/Seurat runtime benchmark)
- **SC3** (R baseline clustered with default recommended settings and timed against FFC)
- **Seurat** (R baseline clustered with default recommended settings and timed against FFC)
- **pegasus** (Baseline via 'pg.infer_cell_types' timed and compared to FFC on MCA runtime and memory)
- **scanpy** (Preprocessing (sc.pp.neighbors, zheng17 recipe) producing the identical inputs each method is timed on)
- **scikit-learn (sklearn)** (adjusted_rand_score to pair each runtime measurement with clustering quality)
- **numba** (Numerical acceleration that makes FFC's measured runtime competitive)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100)
```

## Evaluation signals

- Reproduced pattern: FFC faster than state-of-the-art baselines while yielding similar-quality clusters (ARI/purity comparable or better, e.g. Wilcoxon P = 9.5 × 10−7 for ARI advantage over SC3 on Splatter data)
- Runtime and memory rows exist for every method and every dataset (40 Splatter datasets; MCA) in the CSV output
- Per-dataset ARI values are paired across methods so a one-sided Wilcoxon signed-rank test on FFC vs SC3/Seurat ARI is computable
- MCA benchmark shows ARI improving from ~0.38 to ~0.72 when labels are filtered to PEP<0.1, confirming the timed pipeline produced the expected FFC output
- All methods timed on identically preprocessed inputs (zheng17, 5000 HVGs, 30 PCs) so runtime comparisons are not confounded by preprocessing differences

## Limitations

- For datasets over 1 million points, representing the distance matrix is computationally expensive and must be approximated via KNN, which affects both the runtime measured and its comparability
- Monte Carlo internal validation is computationally intensive at high trial counts; the number of Monte Carlo trials varies with the number of clusters, so FFC runtime is not fixed across datasets
- The fire temperature c strongly influences the size and number of clusters discovered, so runtime and quality profiling must fix or report fire_temp (benchmarks used fire_temp=100)

## Evidence

- [results] FFC faster with similar quality: [evidence span withheld — anchor status: not_found]
- [other] splatter benchmark runtime step: [evidence span withheld — anchor status: not_found]
- [other] mca runtime and memory step: [evidence span withheld — anchor status: not_found]
- [discussion] large dataset distance limitation: "representing the distance matrix for large datasets (>1 million data points) is computationally expensive"
- [readme] readme invocation: "cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)"
