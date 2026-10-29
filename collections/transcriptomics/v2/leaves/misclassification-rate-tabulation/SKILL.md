---
name: misclassification-rate-tabulation
description: Use when a reproduced benchmark table of random-forest misclassification
  percentages (multiple FS algorithms × number of selected features × datasets) is
  available and the goal is to identify the FS algorithm per dataset and compare binary
  vs multi-class winners.
when_to_use_negative:
- Input is the raw data matrix and response rather than an already-reproduced misclassification
  benchmark table — run the FS algorithms and random-forest evaluation first (e.g.,
  via the RRCT/uts pipeline in MATLAB 2021b)
- The task concerns regression or synthetic-dataset probe-discrimination performance
  (assessed via FDR, not misclassification tabulation)
- Only one FS algorithm was benchmarked, so no cross-algorithm winner comparison per
  dataset is possible
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: pandas
  role: Loads the reproduced benchmark table and computes per-dataset column minima
    of misclassification percentage
- name: uts
  role: Produced the ranked feature subsets (K features, descending importance) feeding
    the random-forest misclassification sweep that this skill tabulates
- name: MATLAB 2021b
  role: Environment in which the RF misclassification benchmark experiments were originally
    run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:a0bb9395869795ee7be6f09a094125f4a8c7b0cf724508742f86d4bd6f459a6d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - MATLAB 2021b
  - pandas
  - uts
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# misclassification-rate-tabulation

## Summary

Tabulates, per real-world dataset, the filter feature-selection algorithm that achieves the lowest out-of-sample random-forest misclassification percentage across a 1…min(M,30) feature-count sweep, and summarizes how the winning algorithms differ between binary and multi-class classification settings.

## When to use

Use when a reproduced benchmark table of random-forest misclassification percentages (multiple FS algorithms × number of selected features × datasets) is available and the goal is to identify the best FS algorithm per dataset and compare binary vs multi-class winners.

## When NOT to use

- Input is the raw data matrix and response rather than an already-reproduced misclassification benchmark table — run the FS algorithms and random-forest evaluation first (e.g., via the RRCT/uts pipeline in MATLAB 2021b)
- The task concerns regression or synthetic-dataset probe-discrimination performance (assessed via FDR, not misclassification tabulation)
- Only one FS algorithm was benchmarked, so no cross-algorithm winner comparison per dataset is possible

## Inputs

- Reproduced benchmark CSV table of random-forest misclassification percentages per algorithm, feature count (1…min(M,30)), and dataset (e.g., Spambase [4,601×57], SPECTF [80×44; 187×44], Relathe [1,427×4,322], Ovarian cancer [72×592], Cardiotocography [2,129×21], Handwriting [2,000×649], Lymph [148×18], SRBCT [63×2,308; 25×2,308])
- Mapping of each dataset to binary vs multi-class classification based on its response variable

## Outputs

- best_fs_summary.csv with one row per dataset: dataset, best algorithm, lowest misclassification percentage, and binary/multi-class label
- Summary of how recommended FS algorithms differ between binary and multi-class settings

## How to apply

Load the reproduced benchmark table of random-forest misclassification percentages (recorded for 1…min(M,30) selected features per algorithm, consistent with the uts default K = M, or K = 30 when there are more than 30 features) using pandas. For each dataset, find the algorithm attaining the column-minimum misclassification percentage across the feature-count sweep. Annotate each dataset as binary (Spambase, SPECTF, Relathe, Ovarian cancer) or multi-class (Cardiotocography, Handwriting, SRBCT, Lymph) based on its response variable. Tabulate dataset -> best algorithm -> misclassification rate and write the summary to best_fs_summary.csv; expected winners include HITON for Spambase, Information gain for Cardiotocography, LOGO for Handwriting, L1-LSMI for SPECTF, and RRCT for Relathe and Ovarian cancer. Validation requires each row's rate to equal the reproduced table's column minimum and every benchmarked dataset to appear exactly once.

## Related tools

- **pandas** (Loads the reproduced benchmark table and computes per-dataset column minima of misclassification percentage)
- **uts** (Produced the ranked feature subsets (K features, descending importance) feeding the random-forest misclassification sweep that this skill tabulates)
- **MATLAB 2021b** (Environment in which the RF misclassification benchmark experiments were originally run)

## Examples

```
import pandas as pd; df = pd.read_csv('benchmark_rf_misclassification.csv'); best = df.groupby('dataset')['misclassification_pct'].idxmin(); df.loc[best].to_csv('best_fs_summary.csv', index=False)
```

## Evaluation signals

- Each dataset row's misclassification rate equals the column minimum in the reproduced benchmark table
- Every benchmarked dataset appears exactly once in best_fs_summary.csv (no duplicates, no omissions)
- Feature-count sweep covers 1…min(M,30) features, matching the uts default K = M (or K = 30 when M > 30)
- Binary/multi-class annotations are consistent with each dataset's response variable (binary: Spambase, SPECTF, Relathe, Ovarian cancer; multi-class: Cardiotocography, Handwriting, SRBCT, Lymph)
- Reported winners (e.g., RRCT for Relathe and Ovarian cancer, L1-LSMI for SPECTF, HITON for Spambase) reproduce the article's per-dataset best performers

## Limitations

- Winner selection is over a finite feature-count sweep (max 30 features), so conclusions apply only to that sweep, not to full feature sets
- Comparisons reflect out-of-sample random-forest misclassification on eight real-world datasets only; the article only tentatively suggests generalizability of the best-performing FS algorithms
- The reproduced table is a re-implementation of the original MATLAB 2021b benchmark, so exact rates may differ from the published figures

## Evidence

- [results] RF misclassification sweep: "as a function of the number of selected features presented into the random forest (RF)"
- [methods] uts feature ranking: "features -> The selected features in descending order of importance (ranked by column index)"
- [abstract] generalizability caveat: "RRCT is very competitive in both tasks, and we tentatively make suggestions on the generalizability and application of the best-performing FS algorithms"
- [results] benchmark environment: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
- [readme] RRCT algorithm description: "This function is a computationally efficient, robust approach for feature selection using the RRCT algorithm."
