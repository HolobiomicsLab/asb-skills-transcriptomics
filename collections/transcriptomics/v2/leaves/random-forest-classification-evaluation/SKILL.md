---
name: random-forest-classification-evaluation
description: 'Use when comparing filter FS algorithms on tabular data (data matrix
  + response): each algorithm''s ranked feature list must be translated into downstream
  RF out-of-sample misclassification percentages to judge which selections generalize.'
when_to_use_negative:
- When the goal is to compare selections against known true features on synthetic
  data only — use the paper's false discovery rate (FDR) evaluation at the number
  of true features instead of RF misclassification
- When the dataset has no response variable — the RF evaluation is supervised and
  requires a data matrix plus response
- When more than 30 features need to be evaluated without truncation — the protocol
  caps evaluated subset size at min(M,30), so it will not characterize performance
  beyond the top 30 features
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: uts (RRCT MATLAB implementation)
  role: Invoked on each dataset to produce the ranked feature list that is then fed
    into the RF classifier; supports optional K input (default K=M, K=30 when M>30)
  repo: https://github.com/ThanasisTsanas/RRCT
- name: MATLAB 2021b
  role: Environment in which all experiments (feature selection and RF evaluation)
    were run
- name: FEAST
  role: Provides MATLAB implementations of mutual-information-based filter FS algorithms
    (mim, mrmr, cmim, jmi, disr, cife, relief, fcbf) whose selected indices are evaluated
    via the RF protocol; expects discrete inputs (10 equal-width bins for continuous
    data)
  repo: https://github.com/Craigacp/FEAST
- name: mRMR_Spearman
  role: Simplified mRMR variant using Spearman correlation, one of the 19 baseline
    algorithms evaluated; called as features = mRMR_Spearman(X, y)
  repo: https://github.com/ThanasisTsanas/mRMR_Spearman
- name: CausalExplorer_1.5
  role: Provides HITON_PC/HITON_MB implementations used as one of the 19 baseline
    FS algorithms in the comparison
  repo: https://github.com/mensxmachina/CausalExplorer_1.5
- name: L1-LSMI
  role: MATLAB package for non-linear supervised feature selection with L1-penalized
    squared-loss mutual information, one of the 19 baseline algorithms
  repo: https://github.com/wittawatj/l1lsmi
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:ca36834fd7c19c1d395bb36eeb66f7d682826d4dad74deaa464925cb60f43180
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
  - uts (RRCT MATLAB implementation)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# random-forest-classification-evaluation

## Summary

Evaluate filter feature-selection algorithms by feeding their ranked feature subsets (1 to min(M,30) features) into a random forest (RF) classifier and recording the out-of-sample misclassification percentage, as done in the RRCT benchmark against 19 baseline FS algorithms across eight real-world datasets.

## When to use

Use when comparing filter FS algorithms on tabular data (data matrix + response): each algorithm's ranked feature list must be translated into downstream RF out-of-sample misclassification percentages to judge which selections generalize.

## When NOT to use

- When the goal is to compare selections against known true features on synthetic data only — use the paper's false discovery rate (FDR) evaluation at the number of true features instead of RF misclassification
- When the dataset has no response variable — the RF evaluation is supervised and requires a data matrix plus response
- When more than 30 features need to be evaluated without truncation — the protocol caps evaluated subset size at min(M,30), so it will not characterize performance beyond the top 30 features

## Inputs

- Data matrix X plus response vector y for each benchmark dataset (e.g. Spambase [4,601x57], SPECTF [80x44; 187x44], Relathe [1,427x4,322], Ovarian cancer [72x592], Cardiotocography [2,129x21], Handwriting [2,000x649], Lymph [148x18], SRBCT [63x2,308; 25x2,308]) from the UCI Machine Learning Repository or ASU FS Repository
- Ranked feature list per FS algorithm (features in descending order of importance, ranked by column index), obtained with K=M or K=30 for datasets with more than 30 features

## Outputs

- Out-of-sample misclassification percentage of the RF classifier as a function of the number of selected features (1 to min(M,30)) for each FS algorithm and dataset
- Comparison table (e.g. rrct_misclassification_rates.csv or baseline_fs_comparison.csv) of misclassification percentages across algorithms and datasets; FDR values at the number of true features for synthetic datasets

## How to apply

