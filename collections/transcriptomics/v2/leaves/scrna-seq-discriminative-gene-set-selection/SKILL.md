---
name: scrna-seq-discriminative-gene-set-selection
description: Use when many phenotype groups (e.g., cell-type/tissue combinations)
  have overlapping DE genes and per-group markers fail, and a smallest global feature
  set distinguishing every phenotype pair is needed.
when_to_use_negative:
- Only a small number of distinct groups (e.g., major cell types) with non-overlapping
  DE genes — standard per-group marker selection suffices.
- The goal is an all-relevant feature ranking rather than a minimal discriminative
  cover (mRMR/Boruta-style methods are the wrong contrast here).
- Raw count data that has not been appropriately normalized/log-transformed before
  computing the phenotype × gene score matrix.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: phenotype-cover
  role: Python package providing GreedyPC and CEMPC solvers for the phenotype cover
    marker-selection problem
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Companion pip package installed alongside phenotype-cover for the equivalent
    multiset multicover formulation
- name: CEMPC
  role: Cross-entropy-method solver run with I=500 iterations and Rs=1000 elite samples;
    final markers are features with probability score > 0.98
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: Extended-greedy approximation solver; exposes plot_progress, max_coverage,
    feature_coverage, and pairs_with_incomplete_cover_ diagnostics
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: Provides make_classification for example data and logistic regression for
    train/hold-out evaluation of the selected feature set
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:83bb394bcad8f9bdacd58c02f41cb1707173675cd5c8cb9cb8d0ac039e3366e3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/euxhenh/phenotype-cover
  tools:
  - CEMPC
  - GreedyPC
  - Python
  - multiset-multicover
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# scrna-seq-discriminative-gene-set-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects a global, discriminative set of biomarkers for scRNA-seq phenotypes by solving the phenotype cover (PC) problem — equivalent to NP-complete multiset multicover — using the G-PC greedy and CEM-PC cross-entropy approximation algorithms from the phenotype-cover package.

## When to use

Use when many phenotype groups (e.g., cell-type/tissue combinations) have overlapping DE genes and per-group markers fail, and a smallest global feature set distinguishing every phenotype pair is needed.

## When NOT to use

- Only a small number of distinct groups (e.g., major cell types) with non-overlapping DE genes — standard per-group marker selection suffices.
- The goal is an all-relevant feature ranking rather than a minimal discriminative cover (mRMR/Boruta-style methods are the wrong contrast here).
- Raw count data that has not been appropriately normalized/log-transformed before computing the phenotype × gene score matrix.

## Inputs

- Phenotype × gene score matrix M (e.g., DE scores from IPF, mouse cortex (Zeisel), or HCA (He) scRNA-seq datasets)
- Phenotype-pair coverage requirements for the phenotype cover problem
- Coverage target (e.g., coverage of 8 or 100 depending on task)

## Outputs

- Selected marker gene set with attained phenotype-pair coverage (e.g., cem_pc_ipf_selected_markers.csv)
- Per-iteration coverage and remaining-element diagnostics (coverage_per_iter_, n_elements_remaining_per_iter_)

## How to apply

Build a phenotype × gene score matrix M (e.g., differential-expression scores over the IPF, mouse cortex, or human cell atlas scRNA-seq datasets) and formulate PC as combinatorial optimization: find the smallest set of features such that every phenotype pair (i,j) is covered. Run CEM-PC by initializing the CEMPC solver from phenotype-cover, setting a scoring function that rewards phenotype-pair coverage while penalizing selected-gene-set size, and configuring the cross-entropy loop with I = 500 iterations and Rs = 1000 as the elite-sample parameter; extract the best-scoring gene set and select all features whose probability score exceeds 0.98 after convergence (Methods S1, algorithm 3). Alternatively run the extended-greedy G-PC, which the paper reports outperforms competitors across marker-selection tasks and is fastest via its C++ implementation with Python bindings. Evaluate by training logistic regression on the selected feature subset and assessing performance on left-out data, and by computing the attained phenotype-pair coverage of the final set.

## Related tools

- **phenotype-cover** (Python package providing GreedyPC and CEMPC solvers for the phenotype cover marker-selection problem) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Companion pip package installed alongside phenotype-cover for the equivalent multiset multicover formulation)
- **CEMPC** (Cross-entropy-method solver run with I=500 iterations and Rs=1000 elite samples; final markers are features with probability score > 0.98) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (Extended-greedy approximation solver; exposes plot_progress, max_coverage, feature_coverage, and pairs_with_incomplete_cover_ diagnostics) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Provides make_classification for example data and logistic regression for train/hold-out evaluation of the selected feature set)

## Examples

```
pip install phenotype-cover; from phenotype_cover import CEMPC; cem = CEMPC(); cem.fit(X, y); features = cem.select(8)
```

## Evaluation signals

- Output CSV (e.g., cem_pc_ipf_selected_markers.csv) exists, parses, and records both the selected genes and the attained phenotype-pair coverage.
- For CEM-PC, the selected set equals exactly those features whose probability score exceeds 0.98 after the cross-entropy loop converges.
- Reported coverage is consistent with the coverage-rewarding, size-penalizing scoring function used during optimization.
- GreedyPC diagnostics confirm feasibility: pairs_with_incomplete_cover_ is empty (or the shortfalls are explained), and coverage_per_iter_ trends toward the max_coverage() bound.
- Logistic regression trained on the selected subset retains discriminative performance on left-out data, and stability is assessed by resampling half the data over 5 runs.

## Limitations

- PC/multiset multicover is NP-complete, so both G-PC and CEM-PC are polynomial-time approximations; G-PC does not provide an optimal solution.
- The biological analysis relied on overlap with existing gene marker lists that may be incomplete, and no thorough analysis of selected genes vs. cell types was performed.
- Other coverage-optimizing approximation algorithms remain untested on biological data; batch effects were only assessed on two pancreas datasets.

## Evidence
