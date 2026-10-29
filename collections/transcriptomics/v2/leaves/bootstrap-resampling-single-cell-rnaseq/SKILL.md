---
name: bootstrap-resampling-single-cell-rnaseq
description: Use when scRNAseq cells within a ZT time point must be converted to pseudobulk
  replicates to overcome dropout, so bootstrapped tauFisher circadian-time predictions
  can be compared for phase-heterogeneity differences between cell types.
when_to_use_negative:
- When the goal is to assign circadian time to individual single cells directly —
  tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and thus
  cannot predict circadian time at a single-cell level, which is precisely why pseudobulking
  is required
- When the input is bulk transcriptomic data with a complete training time series
  — the bootstrap/pseudobulk step is unnecessary; run train_tauFisher/test_tauFisher
  directly on the bulk samples
- When predicting timescales of diseased samples using a model trained on healthy
  data — a time series of transcriptomic data from diseased individuals would be required
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3391
tools:
- name: tauFisher
  role: Trains on periodic genes and predicts circadian time for each bootstrapped
    pseudobulk replicate via test_tauFisher; predictions are wrapped to 0-24 h with
    convert_to_24hr
  repo: github.com/micnngo/tauFisher
- name: dplyr
  role: R package dependency used for data manipulation of cell-level and pseudobulk
    expression tables during bootstrapping
- name: stats
  role: R package dependency used for the statistical comparison (Rao's Tests for
    Homogeneity, Wallraff Test) of prediction distributions across cell types
- name: stringr
  role: R package dependency for string handling when assembling pseudobulk replicate
    identifiers and summary outputs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:a5c5bf5cf0b17f59b255e5748a263aded47b6f33e9cfb6bfe125140aae84f958
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dplyr
  - stats
  - stringr
  - tauFisher
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bootstrap-resampling-single-cell-rnaseq

## Summary

Bootstraps single-cell RNAseq data into log2-transformed pseudobulk replicates (sampling cells with replacement, n = 20% of cell count, 500 iterations) so tauFisher can predict circadian time per replicate and quantify circadian phase heterogeneity across cell types (e.g., dermal immune cells vs fibroblasts in GSE223109).

## When to use

Use when scRNAseq cells within a ZT time point must be converted to pseudobulk replicates to overcome dropout, so bootstrapped tauFisher circadian-time predictions can be compared for phase-heterogeneity differences between cell types.

## When NOT to use

- When the goal is to assign circadian time to individual single cells directly — tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single-cell level, which is precisely why pseudobulking is required
- When the input is bulk transcriptomic data with a complete training time series — the bootstrap/pseudobulk step is unnecessary; run train_tauFisher/test_tauFisher directly on the bulk samples
- When predicting timescales of diseased samples using a model trained on healthy data — a time series of transcriptomic data from diseased individuals would be required

## Inputs

- Single-cell RNAseq expression matrix from GSE223109 (mouse dermal skin), with cell-type annotations (dermal immune cells, dermal fibroblasts) and ZT time-point metadata (6 ZTs)
- Trained tauFisher model object (predictor genes, scaled pairwise-difference parameters, PCA embedding) from train_tauFisher

## Outputs

- 500 pseudobulk expression replicates per cell type per ZT, log2(x+1) transformed
- Predicted circadian time (0-24 h) for each bootstrap replicate
- per_zt_prediction_summary.csv containing per-ZT means, standard deviations, and radar-plot data per cell type
- Circular-statistics test results (Rao's Tests for Homogeneity, Wallraff Test of Angular Distances) per ZT

## How to apply

At each of the 6 ZT time points in GSE223109, randomly sample n cells per cell type with replacement, where n equals 20% of that cell type's cell count, repeating for 500 bootstrap iterations. Sum expression over the sampled cells within each iteration to build pseudobulk replicates and log2-transform (log2(x+1)), because tauFisher cannot run on single cells due to scRNAseq dropout. For each pseudobulk replicate, subset to the trained tauFisher predictor genes (top ten statistically significant 24-hour period genes), compute and scale pairwise gene-expression differences, project onto the trained PCA embedding, and predict circadian time with test_tauFisher. Wrap predictions to the 0-24 h clock with convert_to_24hr, then compute per-cell-type means and standard deviations of predicted time at each ZT. Test whether prediction distributions differ between cell types (immune cells vs fibroblasts) using circular statistics: Rao's Tests for Homogeneity and the Wallraff Test of Angular Distances. Save the per-ZT summary (means, SDs, radar-plot data) and test results as per_zt_prediction_summary.csv.

## Related tools

- **tauFisher** (Trains on periodic genes and predicts circadian time for each bootstrapped pseudobulk replicate via test_tauFisher; predictions are wrapped to 0-24 h with convert_to_24hr) — github.com/micnngo/tauFisher
- **dplyr** (R package dependency used for data manipulation of cell-level and pseudobulk expression tables during bootstrapping)
- **stats** (R package dependency used for the statistical comparison (Rao's Tests for Homogeneity, Wallraff Test) of prediction distributions across cell types)
- **stringr** (R package dependency for string handling when assembling pseudobulk replicate identifiers and summary outputs)

## Examples

```
result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes, input_file="pseudobulk_boot.csv"); pred_24h <- convert_to_24hr(result_JTK)
```

## Evaluation signals

- The dispersion (SD) of the 500 bootstrapped circadian-phase predictions should be greater for dermal immune cells than for dermal fibroblasts, with significant differences at five of six ZTs per the paper's finding
- All predictions fall within the 0-24 h circadian clock after convert_to_24hr wrapping
- Each bootstrap iteration's pseudobulk vector equals the sum of exactly n sampled cells (n = 20% of the cell type's cell count) after log2(x+1) transformation
- Pseudobulk replicates must be restricted to the trained tauFisher predictor-gene subset (top ten statistically significant 24-hour period genes) before test_tauFisher projection
- per_zt_prediction_summary.csv exists with per-ZT means, standard deviations, radar-plot data, and Rao/Wallraff test results for both cell types

## Limitations

- tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single-cell level; inference on cells is only possible through bootstrapped pseudobulk aggregates
- The claim of phase-heterogeneity differences between dermal fibroblasts and immune cells relies on computational analysis only, with no experimental validation due to technological constraints
- tauFisher provides no uncertainty quantification / confidence scores for individual predictions, so bootstrap SD is the only dispersion measure available

## Evidence

- [abstract] Greater circadian phase heterogeneity may explain the dampened rhythm of collective core clock gene expression in dermal immune cells compared to dermal fibroblasts: "greater circadian phase heterogeneity may explain the dampened rhythm of collective core clock gene expression in dermal immune cells compared to dermal fibroblasts"
- [intro] Bootstrapping with tauFisher reveals heterogeneity in dermal immune cells: "Incorporating tauFisher with bootstrapping revealed that circadian phase heterogeneity contributes to the dampened collective rhythm as well as fewer rhythmic genes found in dermal immune cells."
- [discussion] tauFisher cannot run on single cells due to dropout: "it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level"
- [methods] test_tauFisher invocation pattern from tauFisher package: "result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes,"
- [readme] tauFisher installation from GitHub README: [evidence span withheld — anchor status: external_doc]
