---
name: circadian-time-prediction-benchmarking
description: Use when comparing circadian phase-prediction methods (e.g., tauFisher
  vs ZeitZeiger vs TimeSignatR) on timestamped bulk RNA time-course data, or when
  needing standardized accuracy/RMSE metrics with a ±2-hour correctness rule.
when_to_use_negative:
- Predicting circadian time from single-cell data without pseudobulk aggregation —
  tauFisher cannot overcome the high sequencing dropout rate in scRNAseq at single-cell
  level
- Training on healthy time-series to predict diseased-sample timescales — a time series
  from diseased individuals is required for that projection
- Datasets with no timestamped training samples or too few features to select ten
  significant 24-hour-period rhythmic genes
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: tauFisher
  role: 'Core pipeline: run_metacycle rhythm detection, find_periodic_genes, train_tauFisher
    (numbasis=5 Fourier basis FDA), test_tauFisher prediction, calc_error scoring'
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: Rhythm detection backend via meta2d (JTK_Cycle and Lomb-Scargle) called through
    tauFisher's run_metacycle wrapper
- name: ZeitZeiger
  role: Comparator method benchmarked on the same 80/20 train/test splits; failures
    due to linear dependency issues are recorded
- name: TimeSignatR
  role: Comparator method trained with two samples 12 hours apart; serves as the within-platform
    accuracy/RMSE baseline
- name: fda
  role: Functional data analysis smoothing of time-expression curves with a K=5 Fourier
    basis in train_tauFisher
- name: nnet
  role: nnet::multinom multinomial logistic regression on PC1 and PC2 for time-label
    classification
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:8cd7e866d62027333287833b44394c2f028b74c14fb70bbe0c6ecd1eb00c0c03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - MetaCycle
  - TimeSignatR
  - ZeitZeiger
  - fda
  - nnet
  - stats
  - tauFisher
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# circadian-time-prediction-benchmarking

## Summary

Benchmark tauFisher's single-sample circadian time predictions against ZeitZeiger and TimeSignatR on GEO bulk transcriptomic time-course datasets, scoring 2-hour-window accuracy and RMSE across randomized train/test partitions. Grounded in the tauFisher Nature Communications 2024 protocol.

## When to use

Use when comparing circadian phase-prediction methods (e.g., tauFisher vs ZeitZeiger vs TimeSignatR) on timestamped bulk RNA time-course data, or when needing standardized accuracy/RMSE metrics with a ±2-hour correctness rule.

## When NOT to use

- Predicting circadian time from single-cell data without pseudobulk aggregation — tauFisher cannot overcome the high sequencing dropout rate in scRNAseq at single-cell level
- Training on healthy time-series to predict diseased-sample timescales — a time series from diseased individuals is required for that projection
- Datasets with no timestamped training samples or too few features to select ten significant 24-hour-period rhythmic genes

## Inputs

- GEO bulk transcriptomic circadian time-course expression matrices (e.g., GSE54650, GSE54651, GSE56931, GSE113883, GSE38622, GSE157077) with circadian time labels
- Normalized expression matrices with genes as rows and samples as columns (e.g., normalized GSE38622 matrix)
- Time-point annotations for each sample (ZT/CT hours)

## Outputs

- benchmark_results.csv with accuracy and RMSE aggregated over 100 partitions for each dataset-by-method combination
- Per-dataset rhythmic predictor gene sets (JTK_Cycle- and Lomb-Scargle-based, top ten significant 24-hour-period genes)
- Per-iteration prediction vs. true-time error records scored by calc_error

## How to apply

Download and preprocess GEO circadian time-course datasets (e.g., GSE54650, GSE54651, GSE56931, GSE113883, GSE38622, GSE157077), harmonizing gene identifiers to uppercase gene names, averaging non-unique genes, filtering GSE56931's 38-hour wakefulness/recovery time points, and using the normalized GSE38622 matrix. Detect rhythmic predictor genes per dataset with tauFisher's run_metacycle wrapper of MetaCycle (method=c('JTK','LS')), then find_periodic_genes selecting JTK_pvalue < 0.05 and the top ten significant 24-hour-period genes. For each of 100 random iterations per dataset, split samples 80/20 into train/test; train tauFisher (train_tauFisher with numbasis=5 Fourier basis), ZeitZeiger, and TimeSignatR (two training samples 12 hours apart), then predict held-out samples with test_tauFisher and comparators. Score each prediction with calc_error under the rule that a prediction is correct if within 2 hours of the true time, aggregate accuracy and RMSE per dataset-by-method combination, and write benchmark_results.csv. Validation: tauFisher with JTK_Cycle genes should outperform the Lomb-Scargle set and match two-sample TimeSignatR accuracy/RMSE on most mouse datasets; record ZeitZeiger failures from linear dependency issues.

