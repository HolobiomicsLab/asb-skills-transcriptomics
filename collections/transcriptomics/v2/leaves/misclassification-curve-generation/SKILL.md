---
name: misclassification-curve-generation
description: Use when comparing filter feature-selection algorithms on a labeled dataset
  by measuring random-forest out-of-sample misclassification as a function of the
  number of selected features.
when_to_use_negative:
- The goal is to evaluate wrapper or embedded FS methods rather than reproduce the
  19 baseline filter algorithms' comparison
- The dataset has no labeled response, since out-of-sample RF misclassification cannot
  be computed
- Only a single fixed feature subset is of interest — this skill specifically requires
  curves over subset sizes 1..min(M,30)
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MATLAB 2021b
  role: Environment in which all experiments (FS algorithms, RF classifier evaluation,
    curve assembly) were run
- name: uts
  role: UTS-style interface for running filter FS algorithms with K = number of features
    to select, returning features in descending order of importance
- name: mRMR_Spearman
  role: One of the 19 baseline filter FS algorithms whose ranked features feed the
    RF misclassification curves
  repo: https://github.com/ThanasisTsanas/mRMR_Spearman
- name: FEAST
  role: MATLAB/C feature selection toolbox providing mutual-information-based filter
    algorithms (mim, mrmr, cmim, jmi, disr, cife, relief, fcbf, etc.) whose outputs
    are evaluated via the RF misclassification curves; expects discrete inputs (10
    equal-width bins suggested)
  repo: https://github.com/Craigacp/FEAST
- name: CausalExplorer_1.5
  role: Provides HITON_PC/HITON_MB and related algorithms used as baseline FS algorithms
    in the benchmark
  repo: https://github.com/mensxmachina/CausalExplorer_1.5
- name: L1-LSMI
  role: MATLAB package for non-linear supervised feature selection with L1-penalized
    squared-loss mutual information, one of the 19 baselines
  repo: https://github.com/wittawatj/l1lsmi
- name: RRCT
  role: The proposed filter FS algorithm whose RF out-of-sample misclassification
    curves are compared against the 19 baselines
  repo: https://github.com/ThanasisTsanas/RRCT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:e4d608fc4da3f5a628eb8dade0d6c08c3325db1bb04a4a6d9f41e7a4abb9c743
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ThanasisTsanas/mRMR_Spearman
  tools:
  - CausalExplorer_1.5
  - FEAST
  - L1-LSMI
  - MATLAB
  - MATLAB 2021b
  - RRCT
  - mRMR_Spearman
  - uts
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# misclassification-curve-generation

## Summary

Generates out-of-sample misclassification percentage curves for filter feature-selection algorithms by feeding each selected feature subset (from 1 up to min(M,30) features) into a random forest classifier, as used to benchmark 19 baseline filter FS algorithms in the RRCT paper (Patterns, 2022).

## When to use

Use when comparing filter feature-selection algorithms on a labeled dataset by measuring random-forest out-of-sample misclassification as a function of the number of selected features.

## When NOT to use

- The goal is to evaluate wrapper or embedded FS methods rather than reproduce the 19 baseline filter algorithms' comparison
- The dataset has no labeled response, since out-of-sample RF misclassification cannot be computed
- Only a single fixed feature subset is of interest — this skill specifically requires curves over subset sizes 1..min(M,30)

## Inputs

- Benchmark binary and multi-class datasets as (n_samples x n_features) data matrices with response labels, e.g. Spambase, SPECTF, SRBCT, Ovarian cancer, Cardiotocography, Handwriting, Relathe, Lymph (UCI Machine Learning Repository and ASU FS Repository)
- Ranked feature lists in descending order of importance output by each of the 19 filter FS algorithms (mRMR Peng, mRMR Spearman, CFS, CBF, CMIM, CIFE, DISR, JMI, MIQ, QPFS, RELIEF, SIMBA, SPECCMI, L1-LSMI, LOGO, GSO, IAMB, HITON, Information gain)

## Outputs

- Misclassification percentage as a function of the number of selected features (1 to min(M,30)) for each FS algorithm and dataset
- False discovery rate (FDR) values across iterative selection steps for the four synthetic datasets, measured at the number of true features
- Comparison table saved as baseline_fs_comparison.csv

## How to apply

