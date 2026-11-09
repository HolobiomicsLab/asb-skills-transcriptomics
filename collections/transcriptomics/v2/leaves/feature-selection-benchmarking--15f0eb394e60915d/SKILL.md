---
name: feature-selection-benchmarking
description: Use when you need to compare filter feature-selection algorithms (e.g.
  RRCT, mRMR, CFS, CMIM, JMI, RELIEF, HITON) on the same data matrix and response,
  matching true feature sets or ranking them by downstream classifier performance.
when_to_use_negative:
- The problem is not feature selection — e.g. the input is already a small, curated
  feature table where no ranking or discarding of probes is needed
- The goal requires wrapper or embedded selection tuned to one specific model rather
  than a generic off-the-shelf filter comparison across algorithms
- The dataset has no response variable or no known true-feature labels, making FDR-based
  evaluation on synthetic ground truth impossible
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MATLAB 2021b
  role: Environment in which all benchmark experiments were run (desktop Windows 10,
    Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM)
- name: uts
  role: 'RRCT implementation exposing the uts-style interface: optional input K (number
    of features to select), output ''features'' in descending order of importance'
  repo: https://github.com/ThanasisTsanas/RRCT
- name: mRMR_Spearman
  role: Baseline filter FS algorithm using Spearman correlation instead of demanding
    mutual-information estimation
  repo: https://github.com/ThanasisTsanas/mRMR_Spearman
- name: FEAST
  role: MATLAB/C toolbox providing mutual-information-based filter FS baselines (mim,
    mrmr, cmim, jmi, disr, cife, relief, etc.); continuous inputs must be discretised
    (10 equal-width bins)
  repo: https://github.com/Craigacp/FEAST
- name: CausalExplorer_1.5
  role: Source of causal-discovery baseline algorithms such as HITON_PC used in the
    benchmark
  repo: https://github.com/mensxmachina/CausalExplorer_1.5
- name: L1-LSMI
  role: Baseline non-linear supervised FS package with L1-penalized squared-loss mutual
    information
  repo: https://github.com/wittawatj/l1lsmi
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:9d88e4cbea88203443d42f105045fab39339e29b547718b60839ae0be4927265
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ThanasisTsanas/RRCT
  tools:
  - CausalExplorer_1.5
  - FEAST
  - L1-LSMI
  - MATLAB
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# feature-selection-benchmarking

## Summary

Benchmark filter feature-selection algorithms by running them (via the uts-style interface with K = M, or K = 30 when the dataset has more than 30 features) on four synthetic and eight real-world datasets, then scoring selected subsets by false discovery rate (FDR) on synthetic data and random-forest out-of-sample misclassification percentage on real-world data.

## When to use

Use when you need to compare filter feature-selection algorithms (e.g. RRCT, mRMR, CFS, CMIM, JMI, RELIEF, HITON) on the same data matrix and response, matching true feature sets or ranking them by downstream classifier performance.

## When NOT to use

- The problem is not feature selection — e.g. the input is already a small, curated feature table where no ranking or discarding of probes is needed
- The goal requires wrapper or embedded selection tuned to one specific model rather than a generic off-the-shelf filter comparison across algorithms
- The dataset has no response variable or no known true-feature labels, making FDR-based evaluation on synthetic ground truth impossible

## Inputs

- Data matrix X with response y (classification or regression) from the UCI Machine Learning Repository or ASU FS Repository
- Four synthetic benchmark datasets with known true features versus irrelevant probes (Breiman 60x30 C2; 1,000x100 C2; Guyon 1,000x500 C10; Guyon 100x500 C8)
- Eight real-world benchmark datasets (Spambase, SPECTF, Relathe, Ovarian cancer, Cardiotocography, Handwriting, Lymph, SRBCT)

## Outputs

- Selected feature subsets per algorithm and dataset, features ranked in descending order of importance (ranked by column index)
- FDR values on the synthetic datasets measured at the number of true features (synthetic_fdr_results.csv)
- RF out-of-sample misclassification percentages for 1...min(M,30) selected features per algorithm (baseline_fs_comparison.csv)
- Summary of the best-performing algorithm per dataset, annotated as binary or multi-class (best_fs_summary.csv)

## How to apply

