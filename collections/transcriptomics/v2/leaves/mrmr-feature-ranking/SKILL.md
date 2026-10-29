---
name: mrmr-feature-ranking
description: Use when benchmarking marker-selection runtimes or ranking features on
  phenotype×feature score matrices, e.g. when reproducing Table 2 of the multiset
  multicover marker-selection paper on the IPF, MC, or HCA scRNA-seq datasets.
when_to_use_negative:
- The goal is a globally discriminative marker set covering every phenotype pair (use
  GreedyPC/CEMPC from phenotype-cover instead of mRMR).
- The target variable is a ranking of already-scored features rather than a supervised
  selection problem.
- Input is raw scRNA-seq counts without conversion to a feature matrix; log-transform
  or preprocess before mrmr_classif/mrmr_regression.
tools:
- name: mrmr (smazzanti/mrmr)
  role: Provides mrmr_classif for minimum Redundancy - Maximum Relevance feature ranking,
    used as a competitor method in the runtime benchmark against G-PC
  repo: https://github.com/smazzanti/mrmr
- name: GreedyPC
  role: The phenotype-cover method whose C++ implementation runtimes are compared
    against mRMR in Table 2
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Cross-entropy phenotype-cover method run alongside mRMR with identical feature
    counts in the benchmark
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: Dependency for data generation (make_classification) and competitor methods
    (decision trees, F-value, ReliefF, mutual information) in the same runtime table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:020ae2996d6de77d758c55f9423ef30b886887250a2554bd5234defeaa45e4f3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/smazzanti/mrmr
  tools:
  - C++
  - CEMPC
  - GreedyPC
  - Python
  - mrmr (smazzanti/mrmr)
  - numpy
  - scikit-learn
  - smazzanti/mrmr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mrmr-feature-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

mRMR (minimum Redundancy - Maximum Relevance) feature ranking, used in Hasanaj et al. 2022 as a competitor marker-selection method benchmarked against G-PC/CEM-PC on IPF, MC, and HCA scRNA-seq datasets, including the Table 2 runtime comparison.

## When to use

Use when benchmarking marker-selection runtimes or ranking features on phenotype×feature score matrices, e.g. when reproducing Table 2 of the multiset multicover marker-selection paper on the IPF, MC, or HCA scRNA-seq datasets.

## When NOT to use

- The goal is a globally discriminative marker set covering every phenotype pair (use GreedyPC/CEMPC from phenotype-cover instead of mRMR).
- The target variable is a ranking of already-scored features rather than a supervised selection problem.
- Input is raw scRNA-seq counts without conversion to a feature matrix; log-transform or preprocess before mrmr_classif/mrmr_regression.

## Inputs

- pandas DataFrame X of gene/feature expression values (phenotype × feature score matrix derived from IPF, MC, or HCA scRNA-seq data)
- pandas Series y of categorical phenotype labels (e.g. cell type/tissue combinations)
- integer K specifying the number of features to select (e.g. 178 IPF, 66 MC, 121 HCA)

## Outputs

- Ranked list of K selected features from mrmr_classif/mrmr_regression
- Wall-clock runtime measurement per dataset for the Table 2 runtime comparison

## How to apply

Install mRMR via pip (pip install mrmr_selection) and import it in Python. Represent the data as a pandas (or polars/Spark/BigQuery) DataFrame of features plus a target Series; for categorical cell-type or tissue phenotypes call mrmr_classif, for numeric targets call mrmr_regression, setting K to the required number of selected features (e.g. 178 for IPF, 66 for MC, 121 for HCA in the runtime benchmark). Note that mrmr_classif returns a ranked list of K features, so truncate the list if fewer features are needed. Record wall-clock runtime alongside the competitor methods (CEM-PC, decision trees, scGeneFit, RankCorr, F-value, ReliefF, mutual information) for inclusion in a runtime_table.csv mirroring Table 2. In the source study, the C++ G-PC implementation ran in under 2 seconds on all three datasets and was the fastest method tested, so mRMR timings serve as comparison points.

## Related tools

- **mrmr (smazzanti/mrmr)** (Provides mrmr_classif for minimum Redundancy - Maximum Relevance feature ranking, used as a competitor method in the runtime benchmark against G-PC) — https://github.com/smazzanti/mrmr
- **GreedyPC** (The phenotype-cover method whose C++ implementation runtimes are compared against mRMR in Table 2) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Cross-entropy phenotype-cover method run alongside mRMR with identical feature counts in the benchmark) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Dependency for data generation (make_classification) and competitor methods (decision trees, F-value, ReliefF, mutual information) in the same runtime table)

## Examples

```
from mrmr import mrmr_classif; selected_features = mrmr_classif(X=X, y=y, K=178)
```

## Evaluation signals

- mrmr_classif returns exactly K selected features, as a ranked list — verify len(selected_features) == K before truncating
- Reproduced runtime row for mRMR in runtime_table.csv uses the same feature counts as the other methods (e.g. 178/66/121 for IPF/MC/HCA)
- G-PC runtimes in the same table should be under 2 seconds on all three datasets, consistent with it being the fastest method tested
- Selected feature list is a ranking, so taking a prefix of it should be valid for further selection steps

## Limitations

- The article notes that G-PC (not mRMR) is the fastest method tested — the C++ implementation with Python bindings outperformed mRMR on the IPF, MC, and HCA runtime benchmark.
- mRMR output is a ranking, not a set cover; it does not guarantee that every phenotype pair is covered by the selected markers.
- Feature selection frequency is a strength for practical ML applications, but the README frames mRMR as a general feature-selection tool, not a phenotype-cover solver.

## Evidence

- [readme] mRMR stands for minimum Redundancy - Maximum Relevance and is a minimal-optimal feature selection algorithm: [evidence span withheld — anchor status: external_doc]
- [readme] mrmr_classif returns a ranking of K selected features, so a further selection takes the first elements of the list: [evidence span withheld — anchor status: external_doc]
- [other] mRMR chosen via smazzanti/mrmr as competitor in the runtime benchmark; C++ G-PC is fastest in Table 2: [evidence span withheld — anchor status: not_found]
