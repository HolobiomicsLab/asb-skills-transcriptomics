---
name: pseudo-time-method-comparison
description: Use when multiple pseudo-time / cell-ordering methods must be ranked
  on data with known stage labels (e.g. cell cycle phases), by sweeping cluster numbers
  k and comparing correlation-scores against the labels.
when_to_use_negative:
- The dataset lacks experimentally determined stage labels (e.g. cell cycle phases),
  since the correlation-score requires known labels to measure agreement of the predicted
  pseudo time-series
- The goal is single-method ordering rather than head-to-head method benchmarking
  — use the reCAT get_ordIndex/get_score workflow directly instead
- Input data are synchronized cell-cycle samples; reCAT is designed for unsynchronized
  scRNA-seq data and the benchmark assumes an unsynchronized setting
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: reCAT
  role: R package supplying get_test_exp, get_ordIndex (consensus-TSP ordering) and
    get_cluster_result used to generate the method orderings and clustering results
  repo: github.com/tinglab/reCAT
- name: TSP
  role: R package underlying the traveling-salesman-problem ordering used by consensus-TSP
    and the arbitrary-insertion baseline
  repo: github.com/tinglab/reCAT
- name: mclust
  role: Gaussian mixture model clustering of single cells into k clusters before ordering
  repo: github.com/tinglab/reCAT
- name: cluster
  role: R package installed as part of the reCAT dependency list for clustering support
  repo: github.com/tinglab/reCAT
- name: doParallel
  role: Parallel backend so get_ordIndex threadnum can be set to the number of available
    cores during the k sweep
  repo: github.com/tinglab/reCAT
- name: ggplot2
  role: Plotting the correlation-score-vs-k comparison curves (Fig. 2c reproduction)
  repo: github.com/tinglab/reCAT
- name: Oscope
  role: Competing oscillation-based ordering baseline whose correlation-score curve
    is compared against consensus-TSP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:c1e4d8cbb3b45eda0fa0d8d8dcf7c0c9ecc7c790b50a798eab2bee2e3c02a9dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Oscope
  - R
  - TSP
  - cluster
  - doParallel
  - ggplot2
  - mclust
  - reCAT
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# pseudo-time-method-comparison

## Summary

Benchmark pseudo-time ordering algorithms by sweeping the cluster number k and scoring each method's ordering against external labels with a PCC-based correlation-score, as reCAT's consensus-TSP was compared against the arbitrary-insertion TSP baseline and Oscope on cell-cycle-labeled mESC-SMARTer data (Fig. 2c).

## When to use

Use when multiple pseudo-time / cell-ordering methods must be ranked on data with known stage labels (e.g. cell cycle phases), by sweeping cluster numbers k and comparing correlation-scores against the labels.

## When NOT to use

- The dataset lacks experimentally determined stage labels (e.g. cell cycle phases), since the correlation-score requires known labels to measure agreement of the predicted pseudo time-series
- The goal is single-method ordering rather than head-to-head method benchmarking — use the reCAT get_ordIndex/get_score workflow directly instead
- Input data are synchronized cell-cycle samples; reCAT is designed for unsynchronized scRNA-seq data and the benchmark assumes an unsynchronized setting

## Inputs

- mESC-SMARTer single-cell RNA-seq expression matrix (cells x genes) with cell-cycle stage labels (e.g. G1, S, G2/M) per cell
- Ordering from the consensus-TSP algorithm (reCAT get_ordIndex) per k in the sweep
- Ordering from the arbitrary-insertion TSP baseline per k
- Ordering from the Oscope algorithm per k
- reCAT ccgene cell cycle base gene set used to filter test_exp

## Outputs

- CSV table of correlation-score values for consensus-TSP, arbitrary-insertion, and Oscope across the k sweep
- ggplot2 figure of correlation-score vs cluster number k reproducing the Fig. 2c comparison
- Per-k pseudo-time orderings (ordIndex) for each method

## How to apply