## Related tools

- **tauFisher** (Core pipeline: run_metacycle rhythm detection, find_periodic_genes, train_tauFisher (numbasis=5 Fourier basis FDA), test_tauFisher prediction, calc_error scoring) — https://github.com/micnngo/tauFisher
- **MetaCycle** (Rhythm detection backend via meta2d (JTK_Cycle and Lomb-Scargle) called through tauFisher's run_metacycle wrapper)
- **ZeitZeiger** (Comparator method benchmarked on the same 80/20 train/test splits; failures due to linear dependency issues are recorded)
- **TimeSignatR** (Comparator method trained with two samples 12 hours apart; serves as the within-platform accuracy/RMSE baseline)
- **fda** (Functional data analysis smoothing of time-expression curves with a K=5 Fourier basis in train_tauFisher)
- **nnet** (nnet::multinom multinomial logistic regression on PC1 and PC2 for time-label classification)

## Examples

```
trained_JTK <- train_tauFisher(periodic_genes=genes$JTK, ...); result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes, ...)
```

## Evaluation signals

- Reproduced benchmark_results.csv must show tauFisher (one test sample) achieving equal or higher 2-hour accuracy than two-sample TimeSignatR on six of ten mouse transcriptomic datasets
- tauFisher with JTK_Cycle-selected genes must outperform the Lomb-Scargle gene set on most datasets
- ZeitZeiger failures on specific datasets must be attributable to linear dependency issues and tracked per platform
- Within disturbed-system LOOCV (GSE83855), reproduce accuracy 0.875 / RMSE 2.236 for ZT5-ZT9 and accuracy 1.0 / RMSE 1.061 for ZT0-ZT4
- Every prediction must be scored with calc_error using the ±2-hour correctness rule; per-method RMSE and accuracy must be aggregated over 100 random 80/20 partitions per dataset

## Limitations

- tauFisher's performance on human blood samples is subpar despite good accuracy and RMSE on mouse benchmark datasets
- ZeitZeiger failed to run for several datasets due to linear dependency issues, and its success was dependent on the assay platform
- tauFisher cannot predict circadian time at single-cell level due to high scRNAseq dropout; pseudobulk aggregation is required
- Choosing the top ten diurnal genes does not guarantee the best performance, though it is a safe and reasonable choice across datasets
- tauFisher's within-sample normalization rests on an unverified assumption that gene-pair differences depend solely on circadian time

## Evidence

- [discussion] For six out of the ten transcriptomic datasets collected from mice, tauFisher achieved equal or higher 2-hour accuracy using one test sample than TimeSignatR using two test samples that are 12 hours apart.: "For six out of the ten transcriptomic datasets collected from mice, tauFisher achieved equal or higher 2-hour accuracy using one test sample than TimeSignatR using two test samples"
- [discussion] ZeitZeiger failed to run for several of the datasets due to linear dependency issues, and its success was dependent on the assay platform.: "ZeitZeiger failed to run for several of the datasets due to linear dependency issues, and its success was dependent on the assay platform"
- [discussion] tauFisher using one test sample achieved similar accuracy and RMSE for most of the datasets when compared to TimeSignatR which required two test samples.: "tauFisher using one test sample achieved similar accuracy and RMSE for most of the datasets when compared to TimeSignatR which required two test samples"
- [results] Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic.: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [readme] tauFisher requires R >=3.5.0 and has been tested on R 4.0.5. It also requires the following R packages: [evidence span withheld — anchor status: external_doc]
- [readme] devtools::install_github("micnngo/tauFisher", build_vignettes = TRUE): "devtools::install_github("micnngo/tauFisher", build_vignettes = TRUE) "
