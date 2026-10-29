---
name: single-cell-rna-seq-marker-selection
description: Use when scRNA-seq data contain many cell types/tissues with overlapping
  DE genes and you need one global marker set that discriminates every phenotype pair
  (e.g., cell-type/tissue combinations in IPF, mouse cortex, or human cell atlas datasets).
when_to_use_negative:
- The goal is per-cell-type markers for a small number of distinct groups with non-overlapping
  DE genes — standard differential expression suffices
- You need the provably optimal minimum marker set — G-PC is a polynomial-time approximation
  and does not provide an optimal solution
- The target variable is numeric or the task is generic ML feature selection without
  phenotype-pair coverage requirements (use mRMR or all-relevant methods instead)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: phenotype-cover
  role: Provides GreedyPC and CEMPC solvers for the phenotype cover marker-selection
    problem; fit on log-transformed X, y and call select(K) at the desired coverage
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Dependency package implementing the multiset multicover formulation to which
    PC is equivalent
- name: C++
  role: C++ implementation of G-PC with Python bindings; fastest marker-selection
    method tested (<2 s on IPF, MC, HCA)
- name: scikit-learn
  role: Supplies competitor feature-selection methods (F-value, mutual information,
    ReliefF-style scoring) and logistic regression for evaluating selected markers
    on left-out data
- name: mRMR (mrmr_selection)
  role: Competitor minimal-optimal feature-selection baseline (mrmr_classif) used
    to benchmark marker sets
  repo: https://github.com/smazzanti/mrmr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:7586ed5d1e62e11a4896e590f59b61f668f643df5b907203cdefd7c20011b2bc
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
  - C++
  - CEMPC
  - GreedyPC
  - Python
  - mRMR (mrmr_selection)
  - multiset-multicover
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-rna-seq-marker-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects a global, discriminative set of markers from single-cell RNA-seq data by formulating marker selection as the phenotype cover (PC) problem — equivalent to NP-complete multiset multicover — and approximating it with GreedyPC (extended greedy set cover) or CEMPC (cross-entropy method) from the phenotype-cover package.

## When to use

Use when scRNA-seq data contain many cell types/tissues with overlapping DE genes and you need one global marker set that discriminates every phenotype pair (e.g., cell-type/tissue combinations in IPF, mouse cortex, or human cell atlas datasets).

## When NOT to use

- The goal is per-cell-type markers for a small number of distinct groups with non-overlapping DE genes — standard differential expression suffices
- You need the provably optimal minimum marker set — G-PC is a polynomial-time approximation and does not provide an optimal solution
- The target variable is numeric or the task is generic ML feature selection without phenotype-pair coverage requirements (use mRMR or all-relevant methods instead)

## Inputs

- scRNA-seq expression matrix (raw counts should be log-transformed) with cell-type/phenotype labels
- Phenotype x feature score matrix M encoding per-pair discriminative scores
- Desired coverage parameter K for GreedyPC.select()
- Comparison marker lists (e.g., HuBMAP ASCT+B marker set) for enrichment analysis

## Outputs

- Ranked/selected global marker set S (e.g., 178 features for IPF, 66 for MC, 121 for HCA)
- Coverage diagnostics: coverage_per_iter_, n_elements_remaining_per_iter_, pairs_with_incomplete_cover_
- Feature-coverage heatmap per selected feature (feature_coverage(i))
- Logistic regression accuracy on left-out data and overlap/enrichment vs. reference marker sets

## How to apply

Preprocess the scRNA-seq data (log-transform raw counts) into a phenotype x feature score matrix M, where each entry scores how well a feature discriminates a phenotype pair. Run GreedyPC from the phenotype-cover package (pip install phenotype-cover and multiset-multicover) via fit(X, y) followed by select(K) with the desired coverage K (e.g., coverage of 8 for GSEA against the HuBMAP ASCT+B marker set), or use CEMPC as the cross-entropy alternative; inspect gpc.coverage_per_iter_, gpc.pairs_with_incomplete_cover_, and gpc.max_coverage() to check cover quality. For large datasets, use the C++ implementation of G-PC with Python bindings, which ran in under 2 seconds on IPF (178 features), MC (66), and HCA (121) — the fastest method tested (Table 2). Compare candidate marker sets against competitors (decision trees, scGeneFit, RankCorr, F-value, ReliefF, mutual information, mRMR) and evaluate discriminative power by training logistic regression on selected features from a data subset and testing on left-out data.

## Related tools

- **phenotype-cover** (Provides GreedyPC and CEMPC solvers for the phenotype cover marker-selection problem; fit on log-transformed X, y and call select(K) at the desired coverage) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Dependency package implementing the multiset multicover formulation to which PC is equivalent)
- **C++** (C++ implementation of G-PC with Python bindings; fastest marker-selection method tested (<2 s on IPF, MC, HCA))
- **scikit-learn** (Supplies competitor feature-selection methods (F-value, mutual information, ReliefF-style scoring) and logistic regression for evaluating selected markers on left-out data)
- **mRMR (mrmr_selection)** (Competitor minimal-optimal feature-selection baseline (mrmr_classif) used to benchmark marker sets) — https://github.com/smazzanti/mrmr

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(8)
```

## Evaluation signals

- GreedyPC reports pairs_with_incomplete_cover_ — verify the number of phenotype pairs failing to reach the requested coverage is small/empty
- coverage_per_iter_ and n_elements_remaining_per_iter_ should show monotone coverage gains until the requested K is attained
- Logistic regression trained on the selected subset should perform well on left-out data (train-on-subset, evaluate-on-holdout protocol)
- Runtime check: C++ G-PC should complete in under 2 seconds on IPF (178 features), MC (66), and HCA (121) feature counts
- GSEA against the HuBMAP ASCT+B marker set at coverage 8 should show enrichment of selected genes among known markers

## Limitations

- G-PC does not provide an optimal solution; it is a polynomial-time approximation to an NP-complete problem
- Biological analysis relied on overlap with existing gene marker lists that may be incomplete
- Raw counts may need log-transformation before fitting, as noted in the README example
- Other coverage-optimizing approximation algorithms remain untested on biological data

## Evidence
