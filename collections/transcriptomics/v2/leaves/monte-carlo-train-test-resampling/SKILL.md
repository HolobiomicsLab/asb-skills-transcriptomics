---
name: monte-carlo-train-test-resampling
description: Use when comparing circadian-time prediction methods on the same time-course
  datasets and you need variance-controlled, partition-averaged accuracy/RMSE rather
  than a single train/test split.
when_to_use_negative:
- 'You only have one data set and a single train/test split suffices (follow the SingleDataset
  vignette: e.g. train on two replicates, predict the third) rather than 100 Monte
  Carlo partitions'
- The dataset lacks timestamped time-course samples with rhythmic genes (no MetaCycle
  periodic gene sets can be derived), so partition-based training of tauFisher/TimeSignatR
  is not meaningful
- You are evaluating single-cell-level circadian time prediction, which tauFisher
  cannot do due to high scRNAseq dropout
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: tauFisher
  role: Method under evaluation; trained per partition via train_tauFisher (numbasis=5
    Fourier basis) and used to predict held-out samples with test_tauFisher
  repo: github.com/micnngo/tauFisher
- name: MetaCycle
  role: Detects rhythmic predictor genes (via run_metacycle wrapper, method=c('JTK','LS'))
    that define the feature sets used in each partition's tauFisher training
- name: ZeitZeiger
  role: Comparator method trained on the same 80% training splits; failures from linear
    dependency issues are recorded in the benchmark
- name: TimeSignatR
  role: Comparator method trained per partition using two training samples 12 hours
    apart, providing the two-sample reference benchmark for one-sample tauFisher
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:03a25048f4661284efacf93cca7f36f5714ae7857b697367a97feeaf75549a47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaCycle
  - TimeSignatR
  - ZeitZeiger
  - tauFisher
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# monte-carlo-train-test-resampling

## Summary

Benchmark protocol that repeatedly partitions a circadian transcriptomic time-course 80/20 into train/test sets over 100 random iterations, trains comparator models (tauFisher, ZeitZeiger, TimeSignatR) per partition, and aggregates 2-hour accuracy and RMSE per dataset-by-method combination.

## When to use

Use when comparing circadian-time prediction methods on the same time-course datasets and you need variance-controlled, partition-averaged accuracy/RMSE rather than a single train/test split.

## When NOT to use

- You only have one data set and a single train/test split suffices (follow the SingleDataset vignette: e.g. train on two replicates, predict the third) rather than 100 Monte Carlo partitions
- The dataset lacks timestamped time-course samples with rhythmic genes (no MetaCycle periodic gene sets can be derived), so partition-based training of tauFisher/TimeSignatR is not meaningful
- You are evaluating single-cell-level circadian time prediction, which tauFisher cannot do due to high scRNAseq dropout

## Inputs

- Preprocessed bulk transcriptomic circadian time-course matrices from GEO (e.g. GSE54650, GSE54651, GSE56931, GSE113883, GSE38622, GSE157077) with uppercase harmonized gene identifiers and sample timestamps
- Periodic predictor gene sets from MetaCycle (JTK_Cycle and Lomb-Scargle, top ten significant 24-hour-period genes per dataset)
- Number of resampling iterations (100) and split fraction (80/20)

## Outputs

- benchmark_results.csv with accuracy and RMSE aggregated over 100 partitions for each dataset-by-method combination (tauFisher-JTK, tauFisher-LS, ZeitZeiger, TimeSignatR)
- Per-iteration prediction errors from calc_error scored at 2-hour tolerance

## How to apply

For each GEO time-course dataset (e.g. GSE54650, GSE54651, GSE56931, GSE113883, GSE38622, GSE157077), run 100 random iterations splitting samples 80/20 into train/test. On each training split, train tauFisher with train_tauFisher using a numbasis=5 Fourier basis (once per JTK_Cycle- and Lomb-Scargle-based periodic gene set selected by find_periodic_genes with JTK_pvalue < 0.05), train ZeitZeiger, and train TimeSignatR using two training samples 12 hours apart. Predict held-out samples with test_tauFisher and the comparator models, score each prediction with calc_error using the rule that a prediction is correct if within 2 hours of the true time, and compute accuracy and RMSE. Aggregate accuracy and RMSE over the 100 partitions per dataset-by-method combination and write benchmark_results.csv.

## Related tools

- **tauFisher** (Method under evaluation; trained per partition via train_tauFisher (numbasis=5 Fourier basis) and used to predict held-out samples with test_tauFisher) — github.com/micnngo/tauFisher
- **MetaCycle** (Detects rhythmic predictor genes (via run_metacycle wrapper, method=c('JTK','LS')) that define the feature sets used in each partition's tauFisher training)
- **ZeitZeiger** (Comparator method trained on the same 80% training splits; failures from linear dependency issues are recorded in the benchmark)
- **TimeSignatR** (Comparator method trained per partition using two training samples 12 hours apart, providing the two-sample reference benchmark for one-sample tauFisher)

## Examples

```
R> for (i in 1:100) { s <- sample(nrow(X), floor(0.8*nrow(X))); m <- train_tauFisher(periodic_genes=genes$JTK, df=X[s,], timepoints=time[s]); r <- test_tauFisher(chosen_genes=m$Genes, df=X[-s,]) }
```

## Evaluation signals

- Each of the 100 iterations should contain a disjoint 80/20 partition; accuracy and RMSE must be computed for every method on every dataset before aggregation into benchmark_results.csv
- tauFisher with JTK_Cycle genes should outperform tauFisher with the Lomb-Scargle gene set, and its accuracy/RMSE should be similar to two-sample TimeSignatR on most mouse benchmark datasets
- A prediction is counted correct only if within 2 hours of the true circadian time as scored by calc_error
- Record and distinguish ZeitZeiger failures caused by linear dependency issues rather than counting them as low accuracy
- Per-method RMSE should be reported alongside accuracy so that near-threshold misses are visible

## Limitations

- ZeitZeiger failed to run on several datasets due to linear dependency issues, and its success was dependent on the assay platform
- tauFisher's benchmark performance on human blood samples is subpar despite strong results on mouse datasets
- The choice of the top ten diurnal predictor genes is a safe but not guaranteed-optimal configuration across datasets

## Evidence

- [methods] For each of 100 random iterations per dataset, split samples 80/20 into train/test, and on the training split train tauFisher (train_tauFisher with a numbasis=5 Fourier basis, once per predictor gene set), ZeitZeiger, and TimeSignatR: [evidence span withheld — anchor status: not_found]
- [methods] Prediction scoring rule and error metric: [evidence span withheld — anchor status: not_found]
- [discussion] Expected comparative outcome of the resampled benchmark: "tauFisher using one test sample achieved similar accuracy and RMSE for most of the datasets when compared to TimeSignatR which required two test samples"
- [discussion] Comparator failure mode that must be tracked during aggregation: "ZeitZeiger failed to run for several of the datasets due to linear dependency issues, and its success was dependent on the assay platform"
- [methods] tauFisher function signatures used per partition: "trained_JTK <- train_tauFisher(periodic_genes=genes$JTK,"