Load the labeled single-cell expression dataset (mESC-SMARTer) and preprocess it in R to log2(TPM+1) values with the reCAT get_test_exp function, retaining only cell cycle base genes. For each cluster number k in the sweep, cluster cells (Gaussian mixture model) and compute the consensus-TSP ordering via get_ordIndex with threadnum set to available cores, step_size = 2 and base_cycle_range = c(6:9); repeat identically for the arbitrary-insertion TSP baseline and the Oscope ordering. For every method and k, compute the correlation-score, a PCC-based scoring function measuring agreement between the predicted pseudo time-series and the experimentally determined cell cycle stage labels. Aggregate the three correlation-score-vs-k curves (get_cluster_result-style score handling), tabulate them in a CSV, and plot with ggplot2 to reproduce the Fig. 2c comparison. The benchmark is valid when the consensus-TSP curve lies above both baselines across the k sweep, matching the reported outperformance of consensus-TSP over Oscope and arbitrary insertion.

## Related tools

- **reCAT** (R package supplying get_test_exp, get_ordIndex (consensus-TSP ordering) and get_cluster_result used to generate the method orderings and clustering results) — github.com/tinglab/reCAT
- **TSP** (R package underlying the traveling-salesman-problem ordering used by consensus-TSP and the arbitrary-insertion baseline) — github.com/tinglab/reCAT
- **mclust** (Gaussian mixture model clustering of single cells into k clusters before ordering) — github.com/tinglab/reCAT
- **cluster** (R package installed as part of the reCAT dependency list for clustering support) — github.com/tinglab/reCAT
- **doParallel** (Parallel backend so get_ordIndex threadnum can be set to the number of available cores during the k sweep) — github.com/tinglab/reCAT
- **ggplot2** (Plotting the correlation-score-vs-k comparison curves (Fig. 2c reproduction)) — github.com/tinglab/reCAT
- **Oscope** (Competing oscillation-based ordering baseline whose correlation-score curve is compared against consensus-TSP)

## Examples

```
source("get_ordIndex.R"); ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- Consensus-TSP correlation-score curve lies above both the arbitrary-insertion and Oscope baselines across the k sweep, matching Fig. 2c
- Input matrix values equal log2(TPM+1) and contain only cell cycle base genes after get_test_exp filtering
- Each method is scored with the identical PCC-based correlation-score against the same experimental cell-cycle stage labels for every k
- k sweep uses consistent get_ordIndex parameters (step_size = 2, base_cycle_range = c(6:9)) for consensus-TSP so ordering paths cover k = 12, 14, ..., n-2, n
- CSV contains three complete score curves (consensus-TSP, arbitrary insertion, Oscope) with one correlation-score value per k per method

## Limitations

- reCAT's performance depends on a preselected gene set (378 genes) rather than data-driven gene selection; the authors note they would prefer semi-supervised selection of cell cycle genes from the data
- The Bayes-scores and mean-scores used to segment stages are noisy and acknowledged by the authors as needing improvement to be less noisy and more informative
- Computational cost grows with cell number; larger datasets (>1000 cells) require bigger step_size values in get_ordIndex to keep run time manageable
- reCAT separates cell cycle analysis from differentiation, which the authors note may introduce some bias, limiting interpretation of ordering differences

## Evidence

- [results] Consensus-TSP outperformed Oscope and the arbitrary insertion algorithm according to the correlation-score, a PCC-based scoring function measuring agreement between predicted pseudo time-series and experimental cell cycle stage labels: [evidence span withheld — anchor status: not_found]
- [methods] correlation-score benchmark workflow: "the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [methods] ordering step: "the order of the clusters was recovered by finding a traveling salesman cycle. Since TSP is a well-known NP-hard problem, we developed a novel and robust heuristic algorithm, termed consensus-TSP"
- [readme] step_size and base_cycle_range defaults: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)"
- [readme] step_size guidance: "if step_size = 2, then reCAT would only calculate and merge the paths for k = 12, 14, 16, 18, …, n-2, n"
- [readme] base_cycle_range options: "This can be set to 6:9 (by default) or 7:10."
- [intro] clustering step: "clustering of single cells using the Gaussian mixture model (GMM) according to a user-defined phase number k"
- [methods] score aggregation function: "***get_cluster_result*** function can get a clustering result, scores which are calculated based on the clustering result, and the time series of the clustering result"
