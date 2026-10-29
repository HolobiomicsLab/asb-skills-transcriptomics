---
name: ggplot2-curve-visualization
description: Use when a per-cell scalar measurement (methylation rate, expression
  score) must be binned and smoothed along an ordered circular pseudo time-series
  (ordIndex/HMM phases G1, S, G2/M) and rendered as an annotated curve figure.
when_to_use_negative:
- Input cells have not been ordered (no ordIndex/HMM result from reCAT) — plotting
  raw unordered cells yields a meaningless curve
- The analysis goal is clustering or reordering of cells (get_ordIndex/get_hmm_order
  territory), not visualization of an already-ordered scalar profile
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ggplot2
  role: R plotting package used to draw the smoothed scalar curve along the recovered
    cell cycle and annotate phase regions
  repo: github.com/tinglab/reCAT
- name: reCAT
  role: Supplies the ordered pseudo time-series (ordIndex) and HMM phase labels that
    define the x-axis of the curve
  repo: github.com/tinglab/reCAT
- name: R
  role: Runtime environment in which reCAT and ggplot2 are executed
  repo: github.com/tinglab/reCAT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:9552ae9ca08581043bb357e2b2e929b98058bc318767601ab0471934242477a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - ggplot2
  - reCAT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ggplot2-curve-visualization

## Summary

Visualization skill for drawing smoothed scalar curves (e.g., whole-genome methylation rate) along the reCAT-recovered cell cycle pseudo time-series using ggplot2 in R, as done to reproduce the methylation profile peaking at G1/S (Fig. 4d of the reCAT paper).

## When to use

Use when a per-cell scalar measurement (methylation rate, expression score) must be binned and smoothed along an ordered circular pseudo time-series (ordIndex/HMM phases G1, S, G2/M) and rendered as an annotated curve figure.

## When NOT to use

- Input cells have not been ordered (no ordIndex/HMM result from reCAT) — plotting raw unordered cells yields a meaningless curve
- The analysis goal is clustering or reordering of cells (get_ordIndex/get_hmm_order territory), not visualization of an already-ordered scalar profile

## Inputs

- reCAT ordIndex (cell ordering along the cell cycle) from get_ordIndex
- HMM phase labels (G1, S, G2/M) from get_hmm_order
- Per-cell whole-genome methylation rates from a single-cell bisulfite methylome matrix (e.g., 61 mESC-MT cells)
- Optional per-cell scores (bayes_score, mean_score) from get_score for analogous score curves

## Outputs

- Smoothed methylation-rate curve along the recovered cell cycle, annotated with G1/S/G2/M regions (e.g., methylation_along_cycle.png)

## How to apply

After reCAT recovers the ordering via get_ordIndex (e.g., threadnum=10, step_size=2, base_cycle_range=c(6:9)) and segments phases with get_hmm_order (cls_num=3 for G1/S/G2/M), map each cell's scalar value — here the whole-genome methylation rate from the 61 mESC-MT single-cell bisulfite methylome matrix — onto its position in the pseudo time-series. Bin cells in sliding windows along the circular ordering, compute the per-bin mean methylation rate, and smooth the binned rates across the cycle. Render the smoothed curve with ggplot2 and annotate the G1, S, and G2/M regions so phase boundaries are visually identifiable; save the figure (e.g., methylation_along_cycle.png). ggplot2 is one of the packages the reCAT authors require alongside doParallel, mclust, cluster, and TSP.

## Related tools

- **ggplot2** (R plotting package used to draw the smoothed scalar curve along the recovered cell cycle and annotate phase regions) — github.com/tinglab/reCAT
- **reCAT** (Supplies the ordered pseudo time-series (ordIndex) and HMM phase labels that define the x-axis of the curve) — github.com/tinglab/reCAT
- **R** (Runtime environment in which reCAT and ggplot2 are executed) — github.com/tinglab/reCAT

## Examples

```
ggplot(meth_bin, aes(x = cycle_pos, y = meth_rate)) + geom_smooth() + annotate("rect", fill = phase) ; ggsave("methylation_along_cycle.png")
```

## Evaluation signals

- The smoothed curve exhibits a visible peak of the methylation rate at the G1/S boundary, matching the qualitative pattern reported in Fig. 4d
- The curve is periodic (returns to its starting value when the circular ordering closes) because the x-axis is a circular pseudo time-series
- Phase annotations (G1, S, G2/M) on the plot align with the HMM region labels (rdata regions, e.g., c(210:230) G1, c(40:60) G2/M, c(170:190) S in the ola_mES_2i example)
- Plotting the bayes_score and mean_score along the same ordIndex (plot_bayes / plot_mean) yields the expected alternating pattern, confirming the ordering used as the x-axis is sane
- Output figure file is produced (e.g., methylation_along_cycle.png) with the three phase regions distinguishable

## Limitations

- Bayes-scores and mean-scores used to derive phase regions are noisy and need improvement to be less noisy and more informative, which can blur the annotated phase boundaries
- reCAT separates cell cycle analysis from differentiation, which may introduce some bias in the ordering underlying the curve
- More delicate time-series profiles require higher quality and quantity of sequencing samples; sparse bisulfite data (61 cells) limits resolution of the methylation curve

## Evidence

- [abstract] Methylation variation was exhibited along the recovered cell cycle: "Moreover, we leverage reCAT to exhibit methylation variation along the recovered cell cycle."
- [methods] ggplot2 is part of the required package list: "the package list is as follows: ***ggplot2***"
- [methods] Scores are drawn along the time series with the plotting functions: "you can draw the scores along the time series use ***plot*** function"
- [readme] get_ordIndex supplies the cell ordering used as the plot x-axis: "you can get its order(cell's time series) easily with ***get_ordIndex*** function"
- [readme] HMM segmentation defines G1, S, G2/M regions annotated on the curve: "cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M)"
