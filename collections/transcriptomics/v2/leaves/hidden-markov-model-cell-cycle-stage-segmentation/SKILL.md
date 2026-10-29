---
name: hidden-markov-model-cell-cycle-stage-segmentation
description: Use when an ordered single-cell pseudo time-series (reCAT ordIndex) plus
  bayes_score/mean_score must be partitioned into cell cycle stages, and judged phase
  regions are available from score plots.
when_to_use_negative:
- Input expression values are not log2(TPM+1) filtered to cycle base genes — get_hmm_order
  assumes the reCAT-standardized test_exp
- The goal is only to recover the cell order (use get_ordIndex) or to cluster cells
  into k groups and score clusters (use get_cluster_result), not to assign phase labels
- Cells are synchronized or already carry reliable stage annotations from another
  source — reCAT targets unsynchronized scRNA-seq data where stages must be inferred
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: reCAT
  role: Provides get_hmm_order (HMM segmentation of the ordered time-series into phase
    classes) and get_start (max-likelihood start selection); implemented in R
  repo: https://github.com/tinglab/reCAT
- name: R
  role: Language in which reCAT and its get_hmm_order/get_start functions are executed
- name: ggplot2
  role: Backing plotting package for plot_bayes/plot_mean used to judge rdata regions
    and to validate HMM labels with cls_result colorbar
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:83347a2f6692d9898695840823f49f71709ab050f6dd194c9da2a97bd202d153
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# hidden-markov-model-cell-cycle-stage-segmentation

## Summary

Segments a reCAT-recovered cell cycle pseudo time-series into discrete phases (G1, S, G2/M, optionally G0) using reCAT's HMM built on Bayes-scores and mean-scores, via get_hmm_order and get_start in R.

## When to use

Use when an ordered single-cell pseudo time-series (reCAT ordIndex) plus bayes_score/mean_score must be partitioned into cell cycle stages, and judged phase regions are available from score plots.

## When NOT to use

- Input expression values are not log2(TPM+1) filtered to cycle base genes — get_hmm_order assumes the reCAT-standardized test_exp
- The goal is only to recover the cell order (use get_ordIndex) or to cluster cells into k groups and score clusters (use get_cluster_result), not to assign phase labels
- Cells are synchronized or already carry reliable stage annotations from another source — reCAT targets unsynchronized scRNA-seq data where stages must be inferred

## Inputs

- test_exp: log2(TPM+1) expression matrix restricted to cycle base genes (from get_test_exp)
- ordIndex: cell time-series ordering from get_ordIndex (GMM clustering via mclust + consensus-TSP via TSP)
- score_result$bayes_score and score_result$mean_score from get_score
- rdata: data.frame of judged class regions (start, end) read off the score plots, e.g. G1 210-230, S 170-190, G2/M 40-60 in ola_mES_2i
- myord: candidate start-anchored ordering, e.g. c(4:1, 295:5)

## Outputs

- hmm_result: per-cell phase labels (G1, S, G2/M or G0/G1/S/G2/M) plus per-cluster ordering, saved as hmm_result.RData
- start: the candidate start whose HMM run achieves the maximum likelihood over all le cells

## How to apply

After get_score returns score_result$bayes_score and score_result$mean_score along the ordIndex pseudo time-series, plot both scores (plot_bayes/plot_mean) and visually judge class regions rdata — in ola_mES_2i, cells c(210:230) are judged G1, c(170:190) S, and c(40:60) G2/M. Call get_hmm_order with its six parameters: bayes_score, mean_score, ordIndex, cls_num = 3 (G1/S/G2/M; use 4 to include G0), myord (a start-anchored ordering such as c(4:1, 295:5)), and rdata, so the HMM segments the series into phases. Resolve the true cycle start with get_start, which iterates all candidate starts 1..le, reruns the HMM for each, and returns the start with maximum likelihood (nthread = 3). Save the per-cell G1/S/G2/M labels and per-cluster ordering as hmm_result.RData; in the full reCAT model a Kalman smoother estimates underlying gene expression levels of the time-series. Validate by plotting the scores with the classified cls_result colorbar: score peaks must align with the three annotated phase regions and the HMM labels must partition cells consistently with the judged regions.

## Related tools

- **reCAT** (Provides get_hmm_order (HMM segmentation of the ordered time-series into phase classes) and get_start (max-likelihood start selection); implemented in R) — https://github.com/tinglab/reCAT
- **R** (Language in which reCAT and its get_hmm_order/get_start functions are executed)
- **ggplot2** (Backing plotting package for plot_bayes/plot_mean used to judge rdata regions and to validate HMM labels with cls_result colorbar)

## Examples

```
hmm_result <- get_hmm_order(bayes_score = score_result$bayes_score, mean_score = score_result$mean_score, ordIndex = ordIndex, cls_num = 3, myord = c(4:1, 295:5), rdata = rdata)
```

## Evaluation signals

- plot_bayes and plot_mean along ordIndex show score peaks aligned with the three judged regions in rdata (ola_mES_2i: G1 210-230, S 170-190, G2/M 40-60)
- HMM labels partition cells consistently with the judged regions rather than crossing region boundaries
- get_start returns the candidate start whose HMM likelihood is the maximum across all le starts (scan performed with nthread = 3)
- cls_num matches the number of regions in rdata (3 for G1/S/G2/M, 4 when G0 is included)
- Final classification round-trips through plot_bayes/plot_mean with cls_result = hmm_result, cls_ord = myord, colorbar = 1 showing coherent phase bands

## Limitations

- Requires user-supplied judged regions (rdata) and a chosen start (myord) read visually from score plots; the true start must then be recovered by scanning all candidates with get_start
- The Bayes-scores and mean-scores driving the HMM are noisy — the authors state they 'need improvements to be less noisy and more informative'
- The model relies on a preselected set of 378 cycle base genes rather than data-driven gene selection; the authors would prefer semi-supervised selection
- reCAT separates cell cycle analysis from differentiation, which the authors note may introduce some bias

## Evidence

- [intro] a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M: "we designed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M, and a Kalman smoother to estimate the underlying gene expression levels"
- [readme] cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M). myord is the real time series: "cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M). myord is the real time series that the you can choose, for example, you choose the 4 is the start, and the real"
- [readme] we can easily judge cells in c(210:230) is G1, cells in c(40:60) is G2/M, and cells in c(170:190) is S: "in ola_mES_2i, we can easily judge cells in c(210:230) is G1, cells in c(40:60) is G2/M, and cells in c(170:190) is S"
- [readme] we can get a likelihood, and then we find the maximum likelihood, this is the real start.: "Everytime we run ***get_hmm_ord*** function, we can get a likelihood, and then we find the maximum likelihood, this is the real start."
- [methods] you can use get_hmm_order function to sort the cells: "you can use ***get_hmm_order*** function to sort the cells"
- [discussion] The scoring metrics (i.e., Bayes-scores and mean-scores) to indicate cell cycle stages also need improvements to be less noisy and more informative: "The scoring metrics (i.e., Bayes-scores and mean-scores) to indicate cell cycle stages also need improvements to be less noisy and more informative"
