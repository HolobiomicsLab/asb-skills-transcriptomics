---
name: benchmark-dataset-evaluation-across-repositories
description: Use when you need to evaluate a filter-based feature-selection algorithm
  on standard UCI/ASU benchmark datasets and compare out-of-sample classification
  performance as a function of the number of selected features.
when_to_use_negative:
- The goal is feature-set recovery on synthetic data with planted probes — the paper
  assesses that via false discovery rate (FDR), not out-of-sample misclassification
- The problem is not representable as a data matrix plus response vector, since RRCT
  was designed as a generic off-the-shelf FS algorithm for exactly that form
- You need mutual-information-based filter methods requiring discretized inputs (e.g.,
  FEAST's mim, cmim, jmi) — RRCT instead uses rank correlations and partial correlations
  directly on continuous data
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: uts (RRCT)
  role: Core feature-selection function invoked on each benchmark dataset to return
    features ranked by RRCT relevance-redundancy-complementarity scores
  repo: https://github.com/ThanasisTsanas/RRCT
- name: MATLAB 2021b
  role: Execution environment in which all benchmark experiments and random-forest
    evaluations were run
- name: FEAST
  role: Comparator filter feature-selection toolbox (mim, mrmr, cmim, jmi, relief,
    etc.) used in the benchmark against 19 FS algorithms
  repo: https://github.com/Craigacp/FEAST
- name: mRMR_Spearman
  role: Spearman-correlation-based mRMR implementation, the mRMR-style baseline related
    to RRCT
  repo: https://github.com/ThanasisTsanas/mRMR_Spearman
- name: CausalExplorer
  role: Comparator MATLAB toolbox of causal/network feature-selection algorithms (MMPC,
    HITON_PC, etc.) in the benchmark
  repo: https://github.com/mensxmachina/CausalExplorer_1.5
- name: L1-LSMI
  role: Comparator non-linear supervised feature-selection package using L1-penalized
    squared-loss mutual information
  repo: https://github.com/wittawatj/l1lsmi
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:218d361e04f89d5ac096e4c69cb6766b75710b18d398f0042c986f14250b188e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ThanasisTsanas/RRCT
  tools:
  - CausalExplorer
  - FEAST
  - L1-LSMI
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  - uts (RRCT)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# benchmark-dataset-evaluation-across-repositories

## Summary

Benchmark a feature-selection algorithm (e.g., RRCT) across the eight real-world datasets of the RRCT study (Cardiotocography, Handwriting, Lymph, Ovarian cancer, Relathe, Spambase, SPECTF, SRBCT) sourced from the UCI ML and ASU FS repositories, by feeding ranked feature subsets into a random forest and recording out-of-sample misclassification rates.

## When to use

Use when you need to evaluate a filter-based feature-selection algorithm on standard UCI/ASU benchmark datasets and compare out-of-sample classification performance as a function of the number of selected features.

## When NOT to use

- The goal is feature-set recovery on synthetic data with planted probes — the paper assesses that via false discovery rate (FDR), not out-of-sample misclassification
- The problem is not representable as a data matrix plus response vector, since RRCT was designed as a generic off-the-shelf FS algorithm for exactly that form
- You need mutual-information-based filter methods requiring discretized inputs (e.g., FEAST's mim, cmim, jmi) — RRCT instead uses rank correlations and partial correlations directly on continuous data

## Inputs

- Data matrix X and response vector y for each of the eight benchmark datasets: Cardiotocography, Handwriting, Lymph, Ovarian cancer, Relathe, Spambase, SPECTF, SRBCT
- Optional integer K = number of features to select (default K = M, or K = 30 when the dataset has more than 30 features)

## Outputs

- Ranked feature list per dataset (selected features in descending order of importance, ranked by column index)
- Out-of-sample misclassification percentage per feature-subset size (1...min(M,30)) per dataset
- Assembled benchmark table, e.g., rrct_misclassification_rates.csv

## How to apply

Load each benchmark dataset (Cardiotocography 2,129x21, Handwriting 2,000x649, Lymph 148x18, Ovarian cancer 72x592, Relathe 1,427x4,322, Spambase 4,601x57, SPECTF 187x44, SRBCT 63x2,308) as a data matrix X and response y from the UCI Machine Learning Repository or ASU FS Repository. For each dataset, invoke the RRCT uts function in MATLAB 2021b, passing K (default K = M, or K = 30 when the dataset has more than 30 features), which scores relevance and redundancy via an information-theoretic transformation of Spearman rank correlations and complementarity via partial correlations. The function returns features ranked in descending order of importance by column index. Feed subsets of size 1...min(M,30) into a random forest classifier and record the out-of-sample misclassification percentage per subset size. Assemble the per-dataset misclassification rates into a single CSV (e.g., rrct_misclassification_rates.csv) and judge success by whether the algorithm is competitive against the 19 FS algorithms benchmarked in the paper (e.g., RRCT won on Relathe and ovarian-cancer, tied on SRBCT).

## Related tools

- **uts (RRCT)** (Core feature-selection function invoked on each benchmark dataset to return features ranked by RRCT relevance-redundancy-complementarity scores) — https://github.com/ThanasisTsanas/RRCT
- **MATLAB 2021b** (Execution environment in which all benchmark experiments and random-forest evaluations were run)
- **FEAST** (Comparator filter feature-selection toolbox (mim, mrmr, cmim, jmi, relief, etc.) used in the benchmark against 19 FS algorithms) — https://github.com/Craigacp/FEAST
- **mRMR_Spearman** (Spearman-correlation-based mRMR implementation, the mRMR-style baseline related to RRCT) — https://github.com/ThanasisTsanas/mRMR_Spearman
- **CausalExplorer** (Comparator MATLAB toolbox of causal/network feature-selection algorithms (MMPC, HITON_PC, etc.) in the benchmark) — https://github.com/mensxmachina/CausalExplorer_1.5
- **L1-LSMI** (Comparator non-linear supervised feature-selection package using L1-penalized squared-loss mutual information) — https://github.com/wittawatj/l1lsmi

## Examples

```
features = uts(X, y, 30); % RRCT in MATLAB 2021b: select K=30 features (or K=M) for dataset X (data matrix), y (response)
```

## Evaluation signals

- Each dataset's uts output must be a feature list in descending order of importance, ranked by column index, of length K (M by default, or 30 for datasets with more than 30 features)
- Misclassification percentages must be recorded for every subset size 1...min(M,30) per dataset and assembled into rrct_misclassification_rates.csv
- Dataset dimensions must match the paper: Relathe 1,427x4,322, Spambase 4,601x57, Ovarian cancer 72x592, SRBCT 63x2,308, SPECTF 187x44, Lymph 148x18, Cardiotocography 2,129x21, Handwriting 2,000x649
- Reproduce the qualitative result: RRCT should be the clear winner on Relathe and ovarian-cancer, tied with a few competitors on SRBCT, and generally strong elsewhere
- Random-forest evaluation must use out-of-sample (not resubstitution) misclassification percentage

## Limitations

- RRCT's advantage is not uniform: it was tied rather than dominant on datasets like SRBCT, and popular competitors (RELIEF, mRMR Peng, CFS, JMI) can still outperform on some data
- Reliance on rank correlations and partial correlations means performance depends on monotonic association structure; FEAST-style mutual-information methods require discretization instead, which is a different failure surface
- Subsets are capped at 30 features (min(M,30)) in the classification evaluation, so behavior with larger feature subsets is not characterized here

## Evidence

- [abstract] Benchmarks RRCT against 19 FS algorithms on eight real-world datasets: "We empirically benchmark the performance of RRCT against 19 FS algorithms across four synthetic and eight real-world datasets"
- [results] Random forest evaluation over feature subsets of size 1..min(M,30): "as a function of the number of selected features presented into the random forest (RF)"
- [results] Datasets sourced from UCI Machine Learning Repository: "Downloaded from the UCI Machine Learning Repository (http://archive.ics.uci.edu/ml/datasets.html)"
- [methods] uts output is a ranked feature list: "features -> The selected features in descending order of importance (ranked by column index)"
- [readme] RRCT is computationally efficient on massive datasets: "The RRCT algorithm is computationally very efficient and can run within a few seconds including on massive datasets with thousands of features."
