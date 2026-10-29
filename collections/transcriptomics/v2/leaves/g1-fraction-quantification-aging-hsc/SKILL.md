---
name: g1-fraction-quantification-aging-hsc
description: Use when comparing cell-cycle phase distributions (especially G1 fraction)
  of stem-cell populations between conditions, e.g., young vs old LT-HSC scRNA-seq
  data, using unsynchronized single-cell transcriptomes.
when_to_use_negative:
- When the input expression matrix has not been normalized to log2(TPM+1) — reCAT's
  get_test_exp expects this scale and will not standardize raw counts
- When cells are synchronized or phase labels are already known experimentally (e.g.,
  FACS-sorted G1/S/G2-M) — reCAT is designed for unsynchronized scRNA-seq data
- When only a single marker gene is available for phase calling — the paper shows
  one marker gene is not sufficient to give reliable cell-cycle stage information
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: reCAT
  role: R package implementing consensus-TSP ordering, Bayes/mean scoring, and HMM
    segmentation of cell-cycle phases applied to the mHSC data
  repo: github.com/tinglab/reCAT
- name: R
  role: Language in which the reCAT pipeline (get_test_exp, get_ordIndex, get_score,
    get_hmm_order, get_start) is run
  repo: github.com/tinglab/reCAT
- name: mclust
  role: Gaussian mixture model clustering of single cells used to build cluster paths
    for the consensus-TSP ordering
  repo: github.com/tinglab/reCAT
- name: TSP
  role: Traveling salesman problem solver underlying the consensus-TSP recovery of
    the cell time-series order
  repo: github.com/tinglab/reCAT
- name: doParallel
  role: Parallel backend for get_ordIndex (threadnum = 10) and get_start (nthread
    = 3)
  repo: github.com/tinglab/reCAT
- name: ggplot2
  role: Plotting Bayes-scores and mean-scores along the time series to judge phase
    regions (rdata)
  repo: github.com/tinglab/reCAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:07328bcffcb0ee45a69cb042887a00ebe13f679581beca08e53308615b50e262
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
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
yes_rate: 0.6923076923076923
n_claims: 13
---

# g1-fraction-quantification-aging-hsc

## Summary

Quantify the proportion of long-term hematopoietic stem cells (LT-HSCs) in the G1 cell-cycle phase between young and old mice by applying reCAT's pseudo-time ordering, Bayes/mean scoring, and HMM phase segmentation to the mHSC SMART-seq dataset, reproducing the paper's 41/167 (young) vs 10/183 (old) G1 fractions (Fig. 4a).

## When to use

Use when comparing cell-cycle phase distributions (especially G1 fraction) of stem-cell populations between conditions, e.g., young vs old LT-HSC scRNA-seq data, using unsynchronized single-cell transcriptomes.

## When NOT to use

- When the input expression matrix has not been normalized to log2(TPM+1) — reCAT's get_test_exp expects this scale and will not standardize raw counts
- When cells are synchronized or phase labels are already known experimentally (e.g., FACS-sorted G1/S/G2-M) — reCAT is designed for unsynchronized scRNA-seq data
- When only a single marker gene is available for phase calling — the paper shows one marker gene is not sufficient to give reliable cell-cycle stage information

## Inputs

- mHSC SMART-seq hematopoietic expression .RData (youngMpp-style) containing young LT-HSC (167 cells) and old LT-HSC (183 cells)
- Judged phase regions table (rdata) with start/end rows for G1, S, and G2/M read from the score plots
- Cell cycle gene set used by reCAT to filter expression to cycle base genes

## Outputs

- lthsc_g1_fraction_counts.csv with G1 counts and phase labels per LT-HSC population
- ordIndex (cell pseudo-time-series order)
- score_result$bayes_score and score_result$mean_score
- hmm_result: per-cell phase assignment (G1, S, G2/M)

## How to apply

