---
name: correlation-score-benchmarking
description: Use when comparing pseudo-time/cell-cycle ordering methods on unsynchronized
  scRNA-seq data by scoring agreement with experimental stage labels (correlation-score)
  across clustering k sweeps.
when_to_use_negative:
- Input data lacks experimental cell-cycle stage labels or a reference ordering, so
  no PCC against ground-truth stages can be computed
- The dataset has already been synchronized or the analysis goal is not cell-cycle
  pseudo-time (e.g. circadian liver or differentiation trajectories reCAT does not
  model), since reCAT separates cell cycle from differentiation
- The expression matrix is not preprocessed to log2(TPM+1) on cycle base genes, so
  get_test_exp input assumptions are violated
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: reCAT
  role: Provides get_test_exp, get_ordIndex (consensus-TSP), get_score, and get_cluster_result
    used to generate and score the orderings being benchmarked
  repo: github.com/tinglab/reCAT
- name: TSP
  role: R package underpinning the traveling-salesman formulation of the consensus-TSP
    ordering compared against baselines
  repo: github.com/tinglab/reCAT
- name: doParallel
  role: Parallelizes get_ordIndex across threads during the k sweep
  repo: github.com/tinglab/reCAT
- name: mclust
  role: Gaussian mixture model clustering of single cells by user-defined phase number
    k
  repo: github.com/tinglab/reCAT
- name: cluster
  role: Clustering support for segmenting cells in the benchmarked ordering workflow
  repo: github.com/tinglab/reCAT
- name: ggplot2
  role: Plots the correlation-score-vs-k curves and method ranking (Fig. 2c/2d reproduction)
  repo: github.com/tinglab/reCAT
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:f00161f3b34d274bfd29e4ecf2a8a06b52fc2f3f96a244621dfc9ee9ea020ed9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - cluster
  - doParallel
  - ggplot2
  - mclust
  - reCAT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# correlation-score-benchmarking

## Summary

Benchmarking protocol from reCAT for comparing cell-cycle pseudo-time orderings (consensus-TSP vs arbitrary-insertion TSP, Oscope, SCUBA, Monocle, TSCAN, Wanderlust/Wishbone, DPT) on scRNA-seq data using a PCC-based correlation-score against known cell-cycle stage labels.

## When to use

Use when comparing pseudo-time/cell-cycle ordering methods on unsynchronized scRNA-seq data by scoring agreement with experimental stage labels (correlation-score) across clustering k sweeps.

## When NOT to use

- Input data lacks experimental cell-cycle stage labels or a reference ordering, so no PCC against ground-truth stages can be computed
- The dataset has already been synchronized or the analysis goal is not cell-cycle pseudo-time (e.g. circadian liver or differentiation trajectories reCAT does not model), since reCAT separates cell cycle from differentiation
- The expression matrix is not preprocessed to log2(TPM+1) on cycle base genes, so get_test_exp input assumptions are violated

## Inputs

- mESC-SMARTer single-cell expression matrix (cell-cycle-labeled), transformed to log2(TPM+1) and filtered to cycle base genes (test_exp)
- Known cell-cycle phase labels (G1, S, G2/M) for the reference ordering
- Cell orderings from baseline pseudo-time methods (arbitrary insertion TSP, Oscope, SCUBA, Monocle, TSCAN, Wanderlust/Wishbone, DPT)
- Cluster number k sweep and thread count parameters

## Outputs

- Correlation-score (PCC) of each method's ordering vs the experimental stage labels, per k
- Tabulated correlation-score-vs-k curves for all methods (CSV)
- Method ranking CSV (e.g. fig2d_method_ranking.csv) with correlation-score and change-index fields
- ggplot2 correlation-score-vs-k comparison plot reproducing Fig. 2c

## How to apply

Preprocess the single-cell expression matrix to the standard reCAT input format log2(TPM+1) restricted to cycle base genes via get_test_exp. For each cluster number k in the sweep, recover cell orderings with get_ordIndex (consensus-TSP; threadnum set to available cores, step_size = 2, base_cycle_range = c(6:9)), and run the baseline methods (arbitrary insertion, Oscope, SCUBA, Monocle, TSCAN, Wanderlust/Wishbone, DPT) on the same cycle-base-gene matrix. Compute the correlation-score, a PCC-based scoring function measuring agreement between the predicted pseudo time-series and experimental cell-cycle stage labels (optionally alongside change-index values on bayes_score/mean_score from get_score), for every method at every k. Aggregate the per-method correlation-score curves with get_cluster_result-style handling, tabulate them in a CSV, and plot correlation-score-vs-k with ggplot2 to reproduce the Fig. 2c comparison and the Fig. 2d method ranking.

## Related tools

- **reCAT** (Provides get_test_exp, get_ordIndex (consensus-TSP), get_score, and get_cluster_result used to generate and score the orderings being benchmarked) — github.com/tinglab/reCAT
- **TSP** (R package underpinning the traveling-salesman formulation of the consensus-TSP ordering compared against baselines) — github.com/tinglab/reCAT
- **doParallel** (Parallelizes get_ordIndex across threads during the k sweep) — github.com/tinglab/reCAT
- **mclust** (Gaussian mixture model clustering of single cells by user-defined phase number k) — github.com/tinglab/reCAT
- **cluster** (Clustering support for segmenting cells in the benchmarked ordering workflow) — github.com/tinglab/reCAT
- **ggplot2** (Plots the correlation-score-vs-k curves and method ranking (Fig. 2c/2d reproduction)) — github.com/tinglab/reCAT

## Examples

```
source("get_ordIndex.R"); ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- The consensus-TSP correlation-score curve lies above both the arbitrary-insertion and Oscope baselines across the k sweep, matching the reported Fig. 2c result
- The ranking CSV (e.g. fig2d_method_ranking.csv) exists and contains one row per evaluated method with correlation-score and change-index fields
- The reCAT entry ranks at the top of the method ranking, consistent with the reported Fig. 2d ordering over SCUBA, Monocle, TSCAN, Wanderlust/Wishbone and DPT
- Correlation-score values are PCC-based agreements between predicted pseudo time-series and experimental cell-cycle stage labels, so their magnitude should reflect agreement rather than raw expression values
- All methods are evaluated on the identical cycle-base-gene log2(TPM+1) matrix, so per-k comparisons are apples-to-apples

## Limitations

- Bayes-scores and mean-scores used for stage discrimination are noisy and need improvement to be more informative
- reCAT relies on a preselected gene set (378 cycle base genes) rather than data-driven gene selection; the authors prefer semi-supervised selection for better future performance
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias, so benchmark scores conflate only the cyclic process
- Computational cost of get_ordIndex grows with k sweep size; step_size up to a maximum of 5 is recommended, and bigger steps are recommended for larger datasets (>1000 cells)

## Evidence

- [readme] log2(TPM+1) input requirement: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] get_ordIndex parameters: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)."
- [readme] base_cycle_range defaults: "the base_cycle_range is a range of four k’s for computing the reference cycle mentioned in the manuscript. This can be set to 6:9 (by default) or 7:10."
- [methods] clustering result aggregation: "get_cluster_result function can get a clustering result, scores which are calculated based on the clustering result, and the time series of the clustering result"
- [methods] order recovery entry point: "you can get its order(cell's time series) easily with ***get_ordIndex*** function"
