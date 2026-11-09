---
name: bayes-mean-score-computation
description: Use when you have reCAT's test_exp (cycle base genes, log2(TPM+1)) and
  a consensus-TSP ordIndex, and need Bayes- and mean-scores to segment cells into
  cycle stages or to compare pseudo-time methods via correlation-scores/change-index
  values.
when_to_use_negative:
- Input expression matrix has not been filtered to cycle base genes or is not in log2(TPM+1)
  scale — reCAT expects get_test_exp-formatted data
- Cells are synchronized or the goal is bulk RNA-seq/qPCR time-course profiling rather
  than unsynchronized single-cell transcriptome cycle recovery
- No consensus-TSP ordIndex has been computed yet — scores are only interpretable
  along the recovered time-series order
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: R package providing get_score (Bayes-score and mean-score computation), get_test_exp,
    get_ordIndex and plotting functions used in this skill
  repo: github.com/tinglab/reCAT
- name: R
  role: Implementation language; reCAT is achieved in R
  repo: github.com/tinglab/reCAT
- name: ggplot2
  role: Required package for plotting the score profiles along the time-series
  repo: github.com/tinglab/reCAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:dec71c6c3e3b3c5fa424bc7363fdbfbf23b2fd568e4609f892de89379d7118d3
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bayes-mean-score-computation

## Summary

Computes the two reCAT cell-cycle stage scoring metrics — Bayes-scores and mean-scores — along the consensus-TSP recovered time-series of unsynchronized single-cell transcriptome data, to discriminate G0/G1/S/G2/M cycle stages before HMM segmentation.

## When to use

Use when you have reCAT's test_exp (cycle base genes, log2(TPM+1)) and a consensus-TSP ordIndex, and need Bayes- and mean-scores to segment cells into cycle stages or to compare pseudo-time methods via correlation-scores/change-index values.

## When NOT to use

- Input expression matrix has not been filtered to cycle base genes or is not in log2(TPM+1) scale — reCAT expects get_test_exp-formatted data
- Cells are synchronized or the goal is bulk RNA-seq/qPCR time-course profiling rather than unsynchronized single-cell transcriptome cycle recovery
- No consensus-TSP ordIndex has been computed yet — scores are only interpretable along the recovered time-series order

## Inputs

- reCAT test_exp matrix (cycle base genes only, values as log2(TPM+1), from get_test_exp)
- ordIndex — cell time-series ordering from get_ordIndex (consensus-TSP)
- R environment with ggplot2, doParallel, mclust, cluster, TSP installed

## Outputs

- score_result$bayes_score — per-position Bayes-scores along the recovered time-series
- score_result$mean_score — per-position mean-scores along the recovered time-series
- plot_bayes / plot_mean score-versus-time plots (optional HMM-annotated overlays)
- change-index and correlation-score summaries when used for method benchmarking (fig2d_method_ranking.csv)

## How to apply

First build the standard reCAT input with get_test_exp, filtering the mESC-SMARTer (or equivalent) expression matrix to the cycle base genes and transforming values to log2(TPM+1), then recover the cell time-series order with get_ordIndex (consensus-TSP; e.g. threadnum = 10, step_size = 2, base_cycle_range = 6:9). Call get_score on the transposed test_exp matrix (get_score(t(test_exp))), which returns a list with bayes_score and mean_score entries. Plot plot_bayes and plot_mean against the ordIndex to visually judge cycle-stage boundaries; these two noisy scoring profiles feed the downstream HMM (get_hmm_order, cls_num = 3 for G1/S/G2M). In benchmarking contexts, the same mean-scores also underpin the change-index used to rank reCAT's consensus-TSP against SCUBA, Monocle, TSCAN, Wanderlust/Wishbone and DPT orderings on the Cyclebase-gene matrix. The authors caution that both scoring metrics are noisy and would benefit from future improvement.

## Related tools

- **reCAT** (R package providing get_score (Bayes-score and mean-score computation), get_test_exp, get_ordIndex and plotting functions used in this skill) — github.com/tinglab/reCAT
- **R** (Implementation language; reCAT is achieved in R) — github.com/tinglab/reCAT
- **ggplot2** (Required package for plotting the score profiles along the time-series) — github.com/tinglab/reCAT

## Examples

```
source("get_score.R"); score_result <- get_score(t(test_exp)); score_result$bayes_score; score_result$mean_score
```

## Evaluation signals

- score_result list exposes non-empty bayes_score and mean_score vectors of length equal to the number of cells
- plot_bayes/plot_mean traces against ordIndex show periodic wave patterns consistent with the two G1 and G2 expression waves
- Scores vary smoothly along the consensus-TSP ordering; abrupt non-cyclic drift suggests a bad ordIndex or unfiltered input
- In benchmark mode, mean-score-based change-index along the reCAT ordering should rank at the top versus SCUBA, Monocle, TSCAN, Wanderlust/Wishbone and DPT (consistent with Fig. 2d)
- Downstream get_hmm_order with cls_num = 3 yields contiguous G1, S, and G2/M segments (validated via get_start maximum likelihood)

## Limitations

- Bayes-scores and mean-scores are noisy; the authors state they need improvements to be less noisy and more informative
- Scoring relies on a preselected gene set (378 cycle genes) rather than data-driven gene selection, which the authors would prefer to replace with semi-supervised selection
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias

## Evidence

- [intro] we designed two scoring methods, Bayes-scores and mean-scores, to discriminate among cycle stages (G0, G1, S, or G2/M): "we designed two scoring methods, Bayes-scores and mean-scores, to discriminate among cycle stages (G0, G1, S, or G2/M)"
- [methods] get_score usage: "In reCAT, there are two scores, bayes score and mean score, you can easily get both when you use ***get_score*** function"
- [methods] input transformation requirement: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [discussion] noisy scores caveat: "The scoring metrics (i.e., Bayes-scores and mean-scores) to indicate cell cycle stages also need improvements to be less noisy and more informative"
- [readme] plotting scores along time series: "Now you have the scores and time series(ordIndex you get before), you can draw the scores along the time series use ***plot*** function"
