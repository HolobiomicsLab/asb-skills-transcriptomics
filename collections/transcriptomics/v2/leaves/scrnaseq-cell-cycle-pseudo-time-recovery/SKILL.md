---
name: scrnaseq-cell-cycle-pseudo-time-recovery
description: Use when you have unsynchronized scRNA-seq expression data and need to
  order cells along the cell cycle (G1/S/G2-M) or associate another single-cell modality
  (e.g., bisulfite methylation) with a recovered cell cycle pseudo time-series.
when_to_use_negative:
- Input expression data is not from single cells or is already synchronized — reCAT
  is designed for unsynchronized scRNA-seq data without cell synchronization artifacts
- You need to infer cell cycle genes de novo from the data — reCAT relies on a preselected
  cycle base gene set (378 genes) rather than data-driven selection
- The analysis requires jointly modeling cell cycle with differentiation or other
  processes — reCAT separates cell cycle analysis from differentiation, which may
  introduce bias
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: Core R package implementing consensus-TSP ordering, GMM clustering, Bayes/mean
    scoring, and HMM phase segmentation of the cell cycle pseudo time-series
  repo: github.com/tinglab/reCAT
- name: TSP
  role: Provides the traveling salesman problem solver underlying the consensus-TSP
    cell ordering heuristic
  repo: github.com/tinglab/reCAT
- name: ggplot2
  role: Draws Bayes/mean score plots along the time series and the smoothed methylation
    curve along the recovered cycle
  repo: github.com/tinglab/reCAT
- name: mclust
  role: Gaussian mixture model clustering of single cells according to the user-defined
    phase number k
  repo: github.com/tinglab/reCAT
- name: doParallel
  role: Parallel execution of consensus-TSP across threads (threadnum depends on available
    cores)
  repo: github.com/tinglab/reCAT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:b1e821c1eb6ef0e34c31fccbf4f8ce1e3ca8260f44fd7123b4eaf24779949d77
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scrnaseq-cell-cycle-pseudo-time-recovery

## Summary

Recover a high-resolution cell cycle pseudo time-series from unsynchronized single-cell RNA-seq data using reCAT, which combines consensus-TSP ordering of GMM clusters with Bayes/mean scores and a hidden Markov model to segment cells into G1, S, and G2/M phases; the recovered ordering can also be leveraged to profile other genomic modalities (e.g., whole-genome methylation) along the cycle.

## When to use

Use when you have unsynchronized scRNA-seq expression data and need to order cells along the cell cycle (G1/S/G2-M) or associate another single-cell modality (e.g., bisulfite methylation) with a recovered cell cycle pseudo time-series.

## When NOT to use

- Input expression data is not from single cells or is already synchronized — reCAT is designed for unsynchronized scRNA-seq data without cell synchronization artifacts
- You need to infer cell cycle genes de novo from the data — reCAT relies on a preselected cycle base gene set (378 genes) rather than data-driven selection
- The analysis requires jointly modeling cell cycle with differentiation or other processes — reCAT separates cell cycle analysis from differentiation, which may introduce bias

## Inputs

- Unsynchronized single-cell RNA-seq expression matrix (cells x genes), preprocessed as log2(TPM+1)
- Cycle base gene set (ccgene, e.g., 378 preselected cell cycle genes) applied via get_test_exp
- Optional: matched single-cell bisulfite methylation matrix for the same cells (e.g., 61 mESC-MT cells)
- Region labels (rdata) with start/end positions for G1, S, G2/M judged from score plots

## Outputs

- ordIndex: circular ordering (pseudo time-series) of cells along the cell cycle
- score_result: bayes_score and mean_score per cell position
- hmm_result: HMM phase labels (G1, S, G2/M) per cell
- Smoothed whole-genome methylation rate curve along the cycle (e.g., methylation_along_cycle.png)

## How to apply

Preprocess expression to log2(TPM+1) and use get_test_exp to retain only cycle base genes, producing the standard input matrix. Recover the cell ordering with get_ordIndex, which runs the consensus-TSP heuristic over GMM cluster paths (threadnum, step_size=2, base_cycle_range=c(6:9)); then compute bayes_score and mean_score with get_score and inspect them with plot_bayes/plot_mean. Segment phases with get_hmm_order (cls_num=3 for G1, S, G2/M or 4 to include G0), supplying the circular myord (e.g., c(4:1, 295:5)) and rdata region labels judged from the score plots; use get_start to find the real cycle start by maximum HMM likelihood. Finally, map each cell — including its methylation profile when a matched single-cell bisulfite matrix exists (e.g., the 61-cell mESC-MT dataset) — onto the ordIndex, bin cells in sliding windows along the circular ordering, smooth the per-bin whole-genome methylation rates, and plot with ggplot2.

## Related tools

- **reCAT** (Core R package implementing consensus-TSP ordering, GMM clustering, Bayes/mean scoring, and HMM phase segmentation of the cell cycle pseudo time-series) — github.com/tinglab/reCAT
- **TSP** (Provides the traveling salesman problem solver underlying the consensus-TSP cell ordering heuristic) — github.com/tinglab/reCAT
- **ggplot2** (Draws Bayes/mean score plots along the time series and the smoothed methylation curve along the recovered cycle) — github.com/tinglab/reCAT
- **mclust** (Gaussian mixture model clustering of single cells according to the user-defined phase number k) — github.com/tinglab/reCAT
- **doParallel** (Parallel execution of consensus-TSP across threads (threadnum depends on available cores)) — github.com/tinglab/reCAT

## Examples

```
ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- The smoothed whole-genome methylation curve along the recovered cycle shows a visible peak at the G1/S boundary, matching the qualitative pattern of Fig. 4d
- get_start returns a start position whose HMM segmentation has maximum likelihood among all candidate starts
- Bayes-score and mean-score plots (plot_bayes/plot_mean) along ordIndex show coherent phase structure consistent with the rdata regions judged for G1, S, and G2/M
- The ordIndex is circular: the recovered ordering wraps around with monotonic transcriptome change between adjacent phases, consistent with reCAT's assumption that transcriptome change is monotonic with time span widths
- HMM segmentation with cls_num=3 yields contiguous G1, S, and G2/M blocks along the pseudo time-series

## Limitations

- Relies on a preselected gene set (378 genes) rather than data-driven gene selection; the authors prefer semi-supervised selection for better future performance
- Bayes-scores and mean-scores are noisy and need improvement to be less noisy and more informative
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias; a combined model considering multiple processes simultaneously is still needed
- step_size for get_ordIndex should not exceed 5; larger steps are recommended only for larger datasets (>1000 cells) to reduce computational time

## Evidence

- [intro] we developed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M, and a Kalman smoother to estimate the underlying gene expression levels: "we designed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M, and a Kalman smoother"
- [readme] in reCAT, the value in test_exp should be log2(TPM+1), and then you can use get_test_exp function to get standard input data: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] The step_size parameter determines the number of k to skip in your consensus path, ie if step_size = 2, then reCAT would only calculate and merge the paths for k = 12, 14, 16: "the base_cycle_range is a range of four k’s for computing the reference cycle mentioned in the manuscript. This can be set to 6:9 (by default) or 7:10."
- [discussion] Instead of the preselected gene set (378 genes), we would prefer semi-supervised selection of cell cycle genes from the data: "Instead of the preselected gene set (378 genes), we would prefer semi-supervised selection of cell cycle genes from the data"
- [readme] cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M): "In this function, there are six parameters: bayes_score, mean_score and ordIndex are values you get before. cls_num is class number you want, generally is 3(G1, S, G2/M)"
