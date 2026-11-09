---
name: binary-versus-multiclass-comparison
description: Use when a feature-selection benchmark table (misclassification percentage
  as a function of number of selected features) must be scanned to find per-dataset
  winners and summarize how recommendations differ between binary and multi-class
  classification problems.
when_to_use_negative:
- The goal is to select features for a single new dataset rather than to compare algorithm
  winners across an existing benchmark of datasets
- Performance is measured by something other than out-of-sample random-forest misclassification
  percentage over a feature-count sweep (e.g., regression RMSE or FDR on synthetic
  probes)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: pandas
  role: Loads and manipulates the reproduced benchmark table to find per-dataset column-minimum
    misclassification rates
- name: uts
  role: MATLAB feature-selection driver whose default K = M (or K = 30 for >30 features)
    defines the feature-count sweep recorded in the benchmark table
  repo: https://github.com/ThanasisTsanas/RRCT
- name: MATLAB 2021b
  role: Environment in which the original benchmark experiments (19 FS algorithms,
    RF misclassification evaluation) were run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:1a05821a91eda6bdbd4e234ba2c14ef8012cc08facd6c23e6b664671f3be71d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ThanasisTsanas/RRCT
  tools:
  - MATLAB
  - MATLAB 2021b
  - pandas
  - uts
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# binary-versus-multiclass-comparison

## Summary

Identifies the best-performing filter feature-selection algorithm per dataset from a reproduced random-forest misclassification benchmark, then contrasts which algorithms win in binary (e.g., Spambase, SPECTF, Relathe, Ovarian cancer) versus multi-class (e.g., Cardiotocography, Handwriting, SRBCT, Lymph) settings.

## When to use

Use when a feature-selection benchmark table (misclassification percentage as a function of number of selected features) must be scanned to find per-dataset winners and summarize how recommendations differ between binary and multi-class classification problems.

## When NOT to use

- The goal is to select features for a single new dataset rather than to compare algorithm winners across an existing benchmark of datasets
- Performance is measured by something other than out-of-sample random-forest misclassification percentage over a feature-count sweep (e.g., regression RMSE or FDR on synthetic probes)

## Inputs

- Reproduced benchmark table (CSV) of random-forest misclassification percentages per FS algorithm and per number of selected features (1…min(M,30))
- Dataset metadata mapping each benchmarked dataset (e.g., Relathe [1,427 × 4,322], Ovarian cancer [72 × 592], SRBCT [63 × 2,308]) to binary or multi-class response type

## Outputs

- best_fs_summary.csv tabulating dataset -> best algorithm -> minimum misclassification rate and binary/multi-class annotation
- A summary statement of how the winning filter FS algorithms differ between binary and multi-class settings

## How to apply

Load the reproduced benchmark table of random-forest misclassification percentages, recorded for 1…min(M,30) selected features per algorithm (the uts default is K = M, or K = 30 when there are more than 30 features), using pandas. For each dataset row-sweep, identify the algorithm achieving the column minimum misclassification percentage. Annotate each dataset as binary (Spambase, SPECTF, Relathe, Ovarian cancer) or multi-class (Cardiotocography, Handwriting, SRBCT, Lymph) based on its response variable. Tabulate dataset -> best algorithm -> misclassification rate and summarize the binary vs multi-class contrast (e.g., RRCT for Relathe and Ovarian cancer, HITON for Spambase, L1-LSMI for SPECTF, Information gain for Cardiotocography, LOGO for Handwriting), noting that RRCT was tied with a few competitors on multi-class SRBCT. Write the summary to best_fs_summary.csv.

## Related tools

- **pandas** (Loads and manipulates the reproduced benchmark table to find per-dataset column-minimum misclassification rates)
- **uts** (MATLAB feature-selection driver whose default K = M (or K = 30 for >30 features) defines the feature-count sweep recorded in the benchmark table) — https://github.com/ThanasisTsanas/RRCT
- **MATLAB 2021b** (Environment in which the original benchmark experiments (19 FS algorithms, RF misclassification evaluation) were run)

## Examples

```
import pandas as pd; df = pd.read_csv('benchmark_rf_misclassification.csv'); best = df.idxmin(axis=1)
```

## Evaluation signals

- Each dataset row's reported misclassification rate equals the column minimum in the reproduced benchmark table
- Every benchmarked dataset (Spambase, SPECTF, Relathe, Ovarian cancer, Cardiotocography, Handwriting, Lymph, SRBCT) appears exactly once in the summary
- Binary/multi-class annotation matches each dataset's response variable
- Feature-count sweep respects the min(M,30) convention consistent with the uts default K = M, or K = 30 if there are more than 30 features
- best_fs_summary.csv is produced with the dataset -> best algorithm -> misclassification rate schema

## Limitations

- Winners are tied on some datasets (RRCT tied with a few competing algorithms on multi-class SRBCT), so a single 'best' algorithm may not be unique
- Conclusions are specific to random-forest out-of-sample misclassification as the evaluation metric and to the eight real-world datasets benchmarked in the paper

## Evidence

- [abstract] RRCT was benchmarked against 19 FS algorithms on four synthetic and eight real-world datasets: "We empirically benchmark the performance of RRCT against 19 FS algorithms across four synthetic and eight real-world datasets"
- [results] Benchmark protocol: RF misclassification as a function of number of selected features: "as a function of the number of selected features presented into the random forest (RF)"
- [methods] uts K parameter defines the feature-count sweep: "K        -> number of features to be selected (integer value)"
- [results] Computing environment for the benchmark: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
- [abstract] RRCT is competitive on out-of-sample classification, motivating per-setting winner analysis: "RRCT is very competitive in both tasks, and we tentatively make suggestions on the generalizability and application of the best-performing FS algorithms"
