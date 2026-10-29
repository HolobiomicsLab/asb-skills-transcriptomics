---
name: kalman-smoothing-pseudotime-expression
description: Use when scRNA-seq data from unsynchronized cells needs a denoised, phase-resolved
  (G1/S/G2/M) expression time-series along a TSP-recovered cell-cycle order, e.g.,
  to surface cycle genes missing from curated bases like Cyclebase.
when_to_use_negative:
- Cells are chemically synchronized, since reCAT is designed to recover the cycle
  without synchronization-induced perturbation
- The expression matrix has not been preprocessed to log2(TPM+1) and filtered to the
  cell-cycle base genes expected by get_test_exp
- The goal is to model multiple processes (e.g., differentiation alongside cell cycle)
  simultaneously — reCAT separates cell cycle analysis from differentiation and may
  introduce bias
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: reCAT
  role: R package providing get_test_exp, get_ordIndex, get_score, get_hmm_order,
    get_start, and the Kalman smoother used to build the smoothed pseudo time-series
  repo: github.com/tinglab/reCAT
- name: TSP
  role: Underlies the consensus-TSP heuristic that recovers the cells' time-series
    order used by the HMM/Kalman step
- name: mclust
  role: Gaussian mixture model clustering of single cells according to the user-defined
    phase number k
- name: doParallel
  role: Parallel backend for reCAT functions such as get_ordIndex (threadnum parameter)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:d93440da6de131916f9ce6380ebed4183e36873c36078a4d89bcfc2447f87661
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - doParallel
  - mclust
  - reCAT
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# kalman-smoothing-pseudotime-expression

## Summary

Apply reCAT's HMM-based segmentation plus Kalman smoother to estimate smoothed gene-expression profiles along a recovered cell-cycle pseudo time-series from unsynchronized single-cell RNA-seq data, enabling downstream ranking of candidate cell-cycle-regulated genes (e.g., Ncapd2) by dCor and KNN-MI.

## When to use

Use when scRNA-seq data from unsynchronized cells needs a denoised, phase-resolved (G1/S/G2/M) expression time-series along a TSP-recovered cell-cycle order, e.g., to surface cycle genes missing from curated bases like Cyclebase.

## When NOT to use

- Cells are chemically synchronized, since reCAT is designed to recover the cycle without synchronization-induced perturbation
- The expression matrix has not been preprocessed to log2(TPM+1) and filtered to the cell-cycle base genes expected by get_test_exp
- The goal is to model multiple processes (e.g., differentiation alongside cell cycle) simultaneously — reCAT separates cell cycle analysis from differentiation and may introduce bias

## Inputs

- Unsynchronized single-cell RNA-seq expression matrix (values as log2(TPM+1)), e.g., the youngMpp dataset restricted to cell-cycle base genes via get_test_exp
- Cell-cycle base gene list used to filter the expression matrix
- Class regions (rdata) judged from the plotted Bayes/mean score profiles, e.g., youngMpp_region
- reCAT scoring outputs: ordIndex, bayes_score, mean_score

## Outputs

- HMM-assigned cell-cycle phase labels (G1, S, G2/M) along the ordered cells
- Kalman-smoothed gene-expression profiles along the pseudo time-series
- Ranked candidate cell-cycle gene lists by dCor and KNN-MI (e.g., top-five lists containing Ncapd2, written to candidate_genes.csv)

## How to apply

Load the expression matrix and restrict it to cell-cycle base genes as log2(TPM+1) with reCAT's get_test_exp. Recover the cells' time-series order with get_ordIndex (e.g., threadnum = 10, step_size = 2, base_cycle_range = c(6:9)), then compute Bayes-scores and mean-scores with get_score. Segment the ordered series into G1, S, and G2/M (cls_num = 3) using get_hmm_order, supplying the class regions (rdata, e.g., youngMpp_region) and the real start obtained via get_start; the HMM built on the two scores drives a Kalman smoother that estimates the underlying smoothed gene-expression profiles along the pseudo time-series. Finally, rank all genes along the smoothed profiles by distance correlation (dCor) and KNN-based mutual information (KNN-MI) significance to extract candidate cycle genes.

## Related tools

- **reCAT** (R package providing get_test_exp, get_ordIndex, get_score, get_hmm_order, get_start, and the Kalman smoother used to build the smoothed pseudo time-series) — github.com/tinglab/reCAT
- **TSP** (Underlies the consensus-TSP heuristic that recovers the cells' time-series order used by the HMM/Kalman step)
- **mclust** (Gaussian mixture model clustering of single cells according to the user-defined phase number k)
- **doParallel** (Parallel backend for reCAT functions such as get_ordIndex (threadnum parameter))

## Examples

```
source("get_hmm.R"); hmm_result <- get_hmm_order(bayes_score=score_result$bayes_score, mean_score=score_result$mean_score, ordIndex=ordIndex, cls_num=3, myord=myord, rdata=rdata)
```

## Evaluation signals

- Output candidate-gene file (candidate_genes.csv) exists and its top-five dCor and KNN-MI lists contain the known cycle-related gene Ncapd2
- HMM result assigns cells to the expected three classes (G1, S, G2/M) consistent with the rdata regions judged from the Bayes/mean score plots
- get_start returns a start maximizing HMM likelihood, and the chosen real order matches the documented myord-style circular ordering c(start:1, N:(start+1))
- Score plots (plot_bayes, plot_mean) along ordIndex show coherent phase waves corresponding to the G1 and G2 checkpoints
- Input test_exp contains only cell-cycle base gene values in log2(TPM+1) scale, as produced by get_test_exp

## Limitations

- reCAT relies on a preselected gene set (378 genes) rather than data-driven gene selection; the authors prefer semi-supervised selection of cell cycle genes from the data for better future performance
- Bayes-scores and mean-scores are noisy and need improvement to be more informative, which can affect HMM segmentation quality
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias; a model considering multiple processes simultaneously is still needed
- More delicate time-series profiles require higher quality and quantity of sequencing samples

## Evidence

- [intro] HMM + Kalman smoother pipeline: "we designed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M, and a Kalman smoother to estimate the underlying gene expression levels"
- [readme] Input preprocessing requirement: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] get_ordIndex parameters: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)."
- [readme] HMM class number: "cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M)."
- [discussion] Unsynchronized recovery rationale: "reCAT can (1) recover transcriptome change without cell synchronization, which might otherwise alter the native processes"
