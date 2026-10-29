---
name: cell-cycle-pseudo-time-reconstruction
description: Use when given an unsynchronized scRNA-seq expression matrix (log2 TPM+1)
  and the goal is to recover cell cycle ordering (G1, S, G2/M) or compare pseudo-time
  methods such as SCUBA, Monocle, TSCAN, Wishbone, or DPT.
when_to_use_negative:
- When cells were chemically synchronized or the data are bulk (qPCR, microarray,
  bulk RNA-seq) rather than single-cell — reCAT targets unsynchronized scRNA-seq
- When the goal is to analyze differentiation or other non-cyclic trajectories — reCAT
  separates cell cycle analysis from differentiation, which may introduce bias
- When a data-driven gene selection is required — the method relies on a preselected
  378-gene set rather than semi-supervised selection from the data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: reCAT
  role: Core R package providing get_test_exp, get_ordIndex (consensus-TSP), get_score,
    get_hmm_order, and get_start for pseudo-time reconstruction
  repo: github.com/tinglab/reCAT
- name: TSP
  role: Traveling salesman problem solver underlying the consensus-TSP ordering step
- name: mclust
  role: Gaussian mixture model clustering of single cells according to the user-defined
    phase number k
- name: doParallel
  role: Parallel backend for threadnum/nthread parameters in get_ordIndex and get_start
- name: ggplot2
  role: Plotting Bayes-scores and mean-scores along the recovered time series
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:c27369f437bffdb34012053ca4cbcf70ca28026bc7aac06878d0c5bdc413cbdf
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# cell-cycle-pseudo-time-reconstruction

## Summary

Reconstruct a high-resolution cell cycle pseudo time-series from unsynchronized single-cell RNA-seq data using the reCAT R package, which combines consensus-TSP ordering, Bayes/mean scoring, and HMM-based phase segmentation on cycle base genes.

## When to use

Use when given an unsynchronized scRNA-seq expression matrix (log2 TPM+1) and the goal is to recover cell cycle ordering (G1, S, G2/M) or compare pseudo-time methods such as SCUBA, Monocle, TSCAN, Wishbone, or DPT.

## When NOT to use

- When cells were chemically synchronized or the data are bulk (qPCR, microarray, bulk RNA-seq) rather than single-cell — reCAT targets unsynchronized scRNA-seq
- When the goal is to analyze differentiation or other non-cyclic trajectories — reCAT separates cell cycle analysis from differentiation, which may introduce bias
- When a data-driven gene selection is required — the method relies on a preselected 378-gene set rather than semi-supervised selection from the data

## Inputs

- Unsynchronized single-cell RNA-seq expression matrix (e.g. mESC-SMARTer, mESC-Quartz, ola_mES_2i.RData, youngMpp.RData) as raw TPM counts
- Preselected cycle base gene list (378 genes, ccgene from github.com/tinglab/reCAT)
- Judged class region table (rdata) with start/end rows for G1, S, G2/M read from the score plots

## Outputs

- ordIndex: recovered cell time-series ordering from consensus-TSP
- score_result with $bayes_score and $mean_score profiles along the ordering
- hmm_result: per-cell phase labels (G1, S, G2/M or G0, G1, S, G2/M)
- start: maximum-likelihood rotation defining the real cycle start
- Phase-fraction counts per group (e.g. young 41/167 vs old 10/183 LT-HSC in G1) and method-ranking tables (correlation-score, change-index)

## How to apply

Filter the expression matrix to the preselected cycle base genes (378 genes) and transform values to log2(TPM+1) with get_test_exp to build the standard reCAT input (test_exp). Recover the cells' time-series order with get_ordIndex (threadnum matched to available cores, e.g. 10; step_size = 2, which skips k values so consensus paths are computed for k = 12, 14, 16, ...; base_cycle_range = c(6:9) or 7:10). Compute Bayes-scores and mean-scores with get_score on the transposed test_exp, judge class regions (rdata start/end table) from plot_bayes/plot_mean curves, then segment cells into G1, S, G2/M with get_hmm_order (cls_num = 3, or 4 to include G0) and locate the true cycle start with get_start, which searches all rotations by maximum HMM likelihood (nthread = 3). Rationale: reCAT assumes cell cycle phases form a cycle and that transcriptome change between phases is monotonic, so consensus-TSP ordering plus the two scoring profiles suffice to recover the cycle without synchronization.

## Related tools

- **reCAT** (Core R package providing get_test_exp, get_ordIndex (consensus-TSP), get_score, get_hmm_order, and get_start for pseudo-time reconstruction) — github.com/tinglab/reCAT
- **TSP** (Traveling salesman problem solver underlying the consensus-TSP ordering step)
- **mclust** (Gaussian mixture model clustering of single cells according to the user-defined phase number k)
- **doParallel** (Parallel backend for threadnum/nthread parameters in get_ordIndex and get_start)
- **ggplot2** (Plotting Bayes-scores and mean-scores along the recovered time series)

## Examples

```
ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9)); score_result <- get_score(t(test_exp)); hmm_result <- get_hmm_order(bayes_score = score_result$bayes_score, mean_score = score_result$mean_score, ordIndex = ordIndex, cls_num = 3, rdata = rdata)
```

## Evaluation signals

- On mESC-SMARTer, consensus-TSP/reCAT should rank above SCUBA, Monocle, TSCAN, Wanderlust/Wishbone, and DPT by correlation-score and change-index (Fig. 2d ranking in fig2d_method_ranking.csv)
- On mESC-Quartz, the recovered reCAT series vs Oscope should reproduce the reported correlation-score of 0.54 (metric_oscope_corr_054)
- HMM segmentation of mHSC data should yield G1 counts of 41/167 young LT-HSC vs 10/183 old LT-HSC, matching Fig. 4a
- Score plots (plot_bayes/plot_mean) should show phase regions consistent with the judged rdata regions (e.g. ola_mES_2i: G1 ≈ 210:230, S ≈ 170:190, G2/M ≈ 40:60)
- get_start should return the rotation with maximum HMM likelihood over all 1..le possible starts

## Limitations

- The method relies on a preselected gene set (378 genes) rather than data-driven gene selection; the authors prefer semi-supervised selection for better future performance
- Bayes-scores and mean-scores are noisy and need improvement to be less noisy and more informative
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias; a combined model considering multiple processes simultaneously is still needed
- Computational cost grows with cell number: step_size up to 5 is tolerated, and bigger steps are recommended for larger datasets (>1000 cells) to reduce runtime

## Evidence

- [methods] reCAT input construction: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [intro] consensus-TSP ordering: "the order of the clusters was recovered by finding a traveling salesman cycle. Since TSP is a well-known NP-hard problem, we developed a novel and robust heuristic algorithm, termed consensus-TSP"
- [intro] HMM segmentation: "we designed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M, and a Kalman smoother"
- [readme] get_ordIndex parameters: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)"
- [readme] step_size rationale: "Usually a step_size of 2 (by default) would suffice and bigger steps are recommended for larger datasets (>1000 cells) in order to reduce computational time."
- [readme] get_start mechanism: "we go through from 1 to le(le is the whole number of the cell) as start to get hmm result. Everytime we run ***get_hmm_ord*** function, we can get a likelihood, and then we find the maximum"
- [discussion] no-synchronization advantage: "reCAT can (1) recover transcriptome change without cell synchronization, which might otherwise alter the native processes"
