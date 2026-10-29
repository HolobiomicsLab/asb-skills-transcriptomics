---
name: cell-cycle-pseudotime-evaluation
description: Use when evaluating an ordering/pseudo-time method for cyclic single-cell
  data against known stage labels (e.g., G1/S/G2/M), by sweeping cluster number k
  and comparing correlation-score curves between a proposed method and baselines.
when_to_use_negative:
- The scRNA-seq data are synchronized along the cycle rather than unsynchronized —
  reCAT is designed to recover time-series without cell synchronization, so this benchmark
  is not the right setting
- There are no experimental cell-cycle stage labels available, since the correlation-score
  is defined as PCC agreement against known stage labels and cannot be computed
- The goal is cell-type or differentiation ordering rather than cell cycle ordering
  — reCAT separates cell cycle analysis from differentiation, which the authors note
  may introduce bias
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: R package providing get_test_exp, get_ordIndex and consensus-TSP ordering
    used to generate the pseudo time-series being evaluated
  repo: https://github.com/tinglab/reCAT
- name: TSP
  role: Traveling salesman problem solver used under the hood for the consensus-TSP
    and arbitrary-insertion TSP orderings
- name: doParallel
  role: Parallel backend for get_ordIndex's threadnum parameter to speed the k sweep
    across cores
- name: mclust
  role: Gaussian mixture model clustering of single cells according to the user-defined
    phase number k
- name: cluster
  role: Clustering support for grouping cells before ordering
- name: ggplot2
  role: Plotting the correlation-score-vs-k comparison curves (Fig. 2c)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:f7e5768ee9cf75ae93db85430a62dfc0650360d15facee4600c99616e79eff23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglab/reCAT
  tools:
  - R
  - TSP
  - cluster
  - doParallel
  - ggplot2
  - mclust
  - reCAT
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cell-cycle-pseudotime-evaluation

## Summary

Benchmark a cell cycle pseudo time-series method by sweeping cluster numbers k and computing a correlation-score (PCC-based agreement between the generated time-series and experimentally determined cell-cycle stage labels), as reCAT's consensus-TSP was compared against arbitrary-insertion TSP and Oscope on mESC-SMARTer (Fig. 2c).

## When to use

Use when evaluating an ordering/pseudo-time method for cyclic single-cell data against known stage labels (e.g., G1/S/G2/M), by sweeping cluster number k and comparing correlation-score curves between a proposed method and baselines.

## When NOT to use

- The scRNA-seq data are synchronized along the cycle rather than unsynchronized — reCAT is designed to recover time-series without cell synchronization, so this benchmark is not the right setting
- There are no experimental cell-cycle stage labels available, since the correlation-score is defined as PCC agreement against known stage labels and cannot be computed
- The goal is cell-type or differentiation ordering rather than cell cycle ordering — reCAT separates cell cycle analysis from differentiation, which the authors note may introduce bias

## Inputs

- mESC-SMARTer single-cell RNA-seq expression matrix (raw TPM values, cells x genes)
- Experimentally determined cell-cycle stage labels (e.g., cell-cycle phase annotations for each cell)
- List of cell-cycle base genes used to filter the expression matrix before ordering

## Outputs

- Consensus-TSP cell ordering (ordIndex) for each cluster number k in the sweep
- Correlation-score (PCC-based) for consensus-TSP, arbitrary-insertion TSP, and Oscope at each k
- CSV table of the three correlation-score-vs-k curves
- ggplot2 figure reproducing the Fig. 2c correlation-score comparison

## How to apply

Load the mESC-SMARTer single-cell expression matrix and preprocess it in R to log2(TPM+1) values using reCAT's get_test_exp, restricting to cell-cycle base genes. For each cluster number k in the sweep, cluster the cells (GMM-based clustering via mclust/cluster) and compute the consensus-TSP ordering with get_ordIndex, setting threadnum to the available cores, step_size = 2 (recommended maximum 5), and base_cycle_range = c(6:9) or 7:10 for the reference cycle. Compute the correlation-score — a PCC-based scoring function measuring agreement between the predicted pseudo time-series and the experimental cell-cycle stage labels — for consensus-TSP, for an arbitrary-insertion TSP baseline ordering, and for the Oscope ordering at every k. Aggregate the three correlation-score-vs-k curves (e.g., with get_cluster_result-style score handling), tabulate them in a CSV, and plot with ggplot2 to reproduce the Fig. 2c comparison.

## Related tools

- **reCAT** (R package providing get_test_exp, get_ordIndex and consensus-TSP ordering used to generate the pseudo time-series being evaluated) — https://github.com/tinglab/reCAT
- **TSP** (Traveling salesman problem solver used under the hood for the consensus-TSP and arbitrary-insertion TSP orderings)
- **doParallel** (Parallel backend for get_ordIndex's threadnum parameter to speed the k sweep across cores)
- **mclust** (Gaussian mixture model clustering of single cells according to the user-defined phase number k)
- **cluster** (Clustering support for grouping cells before ordering)
- **ggplot2** (Plotting the correlation-score-vs-k comparison curves (Fig. 2c))

## Examples

```
source("get_test_exp.R"); test_exp <- get_test_exp(t(test_exp)); source("get_ordIndex.R"); ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- The consensus-TSP correlation-score curve lies above both the arbitrary-insertion and Oscope baselines across the k sweep, matching the reported Fig. 2c result on mESC-SMARTer
- test_exp contains only cell-cycle base gene values in log2(TPM+1) form after get_test_exp filtering
- get_ordIndex produces a .RData file recording the clustering result for each k (e.g., bestEnsembleComplexTSP output), confirming the k sweep completed
- step_size does not exceed 5 and base_cycle_range is c(6:9) or 7:10, per the recommended parameters
- Each method's correlation-score is computed identically over the same k values and stage labels before plotting

## Limitations

- reCAT relies on a preselected gene set (378 genes) rather than data-driven gene selection; the authors prefer semi-supervised selection of cell cycle genes for better future performance
- The scoring metrics (Bayes-scores and mean-scores) used to indicate cell cycle stages are noisy and need improvement to be less noisy and more informative
- For large datasets (>1000 cells), larger step_size values are recommended to reduce computational time, at some cost to performance

## Evidence

- [results] consensus-TSP outperformed Oscope and the arbitrary insertion algorithm according to the correlation-score: [evidence span withheld — anchor status: not_found]
- [intro] clustering of single cells using a Gaussian mixture model according to a user-defined phase number k: "The first step is data processing, including quality control, normalization, and clustering of single cells using the Gaussian mixture model (GMM) according to a user-defined phase number k"
- [methods] the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp***: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] the step_size parameter determines the number of k to skip in your consensus path: "The step_size parameter determines the number of k to skip in your consensus path, ie if step_size = 2, then reCAT would only calculate and merge the paths for k = 12, 14, 16, 18, …, n-2, n."
- [methods] get_cluster_result function can get a clustering result, scores which are calculated based on the clustering result: "***get_cluster_result*** function can get a clustering result, scores which are calculated based on the clustering result, and the time series of the clustering result"