Run each filter FS algorithm through the UTS-style interface with K set to M (number of features), or K=30 when the dataset has more than 30 features, obtaining features in descending order of importance. For each algorithm and dataset (e.g. Spambase 4,601x57, SPECTF 80/187x44, SRBCT 63x2,308, Ovarian cancer 72x592, Cardiotocography 2,129x21, Handwriting 2,000x649, Relathe 1,427x4,322, Lymph 148x18), take the ranked feature list and, for each subset size from 1 up to min(M,30), present the selected features into a random forest (RF) classifier and record the misclassification percentage. Plot/assemble misclassification percentage as a function of the number of selected features so algorithms can be compared curve-by-curve. For synthetic datasets, additionally record false discovery rate (FDR) outputs across iterative selection steps, measured at the number of true features. Spot-check reproduced values against the paper's reported figures (HITON 4.26% on Spambase, Information gain 8.49% on Cardiotocography, LOGO 0.90% on Handwriting, L1-LSMI 22.46% on SPECTF, SIMBA 0.00% on SRBCT with 6 features, CMIM 16.67% on Ovarian cancer, mRMR Peng FDR 0.67 on Synthetic 1) and save the table as baseline_fs_comparison.csv.

## Related tools

- **MATLAB 2021b** (Environment in which all experiments (FS algorithms, RF classifier evaluation, curve assembly) were run)
- **uts** (UTS-style interface for running filter FS algorithms with K = number of features to select, returning features in descending order of importance)
- **mRMR_Spearman** (One of the 19 baseline filter FS algorithms whose ranked features feed the RF misclassification curves) — https://github.com/ThanasisTsanas/mRMR_Spearman
- **FEAST** (MATLAB/C feature selection toolbox providing mutual-information-based filter algorithms (mim, mrmr, cmim, jmi, disr, cife, relief, fcbf, etc.) whose outputs are evaluated via the RF misclassification curves; expects discrete inputs (10 equal-width bins suggested)) — https://github.com/Craigacp/FEAST
- **CausalExplorer_1.5** (Provides HITON_PC/HITON_MB and related algorithms used as baseline FS algorithms in the benchmark) — https://github.com/mensxmachina/CausalExplorer_1.5
- **L1-LSMI** (MATLAB package for non-linear supervised feature selection with L1-penalized squared-loss mutual information, one of the 19 baselines) — https://github.com/wittawatj/l1lsmi
- **RRCT** (The proposed filter FS algorithm whose RF out-of-sample misclassification curves are compared against the 19 baselines) — https://github.com/ThanasisTsanas/RRCT

## Examples

```
features = mRMR_Spearman(X, y); % then feed subsets of features(1:k), k=1..min(M,30), into a MATLAB random forest and record misclassification percentage
```

## Evaluation signals

- Reproduced misclassification percentages match the paper's spot-check figures: HITON 4.26% (Spambase), Information gain 8.49% (Cardiotocography), LOGO 0.90% (Handwriting), L1-LSMI 22.46% (SPECTF), SIMBA 0.00% (SRBCT with 6 features), CMIM 16.67% (Ovarian cancer)
- Synthetic-dataset FDR at the number of true features matches reported values, e.g. mRMR Peng FDR 0.67 on Synthetic 1 and GSO/IAMB/RRCT FDR of 0 on the synthetic datasets
- Each curve spans subset sizes 1 to min(M,30) features, with K capped at 30 when the dataset has more than 30 features
- Output file baseline_fs_comparison.csv contains misclassification percentages (and FDR for synthetic datasets) for all 19 algorithms across the four synthetic and eight real-world datasets
- Curves show no single baseline algorithm consistently dominating, consistent with the paper's finding that there was no clear winner among the competing FS algorithms

## Limitations

- Discretization affects MI-based filter implementations: FEAST expects discrete inputs and produces unreliable results with continuous inputs, so continuous values must be discretised (10 equal-width bins suggested as suitable, though this is data set size dependent)
- The mRMR criterion via proper mutual information density estimation is extremely demanding, and crude histogram-based MI estimation is likely problematic; the mRMR_Spearman variant uses Spearman rank correlation instead
- Popular algorithms such as RELIEF, mRMR Peng, CFS, and JMI may erroneously select probes in synthetic datasets, so their curves can reflect probe selection rather than true signal
- Performance is dataset-dependent: no baseline FS algorithm consistently dominates across the real-world datasets, so single-dataset comparisons may be misleading

## Evidence

- [results] Selected feature subsets from each FS algorithm were presented into a random forest (RF) classifier, with 1…min(M,30) features, recording misclassification percentage: "as a function of the number of selected features presented into the random forest (RF)"
- [methods] UTS K parameter and ranking: "K        -> number of features to be selected (integer value)"
- [abstract] Benchmark scope: "We empirically benchmark the performance of RRCT against 19 FS algorithms across four synthetic and eight real-world datasets"
- [readme] FEAST discretization caveat: [evidence span withheld — anchor status: external_doc]
- [readme] FEAST MATLAB invocation: [evidence span withheld — anchor status: external_doc]