Load the mHSC SMART-seq hematopoietic expression data (youngMpp-style .RData from the reCAT package) and preprocess to log2(TPM+1) with get_test_exp, which filters to cell-cycle base genes. Recover the cell pseudo-time-series order with get_ordIndex (threadnum = 10, step_size = 2, base_cycle_range = c(6:9)) using the consensus-TSP heuristic over Gaussian-mixture cluster paths. Compute both Bayes-scores and mean-scores with get_score, then supply the judged phase regions (a start/end rdata table read from the score plots) to get_hmm_order with cls_num = 3 to segment ordered cells into G1, S, and G2/M; optionally confirm the true cycle start with get_start (maximum-likelihood search over all starts, nthread = 3). Finally, count young LT-HSC cells (n = 167) and old LT-HSC cells (n = 183) assigned to G1 and write counts plus phase labels to lthsc_g1_fraction_counts.csv.

## Related tools

- **reCAT** (R package implementing consensus-TSP ordering, Bayes/mean scoring, and HMM segmentation of cell-cycle phases applied to the mHSC data) — github.com/tinglab/reCAT
- **R** (Language in which the reCAT pipeline (get_test_exp, get_ordIndex, get_score, get_hmm_order, get_start) is run) — github.com/tinglab/reCAT
- **mclust** (Gaussian mixture model clustering of single cells used to build cluster paths for the consensus-TSP ordering) — github.com/tinglab/reCAT
- **TSP** (Traveling salesman problem solver underlying the consensus-TSP recovery of the cell time-series order) — github.com/tinglab/reCAT
- **doParallel** (Parallel backend for get_ordIndex (threadnum = 10) and get_start (nthread = 3)) — github.com/tinglab/reCAT
- **ggplot2** (Plotting Bayes-scores and mean-scores along the time series to judge phase regions (rdata)) — github.com/tinglab/reCAT

## Examples

```
hmm_result <- get_hmm_order(bayes_score = score_result$bayes_score, mean_score = score_result$mean_score, ordIndex = ordIndex, cls_num = 3, myord = myord, rdata = rdata)
```

## Evaluation signals

- G1 counts for young LT-HSC equal 41/167 and for old LT-HSC equal 10/183, matching Fig. 4a of the paper
- hmm_result assigns every ordered cell to exactly one of cls_num = 3 classes (G1, S, G2/M) with no unassigned cells
- sum of per-phase counts equals the population sizes (167 young, 183 old LT-HSC), and young G1 fraction (~24.6%) exceeds old (~5.5%)
- get_start returns a cycle start whose HMM likelihood is the maximum over all possible starts
- Score plots (plot_bayes / plot_mean) show coherent waves of bayes_score and mean_score along ordIndex consistent with the judged rdata regions

## Limitations

- Bayes-scores and mean-scores are noisy and need improvement to be more informative, which can blur phase boundaries
- reCAT relies on a preselected gene set (378 genes) rather than data-driven gene selection; authors prefer semi-supervised selection for better future performance
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias when comparing aged vs young HSCs
- Judged phase regions (rdata) and the real start depend on visual inspection of score plots; wrong regions or start can misassign phases, though get_start can search the maximum-likelihood start

## Evidence

- [intro] we designed two scoring methods, Bayes-scores and mean-scores, to discriminate among cycle stages (G0, G1, S, or G2/M): "we designed two scoring methods, Bayes-scores and mean-scores, to discriminate among cycle stages (G0, G1, S, or G2/M)"
- [intro] HMM segmentation into phases: "we designed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M"
- [readme] get_hmm_order parameters: "cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M)"
- [readme] get_start maximum-likelihood search: "we go through from 1 to le(le is the whole number of the cell) as start to get hmm result. Everytime we run ***get_hmm_ord*** function, we can get a likelihood, and then we find the maximum likelihood"
- [readme] get_ordIndex parameters: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)."
- [intro] one marker gene is insufficient: "we first show one marker gene is not sufficient to give reliable information about cell cycle stages in scRNA-seq data sets"
