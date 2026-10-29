---
name: single-cell-clustering-sweep
description: Use when benchmarking a pseudo-time reconstruction (e.g. consensus-TSP
  vs arbitrary insertion or Oscope) on cell-cycle-labeled scRNA-seq data across a
  sweep of cluster numbers k, needing correlation-scores per k.
when_to_use_negative:
- Data is not unsynchronized scRNA-seq with reliable cell-cycle phase labels — the
  correlation-score against stage labels cannot be computed
- Goal is a single ordering rather than benchmarking performance across cluster numbers;
  a single get_ordIndex call without the k sweep suffices
- Comparing pseudo-time methods on data lacking the two-wave G1/G2 cyclic structure
  reCAT assumes (phases forming a cycle with monotonic transcriptome change)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: reCAT
  role: R package providing get_test_exp, get_ordIndex, and get_cluster_result used
    to cluster cells and compute consensus-TSP orderings at each k in the sweep
  repo: github.com/tinglab/reCAT
- name: TSP
  role: R package underlying the traveling-salesman-cycle computation in the consensus-TSP
    ordering
  repo: github.com/tinglab/reCAT
- name: mclust
  role: Gaussian mixture model clustering of single cells by user-defined phase number
    k
  repo: github.com/tinglab/reCAT
- name: cluster
  role: R package dependency for clustering in the reCAT pipeline
  repo: github.com/tinglab/reCAT
- name: doParallel
  role: Parallel backend for the threadnum-controlled consensus-TSP computation across
    the k sweep
  repo: github.com/tinglab/reCAT
- name: ggplot2
  role: Plotting the three correlation-score-vs-k curves for the benchmark comparison
  repo: github.com/tinglab/reCAT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:d2f23f2b6d1a13e74267eab70fd4e32873b496bd42c0618ddc9599e3960ca2be
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-clustering-sweep

## Summary

Sweeps the number of single-cell clusters k in the reCAT consensus-TSP pipeline, recomputing the cell ordering at each k and benchmarking the resulting pseudo time-series against baselines via a PCC-based correlation-score, as done for the mESC-SMARTer comparison (Fig. 2c).

## When to use

Use when benchmarking a pseudo-time reconstruction (e.g. consensus-TSP vs arbitrary insertion or Oscope) on cell-cycle-labeled scRNA-seq data across a sweep of cluster numbers k, needing correlation-scores per k.

## When NOT to use

- Data is not unsynchronized scRNA-seq with reliable cell-cycle phase labels — the correlation-score against stage labels cannot be computed
- Goal is a single ordering rather than benchmarking performance across cluster numbers; a single get_ordIndex call without the k sweep suffices
- Comparing pseudo-time methods on data lacking the two-wave G1/G2 cyclic structure reCAT assumes (phases forming a cycle with monotonic transcriptome change)

## Inputs

- mESC-SMARTer (or similar unsynchronized) single-cell expression matrix to be preprocessed as log2(TPM+1) via get_test_exp
- Experimentally determined cell-cycle stage labels (e.g. G1/S/G2/M) for the cells
- Baseline orderings: arbitrary-insertion TSP ordering and Oscope ordering
- get_ordIndex parameters: threadnum (core count), step_size, base_cycle_range

## Outputs

- Per-k consensus-TSP orderings (ordIndex) across the k sweep
- Correlation-score values per k for consensus-TSP, arbitrary-insertion, and Oscope
- CSV table of the three correlation-score-vs-k curves
- ggplot2 figure reproducing the Fig. 2c comparison

## How to apply

Load the single-cell expression matrix and convert it to reCAT's standard input with get_test_exp, ensuring values are log2(TPM+1). For each k in the sweep, call get_ordIndex on the filtered cycle-gene matrix with threadnum set to available cores, step_size = 2 (recommended maximum 5; larger steps for >1000 cells), and base_cycle_range = c(6:9) or c(7:10) for the reference cycle. At each k also compute baseline orderings (arbitrary insertion for TSP, and the Oscope ordering), then evaluate each ordering with the correlation-score — a PCC-based scoring function measuring agreement between the predicted pseudo time-series and experimental cell-cycle stage labels. Aggregate the three score-over-k curves with get_cluster_result-style score handling, tabulate to CSV, and plot the curves with ggplot2. Validation: the consensus-TSP curve should lie above both baselines across the k sweep, matching the reported mESC-SMARTer result.

## Related tools

- **reCAT** (R package providing get_test_exp, get_ordIndex, and get_cluster_result used to cluster cells and compute consensus-TSP orderings at each k in the sweep) — github.com/tinglab/reCAT
- **TSP** (R package underlying the traveling-salesman-cycle computation in the consensus-TSP ordering) — github.com/tinglab/reCAT
- **mclust** (Gaussian mixture model clustering of single cells by user-defined phase number k) — github.com/tinglab/reCAT
- **cluster** (R package dependency for clustering in the reCAT pipeline) — github.com/tinglab/reCAT
- **doParallel** (Parallel backend for the threadnum-controlled consensus-TSP computation across the k sweep) — github.com/tinglab/reCAT
- **ggplot2** (Plotting the three correlation-score-vs-k curves for the benchmark comparison) — github.com/tinglab/reCAT

## Examples

```
ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- Consensus-TSP correlation-score curve lies above both the arbitrary-insertion and Oscope curves across the k sweep, matching Fig. 2c
- Each per-k ordering is produced with the same get_ordIndex settings (step_size = 2, base_cycle_range = c(6:9)) so scores are comparable
- Input matrix values verified as log2(TPM+1) before ordering
- Correlation-score values are PCC-based and in [-1, 1] with agreement against experimental cell-cycle stage labels
- The k sweep follows the consensus path skipping pattern implied by step_size (e.g. k = 12, 14, 16, ... for step_size = 2)

## Limitations

- step_size > 5 is not recommended; larger steps are only advised for datasets with more than 1000 cells to cut compute time
- reCAT relies on a preselected gene set (378 genes) rather than data-driven gene selection; authors prefer semi-supervised selection
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias; a combined model considering multiple processes is still needed
- Bayes-scores and mean-scores are noisy and need improvement to be more informative

## Evidence

- [methods] in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function"
- [readme] get_ordIndex additional parameters: "The step_size parameter determines the number of k to skip in your consensus path"
- [readme] base_cycle_range definition: "the base_cycle_range is a range of four k’s for computing the reference cycle mentioned in the manuscript"
- [intro] clustering by user-defined phase number k: "clustering of single cells using the Gaussian mixture model (GMM) according to a user-defined phase number k"
- [results] benchmark finding on mESC-SMARTer: [evidence span withheld — anchor status: not_found]
