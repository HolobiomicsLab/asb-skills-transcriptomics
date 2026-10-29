---
name: consensus-tsp-ordering
description: Use when an unsynchronized scRNA-seq matrix restricted to cell-cycle
  base genes (log2(TPM+1) test_exp) must be converted into a cell time-series ordering
  for downstream G1/S/G2/M segmentation or pseudo-time scoring.
when_to_use_negative:
- Cells are synchronized or already carry trusted experimental time/stage labels —
  no pseudo-time ordering is needed
- 'The goal is a differentiation or developmental trajectory: reCAT separates cell-cycle
  analysis from differentiation, which the authors note may introduce bias'
- The expression matrix cannot be restricted to cycle base genes (e.g., genes absent
  from the 378-gene Cyclebase-based ccgene set), leaving the consensus cycle without
  its expected signal
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: reCAT
  role: Core R package whose get_ordIndex function implements the consensus-TSP ordering
    heuristic
  repo: github.com/tinglab/reCAT
- name: mclust
  role: Gaussian mixture model clustering of single cells at each user-defined cluster
    number k before TSP path construction
- name: TSP
  role: R package used to find the traveling-salesman cycle for each clustering that
    the consensus step merges
- name: doParallel
  role: Parallel backend behind the threadnum parameter that speeds up the multi-k
    TSP sweep
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:01e98433d78e31f941dd3cac4dd14e15055f3a3e835c57b1a36706764bbdebd9
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# consensus-tsp-ordering

## Summary

Order unsynchronized single-cell transcriptomes into a cell-cycle pseudo time-series by merging traveling-salesman cycles computed over Gaussian-mixture clusterings at multiple cluster numbers k, as implemented in reCAT's consensus-TSP heuristic.

## When to use

Use when an unsynchronized scRNA-seq matrix restricted to cell-cycle base genes (log2(TPM+1) test_exp) must be converted into a cell time-series ordering for downstream G1/S/G2/M segmentation or pseudo-time scoring.

## When NOT to use

- Cells are synchronized or already carry trusted experimental time/stage labels — no pseudo-time ordering is needed
- The goal is a differentiation or developmental trajectory: reCAT separates cell-cycle analysis from differentiation, which the authors note may introduce bias
- The expression matrix cannot be restricted to cycle base genes (e.g., genes absent from the 378-gene Cyclebase-based ccgene set), leaving the consensus cycle without its expected signal

## Inputs

- Raw single-cell expression matrix (e.g., ola_mES_2i.RData or mESC-SMARTer TPM values)
- Standard reCAT test_exp matrix: log2(TPM+1) values restricted to cycle base genes via get_test_exp
- Parameter set: threadnum (core count), step_size (k skip, default 2), base_cycle_range (c(6:9) or c(7:10))

## Outputs

- ordIndex — integer ordering of the cells along the recovered cell-cycle time series
- A saved .RData file with the clustering/ensemble TSP result (e.g., ensembleResultLst, resultLst) for later use with get_cluster_result
- Input ordering for downstream get_score (bayes_score/mean_score) and get_hmm_order (G1/S/G2/M segmentation)

## How to apply

First produce the standard reCAT input with get_test_exp, which filters the raw matrix to cycle base genes only and stores values as log2(TPM+1). Then run get_ordIndex(test_exp, threadnum, step_size, base_cycle_range): set threadnum to the available core count (e.g., 10 on a laptop, ~20 on a server) since consensus-TSP is parallelized via doParallel; keep step_size = 2 (recommended maximum 5) so TSP paths are computed and merged only for k = 12, 14, 16, ..., n-2, n; and pass base_cycle_range = c(6:9) (or 7:10), the four k values used to compute the reference cycle. The rationale is that TSP is NP-hard, so reCAT solves many smaller TSP cycles across GMM (mclust) clusterings and merges them into a robust consensus ordering rather than solving one large problem. The call returns the ordIndex (the cell time series) and also writes a .RData file holding the clustering/ensemble result (e.g., ensembleResultLst and resultLst), which get_cluster_result can later consume with a chosen cls_num.

## Related tools

- **reCAT** (Core R package whose get_ordIndex function implements the consensus-TSP ordering heuristic) — github.com/tinglab/reCAT
- **mclust** (Gaussian mixture model clustering of single cells at each user-defined cluster number k before TSP path construction)
- **TSP** (R package used to find the traveling-salesman cycle for each clustering that the consensus step merges)
- **doParallel** (Parallel backend behind the threadnum parameter that speeds up the multi-k TSP sweep)

## Examples

```
ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- Plot bayes_score and mean_score along ordIndex with plot_bayes/plot_mean: score peaks should align with the annotated G1, S and G2/M class regions (e.g., rdata regions 210-230, 170-190, 40-60 in ola_mES_2i)
- Benchmark correlation-score (PCC agreement with cell-cycle stage labels): the consensus-TSP curve should lie above both the arbitrary-insertion TSP baseline and Oscope across the k sweep on mESC-SMARTer (Fig. 2c)
- Method ranking by correlation-score and change-index: reCAT/consensus-TSP should outrank SCUBA, Monocle, TSCAN, Wanderlust/Wishbone and DPT (Fig. 2d / Supplementary Fig. 7)
- Schema check: ordIndex is a permutation of length equal to the number of cells, and the side .RData clustering result exists with the expected ensembleResultLst/resultLst objects
- Sweep invariants: with step_size = 2 and base_cycle_range = c(6:9), merged paths exist for k = 12, 14, ..., n-2, n, and the reference cycle derives from k = 6:9

## Limitations

- Relies on a preselected 378-gene cycle base set rather than data-driven gene selection; the authors would prefer semi-supervised selection of cell cycle genes from the data
- Downstream Bayes-scores and mean-scores are noisy and, per the authors, need improvement to be less noisy and more informative
- Computational cost scales with cell number: bigger step_size (up to 5) is recommended for datasets with more than 1000 cells to reduce runtime
- reCAT separates cell cycle analysis from differentiation, which may introduce bias; combined multi-process models are still needed

## Evidence

- [intro] the order of the clusters was recovered by finding a traveling salesman cycle. Since TSP is a well-known NP-hard problem, we developed a novel and robust heuristic algorithm, termed consensus-TSP, to: "the order of the clusters was recovered by finding a traveling salesman cycle. Since TSP is a well-known NP-hard problem, we developed a novel and robust heuristic algorithm, termed consensus-TSP, to"
- [readme] you can get its order(cell's time series) easily with ***get_ordIndex*** function: "you can get its order(cell's time series) easily with ***get_ordIndex*** function"
- [readme] The step_size parameter determines the number of k to skip in your consensus path, ie if step_size = 2: "The step_size parameter determines the number of k to skip in your consensus path, ie if step_size = 2"
- [methods] the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