Load the benchmark datasets (four synthetics: Breiman 60x30 C2, 1,000x100 C2, Guyon 1,000x500 C10, Guyon 100x500 C8; eight real-world: Spambase 4,601x57, SPECTF, Relathe 1,427x4,322, Ovarian cancer 72x592, Cardiotocography 2,129x21, Handwriting 2,000x649, Lymph 148x18, SRBCT 63x2,308) in MATLAB 2021b. Run each of the 19 filter FS algorithms through the uts-style interface with the default K setting (K = M, or K = 30 when the dataset has more than 30 features), collecting the 'features' output ranked in descending order of importance by column index. For the synthetic datasets, label truly relevant features versus irrelevant probes and compute the false discovery rate at the number of true features (e.g. RRCT, GSO, and IAMB achieved FDR of 0, 0, 0, and 0.10 on Synthetics 1-4, while RELIEF, mRMR Peng, CFS, and JMI erroneously selected probes). For real-world datasets, feed selected feature subsets of 1...min(M,30) features into a random forest (RF) classifier and record the misclassification percentage as a function of the number of selected features. Assemble misclassification percentages and FDR values into a comparison table and spot-check against reported values (e.g. HITON 4.26% on Spambase, Information gain 8.49% on Cardiotocography, LOGO 0.90% on Handwriting, L1-LSMI 22.46% on SPECTF, SIMBA 0.00% on SRBCT with 6 features, CMIM 16.67% on Ovarian cancer, mRMR Peng FDR 0.67 on Synthetic 1).

## Related tools

- **MATLAB 2021b** (Environment in which all benchmark experiments were run (desktop Windows 10, Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM))
- **uts** (RRCT implementation exposing the uts-style interface: optional input K (number of features to select), output 'features' in descending order of importance) — https://github.com/ThanasisTsanas/RRCT
- **mRMR_Spearman** (Baseline filter FS algorithm using Spearman correlation instead of demanding mutual-information estimation) — https://github.com/ThanasisTsanas/mRMR_Spearman
- **FEAST** (MATLAB/C toolbox providing mutual-information-based filter FS baselines (mim, mrmr, cmim, jmi, disr, cife, relief, etc.); continuous inputs must be discretised (10 equal-width bins)) — https://github.com/Craigacp/FEAST
- **CausalExplorer_1.5** (Source of causal-discovery baseline algorithms such as HITON_PC used in the benchmark) — https://github.com/mensxmachina/CausalExplorer_1.5
- **L1-LSMI** (Baseline non-linear supervised FS package with L1-penalized squared-loss mutual information) — https://github.com/wittawatj/l1lsmi

## Examples

```
features = uts(X, y); % optional K input: K = M, or K = 30 when the dataset has more than 30 features
```

## Evaluation signals

- FDR at the number of true features on the synthetics should reproduce the reported values (RRCT, GSO, IAMB: 0, 0, 0, 0.10 on Synthetics 1-4; mRMR Peng FDR 0.67 on Synthetic 1)
- Reproduced misclassification percentages should match reported spot-checks: HITON 4.26% on Spambase, Information gain 8.49% on Cardiotocography, LOGO 0.90% on Handwriting, L1-LSMI 22.46% on SPECTF, SIMBA 0.00% on SRBCT with 6 features, CMIM 16.67% on Ovarian cancer
- Each dataset row's best-algorithm misclassification rate must equal the column minimum in the benchmark table, and every benchmarked dataset must appear exactly once
- Feature counts swept must span 1...min(M,30) per dataset, consistent with the uts default K = M or K = 30 when there are more than 30 features
- Selected-feature output must be in descending order of importance ranked by column index

## Limitations

- FEAST implementations expect discrete inputs; continuous values must be discretised (10 equal-width bins suggested), otherwise results are unreliable, slow, and memory-intensive
- mRMR's original mutual information implementation is computationally demanding via proper density estimation and likely problematic with crude histograms, which is why mRMR_Spearman substitutes Spearman correlation
- No baseline FS algorithm consistently dominates: there was no clear winner in RF out-of-sample performance across the real-world datasets, although RRCT worked very well generally
- Popular algorithms such as RELIEF, mRMR Peng (the default implementation most studies use), CFS, and JMI may stumble and erroneously select probes on synthetic data

## Evidence

- [results] GSO, IAMB, and RRCT are all very effective at discarding probes: "The results in Figure 1 and Table 1 illustrate that GSO, IAMB, and RRCT are all very effective at discarding probes."
- [methods] features ranked in descending order of importance: "features -> The selected features in descending order of importance (ranked by column index)"
- [results] misclassification as a function of number of selected features into RF: "as a function of the number of selected features presented into the random forest (RF)"
- [readme] FEAST provides MI-based filter FS baselines: [evidence span withheld — anchor status: external_doc]
- [methods] uts optional input K controls the number of selected features: "K        -> number of features to be selected (integer value)"
