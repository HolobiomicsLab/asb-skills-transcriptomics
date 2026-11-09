---
name: cross-cell-type-phase-heterogeneity-analysis
description: Use when single-cell RNAseq data from multiple cell types is sampled
  across circadian time points and the goal is to test whether circadian phase heterogeneity
  (not just mean phase) differs between cell types.
when_to_use_negative:
- When tauFisher's performance is already subpar for the dataset type (e.g., human
  blood samples) — dispersion comparisons inherit prediction noise
- When the goal is mean circadian phase estimation of bulk samples rather than comparing
  within-cell-type phase dispersion
- When single-cell data lacks multiple ZT time points — bootstrapping at each ZT requires
  a sampled time series to compare per time point
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: tauFisher
  role: Trains the circadian-time predictor (train_tauFisher) and predicts circadian
    time for each bootstrapped pseudobulk replicate (test_tauFisher); convert_to_24hr
    wraps predictions to the 0-24 h clock
  repo: https://github.com/micnngo/tauFisher
- name: dplyr
  role: Data manipulation for building bootstrap pseudobulk replicates and summarizing
    per-ZT prediction statistics
- name: stats
  role: R package providing the statistical routines used in computing prediction
    distribution summaries and supporting circular-statistics tests
- name: stringr
  role: String handling for matching cell-type labels and constructing per-ZT summary
    identifiers
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:9f101090edb75ace2a3a4ab301f34ce7721656eb324a1832f6fc13f73e5e8622
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - dplyr
  - stats
  - stringr
  - tauFisher
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-cell-type-phase-heterogeneity-analysis

## Summary

Quantifies whether circadian phases are more heterogeneous in one cell type than another by bootstrapping pseudobulk replicates from single-cell data, predicting circadian time per replicate with tauFisher, and comparing prediction dispersion (SD) and circular distributions across cell types at each ZT time point.

## When to use

Use when single-cell RNAseq data from multiple cell types is sampled across circadian time points and the goal is to test whether circadian phase heterogeneity (not just mean phase) differs between cell types.

## When NOT to use

- When tauFisher's performance is already subpar for the dataset type (e.g., human blood samples) — dispersion comparisons inherit prediction noise
- When the goal is mean circadian phase estimation of bulk samples rather than comparing within-cell-type phase dispersion
- When single-cell data lacks multiple ZT time points — bootstrapping at each ZT requires a sampled time series to compare per time point

## Inputs

- Single-cell RNAseq expression matrix from mouse dermal skin (e.g., GEO accession GSE223109), cell-type annotated, with 6 ZT time points
- Trained tauFisher predictor: chosen_genes and PCA embedding from train_tauFisher run on a reference circadian dataset
- Cell-type labels and ZT time-point metadata per cell

## Outputs

- per_zt_prediction_summary.csv containing per-ZT mean and standard deviation of predicted circadian time per cell type, plus radar-plot data
- Circular-statistics test results (Rao's Tests for Homogeneity, Wallraff Test of Angular Distances) comparing prediction distributions per ZT
- 500 bootstrapped predicted-time values per cell type per ZT (0-24 h clock)

## How to apply

At each of the 6 ZT time points in GSE223109, randomly sample n cells per cell type with replacement (n = 20% of the immune-cell count) for 500 bootstrap iterations, summing expression over sampled cells per iteration to build pseudobulk replicates, then log2-transform (log2(x+1)) since tauFisher cannot run on single cells due to dropout. For each pseudobulk replicate, subset to the trained tauFisher predictor genes (top ten statistically significant 24-hour period genes), compute and scale pairwise gene-expression differences, project onto the trained PCA embedding, and use test_tauFisher to predict circadian time for all 500 replicates per condition. Wrap predictions to the 0-24 h clock with convert_to_24hr and compute per-cell-type means and standard deviations of predicted time at each ZT. Finally, compare prediction distributions across cell types with Rao's Tests for Homogeneity and the Wallraff Test of Angular Distances (circular statistics), saving per-ZT means, standard deviations, and test results as per_zt_prediction_summary.csv.

## Related tools

- **tauFisher** (Trains the circadian-time predictor (train_tauFisher) and predicts circadian time for each bootstrapped pseudobulk replicate (test_tauFisher); convert_to_24hr wraps predictions to the 0-24 h clock) — https://github.com/micnngo/tauFisher
- **dplyr** (Data manipulation for building bootstrap pseudobulk replicates and summarizing per-ZT prediction statistics)
- **stats** (R package providing the statistical routines used in computing prediction distribution summaries and supporting circular-statistics tests)
- **stringr** (String handling for matching cell-type labels and constructing per-ZT summary identifiers)

## Examples

```
result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes, input_file=pseudobulk_boot_matrix)
```

## Evaluation signals

- The dispersion (standard deviation) of bootstrapped circadian-phase predictions should be greater for dermal immune cells than for dermal fibroblasts, consistent with the paper's finding (greater SD at five of six ZTs)
- Prediction distributions should differ significantly across cell types at individual ZTs per Rao's Tests for Homogeneity and the Wallraff Test of Angular Distances
- All 500 bootstrap predictions per cell type per ZT must lie within the wrapped 0-24 h clock after convert_to_24hr
- Pseudobulk replicates must be restricted to the trained tauFisher predictor genes (top ten 24-hour period genes) before prediction
- per_zt_prediction_summary.csv must contain per-ZT means, standard deviations, radar-plot data, and statistical test results for both cell types

## Limitations

- tauFisher cannot predict circadian time at the single-cell level due to high scRNAseq dropout, hence the pseudobulk bootstrapping workaround
- The claim of phase heterogeneity differences relies only on computational analysis, with no experimental validation
- tauFisher outputs no uncertainty quantification or confidence scores on individual predictions

## Evidence

- [results] Bootstrapping pseudobulk replicates and predicting circadian time with tauFisher showed the prediction distribution had a significantly greater standard deviation for immune cells than fibroblasts at five of six ZTs: [evidence span withheld — anchor status: not_found]
- [intro] Incorporating tauFisher with bootstrapping revealed that circadian phase heterogeneity contributes to the dampened collective rhythm as well as fewer rhythmic genes found in dermal immune cells.: "Incorporating tauFisher with bootstrapping revealed that circadian phase heterogeneity contributes to the dampened collective rhythm as well as fewer rhythmic genes found in dermal immune cells."
- [discussion] it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level: "it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level"
- [discussion] Due to technological constraints, our claim on differences in phase heterogeneity between dermal fibroblasts and immune cells relies on the computational analysis: "Due to technological constraints, our claim on differences in phase heterogeneity between dermal fibroblasts and immune cells relies on the computational analysis"
- [methods] result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes,: "result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes,"