For each FS algorithm (e.g. RRCT, mRMR Peng, mRMR Spearman, CFS, CMIM, CIFE, DISR, JMI, RELIEF, SIMBA, HITON, L1-LSMI, LOGO), first obtain the feature list ranked in descending order of importance (ranked by column index), invoking the selection function with K set to M, or K=30 when the dataset has more than 30 features. Then feed selected feature subsets of size 1...min(M,30) into a random forest (RF) classifier and record the misclassification percentage as a function of the number of selected features. Assemble the percentages into a comparison table (e.g. baseline_fs_comparison.csv) so algorithms can be judged dataset by dataset. The rationale, per the paper, is that filter FS algorithms differ in how well their selections generalize: RRCT is benchmarked this way and was found 'very competitive in both matching the true feature set and out-of-sample classification performance', with no baseline algorithm consistently dominating. For synthetic datasets, complement the RF evaluation with false discovery rate (FDR) measured at the number of true features.

## Related tools

- **uts (RRCT MATLAB implementation)** (Invoked on each dataset to produce the ranked feature list that is then fed into the RF classifier; supports optional K input (default K=M, K=30 when M>30)) — https://github.com/ThanasisTsanas/RRCT
- **MATLAB 2021b** (Environment in which all experiments (feature selection and RF evaluation) were run)
- **FEAST** (Provides MATLAB implementations of mutual-information-based filter FS algorithms (mim, mrmr, cmim, jmi, disr, cife, relief, fcbf) whose selected indices are evaluated via the RF protocol; expects discrete inputs (10 equal-width bins for continuous data)) — https://github.com/Craigacp/FEAST
- **mRMR_Spearman** (Simplified mRMR variant using Spearman correlation, one of the 19 baseline algorithms evaluated; called as features = mRMR_Spearman(X, y)) — https://github.com/ThanasisTsanas/mRMR_Spearman
- **CausalExplorer_1.5** (Provides HITON_PC/HITON_MB implementations used as one of the 19 baseline FS algorithms in the comparison) — https://github.com/mensxmachina/CausalExplorer_1.5
- **L1-LSMI** (MATLAB package for non-linear supervised feature selection with L1-penalized squared-loss mutual information, one of the 19 baseline algorithms) — https://github.com/wittawatj/l1lsmi

## Examples

```
selectedIndices = feast('mrmr',10,data,labels); % then feed subsets of size 1..min(M,30) into the RF and record misclassification percentage
```

## Evaluation signals

- Spot-check reproduced misclassification percentages against reported figures: HITON 4.26% on Spambase, Information gain 8.49% on Cardiotocography, LOGO 0.90% on Handwriting, L1-LSMI 22.46% on SPECTF, SIMBA 0.00% on SRBCT with 6 features, CMIM 16.67% on Ovarian cancer
- Verify the feature ranking output is in descending order of importance (ranked by column index) before subsetting
- Confirm the evaluated subset sizes cover exactly 1...min(M,30) features, with K=30 applied when the dataset has more than 30 features
- Check that the misclassification percentage curve is monotone-plausible as a function of subset size and that the CSV (rrct_misclassification_rates.csv / baseline_fs_comparison.csv) contains one curve per algorithm per dataset
- On synthetic datasets, verify FDR at the number of true features matches expectations (e.g. mRMR Peng FDR 0.67 on Synthetic 1; GSO, IAMB, and RRCT reported as effective at discarding probes)

## Limitations

- No baseline FS algorithm consistently dominates across the real-world datasets: 'there was no clear winner among the competing FS algorithms in terms of RF out-of-sample performance, although RRCT worked very well generally' — results are dataset-dependent, not universally transferable
- Popular algorithms such as RELIEF, mRMR Peng (the default implementation most studies use when referring to mRMR), CFS, and JMI may erroneously select probes in synthetic data, so their RF rankings can be misleading about true features
- FEAST-based algorithms require discretized inputs: continuous values must be discretised (10 equal-width bins suggested) or FEAST 'produces unreliable results when used with continuous inputs'
- The evaluation protocol only assesses the top min(M,30) ranked features, capping the analysis for fat datasets like Relathe (4,322 features) and Ovarian cancer (592 features)

## Evidence

- [results] Selected feature subsets from each FS algorithm were presented into a random forest (RF) classifier, with 1…min(M,30) features, recording misclassification percentage: "as a function of the number of selected features presented into the random forest (RF)"
- [methods] uts output specification: ranked features: "features -> The selected features in descending order of importance (ranked by column index)"
- [abstract] RRCT competitiveness in out-of-sample classification: [evidence span withheld — anchor status: not_found]
- [results] Computing environment for all RF evaluation experiments: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
- [readme] FEAST MATLAB call returning selected feature indices for RF evaluation: [evidence span withheld — anchor status: external_doc]
