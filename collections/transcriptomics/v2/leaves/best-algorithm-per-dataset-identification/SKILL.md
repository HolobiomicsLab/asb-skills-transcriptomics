---
name: best-algorithm-per-dataset-identification
description: Use when a reproduced benchmark table of random-forest misclassification
  percentages (1…min(M,30) selected features per FS algorithm) is available and you
  need to name the algorithm per dataset and compare binary vs multi-class winners.
when_to_use_negative:
- You only have synthetic-dataset results (FDR-based probe-discard analysis), not
  out-of-sample random-forest misclassification percentages on real-world datasets
- The benchmark table was produced with a different feature-count sweep or classifier
  than RF with 1…min(M,30) features, so column minima are not comparable to the paper's
  setup
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: pandas
  role: Load and reduce the reproduced misclassification benchmark table to per-dataset
    column minima
- name: uts
  role: MATLAB tool whose default K = M (or K = 30 if more than 30 features) defines
    the feature-count sweep recorded in the benchmark table
  repo: github.com/ThanasisTsanas/RRCT
- name: MATLAB 2021b
  role: Environment in which the RRCT-vs-19-algorithms benchmark and random-forest
    misclassification experiments were run
  repo: github.com/ThanasisTsanas/RRCT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:e65c6b50b4727e25b2c5960e5ac9b4d2343a7eb6bf89f848514a468b50fa8c36
schema_version: 0.2.0
metadata:
  license_tier: open
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

# best-algorithm-per-dataset-identification

## Summary

Identifies which filter feature-selection algorithm achieved the lowest out-of-sample random-forest misclassification percentage on each real-world dataset, and contrasts the winners between binary (e.g., Relathe, Ovarian cancer, Spambase, SPECTF) and multi-class (e.g., SRBCT, Cardiotocography, Handwriting, Lymph) settings.

## When to use

Use when a reproduced benchmark table of random-forest misclassification percentages (1…min(M,30) selected features per FS algorithm) is available and you need to name the best algorithm per dataset and compare binary vs multi-class winners.

## When NOT to use

- You only have synthetic-dataset results (FDR-based probe-discard analysis), not out-of-sample random-forest misclassification percentages on real-world datasets
- The benchmark table was produced with a different feature-count sweep or classifier than RF with 1…min(M,30) features, so column minima are not comparable to the paper's setup

## Inputs

- Reproduced benchmark table (CSV/pandas DataFrame) of random-forest misclassification percentages, rows = number of selected features (1…min(M,30)), columns = FS algorithms, per real-world dataset
- Dataset metadata mapping each dataset to binary or multi-class response type (Spambase [4,601×57], SPECTF [187×44], Relathe [1,427×4,322], Ovarian cancer [72×592], Cardiotocography [2,129×21], Handwriting [2,000×649], Lymph [148×18], SRBCT [63×2,308])

## Outputs

- best_fs_summary.csv with one row per dataset: dataset -> best algorithm -> minimum misclassification rate
- Summary statement of binary vs multi-class winner differences

## How to apply

Load the reproduced benchmark table of random-forest misclassification percentages with pandas; the table records results for 1…min(M,30) selected features per algorithm, consistent with the uts default K = M, or K = 30 when there are more than 30 features. For each dataset column, find the feature-selection algorithm achieving the minimum misclassification percentage across the feature-count sweep. Annotate each dataset as binary (Spambase, SPECTF, Relathe, Ovarian cancer) or multi-class (Cardiotocography, Handwriting, SRBCT, Lymph) based on its response variable. Tabulate dataset -> best algorithm -> misclassification rate and summarize how recommendations differ, e.g., HITON best for Spambase, Information gain for Cardiotocography, LOGO for Handwriting, L1-LSMI for SPECTF, and RRCT for Relathe and Ovarian cancer (tied with a few competitors on multi-class SRBCT). Write the summary to best_fs_summary.csv. Rationale: the paper benchmarked RRCT against 19 FS algorithms on eight real-world datasets, feeding 1…min(M,30) selected features into a random forest classifier and recording misclassification percentage, so the column minimum is the correct within-dataset winner criterion.

## Related tools

- **pandas** (Load and reduce the reproduced misclassification benchmark table to per-dataset column minima)
- **uts** (MATLAB tool whose default K = M (or K = 30 if more than 30 features) defines the feature-count sweep recorded in the benchmark table) — github.com/ThanasisTsanas/RRCT
- **MATLAB 2021b** (Environment in which the RRCT-vs-19-algorithms benchmark and random-forest misclassification experiments were run) — github.com/ThanasisTsanas/RRCT

## Examples

```
import pandas as pd; df = pd.read_csv('rf_misclassification_benchmark.csv'); best = df.min().idxmin() if False else df.idxmin(); best.to_csv('best_fs_summary.csv')
```

## Evaluation signals

- Each dataset row's misclassification rate equals the column minimum in the reproduced benchmark table
- Every benchmarked real-world dataset (Spambase, SPECTF, Relathe, Ovarian cancer, Cardiotocography, Handwriting, Lymph, SRBCT) appears exactly once in best_fs_summary.csv
- Binary datasets (Spambase, SPECTF, Relathe, Ovarian cancer) and multi-class datasets (Cardiotocography, Handwriting, SRBCT, Lymph) are each labeled correctly
- RRCT is reported as winner on binary Relathe and Ovarian cancer and tied with a few competitors on multi-class SRBCT, consistent with the paper's finding

## Limitations

- Winners are dataset-specific; RRCT was a clear winner only on the binary Relathe and ovarian-cancer datasets and was tied on multi-class SRBCT, so no single algorithm dominates everywhere
- Popular algorithms such as RELIEF, mRMR Peng (the default implementation most studies use), CFS, and JMI may stumble and erroneously select probes on synthetic data, so their rankings do not transfer directly
- Sweep is capped at min(M,30) features; performance beyond 30 selected features is not captured

## Evidence

- [methods] uts default K = M, or K = 30 if there are more than 30 features: "K        -> number of features to be selected (integer value)"
- [results] 1…min(M,30) features into random forest, misclassification percentage recorded: "as a function of the number of selected features presented into the random forest (RF)"
- [abstract] Benchmarked against 19 FS algorithms on four synthetic and eight real-world datasets: "We empirically benchmark the performance of RRCT against 19 FS algorithms across four synthetic and eight real-world datasets"
- [results] CFS and JMI may stumble and erroneously select probes: "we remark that popular algorithms such as RELIEF, mRMR Peng (which is the default implementation most studies use when referring to mRMR), correlation-based FS (CFS), and JMI may stumble and"
- [results] All experiments run on MATLAB 2021b: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
