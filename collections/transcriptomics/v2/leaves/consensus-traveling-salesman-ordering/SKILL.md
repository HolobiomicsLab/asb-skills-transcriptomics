---
name: consensus-traveling-salesman-ordering
description: Use when you have log2(TPM+1) scRNA-seq expression filtered to cycle
  base genes and need to order unsynchronized cells along the cell cycle without synchronization.
when_to_use_negative:
- The data are synchronized bulk or qPCR/microarray measurements rather than unsynchronized
  single-cell transcriptome data
- Expression values have not been log2(TPM+1) normalized and filtered to cycle base
  genes via get_test_exp
- The goal is assigning cells to discrete G1/S/G2/M phases rather than recovering
  a continuous time-series ordering (use get_hmm_order/get_start instead)
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: reCAT
  role: R package providing get_test_exp and get_ordIndex implementing the consensus-TSP
    heuristic for cell ordering
  repo: github.com/tinglab/reCAT
- name: TSP
  role: R dependency used to solve traveling salesman cycles underlying the consensus-TSP
    ordering
- name: doParallel
  role: R dependency for multithreaded execution of get_ordIndex (threadnum parameter)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:c821b1f7806380f4d3825c98b93638a3362e9af940c6aad5ca3a72e7ccf6684b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - cluster
  - doParallel
  - mclust
  - reCAT
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# consensus-traveling-salesman-ordering

## Summary

Recover the cell-cycle time-series ordering of unsynchronized single cells using reCAT's consensus-TSP heuristic, which merges Traveling Salesman Problem cycles computed across a range of cluster counts k into a robust pseudo-time ordering of cells.

## When to use

Use when you have log2(TPM+1) scRNA-seq expression filtered to cycle base genes and need to order unsynchronized cells along the cell cycle without synchronization.

## When NOT to use

- The data are synchronized bulk or qPCR/microarray measurements rather than unsynchronized single-cell transcriptome data
- Expression values have not been log2(TPM+1) normalized and filtered to cycle base genes via get_test_exp
- The goal is assigning cells to discrete G1/S/G2/M phases rather than recovering a continuous time-series ordering (use get_hmm_order/get_start instead)

## Inputs

- test_exp matrix: log2(TPM+1) expression values of single cells filtered to cycle base genes (from get_test_exp)
- threadnum: number of CPU cores to parallelize TSP solving
- step_size: number of k to skip in the consensus path (default 2)
- base_cycle_range: range of four k's for the reference cycle (default 6:9 or 7:10)

## Outputs

- ordIndex: ordering of cells representing the recovered cell-cycle time series
- A .RData file recording the clustering result (written by get_ordIndex)

## How to apply

First preprocess the expression matrix with get_test_exp so values are log2(TPM+1) and only cycle base genes are retained, transposing so cells are rows. Then call get_ordIndex(test_exp, threadnum, step_size, base_cycle_range): set threadnum to the number of available cores (e.g., 10 on a server), step_size to the number of k values skipped when merging consensus paths (default 2, up to 5; larger steps recommended for >1000 cells), and base_cycle_range to c(6:9) or c(7:10) for the four k's defining the reference cycle. The function solves the NP-hard TSP via a consensus-TSP heuristic across multiple k, writing a clustering .RData file as a side effect and returning the ordIndex cell ordering. Rationale: different cell cycle phases form a cycle and transcriptome change between phases is monotonic, so a traveling salesman cycle approximates the true time-series.

## Related tools

- **reCAT** (R package providing get_test_exp and get_ordIndex implementing the consensus-TSP heuristic for cell ordering) — github.com/tinglab/reCAT
- **TSP** (R dependency used to solve traveling salesman cycles underlying the consensus-TSP ordering)
- **doParallel** (R dependency for multithreaded execution of get_ordIndex (threadnum parameter))

## Examples

```
ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- Plotted Bayes-scores and mean-scores along the ordIndex show smooth, cyclic waves corresponding to G1 and G2/M checkpoints (inspect via plot_bayes/plot_mean)
- The recovered reCAT time-series correlates with an independent ordering method; e.g., correlation-score of 0.54 against Oscope on the cell-cycle-labeled mESC-Quartz dataset reproduces the reported benchmark
- ordIndex length equals the number of cells and contains each cell exactly once (a valid permutation/cycle)
- Side-effect .RData clustering file is produced and loadable for downstream get_hmm_order / get_cluster_result steps
- Downstream get_start rotation with maximum HMM likelihood yields a consistent start position, confirming the ordering is cyclically coherent

## Limitations

- Consensus-TSP ordering assumes cell cycle phases form a cycle with monotonic transcriptome change between phases; other cyclic processes or differentiating cells may violate this
- For large datasets (>1000 cells) full-resolution consensus computation is expensive; step_size up to 5 is recommended but reduces path resolution
- Preselected cycle base gene set (378 genes) rather than data-driven gene selection may bias ordering; authors prefer semi-supervised selection
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias

## Evidence

- [intro] the order of the clusters was recovered by finding a traveling salesman cycle. Since TSP is a well-known NP-hard problem, we developed a novel and robust heuristic algorithm, termed consensus-TSP: "the order of the clusters was recovered by finding a traveling salesman cycle. Since TSP is a well-known NP-hard problem, we developed a novel and robust heuristic algorithm, termed consensus-TSP"
- [methods] you can get its order(cell's time series) easily with get_ordIndex function: "you can get its order(cell's time series) easily with ***get_ordIndex*** function"
- [readme] ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range).: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)."
- [readme] We recommend step_size of up to a maximum of 5 while preserving the performance of reCAT.: "We recommend step_size of up to a maximum of 5 while preserving the performance of reCAT."
- [other] a correlation-score of 0.54 was reported for the recovered time-series against Oscope on mESC-Quartz.: [evidence span withheld — anchor status: not_found]
