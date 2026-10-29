---
name: single-cell-bisulfite-methylation-quantification
description: Use when single-cell bisulfite methylation matrices must be coupled to
  an independently recovered single-cell ordering (e.g., reCAT pseudo time-series
  from paired scRNA-seq) to visualize methylation dynamics along a cyclic biological
  process.
when_to_use_negative:
- When cells are synchronized by experimental perturbation — reCAT is designed to
  recover the cycle without synchronization, which might otherwise alter the native
  processes
- When no paired single-cell transcriptome data exist to recover the ordering — the
  methylation profile is only interpretable once mapped onto the reCAT-recovered pseudo
  time-series
- When the goal is single-marker-gene-based cell cycle staging — the article shows
  one marker gene is not sufficient to give reliable information about cell cycle
  stages
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: reCAT
  role: recovers the cell cycle pseudo time-series (get_test_exp, get_ordIndex, get_score,
    get_hmm_order) onto which methylation profiles are mapped
  repo: github.com/tinglab/reCAT
- name: R
  role: implementation language for the methylation-along-cycle analysis
- name: ggplot2
  role: draws the smoothed methylation curve along the cycle with G1, S, G2/M annotations
- name: TSP
  role: provides the traveling salesman problem solver underlying the consensus-TSP
    ordering used to place cells (and their methylation rates) on the cycle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:813a64ae76227b0699db60e6156593dc740f31affa75ef3982b28713e4006dda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - ggplot2
  - reCAT
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-bisulfite-methylation-quantification

## Summary

Maps whole-genome methylation profiles from single-cell bisulfite methylome data (61 mESC-MT cells) onto a reCAT-recovered cell cycle pseudo time-series, revealing that methylation rates peak at the G1/S boundary (Fig. 4d).

## When to use

Use when single-cell bisulfite methylation matrices must be coupled to an independently recovered single-cell ordering (e.g., reCAT pseudo time-series from paired scRNA-seq) to visualize methylation dynamics along a cyclic biological process.

## When NOT to use

- When cells are synchronized by experimental perturbation — reCAT is designed to recover the cycle without synchronization, which might otherwise alter the native processes
- When no paired single-cell transcriptome data exist to recover the ordering — the methylation profile is only interpretable once mapped onto the reCAT-recovered pseudo time-series
- When the goal is single-marker-gene-based cell cycle staging — the article shows one marker gene is not sufficient to give reliable information about cell cycle stages

## Inputs

- single-cell bisulfite methylation matrix (61 mESC-MT cells, whole-genome methylation rates per cell)
- paired mESC-MT scRNA-seq expression matrix (preprocessed as log2(TPM+1))
- reCAT cycle base gene set (ccgene from github.com/tinglab/reCAT)
- ordIndex (recovered cell pseudo time-series) and HMM phase labels (G1, S, G2/M)

## Outputs

- smoothed whole-genome methylation curve along the recovered cell cycle (methylation_along_cycle.png, ggplot2)
- binned per-window methylation rates across the circular cell ordering

## How to apply

Load the 61-cell mESC-MT single-cell bisulfite methylation matrix together with the paired mESC-MT scRNA-seq expression matrix, preprocessing expression as log2(TPM+1) and filtering to cycle base genes with reCAT's get_test_exp. Recover the cell cycle ordering with get_ordIndex (consensus-TSP over GMM cluster paths; threadnum=10, step_size=2, base_cycle_range=c(6:9)) and segment phases with get_hmm_order (cls_num=3 for G1, S, G2/M) using the bayes_score and mean_score from get_score. Map each cell's methylation profile onto its position in the recovered pseudo time-series via the ordIndex and HMM phase labels, then compute whole-genome methylation rates for cells binned in sliding windows along the cycle and smooth the binned rates across the circular ordering. Draw the smoothed methylation curve with ggplot2, annotating the G1, S, and G2/M regions, and save it as methylation_along_cycle.png. The rationale is that reCAT's recovered ordering is monotonic across phases, so per-cell methylation rates can be read directly against the cycle to expose phase-specific methylation variation.

## Related tools

- **reCAT** (recovers the cell cycle pseudo time-series (get_test_exp, get_ordIndex, get_score, get_hmm_order) onto which methylation profiles are mapped) — github.com/tinglab/reCAT
- **R** (implementation language for the methylation-along-cycle analysis)
- **ggplot2** (draws the smoothed methylation curve along the cycle with G1, S, G2/M annotations)
- **TSP** (provides the traveling salesman problem solver underlying the consensus-TSP ordering used to place cells (and their methylation rates) on the cycle)

## Examples

```
source("get_ordIndex.R"); ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- The saved methylation_along_cycle.png must exhibit a visible peak of the smoothed methylation rate at the G1/S boundary, matching Fig. 4d
- get_test_exp must return a matrix containing only cycle base genes' log2(TPM+1) values from the original data
- The HMM segmentation with cls_num=3 must yield three contiguous phase regions (G1, S, G2/M) along the circular ordIndex
- The smoothed curve must be periodic/circular — methylation returns to its starting value at the cycle boundary
- Bayes-score and mean-score profiles (plot_bayes / plot_mean against ordIndex) must be consistent with the assigned phase regions

## Limitations

- Bayes-scores and mean-scores used to assign phases are noisy and need improvement to be more informative
- reCAT relies on a preselected gene set (378 genes) rather than data-driven gene selection; authors prefer semi-supervised selection for better future performance
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias
- More delicate time-series profiles require higher quality and quantity of sequencing samples

## Evidence

- [abstract] Methylation variation was exhibited along the recovered cell cycle: "Moreover, we leverage reCAT to exhibit methylation variation along the recovered cell cycle."
- [readme] reCAT input preprocessing requirement: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] HMM phase segmentation parameters: "cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M)."
- [intro] consensus-TSP ordering and GMM clustering described in article: "the order of the clusters was recovered by finding a traveling salesman cycle. Since TSP is a well-known NP-hard problem, we developed a novel and robust heuristic algorithm, termed consensus-TSP, to"
- [intro] HMM segmentation of the time-series into cycle stages: "we designed a hidden Markov model (HMM) based on these two scoring methods to segment the time-series into G0, G1, S and G2/M"
