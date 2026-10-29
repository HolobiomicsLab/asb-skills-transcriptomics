---
name: prediction-accuracy-and-rmse-evaluation
description: Use when you have predicted vs. true circadian times (ZT hours) from
  a transcriptomic time-prediction pipeline and need standardized 2-hour accuracy
  and RMSE metrics, e.g. for 80/20-split benchmarks or leave-one-out cross-validation.
when_to_use_negative:
- You need single-cell-level time prediction — tauFisher cannot overcome the high
  sequencing dropout rate in scRNAseq and cannot predict circadian time at a single-cell
  level
- You want uncertainty quantification or confidence scores — tauFisher does not output
  confidence scores, so accuracy/RMSE cannot be complemented with per-prediction confidence
  here
- The test data come from a diseased system and you expect healthy-trained predictions
  on the diseased timescale — a time series of transcriptomic data from diseased individuals
  is required for that
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tauFisher
  role: Provides calc_error for scoring predictions (within 2 hours = correct) and
    train_tauFisher/test_tauFisher for generating predictions to evaluate
  repo: github.com/micnngo/tauFisher
- name: ZeitZeiger
  role: Comparator method whose accuracy and RMSE are computed under the same 2-hour
    calc_error rule in the benchmark
- name: TimeSignatR
  role: Comparator method (two training samples 12 hours apart) evaluated with identical
    accuracy and RMSE metrics
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:24a8fdd12b0d8fac3f333fdcbf64aa20aad8b3a412166c6d03d447f96ad7716a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TimeSignatR
  - ZeitZeiger
  - fda
  - nnet
  - stats
  - tauFisher
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# prediction-accuracy-and-rmse-evaluation

## Summary

Evaluation protocol for circadian-time prediction methods (tauFisher, ZeitZeiger, TimeSignatR) that scores each predicted circadian time with tauFisher's calc_error rule — a prediction is correct if within 2 hours of the true time — and reports 2-hour accuracy and RMSE per dataset, method, and cross-validation scheme.

## When to use

Use when you have predicted vs. true circadian times (ZT hours) from a transcriptomic time-prediction pipeline and need standardized 2-hour accuracy and RMSE metrics, e.g. for 80/20-split benchmarks or leave-one-out cross-validation.

## When NOT to use

- You need single-cell-level time prediction — tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and cannot predict circadian time at a single-cell level
- You want uncertainty quantification or confidence scores — tauFisher does not output confidence scores, so accuracy/RMSE cannot be complemented with per-prediction confidence here
- The test data come from a diseased system and you expect healthy-trained predictions on the diseased timescale — a time series of transcriptomic data from diseased individuals is required for that

## Inputs

- Predicted circadian times (hours) from test_tauFisher, ZeitZeiger, or TimeSignatR
- True circadian time labels (ZT hours) for each sample
- Expression matrices (genes as rows, samples as columns) for benchmark datasets GSE54650, GSE54651, GSE56931, GSE113883, GSE38622, GSE157077, and GSE83855 ZT5-ZT9 / ZT0-ZT4 schedules
- Cross-validation partition definitions (100 random 80/20 train/test splits, or leave-one-out)

## Outputs

- benchmark_results.csv with accuracy and RMSE aggregated over 100 partitions for each dataset-by-method combination
- gse83855_loocv_metrics.csv with accuracy and RMSE for both time-restricted feeding schedules
- Per-prediction correct/incorrect flags from calc_error (correct if within 2 hours of true time)

## How to apply

Score every prediction with tauFisher's calc_error function, counting a prediction as correct if it is within 2 hours of the true circadian time, then compute accuracy (fraction correct) and RMSE over the held-out samples. For the within-platform benchmark, run 100 random iterations per dataset (e.g. GSE54650, GSE54651, GSE56931, GSE113883, GSE38622, GSE157077) splitting samples 80/20 into train/test, predict held-out samples with test_tauFisher and comparator models (TimeSignatR with two training samples 12 hours apart), and aggregate accuracy and RMSE per dataset-by-method combination into benchmark_results.csv. For the disturbed-system evaluation (GSE83855 ZT5-ZT9 and ZT0-ZT4 time-restricted feeding schedules), run leave-one-out cross-validation: retrain on the remaining samples for each held-out sample, predict, score, and assemble accuracy and RMSE for both schedules into gse83855_loocv_metrics.csv. Validate by checking the LOOCV table reproduces the reported values — accuracy 0.875 and RMSE 2.236 for ZT5-ZT9, accuracy 1.0 and RMSE 1.061 for ZT0-ZT4 — and confirm tauFisher with JTK_Cycle genes outperforms the Lomb-Scargle gene set and matches two-sample TimeSignatR on most mouse datasets.

## Related tools

- **tauFisher** (Provides calc_error for scoring predictions (within 2 hours = correct) and train_tauFisher/test_tauFisher for generating predictions to evaluate) — github.com/micnngo/tauFisher
- **ZeitZeiger** (Comparator method whose accuracy and RMSE are computed under the same 2-hour calc_error rule in the benchmark)
- **TimeSignatR** (Comparator method (two training samples 12 hours apart) evaluated with identical accuracy and RMSE metrics)

## Examples

```
correct <- (abs(predicted_zt - true_zt) <= 2); accuracy <- mean(correct); rmse <- sqrt(mean((predicted_zt - true_zt)^2))
```

## Evaluation signals

- Every prediction is scored with calc_error under the 2-hour correctness rule; accuracy = fraction of predictions within 2 hours of true time
- GSE83855 LOOCV table reproduces reported values: accuracy 0.875 / RMSE 2.236 for ZT5-ZT9, accuracy 1.0 / RMSE 1.061 for ZT0-ZT4
- RMSE is computed per schedule/dataset-method combination over held-out samples and aggregated over the 100 random 80/20 partitions
- tauFisher with JTK_Cycle predictor genes outperforms the Lomb-Scargle gene set on the benchmark datasets
- ZeitZeiger failures from linear dependency issues are recorded rather than silently dropped from the benchmark table

## Limitations

- tauFisher performance on human blood samples is subpar, so benchmark accuracy/RMSE conclusions are strongest for mouse datasets
- ZeitZeiger failed to run for several of the datasets due to linear dependency issues, and its success was dependent on the assay platform
- tauFisher was not yet validated on circadian-disturbed/diseased transcriptomic datasets beyond the time-restricted feeding evaluation

## Evidence

- [results] scoring each prediction with calc_error under the rule that a prediction is correct if within 2 hours of the true time, and compute accuracy and RMSE per method: [evidence span withheld — anchor status: not_found]
- [results] LOOCV reported metrics: [evidence span withheld — anchor status: not_found]
- [discussion] tauFisher comparable to TimeSignatR: "tauFisher using one test sample achieved similar accuracy and RMSE for most of the datasets when compared to TimeSignatR which required two test samples"
- [discussion] ZeitZeiger failure mode: "ZeitZeiger failed to run for several of the datasets due to linear dependency issues, and its success was dependent on the assay platform"
- [readme] README runtime expectation: [evidence span withheld — anchor status: external_doc]
