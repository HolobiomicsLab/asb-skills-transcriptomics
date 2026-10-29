---
name: circadian-phase-prediction-pseudobulk
description: Use when single-cell transcriptomic data lack timestamps and you need
  per-cell-type circadian time estimates, e.g., comparing circadian phase heterogeneity
  across cell types via bootstrapped pseudobulk tauFisher predictions.
when_to_use_negative:
- When you need circadian time of individual cells rather than cell-type pseudobulk
  aggregates — tauFisher cannot overcome the high sequencing dropout rate in scRNAseq
  and cannot predict circadian time at a single-cell level
- When the sample comes from human blood or a circadian-disturbed/diseased context
  — tauFisher's performance on human blood samples is subpar and it is not yet validated
  on diseased transcriptomic datasets
- When you need confidence scores or uncertainty quantification per prediction — tauFisher
  does not output confidence scores (consider alternatives like Tempo)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: tauFisher
  role: train (train_tauFisher) and apply (test_tauFisher) the circadian time predictor
    on pseudobulk replicates; provides convert_to_24hr for wrapping predictions
  repo: github.com/micnngo/tauFisher
- name: MetaCycle
  role: identify periodic genes (find_periodic_genes / JTK) used to select the top
    ten 24-hour predictor genes
- name: dplyr
  role: data manipulation for bootstrapping and summarizing per-ZT predictions
- name: nnet
  role: multinomial logistic regression on the first two principal components in tauFisher
    training
- name: fda
  role: functional data analysis with Fourier basis expansion (K=5) to smooth time-expression
    curves
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:7ddbb2c605cdb61f2b4c8e204d299f81233aa164f4bf8eb9794029c8638e293c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaCycle
  - dplyr
  - fda
  - nnet
  - stats
  - stringr
  - tauFisher
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# circadian-phase-prediction-pseudobulk

## Summary

Predicts circadian time from single-cell RNA-seq by summing cell expression into pseudobulk replicates and applying the tauFisher pipeline (trained on periodic genes), enabling per-cell-type circadian phase assignment and bootstrapping-based phase heterogeneity analysis.

## When to use

Use when single-cell transcriptomic data lack timestamps and you need per-cell-type circadian time estimates, e.g., comparing circadian phase heterogeneity across cell types via bootstrapped pseudobulk tauFisher predictions.

## When NOT to use

- When you need circadian time of individual cells rather than cell-type pseudobulk aggregates — tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and cannot predict circadian time at a single-cell level
- When the sample comes from human blood or a circadian-disturbed/diseased context — tauFisher's performance on human blood samples is subpar and it is not yet validated on diseased transcriptomic datasets
- When you need confidence scores or uncertainty quantification per prediction — tauFisher does not output confidence scores (consider alternatives like Tempo)

## Inputs

- raw scRNAseq sequencing output (processed with CellRanger 3.1.0, MM10 reference)
- QC-filtered, SCTransform-normalized single-cell expression matrix with cell-type labels and ZT time-point metadata
- trained tauFisher predictor gene list (top ten significant 24-hour period genes, JTK_pvalue < 0.05) with PCA embedding from train_tauFisher
- GEO reference time-series datasets (e.g., GSE157077, GSE117295, GSE223109)

## Outputs

- per-replicate predicted circadian times (wrapped to 0–24 h) for each cell type and ZT time point
- per-ZT summary table of predicted-time means and standard deviations per cell type (per_zt_prediction_summary.csv) with radar-plot data
- circular-statistics test results (Rao's Tests for Homogeneity, Wallraff Test of Angular Distances) comparing prediction distributions across cell types

## How to apply

Process raw scRNAseq with CellRanger (MM10 reference), apply QC filters (850–7800 features, <13% mitochondrial genes), then SCTransform normalization and integration of 3250 features, followed by PCA and Louvain clustering to define cell types. Because tauFisher cannot run at single-cell level due to dropout, sum expression over randomly sampled cells (with replacement) into pseudobulk replicates per cell type/time point and log2-transform (log2(x+1)). Train tauFisher on a reference time series by selecting the top ten statistically significant 24-hour period genes (JTK_pvalue < 0.05), computing and scaling pairwise gene-expression differences, projecting via PCA, and fitting multinomial logistic regression on the first two principal components; then use test_tauFisher to predict circadian time of pseudobulk replicates. Wrap predictions to the 0–24 h clock with convert_to_24hr, and quantify phase heterogeneity via per-cell-type standard deviations plus circular statistics (Rao's Tests for Homogeneity, Wallraff Test of Angular Distances).

## Related tools

- **tauFisher** (train (train_tauFisher) and apply (test_tauFisher) the circadian time predictor on pseudobulk replicates; provides convert_to_24hr for wrapping predictions) — github.com/micnngo/tauFisher
- **MetaCycle** (identify periodic genes (find_periodic_genes / JTK) used to select the top ten 24-hour predictor genes)
- **dplyr** (data manipulation for bootstrapping and summarizing per-ZT predictions)
- **nnet** (multinomial logistic regression on the first two principal components in tauFisher training)
- **fda** (functional data analysis with Fourier basis expansion (K=5) to smooth time-expression curves)

## Examples

```
devtools::install_github("micnngo/tauFisher", build_vignettes = TRUE); genes <- find_periodic_genes(input_file=meta2d_file, ...); trained_JTK <- train_tauFisher(periodic_genes=genes$JTK, ...); result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes, ...)
```

## Evaluation signals

- Predicted-time dispersion check: standard deviation of bootstrapped circadian-phase predictions should be greater for dermal immune cells than for fibroblasts, consistent with the paper's heterogeneity finding (significant at five of six ZTs)
- Prediction distributions across cell types at each ZT should show significance in Rao's Tests for Homogeneity and/or the Wallraff Test of Angular Distances where heterogeneity exists
- Per-ZT summary table must contain means, standard deviations, and radar-plot data for each cell type, with all predictions in the 0–24 h range after convert_to_24hr
- Bootstrap settings invariant: 500 iterations sampling n = 20% of the immune-cell count with replacement at each of the 6 ZT time points
- Cross-check trained predictor set: exactly the top ten statistically significant 24-hour period genes (JTK_pvalue < 0.05) should feed test_tauFisher

## Limitations

- tauFisher cannot predict circadian time at a single-cell level due to the high sequencing dropout rate in scRNAseq, hence pseudobulking is required
- The claim of phase heterogeneity differences relies only on computational analysis with no experimental validation
- tauFisher does not output confidence scores, and the choice of top ten diurnal genes is not guaranteed optimal
- Within-sample normalization assumes gene-pair differences depend solely on circadian time; performance on human blood samples is subpar

## Evidence

- [other] Bootstrapping pseudobulk replicates and predicting circadian time with tauFisher showed the prediction distribution had a significantly greater standard deviation for immune cells than fibroblasts at five of six ZTs: [evidence span withheld — anchor status: not_found]
- [discussion] tauFisher cannot run on single cells due to dropout: "it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level"
- [methods] training and testing functions from methods: "trained_JTK <- train_tauFisher(periodic_genes=genes$JTK,"
- [discussion] multinomial regression on first two PCs: "linearly transforming the scaled differences using principal component analysis and fitting a multinomial logistic regression on the first two principal components"
- [discussion] top ten predictor genes filter: "It then selects the top ten statistically significant genes with a 24-hour period length."
- [readme] R package dependencies from README: [evidence span withheld — anchor status: external_doc]
- [abstract] heterogeneity finding from abstract: "greater circadian phase heterogeneity may explain the dampened rhythm of collective core clock gene expression in dermal immune cells compared to dermal fibroblasts"
