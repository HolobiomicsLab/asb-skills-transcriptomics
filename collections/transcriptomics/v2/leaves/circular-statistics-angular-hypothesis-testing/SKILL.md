---
name: circular-statistics-angular-hypothesis-testing
description: Use when comparing two or more groups of predicted circadian times (angles
  on a 0-24 h clock) to formally test whether the angular distributions differ in
  homogeneity or dispersion, e.g., bootstrapped tauFisher predictions per cell type
  at each ZT time point.
when_to_use_negative:
- The quantities to compare are linear (non-angular) measurements where circular wraparound
  does not apply — use ordinary dispersion tests instead
- Predictions are at the single-cell level; tauFisher cannot run on single cells due
  to high scRNAseq dropout, so pseudobulk replicates must be built first
- There is only one phase prediction per group per ZT (no bootstrap replicates), leaving
  insufficient samples for a distributional comparison
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0203
tools:
- name: tauFisher
  role: Predicts circadian time from each bootstrapped pseudobulk replicate; predictions
    are the angles fed into the circular tests
  repo: https://github.com/micnngo/tauFisher
- name: dplyr
  role: R package used for per-ZT grouping and summarizing of predicted times by cell
    type
- name: stats
  role: R stats package used for the circular hypothesis tests (Rao's Tests for Homogeneity,
    Wallraff Test of Angular Distances)
- name: stringr
  role: R string-handling package used in assembling the per-ZT summary outputs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:635139399a81f28539ceeb9ebbb064dd690c1b4e059a2a65ffc834dabfc755ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - dplyr
  - stats
  - stringr
  - tauFisher
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# circular-statistics-angular-hypothesis-testing

## Summary

Applies circular statistical hypothesis tests (Rao's Tests for Homogeneity and the Wallraff Test of Angular Distances) to distributions of bootstrapped circadian-phase predictions (wrapped to the 0-24 h clock) to determine whether predicted phases are significantly more dispersed in one cell type than another.

## When to use

Use when comparing two or more groups of predicted circadian times (angles on a 0-24 h clock) to formally test whether the angular distributions differ in homogeneity or dispersion, e.g., bootstrapped tauFisher predictions per cell type at each ZT time point.

## When NOT to use

- The quantities to compare are linear (non-angular) measurements where circular wraparound does not apply — use ordinary dispersion tests instead
- Predictions are at the single-cell level; tauFisher cannot run on single cells due to high scRNAseq dropout, so pseudobulk replicates must be built first
- There is only one phase prediction per group per ZT (no bootstrap replicates), leaving insufficient samples for a distributional comparison

## Inputs

- Bootstrapped circadian-time predictions from tauFisher (500 pseudobulk replicates per cell type per ZT, wrapped to 0-24 h clock via convert_to_24hr)
- Group labels (e.g., dermal immune cells vs dermal fibroblasts) per ZT time point in GSE223109

## Outputs

- per_zt_prediction_summary.csv with per-cell-type means, standard deviations, and radar-plot data of predicted circadian time at each ZT
- Circular statistical test results (Rao's Tests for Homogeneity; Wallraff Test of Angular Distances) comparing prediction distributions at each ZT

## How to apply

First generate replicate circadian-phase estimates per group: in the tauFisher dermal-skin workflow, sample n cells per cell type with replacement (n = 20% of the immune-cell count) for 500 bootstrap iterations per ZT, sum expression into pseudobulk replicates, log2-transform (log2(x+1)), and predict circadian time with test_tauFisher (predictions wrapped to the 0-24 h clock via convert_to_24hr). Then compute per-group means and standard deviations of predicted time at each ZT. Finally, compare prediction distributions across cell types at each ZT using circular statistics: Rao's Tests for Homogeneity and the Wallraff Test of Angular Distances, treating predicted times as angles so the circular wrap (e.g., 23 h vs 1 h are adjacent) is respected. Save the per-ZT summary table (means, SDs, radar-plot data) and test results as per_zt_prediction_summary.csv.

## Related tools

- **tauFisher** (Predicts circadian time from each bootstrapped pseudobulk replicate; predictions are the angles fed into the circular tests) — https://github.com/micnngo/tauFisher
- **dplyr** (R package used for per-ZT grouping and summarizing of predicted times by cell type)
- **stats** (R stats package used for the circular hypothesis tests (Rao's Tests for Homogeneity, Wallraff Test of Angular Distances))
- **stringr** (R string-handling package used in assembling the per-ZT summary outputs)

## Examples

```
result <- test_tauFisher(chosen_genes=trained_JTK$Genes, ...); pred_24h <- convert_to_24hr(result); per_zt_summary <- pred_24h %>% group_by(cell_type, ZT) %>% summarise(mean=mean(time), sd=sd(time))
```

## Evaluation signals

- Per-ZT summary table contains, for each cell type and each of the 6 ZT time points, a mean and standard deviation of predicted circadian time (in hours, 0-24)
- Standard deviation of bootstrapped predictions should be greater for dermal immune cells than for dermal fibroblasts (significant at five of six ZTs), matching the paper's finding of greater phase heterogeneity in immune cells
- Rao's Test for Homogeneity and/or Wallraff Test of Angular Distances p-values are reported for every ZT comparison, not just the significant ones
- All predicted times entering the tests are wrapped to the 0-24 h clock via convert_to_24hr (no out-of-range values such as negative hours or hours > 24)
- Each test is based on exactly 500 bootstrap predictions per cell type per ZT, as specified in the workflow

## Limitations

- The claim of phase-heterogeneity differences between dermal fibroblasts and immune cells relies only on computational analysis with no experimental validation ('Due to technological constraints, our claim on differences in phase heterogeneity between dermal fibroblasts and immune cells relies on the computational analysis')
- tauFisher cannot predict circadian time at the single-cell level due to high scRNAseq dropout, so the angular tests operate on pseudobulk bootstrap replicates rather than single-cell phases
- tauFisher outputs no uncertainty or confidence scores, so dispersion must be estimated by bootstrapping rather than by per-prediction error bars

## Evidence

- [other] Compare prediction distributions across cell types at each ZT with Rao's Tests for Homogeneity and the Wallraff Test of Angular Distances (circular statistics).: [evidence span withheld — anchor status: not_found]
- [results] Bootstrapping pseudobulk replicates and predicting circadian time with tauFisher showed the prediction distribution had a significantly greater standard deviation for immune cells than fibroblasts at five of six ZTs.: [evidence span withheld — anchor status: not_found]
- [intro] Incorporating tauFisher with bootstrapping revealed that circadian phase heterogeneity contributes to the dampened collective rhythm as well as fewer rhythmic genes found in dermal immune cells.: "Incorporating tauFisher with bootstrapping revealed that circadian phase heterogeneity contributes to the dampened collective rhythm as well as fewer rhythmic genes found in dermal immune cells."
- [discussion] Due to technological constraints, our claim on differences in phase heterogeneity between dermal fibroblasts and immune cells relies on the computational analysis: "Due to technological constraints, our claim on differences in phase heterogeneity between dermal fibroblasts and immune cells relies on the computational analysis"
- [readme] tauFisher requires R >=3.5.0 and has been tested on R 4.0.5: [evidence span withheld — anchor status: external_doc]
