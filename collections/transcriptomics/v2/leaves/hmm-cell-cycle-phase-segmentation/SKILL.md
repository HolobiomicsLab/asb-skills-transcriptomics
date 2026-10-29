---
name: hmm-cell-cycle-phase-segmentation
description: Use when you have an ordered single-cell pseudo time-series (e.g., reCAT
  ordIndex from consensus-TSP) with Bayes/mean scores and need to assign each cell
  to a cell cycle stage such as G1, S, or G2/M.
when_to_use_negative:
- The goal is de novo trajectory inference on non-cyclic processes (e.g. differentiation
  ordering) where methods like Monocle, TSCAN, Wanderlust/Wishbone, SCUBA or DPT are
  the appropriate baselines
- Expression values are not on the log2(TPM+1) scale and have not been filtered to
  cell-cycle base genes, since get_hmm_order requires the standard reCAT input
- Only a single marker gene is available to call cell cycle stage, which the paper
  shows is insufficient to give reliable stage information in scRNA-seq data
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: reCAT
  role: Provides get_hmm_order (HMM segmentation of the ordered cells into cycle phases),
    get_score (Bayes/mean scores), and get_start (maximum-likelihood cycle-start search)
  repo: github.com/tinglab/reCAT
- name: R
  role: Implementation language for reCAT and the HMM segmentation workflow
  repo: github.com/tinglab/reCAT
- name: mclust
  role: Dependency package used by reCAT (Gaussian mixture clustering of cells feeding
    the scoring/segmentation steps)
  repo: github.com/tinglab/reCAT
- name: TSP
  role: Dependency package for the consensus-TSP pseudo-time ordering that precedes
    HMM segmentation
  repo: github.com/tinglab/reCAT
- name: doParallel
  role: Parallel backend for reCAT functions such as get_ordIndex and get_start (nthread/threadnum
    parameters)
  repo: github.com/tinglab/reCAT
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:c18e7d2c08ab19205d513e0f564380cf6ab2ab11ee11fe44fb5358ed8eaae7e3
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# hmm-cell-cycle-phase-segmentation

## Summary

Segments unsynchronized single-cell RNA-seq data into cell cycle phases (G1, S, G2/M, or G0/G1/S/G2/M) using reCAT's hidden Markov model driven by two scores — Bayes-scores and mean-scores — computed along the recovered pseudo time-series order.

## When to use

Use when you have an ordered single-cell pseudo time-series (e.g., reCAT ordIndex from consensus-TSP) with Bayes/mean scores and need to assign each cell to a cell cycle stage such as G1, S, or G2/M.

## When NOT to use

- The goal is de novo trajectory inference on non-cyclic processes (e.g. differentiation ordering) where methods like Monocle, TSCAN, Wanderlust/Wishbone, SCUBA or DPT are the appropriate baselines
- Expression values are not on the log2(TPM+1) scale and have not been filtered to cell-cycle base genes, since get_hmm_order requires the standard reCAT input
- Only a single marker gene is available to call cell cycle stage, which the paper shows is insufficient to give reliable stage information in scRNA-seq data

## Inputs

- Bayes-scores (score_result$bayes_score) from reCAT get_score
- Mean-scores (score_result$mean_score) from reCAT get_score
- Cell pseudo time-series ordering (ordIndex) from get_ordIndex
- Judged class-region table rdata with start/end rows for G1, S, G2/M
- log2(TPM+1) expression matrix of cycle base genes (test_exp from get_test_exp)
- Optional chosen cycle start (myord), e.g. c(4:1, 295:5) for 295 cells

## Outputs

- Per-cell cell-cycle phase labels (hmm_result / cls_result) for G1, S, G2/M (or G0, G1, S, G2/M)
- Maximum-likelihood cycle start from get_start
- Phase-fraction counts per group (e.g. lthsc_g1_fraction_counts.csv with 41/167 young vs 10/183 old LT-HSC in G1)

## How to apply

