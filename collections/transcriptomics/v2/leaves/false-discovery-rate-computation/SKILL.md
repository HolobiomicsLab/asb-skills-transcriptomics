---
name: false-discovery-rate-computation
description: Use when benchmarking filter feature-selection algorithms on synthetic
  datasets where the truly relevant features are known, so the fraction of irrelevant
  probes among the selected features (FDR) can be measured.
when_to_use_negative:
- Real-world datasets where the truly relevant features are unknown — use out-of-sample
  random-forest misclassification performance instead
- When the goal is classification accuracy as a function of the number of selected
  features rather than probe-discard effectiveness
- Datasets with no labelled set of irrelevant probe features, since the FDR cannot
  be defined without knowing which features are probes
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: uts
  role: Implements the RRCT feature-selection algorithm; its 'features' output returns
    the selected features in descending order of importance (ranked by column index),
    which are then scored for FDR
  repo: https://github.com/ThanasisTsanas/RRCT
- name: MATLAB 2021b
  role: Environment in which the synthetic datasets are generated, the FS algorithms
    are run, and the FDR results are computed and written to synthetic_fdr_results.csv
- name: mRMR_Spearman
  role: Comparison filter FS algorithm (mRMR variant using the Spearman correlation
    coefficient criterion) whose selected features can be scored for FDR alongside
    RRCT
  repo: https://github.com/ThanasisTsanas/mRMR_Spearman
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:074fce85cfb278821186d6fbdf0cbf5bec716853b2f89062c2091f0b5b23b5a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ThanasisTsanas/RRCT
  tools:
  - MATLAB
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# false-discovery-rate-computation

## Summary

Quantifies how effectively a feature-selection algorithm discards irrelevant probe features by computing the false discovery rate (FDR) — the fraction of selected features that are irrelevant probes — on synthetic datasets with known true features, as done for RRCT, GSO, and IAMB.

## When to use

Use when benchmarking filter feature-selection algorithms on synthetic datasets where the truly relevant features are known, so the fraction of irrelevant probes among the selected features (FDR) can be measured.

## When NOT to use

- Real-world datasets where the truly relevant features are unknown — use out-of-sample random-forest misclassification performance instead
- When the goal is classification accuracy as a function of the number of selected features rather than probe-discard effectiveness
- Datasets with no labelled set of irrelevant probe features, since the FDR cannot be defined without knowing which features are probes

## Inputs

- Synthetic benchmark dataset as a MATLAB data matrix plus response vector, with known true features and irrelevant probe features (e.g. Breiman 60x30 C2; 1,000x100 C2; Guyon 1,000x500 C10; Guyon 100x500 C8)
- Selected feature list returned by a filter FS algorithm (features ranked in descending order of importance, ranked by column index)
- Optional K input: number of features to be selected (integer; default K = M, or K = 30 when the dataset has more than 30 features)

## Outputs

- FDR value per synthetic dataset, measured at the number of true features (fraction of selected features that are irrelevant probes)
- FDR results table for RRCT, GSO, and IAMB on all four synthetics (synthetic_fdr_results.csv)

## How to apply

Generate the synthetic benchmark datasets with their specified dimensions and class counts (Breiman 60x30 C2; 1,000x100 C2; Guyon 1,000x500 C10; Guyon 100x500 C8), labelling the truly relevant features versus the irrelevant probes. Run the feature-selection algorithm (e.g. the uts function implementing RRCT, with optional K input set to M, or K=30 when the dataset has more than 30 features) and collect its 'features' output, which returns the selected features in descending order of importance ranked by column index. Compute the FDR at the number of true features, i.e. the fraction of selected features that are irrelevant probes. Repeat the measurement for comparison algorithms (e.g. GSO and IAMB) on the same synthetic data to characterise their behaviour, then write the results (e.g. to synthetic_fdr_results.csv).

## Related tools

- **uts** (Implements the RRCT feature-selection algorithm; its 'features' output returns the selected features in descending order of importance (ranked by column index), which are then scored for FDR) — https://github.com/ThanasisTsanas/RRCT
- **MATLAB 2021b** (Environment in which the synthetic datasets are generated, the FS algorithms are run, and the FDR results are computed and written to synthetic_fdr_results.csv)
- **mRMR_Spearman** (Comparison filter FS algorithm (mRMR variant using the Spearman correlation coefficient criterion) whose selected features can be scored for FDR alongside RRCT) — https://github.com/ThanasisTsanas/mRMR_Spearman

## Examples

```
features = uts(X, y, K);  fdr = mean(~ismember(features(1:n_true), true_idx));
```

## Evaluation signals

- RRCT, GSO, and IAMB should achieve FDR of 0, 0, 0, and 0.10 on Synthetics 1-4 respectively (measured at the number of true features), recovering all true features on Synthetics 1-3 and missing only the last true feature on the fat-dataset Synthetic 4
- Cross-check: mRMR Peng should yield FDR 0.67 on Synthetic 1, reflecting its tendency to erroneously select probes (like RELIEF, CFS, and JMI)
- FDR must be evaluated at the number of true features (not the full selected list) and expressed as the fraction of selected features that are irrelevant probes
- The 'features' output must be in descending order of importance ranked by column index, so the top n_true_features entries are the subset scored
- Output table should contain FDR values for RRCT, GSO, and IAMB across all four synthetic datasets

## Limitations

- FDR can only be computed on synthetic datasets where the true features and irrelevant probes are known by construction; it is not applicable to the real-world benchmark datasets (e.g. Spambase, SPECTF, SRBCT)
- Even well-performing algorithms may fail on fat datasets: on Synthetic 4 (the fat dataset) RRCT/GSO/IAMB missed the last true feature, giving FDR 0.10
- Popular algorithms such as RELIEF, mRMR Peng (the default implementation most studies use when referring to mRMR), CFS, and JMI may stumble and erroneously select probes

## Evidence

- [results] RRCT achieved an FDR of 0, 0, 0, and 0.10 on Synthetics 1-4 respectively (measured at the number of true features): "The results in Figure 1 and Table 1 illustrate that GSO, IAMB, and RRCT are all very effective at discarding probes."
- [results] FDR measured across iterative selection steps: "reporting the false discovery rate (FDR) outputs for the four synthetic datasets"
- [methods] uts returns ranked features used for the FDR subset: "features -> The selected features in descending order of importance (ranked by column index)"
- [methods] uts optional K input controlling how many features are selected: "optional inputs:  
  
K        -> number of features to be selected (integer value)"
- [abstract] RRCT benchmarked against 19 FS algorithms on four synthetic and eight real-world datasets: "We empirically benchmark the performance of RRCT against 19 FS algorithms across four synthetic and eight real-world datasets"
