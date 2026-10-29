---
name: synthetic-dataset-generation
description: Use when you need to benchmark a feature-selection algorithm on data
  where the truly relevant features are known by construction, e.g. to measure false
  discovery rate on probe-laden synthetic matrices.
when_to_use_negative:
- The dataset is a real-world dataset without known ground-truth relevant features,
  where FDR cannot be computed and out-of-sample classification via a random forest
  is the appropriate evaluation instead
- The goal is to test discrete-input mutual-information algorithms such as those in
  FEAST, which require continuous values to be discretised (e.g. 10 equal-width bins)
  rather than synthetic probe/relevant-feature structures
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MATLAB 2021b
  role: Environment in which the four synthetic benchmark datasets are generated and
    all experiments are run
- name: uts (RRCT)
  role: Feature-selection function run on each synthetic dataset with optional K input;
    returns features in descending order of importance
  repo: https://github.com/ThanasisTsanas/RRCT
- name: mRMR_Spearman
  role: Comparison feature-selection approach using the Spearman correlation coefficient
    criterion on the synthetic data
  repo: https://github.com/ThanasisTsanas/mRMR_Spearman
- name: FEAST
  role: MATLAB/C toolbox providing implementations of comparison filter algorithms
    (mrmr, jmi, relief, etc.) benchmarked on the same data
  repo: https://github.com/Craigacp/FEAST
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:5a5749c8e041a477cc84607b9853156d86e1b4b2567827c9105a1cd3277b80c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ThanasisTsanas/RRCT
  tools:
  - FEAST
  - MATLAB
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  - uts (RRCT)
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# synthetic-dataset-generation

## Summary

Generate the four synthetic benchmark datasets (with known true features versus irrelevant probes) used to evaluate feature-selection algorithms by false discovery rate (FDR) in the RRCT paper, enabling controlled recovery experiments.

## When to use

Use when you need to benchmark a feature-selection algorithm on data where the truly relevant features are known by construction, e.g. to measure false discovery rate on probe-laden synthetic matrices.

## When NOT to use

- The dataset is a real-world dataset without known ground-truth relevant features, where FDR cannot be computed and out-of-sample classification via a random forest is the appropriate evaluation instead
- The goal is to test discrete-input mutual-information algorithms such as those in FEAST, which require continuous values to be discretised (e.g. 10 equal-width bins) rather than synthetic probe/relevant-feature structures

## Inputs

- Synthetic data matrix specifications: Breiman 60x30 C2; 1,000x100 C2; Guyon 1,000x500 C10; Guyon 100x500 C8
- Labels of truly relevant features versus irrelevant probe features
- Optional K input to the uts function (number of features to select, default K = M or K = 30 when M > 30)

## Outputs

- Four synthetic datasets (data matrix plus response) with known true-feature ground truth
- Selected feature indices ranked in descending order of importance (ranked by column index) from uts
- FDR values per dataset measured at the number of true features (e.g. RRCT: 0, 0, 0, 0.10 on Synthetics 1-4)
- Comparison table of FDR results for RRCT, GSO, and IAMB (e.g. synthetic_fdr_results.csv)

## How to apply

In MATLAB 2021b, construct the four synthetic datasets with their specified dimensions and class counts: Breiman 60x30 with 2 classes, 1,000x100 with 2 classes, Guyon 1,000x500 with 10 classes, and the fat-dataset Guyon 100x500 with 8 classes, explicitly labelling truly relevant features versus irrelevant probes. Run the RRCT uts function on each dataset using the optional K input with the default setting (K = M, or K = 30 when the dataset has more than 30 features), and collect the 'features' output ranked in descending order of importance by column index. Compute FDR at the number of true features, i.e. the fraction of selected features that are irrelevant probes, and repeat the measurement for comparison algorithms such as GSO, IAMB, and mRMR (Peng) to characterise their behaviour on identical synthetic data. Write results (e.g. synthetic_fdr_results.csv) so algorithm-specific probe selection can be compared across datasets.

## Related tools

- **MATLAB 2021b** (Environment in which the four synthetic benchmark datasets are generated and all experiments are run)
- **uts (RRCT)** (Feature-selection function run on each synthetic dataset with optional K input; returns features in descending order of importance) — https://github.com/ThanasisTsanas/RRCT
- **mRMR_Spearman** (Comparison feature-selection approach using the Spearman correlation coefficient criterion on the synthetic data) — https://github.com/ThanasisTsanas/mRMR_Spearman
- **FEAST** (MATLAB/C toolbox providing implementations of comparison filter algorithms (mrmr, jmi, relief, etc.) benchmarked on the same data) — https://github.com/Craigacp/FEAST

## Examples

```
features = uts(X, y);  % MATLAB: select features ranked by importance, default K = M (or 30 when M > 30)
```

## Evaluation signals

- RRCT (and GSO, IAMB) should achieve FDR of 0, 0, 0, and 0.10 on Synthetics 1-4 measured at the number of true features
- All true features should be recovered on Synthetics 1-3; only the last true feature of the fat-dataset Synthetic 4 should be missed
- The uts 'features' output must list selected features in descending order of importance ranked by column index
- Popular algorithms such as RELIEF, mRMR Peng, CFS, and JMI may erroneously select probes — cross-checking against them validates the FDR scoring is discriminating
- K defaults to M, or K = 30 when the dataset has more than 30 features, so selection counts must match this rule

## Limitations

- Synthetic 4 is a 'fat' dataset (100x500, 8 classes) where even strong algorithms miss one true feature (FDR 0.10), so perfect probe rejection is not guaranteed in fat datasets
- Popular algorithms (RELIEF, mRMR Peng as the default implementation, CFS, JMI) may stumble and erroneously select probes, so absolute FDR=0 is not an algorithm-agnostic expectation

## Evidence

- [results] RRCT achieved an FDR of 0, 0, 0, and 0.10 on Synthetics 1-4 respectively: "The results in Figure 1 and Table 1 illustrate that GSO, IAMB, and RRCT are all very effective at discarding probes."
- [methods] uts returns ranked selected features: "features -> The selected features in descending order of importance (ranked by column index)"
- [methods] uts optional K input: "uts

optional inputs:  

K        -> number of features to be selected (integer value)"
- [results] Experiments run in MATLAB 2021b: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
- [results] Popular algorithms select probes erroneously: "we remark that popular algorithms such as RELIEF, mRMR Peng (which is the default implementation most studies use when referring to mRMR), correlation-based FS (CFS), and JMI may stumble and"
