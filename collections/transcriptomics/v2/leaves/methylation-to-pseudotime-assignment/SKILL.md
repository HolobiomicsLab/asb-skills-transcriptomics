---
name: methylation-to-pseudotime-assignment
description: Use when you have paired single-cell methylation and transcriptome data
  and want to map methylation rates onto an independently recovered cell cycle ordering,
  or reproduce Fig. 4d-style methylation curves along G1/S/G2-M.
when_to_use_negative:
- Methylation and transcriptome data come from different cells or unsynchronized samples
  without matched pairing, since each cell's methylation must be placed at its own
  pseudo-time position
- The goal is cell cycle ordering itself from methylation data alone — reCAT recovers
  order from transcriptome data, not from methylation profiles
- 'Only a few cells are available: binning methylation rates in sliding windows along
  the cycle requires enough cells per window for a stable smoothed curve'
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0204
tools:
- name: reCAT
  role: Recovers the cell cycle ordering (get_ordIndex, get_score, get_hmm_order)
    from matched scRNA-seq data onto which methylation profiles are projected
  repo: github.com/tinglab/reCAT
- name: R
  role: Implementation environment for the reCAT pipeline and the methylation mapping/plotting
    script
- name: ggplot2
  role: Draws the smoothed whole-genome methylation curve along the cycle with phase
    annotations
- name: TSP
  role: Underlies the consensus-TSP traveling salesman cycle used by get_ordIndex
    to order GMM clusters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:e7586895238b06946e6a980d1b6e415b211e29d37af84fa613b8a4f61a15b80f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - ggplot2
  - reCAT
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# methylation-to-pseudotime-assignment

## Summary

Projects single-cell bisulfite methylation profiles onto a reCAT-recovered cell cycle pseudo time-series derived from matched scRNA-seq data, to reveal whole-genome methylation variation along the cycle (e.g., higher methylation at G1/S in mESC-MT, Fig. 4d).

## When to use

Use when you have paired single-cell methylation and transcriptome data and want to map methylation rates onto an independently recovered cell cycle ordering, or reproduce Fig. 4d-style methylation curves along G1/S/G2-M.

## When NOT to use

- Methylation and transcriptome data come from different cells or unsynchronized samples without matched pairing, since each cell's methylation must be placed at its own pseudo-time position
- The goal is cell cycle ordering itself from methylation data alone — reCAT recovers order from transcriptome data, not from methylation profiles
- Only a few cells are available: binning methylation rates in sliding windows along the cycle requires enough cells per window for a stable smoothed curve

## Inputs

- Single-cell bisulfite methylation matrix (e.g., 61 mESC-MT cells with whole-genome methylation rates per cell)
- Matched single-cell RNA-seq expression matrix (log2(TPM+1) values for the same cells)
- Cycle base gene list for filtering expression to cell-cycle genes (reCAT get_test_exp step)

## Outputs

- Cell ordering along the recovered cell cycle (ordIndex) with HMM phase labels (G1, S, G2/M)
- Whole-genome methylation rates binned in sliding windows along the pseudo time-series, smoothed across the circular ordering
- Plot of smoothed methylation rate along the cycle with annotated phases (methylation_along_cycle.png)

## How to apply

Preprocess the matched scRNA-seq expression matrix as log2(TPM+1), filter it to cycle base genes with reCAT's get_test_exp, then recover the cell ordering with get_ordIndex (consensus-TSP over GMM cluster paths; threadnum e.g. 10, step_size 2, base_cycle_range c(6:9)) and obtain bayes_score/mean_score via get_score. Segment the circular ordering into G1, S, and G2/M with get_hmm_order (cls_num = 3), using get_start if the real cycle start is unknown. With ordIndex and HMM phase labels in hand, assign each of the 61 mESC-MT single-cell bisulfite methylation profiles to its position in the pseudo time-series, compute whole-genome methylation rates for cells binned in sliding windows along the cycle, and smooth the binned rates across the circular ordering. Draw the smoothed curve with ggplot2, annotating G1, S, and G2/M regions, and validate that a visible methylation-rate peak occurs at the G1/S boundary as reported in Fig. 4d.

## Related tools

- **reCAT** (Recovers the cell cycle ordering (get_ordIndex, get_score, get_hmm_order) from matched scRNA-seq data onto which methylation profiles are projected) — github.com/tinglab/reCAT
- **R** (Implementation environment for the reCAT pipeline and the methylation mapping/plotting script)
- **ggplot2** (Draws the smoothed whole-genome methylation curve along the cycle with phase annotations)
- **TSP** (Underlies the consensus-TSP traveling salesman cycle used by get_ordIndex to order GMM clusters)

## Examples

```
ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9)); score_result <- get_score(t(test_exp)); hmm_result <- get_hmm_order(bayes_score = score_result$bayes_score, mean_score = score_result$mean_score, ordIndex = ordIndex, cls_num = 3, myord = myord, rdata = rdata)
```

## Evaluation signals

- The saved methylation_along_cycle.png shows a visible peak of the smoothed methylation rate at the G1/S boundary, matching the qualitative pattern in Fig. 4d
- HMM segmentation with cls_num = 3 yields contiguous G1, S, and G2/M regions along the circular ordIndex, consistent with bayes_score/mean_score patterns from plot_bayes/plot_mean
- ordIndex is reproducible across runs with the same parameters (threadnum 10, step_size 2, base_cycle_range c(6:9)), up to cyclic rotation of the starting cell
- Every cell's methylation profile maps to exactly one position in the recovered pseudo time-series; no cells are dropped or double-assigned
- Smoothed methylation rates remain within plausible [0,1] methylation-range bounds across the circular ordering

## Limitations

- reCAT's cell cycle analysis relies on a preselected gene set (378 genes) rather than data-driven gene selection; the authors prefer semi-supervised selection for better future performance
- Bayes-scores and mean-scores used for HMM phase segmentation are noisy and need improvement to be less noisy and more informative
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias in the pseudo time-series that methylation is mapped onto

## Evidence

- [abstract] Methylation variation was exhibited along the recovered cell cycle: "Moreover, we leverage reCAT to exhibit methylation variation along the recovered cell cycle."
- [readme] get_ordIndex recovers cell ordering: "When you preprocessing your test data, you can get its order(cell's time series) easily with ***get_ordIndex*** function."
- [readme] get_hmm_order segments cells into cycle phases: "you can use ***get_hmm_order*** function
to sort the cells."
- [readme] get_test_exp filters to cycle base genes: "in test_exp, there are only cycle base gene's value, we just do a filter on the original data"
- [readme] additional parameters for get_ordIndex: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)."