First recover the cell time-series order with reCAT's get_ordIndex (consensus-TSP, threadnum ~10, step_size = 2, base_cycle_range = c(6:9)) on a log2(TPM+1) expression matrix restricted to cycle base genes, then compute score_result$bayes_score and score_result$mean_score with get_score on the transposed matrix. Supply the scores, ordIndex, and a judged class-region table (rdata: start/end rows identifying which cells are G1, S, and G2/M, read from the score plots, e.g. rows 210-230 = G1, 170-190 = S, 40-60 = G2/M in ola_mES_2i) to get_hmm_order with cls_num = 3 (G1, S, G2/M) or 4 (G0, G1, S, G2/M). If the true cycle start is unknown, use get_start, which iterates over all possible starts, runs get_hmm_order each time, and selects the maximum-likelihood start (nthread = 3); the returned myord (e.g. c(4:1, 295:5)) can then be passed back to get_hmm_order. The HMM output is a per-cell class assignment that downstream analyses use to quantify phase fractions (e.g. 41/167 young vs 10/183 old LT-HSC in G1) or to feed a Kalman smoother that estimates underlying smoothed gene-expression profiles along the cycle.

## Related tools

- **reCAT** (Provides get_hmm_order (HMM segmentation of the ordered cells into cycle phases), get_score (Bayes/mean scores), and get_start (maximum-likelihood cycle-start search)) — github.com/tinglab/reCAT
- **R** (Implementation language for reCAT and the HMM segmentation workflow) — github.com/tinglab/reCAT
- **mclust** (Dependency package used by reCAT (Gaussian mixture clustering of cells feeding the scoring/segmentation steps)) — github.com/tinglab/reCAT
- **TSP** (Dependency package for the consensus-TSP pseudo-time ordering that precedes HMM segmentation) — github.com/tinglab/reCAT
- **doParallel** (Parallel backend for reCAT functions such as get_ordIndex and get_start (nthread/threadnum parameters)) — github.com/tinglab/reCAT

## Examples

```
hmm_result <- get_hmm_order(bayes_score = score_result$bayes_score, mean_score = score_result$mean_score, ordIndex = ordIndex, cls_num = 3, myord = c(4:1, 295:5), rdata = rdata); start = get_start(bayes_score = score_result$bayes_score, mean_score = score_result$mean_score, ordIndex = ordIndex, cls_num = 3, rdata = rdata, nthread = 3)
```

## Evaluation signals

- On the mHSC SMART-seq data, G1 counts from the HMM assignment must equal 41/167 young LT-HSC and 10/183 old LT-HSC, matching Fig. 4a of the paper
- get_start's maximum-likelihood search returns a start whose get_hmm_order likelihood exceeds those of all other candidate starts
- Class-region rdata rows must each map a contiguous span of the ordered series to exactly one phase (e.g. 210-230 = G1, 170-190 = S, 40-60 = G2/M in ola_mES_2i)
- cls_num must match the intended phase model: 3 for G1/S/G2/M or 4 for G0/G1/S/G2/M; the output labels must be consistent with that choice
- Plotting plot_bayes/plot_mean with cls_result and cls_ord should show phase color bars that align with the score peaks

## Limitations

- The HMM requires a user-judged rdata region table read from score plots, so segmentation is semi-supervised and depends on the user correctly identifying G1/S/G2/M regions
- The scoring metrics driving the HMM (Bayes-scores and mean-scores) are noisy and the authors state they 'need improvements to be less noisy and more informative'
- The method relies on a preselected cell-cycle gene set (378 genes) rather than data-driven gene selection; the authors note a preference for semi-supervised selection
- The cycle must start at the correct point for meaningful phase labels; an incorrect myord start changes the segmentation, so get_start's maximum-likelihood search may be required

## Evidence

- [intro] we designed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M, and a Kalman smoother to estimate the underlying gene expression levels: "we designed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M, and a Kalman smoother to estimate the underlying gene expression levels"
- [readme] cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M). myord is the real time series that the you can choose: "cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M). myord is the real time series that the you can choose"
- [readme] we go through from 1 to le(le is the whole number of the cell) as start to get hmm result. Everytime we run ***get_hmm_ord*** function, we can get a likelihood: "we go through from 1 to le(le is the whole number of the cell) as start to get hmm result. Everytime we run ***get_hmm_ord*** function, we can get a likelihood"
- [methods] you can use ***get_hmm_order*** function to sort the cells: "you can use ***get_hmm_order*** function to sort the cells"
